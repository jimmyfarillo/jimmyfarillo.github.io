---
layout: post
title: "Learn Design Patterns: Template Method Pattern"
date: 2017-02-19
---

A couple friends and I have a web development book club. The goal is to learn
and grow together as software engineers. But really, sometimes it's more like a
support group, where we use a lot of 😞 and 😢 to discuss our respective
screw-ups at work and the impossible vastness of information about web
development and how we only know the smallest tiniest minusculest amount of it.
We're a fun group...

The first book we read together was _Practical Object-Oriented Design in Ruby_
by Sandi Metz. Really amazing resource for new Ruby devs (or probably any new
dev using an OO language). For our next book, we just started reading _Design
Patterns in Ruby_ by Russ Olsen, so this will be the first in a planned series
about the various design patterns discussed in the book.

The first pattern is the template method pattern.

## Inheritance + Polymorphism

The template method pattern is an object-oriented design pattern that relies on
inheritance and polymorphism. By inheritance, I am talking about the
parent-child relationship between classes, where a class can be defined as a
subclass of another and inherits all of that superclass' methods. By
polymorphism, I am talking about the ability to invoke the same method on
multiple objects of different classes.

```
# Inheritance

class Computer
  def initialize
    @on = false
  end

  def power_on
    @on = true
    puts 'Booting up...'
  end
end

class Laptop < Computer
  def close_lid
    ...
  end

  def open_lid
    ...
  end
end

my_laptop = Laptop.new
my_laptop.power_on          # => Booting up...
```

In the basic inheritance example above, the `Computer` class defines the
`power_on` method. The `Laptop` class is defined as a subclass of `Computer` and
also defines its own set of methods. But since `Computer` is its superclass,
instances of `Laptop` inherit the `power_on` method and are able to be sent that
message.

```
# Polymorphism

def too_big?(obj)
  obj.length > 10
end

str = 'Hello, World!'
arr = [1, 2, 3]

too_big?(str)               # => true
too_big?(arr)               # => false
```

The polymorphism example is showing that both the `String` and `Array` classes
implement the `length` method. There is some role that they are both playing,
perhaps the role of a `Countable`. The `too_big?` method doesn't care whether it
is being passed a string or an array; it just cares whether it is being passed
an object that is countable and has implemented `length`. Being able to send the
same message to objects that are instances of different classes is polymorphism.

## The Template Method Pattern

The template method pattern combines the concepts of inheritance and
polymorphism to solve the common programming problem of needing to allow for
some level of variation in an algorithm.

If you were a strange person, you might think of your life in terms of Ruby
objects and messages that you send to these objects. So if you wanted to throw a
birthday party for your niece, you might have a `Party` class with a
`throw_party` method that does all the steps necessary for giving your niece a
successful birthday party.

```
class Party
  def throw_party
    invite_guests
    clean_house
    buy_birthday_cake
    welcome_guests
    sing_happy_birthday
    kick_out_guests
    clean_house
  end

  private

  ...all the private methods being used in throw_party...

end
```

What if you wanted to throw another party, maybe a game night for you and some
friends? It would be great to reuse this `Party` class and the `throw_party`
method, but while some parts of the method are general to any party, some parts
are very specific to birthday parties.

We could put in some conditionals:

```
class Party
  def throw_party(type)
    invite_guests
    clean_house

    if type == 'birthday'
      buy_birthday_cake
    elsif type == 'game_night'
      buy_board_games
    end

    welcome_guests

    if type == 'birthday'
      sing_happy_birthday
    elsif type == 'game_night'
      play_games
    end

    kick_out_guests
    clean_house
  end

  private

  ...all the private methods being used in throw_party...

end
```

Yikes! That is not the best solution. Any time you want to throw a different
kind of party, you will have to modify this method until it grows out of
control. And you'll have to add more and more private methods to this class. But
having completely separate classes for each party type isn't ideal, either,
since we saw that most of the `throw_party` logic is common to any type of
party.

The template method pattern is a way of separating the parts that are the same
from the parts that are different. The parts that are the same get implemented
in a parent class while the parts that are different are implemented in the
subclasses.

The only parts that differ between the two party types are the buying of
supplies and the party activities. So we can create an abstract Party class
(abstract because there will be no instances of it) that will serve as the
superclass for the `BirthdayParty` and `GameNight` classes.

```
class Party
  def throw_party
    invite_guests
    buy_supplies
    clean_house
    welcome_guests
    do_activities
    kick_out_guests
    clean_house
  end

  private

  def buy_supplies
    raise 'Not implemented: buy_supplies'
  end

  def do_activities
    raise 'Not implemented: do_activities'
  end

  ...all the other private methods being used in throw_party...

end
```

Now, all the parts that are the same can stay in `Party`, and the parts that are
different can be defined in our `BirthdayParty` and `GameNight` classes.

```
class BirthdayParty < Party

  private

  def buy_supplies
    buy_birthday_cake
  end

  def do_activities
    sing_happy_birthday
  end

  def buy_birthday_cake
    ...
  end

  def sing_happy_birthday
    ...
  end
end

class GameNight < Party

  private

  def buy_supplies
    buy_board_games
  end

  def do_activities
    play_games
  end

  def buy_board_games
    ...
  end

  def play_games
    ...
  end
end
```

The superclass is the template for what happens for a party, and each subclass
implements just the parts that vary from the template.

In the current implementation of `Party`, an error is raised for methods that
are meant to be implemented in the subclasses. This provides a clear signal to
anyone wanting to implement a new type of party that `buy_supplies` and
`do_activities` are methods that should be implemented in the new party type.
Another approach would be to implement these methods in `Party` with some
default behavior that the subclasses then override.

Back to the use of inheritance and polymorphism. It is fairly clear from the
code that `BirthdayParty` and `GameNight` are both subclasses of `Party`. So
every method defined in `Party` is also available to the subclasses. In our
example, the only publicly available method in either subclass is the
`throw_party` method defined in `Party`. That's where the polymorphism comes in.
Whatever object decides to throw a party can do so by sending the `throw_party`
message to an instance of either `BirthdayParty` or `GameNight`. It doesn't care
which type of object it sends the message to, and even though the results may be
different, both types of objects will respond to the message.
