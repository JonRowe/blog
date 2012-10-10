---
title: Adventures In The Hexagonal (Part 1)
layout: post
abstract: Decoupling your Rails
---
Adventures In The Hexagonal: Part 1
=========================================

One of the key challenges I've consistently faced in keeping my code lean and
my tests mean in Rails applications has been the controller. Wether refactoring
a fat controller down or a dealing with a skinny boiler plate controller, I
find they suffer from cruft, issues with integration testing and worries about
repetition. As I've discussed before it's possible to unit test controllers, but
that doesn't eliminate the problems with architecture design; it only helps to
sooth the aches and pains that arise.

Traditionally I attempted to remove business logic from my controllers by
segregating that logic into a service layer. All the better for segregation
of responsibilities, and for testing responsibility; even hopefully improving
design along the but I had a problem. How do you handle the different resulting
conditions?  I found even simple CRUD could require a response beyond the
boolean true/false. Have even semi-complex business logic and I became
stuck, how do you handle the response when you have more than that
pass/fail matrix. You could return an object with state, return
symbols/strings to represent the result, but that would probably leave
you with a mess of complex conditionals, yuk! I settled on throwing
exceptions, using the controller to catch them and perform the relevant
task. It worked well, despite violating the rule of "exceptions should be
truly exceptional", essentially the services were telling the
controller what to do by the means of all hell breaking loose.
However, back in the summer, I attended the Scottish Ruby Conference, and
saw [@MattWynne]( http://twitter.com/mattwynne )'s hexagonal rails talk; in
that talk Matt proposed using the controllers themselves as response objects.
Injecting themselves as a dependency to the service layer turning this:

<script src="https://gist.github.com/e7aeb5dea41e29874f32.js">
</script>

Into something more like this:

<script src="https://gist.github.com/ac51b08035a5f0d398fd.js">
</script>

I became a big fan of this idea, by giving the business layer something
concrete to attach to, you are definitively telling the controller what
to do. Additionally you end up defining an API for talking between the
controller and the business layer, meaning that the business layer knows
nothing of the how the response is returned, only which action it has
triggered. This means you can stack different layers together, you can
add in wrappers, all without changing underlying code. You can use
this interface in your integration tests, or from a command line
interface or anything you want.

However there are still some issues with this approach, it feels wrong
to have public methods like this on Rails controllers, especially given
their special treatment by Rails. At the post-talk BOF session, a few of
us experimented with creating response objects to solve this problem,
but didn't come across a neat solution. However recently I came across
[this]( https://www.theagileplanner.com/blog/building-agile-planner/refactoring-with-hexagonal-rails )
refactoring by [@GrahamAshton]( http://twitter.com/grahamashton ) with
input from [@MattWynne]( http://twitter.com/mattwynne ) :

<script src="https://gist.github.com/55e488edffa8e88ea754.js">
</script>

Suddenly I see a clearer path towards nice hexagonal controllers, these
simple response objects make for simple testing, nicely segregated
responsibility and can be easily reused if you so desired. I haven't
settled on a pattern for the placement of these responses, do they
belong to their parents controllers? Do they get placed somewhere in the
main namespace on there own? Are they worth reusing at all? 

I still see problems with the controllers too, testing them (when they have
large level of constants and class creation) still (to me) seems worthwhile, but
is fraught with things I don't like doing (like mocking/stubbing new to
produce a fake instance). I'd rather inject the dependent services but
Rails gives me no means to do so, if Rails did it would also give me the
means to build single CRUD controllers with no magic/inheritance to
handle multiple types of process, inject different services and
responses to do different things.
This is something that [Raptor]( https://github.com/garybernhardt/raptor)
does and while I feel uncomfortable with *no* controllers I feel the
router and injection is moving along the right lines.

So... Who wants to help make this possible with Rails::Metal and the
Rails router ;)

As usual... Thoughts? Questions? Queries? Trolling? Find me on
[Twitter]( https://twitter.com/jonrowe)
