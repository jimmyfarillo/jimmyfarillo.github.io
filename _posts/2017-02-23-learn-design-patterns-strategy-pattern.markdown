---
layout: post
title: "Learn Design Patterns: Strategy Pattern"
date: 2017-02-23
---

The strategy pattern is an alternative to the
[template method pattern]({% post_url 2017-02-19-learn-design-patterns-template-method-pattern %})
I wrote about in my previous post, providing a different solution to the common
programming problem of requiring some variation within an algorithm. Both
patterns use polymorphism in their approaches, but while the template method
pattern relies on inheritance, the strategy pattern relies on composition and
delegation.

## Composition

To use the `Party` example from the post on the template method pattern, you
might choose to think about parties as being made up of different parts, such as
a venue, guests, supplies, and activities. Modeled in code, these parts could be
defined as instance variables with accessor methods.

```
class Party
  attr_accessor :venue, :guests, :supplies, :activities

  def initialize(args)
    @venue = args[:venue]
    @guests = args[:guests]
    @supplies = args[:supplies]
    @activities = args[:activities]
  end
end
```

Composition describes the "has a" relationship between the object and the parts
that it is made up of. From this perspective, a `Party` "has a" `venue`, "has a"
`guests`, "has a" `supplies`, and "has a" `activities`.

This is in contrast with inheritance, which describes a "is a" relationship. A
`BirthdayParty` "is a" `Party`.

So another way to look at the different types a `Party` could be is to think
about them in terms of their parts. Here's an example of a birthday party and a
game night party implemented using composition rather than inheritance.

```
class Party
  attr_accessor :venue, :guests, :supplies, :activities

  def initialize(args)
    @venue = args[:venue]
    @guests = args[:guests]
    @supplies = args[:supplies]
    @activities = args[:activities]
  end
end

birthday_party = Party.new(
  venue: Place.new("Chuck E. Cheese's"),
  guests: [User.new('Margaret Atwood'), User.new('David Mitchell')],
  supplies: [Food.new('birthday cake')],
  activities: [Activity.new('sing happy birthday')]
)

game_night = Party.new(
  venue: Place.new('246 Maple Lane'),
  guests: [User.new('Ruth Ozeki'), User.new('Kazuo Ishiguro')],
  supplies: [Game.new('Catan'), Game.new('Bananagrams')],
  activities: [Activity.new('play games')]
)
```

In this example, each instance of `Party` is instantiated with different parts.
Specifically, the array supplies for `birthday_party` contains a single object
of type `Food`, while `game_night` contains two objects of type `Game`. We'll
come back to this example below, when we get into the specifics of the strategy
pattern.

Instead of defining different kinds of parties as subclasses of the abstract
`Party` class, instances of `Party` are made to be different because they are
composed of different parts. The advantage of using composition over inheritance
is that there is no coupling of subclasses to a superclass in composition,
reducing the number of dependencies in your code.

## Delegation

Delegation often comes up when composition is used. Delegation is the concept of
passing off some work to another object.

There will be some logic involved for inviting our guests to the party, and we
could put that logic inside our `Party` class.

```
class Party
  attr_accessor :venue, :guests, :supplies, :activities

  def initialize(args)
    @venue = args[:venue]
    @guests = args[:guests]
    @supplies = args[:supplies]
    @activities = args[:activities]
  end

  def invite_guests
    guests.each { |guest| invite(guest) }
  end

  def invite(guest)
    puts "Hi #{guest.name}, please come to my party at #{venue}!"
  end
end
```

But we could also delegate this task to each guest by placing that logic inside
the `User` class and sending that message to the guest objects.

```
class User
  attr_accessor :name

  def initialize(name)
    @name = name
  end

  def send_invitation(context)
    puts "Hi #{name}, please come to my party at #{context.venue}!"
  end
end

class Party
  attr_accessor :venue, :guests, :supplies, :activities

  def initialize(args)
    @venue = args[:venue]
    @guests = args[:guests]
    @supplies = args[:supplies]
    @activities = args[:activities]
  end

  def invite_guests
    guests.each { |guest| guest.send_invitation(self) }
  end
end
```

In the above example, the `invite_guests` method on the `Party` class sends the
`send_invitation` message to each guest, which is defined in the `User` class.
The instance of `Party` also passes a reference of itself, as `self`, as an
argument to `send_invitation`, allowing the method to know the context in which
it should be performed. The logic of how to invite a guest has been passed off
to each guest object.

## The Strategy Pattern

Now that we've discussed composition and delegation, we can use them in the
strategy pattern. Let's Party on! 🎉

You might remember the `throw_party` method from the previous post about the
template method pattern. It contains all the logic necessary for throwing a
great party.

```
def throw_party
  invite_guests
  buy_supplies
  clean_house
  welcome_guests
  do_activities
  kick_out_guests
  clean_house
end
```

We can create variation in that algorithm by delegating the responsibilities to
the different parts that compose each party. If you remember, the only parts
that differ between birthday parties and game nights are the `supplies` and the
`activities`, so I'm going to ignore the methods the deal with `venue` and
`guests`.

```
class Party
  attr_accessor :venue, :guests, :supplies, :activities

  def initialize(args)
    @venue = args[:venue]
    @guests = args[:guests]
    @supplies = args[:supplies]
    @activities = args[:activities]
  end

  def throw_party
    invite_guests
    buy_supplies
    clean_house
    welcome_guests
    do_activities
    kick_out_guests
    clean_house
  end

  def buy_supplies
    supplies.each { |supply| supply.buy }
  end

  def do_activities
    activities.each { |activity| activity.perform(self) }
  end
end

class Food
  def buy
    ...
  end
end

class Game
  def buy
    ...
  end
end

class Activity
  def perform
    ...
  end
end
```

The `supplies` and `activities` are the objects that compose our `Party`. All
`supplies` that make up our `Party` should implement the `buy` method, whether
each object is a `Game`, `Food`, or any other party supply. These objects all
implement a common interface, even though the implementations will likely be
different for each type of object. Similarly, all activities will implement the
`perform` method. Without knowing the internals of `Activity#perform`, we can
safely assume that the `perform` method for the `Activity` object for singing
happy birthday will likely have different side effects than the one for playing
games. All of these sections of the algorithm have been delegated to the parts
of the `Party` object that are involved in their execution.

The strategy pattern solves the same problem that the template method pattern
solves, but it reduces the amount of coupling between objects by using
composition over inheritance. Instead of identifying the sections of your
algorithm that can vary and then placing those sections in subclasses of an
abstract superclass, the algorithm has been delegated to the different parts
that compose your object, and those objects implement their respective portions
of the algorithm however they see fit.
