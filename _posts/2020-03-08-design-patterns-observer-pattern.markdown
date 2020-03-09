---
layout: post
title: "Design Patterns: Observer Pattern"
date: 2020-03-08
---

In object-oriented programming, we have objects passing many different messages
to each other. But we must also balance this with a desire to prevent our
classes from being tightly coupled to each other and therefore having many
interlocking dependencies that make it hard to maintain the code.

One common issue where objects can become tightly coupled is when an object or
multiple objects needs to perform some action when another object is updated.
Using our real worlf party example from previous posts, if the details of a
party are updated, then the guests need to be informed so that they can plan
for the changes. The observer pattern can help us with these types of
situations.

## Perils of Party Planning

Let's revist our `Party` and `User` classes:

```
class Party
  attr_accessor :date, :venue, :guests, :supplies, :activities

  def initialize(args)
    @date = args[:venue]
    @venue = args[:venue]
    @guests = args[:guests]
    @supplies = args[:supplies]
    @activities = args[:activities]
  end

  def throw_party
    invite_guests
    puts "Doing all the things necessary to throw a great party!"
  end

  def invite_guests
    guests.each { |guest| guest.send_invitation(self) }
  end
end

class User
  attr_accessor :name, :event_calendar

  def initialize(name)
    @name = name
  end

  def send_invitation(event)
    puts "Hi #{name}, please come to my party at #{event.venue}!"
    plan_for_event(event)
  end

  def plan_for_event(event)
    puts "Doing all the things to plan for #{event}!"
  end
end
```

For recap, we can initialize an instance of `Party` with all the details,
including the guests. When we throw the party, we first invite all the guests,
which sends the `send_invitation` message to all the guests so that they can
plan for the event.

```
party = Party.new(
    venue: Venue.new('246 Maple Lane'),
  guests: [User.new('Ruth Ozeki'), User.new('Kazuo Ishiguro')],
  supplies: [Game.new('Catan'), Game.new('Bananagrams')],
  activities: [Activity.new('play games')]
)

party.throw_party
```

What happens when the date of our party changes, though? Our `Party` class is
using `attr_accessor` so it is fairly trivial to update the date. But then our
guests won't know about the change in date. And I'm sure you nobody will be
having a good time if your friends show up to the venue the day before the party
is actually supposed to happen. Let's try to keep our friends happy!

One way we could do this is by just letting the guests know directly that the
date has changed:

```
class Party
  attr_accessor :date, :venue, :guests, :supplies, :activities

  def initialize(args)
    @date = args[:venue]
    @venue = args[:venue]
    @guests = args[:guests]
    @supplies = args[:supplies]
    @activities = args[:activities]
  end

  def throw_party
    invite_guests
    puts "Doing all the things necessary to throw a great party!"
  end

  def invite_guests
    guests.each { |guest| guest.send_invitation(self) }
  end

  def date=(new_date)
    @date = new_date
    guests.each { |guest| guest.notify_event_change(self) }
  end
end

class User
  attr_accessor :name, :event_calendar

  def initialize(name)
    @name = name
  end

  def send_invitation(event)
    puts "Hi #{name}, please come to my party at #{event.venue}!"
    plan_for_event(event)
  end

  def plan_for_event(event)
    puts "Doing all the things to plan for #{event}!"
  end

  def notify_event_change(event)
    puts "Changing my plans for #{event}"
  end
end
```

That's not too bad. But then we realize that we also need to let the venue know
that the date changed. That means going back into `date=` and adding some code
to inform the venue. And if this is a more complex party, you might need to add
code for informing the caterer and the photographer and anyone else that needs
to know about any changes. This is an example of how classes can quickly become
tightly coupled. Luckily, there is a better way.

## Make It Observable

Ruby has an `Observable` module that we can take advantage of in our `Party`
class.

```
class Party
  include Observable

  attr_accessor :date, :venue, :guests, :supplies, :activities

  def initialize(args)
    @date = args[:venue]
    @venue = args[:venue]
    @guests = args[:guests]
    @supplies = args[:supplies]
    @activities = args[:activities]
  end

  def throw_party
    invite_guests
    puts "Doing all the things necessary to throw a great party!"
  end

  def invite_guests
    guests.each { |guest| guest.send_invitation(self) }
  end

  def date=(new_date)
    original_date = date
    @date = new_date

    if original_date != date
      changed
      notify_observers(self)
    end
  end
end

class User
  attr_accessor :name, :event_calendar

  def initialize(name)
    @name = name
  end

  def send_invitation(event)
    puts "Hi #{name}, please come to my party at #{event.venue}!"
    event.add_observer(self)
    plan_for_event(event)
  end

  def plan_for_event(event)
    puts "Doing all the things to plan for #{event}!"
  end

  def update(event)
    puts "Changing my plans for #{event}"
  end
end
```

Most of the code above is similar to what we saw before, but now we're doing
`include Observable` in the `Party` class. This adds a few methods for adding
and deleting observers, and defines our party as the subject. We're using the
party's `add_observer` method in the `User#send_invitation` method. This will
add that user to the list of observers of the party. Now, when the date of the
party is being modified to a different date, we call the `changed` method which
is also supplied by the `Observable` module. This simply sets a boolean value
that something on the object has changed. Then we call the `notify_observers`
method with `self`. This will call the `update` method on each observer with
whatever arguments are passed to `notify_observers` - `self`, in our case.

To further illustrate how this prevents our classes from being tightly coupled,
I'll implement this pattern with the venue:

```
class Party
  include Observable

  attr_accessor :date, :venue, :guests, :supplies, :activities

  def initialize(args)
    @date = args[:venue]
    @venue = args[:venue]
    @guests = args[:guests]
    @supplies = args[:supplies]
    @activities = args[:activities]
  end

  def throw_party
    book_venue
    invite_guests
    puts "Doing all the things necessary to throw a great party!"
  end

  def book_venue
    venue.book(self)
  end

  def invite_guests
    guests.each { |guest| guest.send_invitation(self) }
  end

  def date=(new_date)
    original_date = date
    @date = new_date

    if original_date != date
      changed
      notify_observers(self)
    end
  end
end

class Venue
  def initialize(address)
    @address = address
  end

  def book(event)
    event.add_observer(self)
    puts "Block out #{event.date} in calendar so no other events can be booked."
  end

  def update(event)
    puts "Modify booking for #{event}"
  end
end
```

In the code above, no change needed to be made to the `date=` method. The venue
adds itself to the party's observers when the venue is booked. This allows the
venue to be automatically notified when the subject (party) is changed.

The observer pattern can be thought of as a simplified version of the
publish-subscribe pattern. The main difference between observer and
publish-subscribe is that a subject directly notifies the observers of changes
while a publisher notifies a third-party of changes and a subscriber tells that
third-party that it wants to be notified of those changes. This third-party
broker manages all the update messages from the publisher and sends them out the
the appropriate subscribers. The observer pattern does not utilize a
third-party.

{% include design_patterns.markdown %}
