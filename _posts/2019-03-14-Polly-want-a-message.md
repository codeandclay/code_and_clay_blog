---
layout: post
title: Polly want a message
---
I wrote some notes whilst watching this video. No one talks about OOP the way [Sandi Metz](https://www.sandimetz.com) does. It's fair to say, I didn't really get OOP until I read [POODR](https://www.poodr.com).

<iframe allowfullscreen="" frameborder="0" mozallowfullscreen="" src="https://player.vimeo.com/video/290378227" webkitallowfullscreen=""></iframe>
<br>
The video and its transcription is [here](https://www.deconstructconf.com/2018/sandi-metz-polly-want-a-message).

### How OOP apps can go wrong. How to fix them

- Design stamina hypothesis -- Martin Fowler
  - Don't design too early.

- Procedures vs OO
  - OO is harder to understand because it works on messages. Messages add a layer of indirection.
  - If you can solve a problem with a simple procedure, you probably should.
  - But procedures do not scale. Procedures bloat with conditionals.
  - Just as bad is OO with wrong abstractions (Faux-O).

- Churn vs Complexity -- Michael Feathers
  - Simple code changes a lot. Super complex code does not.
  - Beware super large classes that are a central part of the app domain (user, contract, etc..)

  - 1 Design Stamina Hypothesis
  - 2 Simple procedures
  - 3 Churn vs Complexity
  - 4 What matters suffers.

- What matters has most tickets. Changes reflect what already exists. "People do more of what's there."

- What brought you success will bring you to failure.
  - There is a point in development when you need to do something different.

### OO Affords … objects
#### Anthropomorphic
Think like an object. OO is play with living beings in a world.
#### Polymorphic
Objects sharing the same form at the message response level.
#### Loosely-coupled
Objects in a system can be easily swapped for others.
#### Role-playing
Objects are players of their roles. They are substitutable.
#### Factory-created
Factories are places where you hide the rules for picking the right player of a role.
#### Message-sending
I know what I want, you know how to do it.

Inject a smarter thing.

Isolate the thngs you want to vary.

Push conditionals back on the stack.
  - Push them back as far as you can. Back to the place where someone could have had enough information to pick the right object.

Dependency injection is your friend! As long as you inject smart things.

### Factory Objects
Create factories to encapsulate conditionals.

```ruby
 Clump
  Clump::Comment#lines
  Clump::LineNumber#lines
```
- Factories are where the conditionals go.
- Factories are: shared, isolated, simple, ignorable.
- They have a tiny amount of code. They know the rules of switching and objects that provide the intended bevhaviour.

- If you create a factory to produce a player of a role, you must use that factory whenever you have that role. You must not reference the names of these classes elsewhere.

- OO gives you the opportunity to maximise the ignorance of every object. If you take advantage of that opportunity you can make your entire application smarter.
