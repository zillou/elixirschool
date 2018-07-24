---
author: Sean Callan
author_link: https://github.com/doomspork
categories: general
date: 2018-08-01
ayout: post
title: Umbrellas: only when it rains?
excerpt: >
  A look at Umbrella applications and how they can help us write cleaner maintainable code.
---

The Umbrella application is powerful tool available to us but many people are unfamiliar with them.
In this post we'll look at Umbrella applications and why we might want to consider using them for our next project.

## Project types

Before we can understand when to use Umbrellas and the role they play, let's review the types of projects available.
For all intents and purposes there are three project types in the Erlang and Elixir ecosystem: Applications, Libraries, and Umbrellas.

Libraries are packages of code that can be re-used by other projects and applications.
These projects lack a supervision tree.
In Elixir a Library would lack the `:mod` key in the Mix `application/0` function.

Applications are like Libraries but contain a supervision tree.
If you open the `mix.exs` file for a project and see that the `application/0` function defines a `:mod` entry point: you're working with an Application.
This is a dependency that has and maintains state and additional processes.
These aren't simply modules with simple functions.

Lastly we have Umbrellas applications which are little more than syntactic sugar for managing a collection of Libraries and Applications as a single project.
Why we might use Umbrellas is what we'll explore here today.

## Getting to know Umbrella Applications

...

If you haven't already, take a peek at the [Umbrella]() lesson which provides a great overview of Umbrella applications.

## Separating concerns

In my humble opinion one of the great strengths to Umbrellas is the forced separation of concerns and decoupling of code.
We have to be explicit with our individual application's configuration and this includes dependencies on other peer applications.
If our presentation layer doesn't have the internal data application as a dependency there's no chance those two components will become coupled.

While the same separation can be achieved in theory using a singular Application, the Umbrella gives us the added benefit of enforcing it through dependencies.
I've never seen "in theory" stand-up in a professional setting when skill levels and experience vary, Umbrellas remove any questions and confusion.

Above any other reason, this is often the deciding factor for whether or not I leverage an Umbrella application.

## Service Oriented Architect lite

If we know we building a complex application who's internal components will grow in size with distinct roles and responsibilities, Umbrellas can be a valuable tool in the toolchest.

With one codebase and repository we're able to maintain any number of sub applications as part of our overall Umbrella.
If we're building an online marketplace we could use this to keep our payment transaction code isolated from our other modules.
We can implement integrations with third-party services as standalone applications inside our Umbrella.
This not only keeps those concerns separated but allows us to work on components individual and more importantly: test them individually.

Nothing is keeping us from running multiple Phoenix applications in a single Umbrella.
We can develop our admin web portal independently of our customer facing site, running on a separate port making it a matter of network configuration to limit access to VPN users only.

If the scope of an application grows too much that we need to remove it from the Umbrella, no problem!
Doing this is simple with Umbrellas because of those explicit dependencies, updating those references is all we need to do.

An added benefit of developing our project as disparate micro-services: large teams can work concurrently on one codebase while minimizing code conflicts.

## One deployment to rule them all

What's easier: deploying changes to 12 applications or 1?

Releases aren't just easier with Umbrellas, they're more powerful.
We not only have a single artifact we can deploy, we can configura

## To Umbrella or not to Umbrella

Contrary to what some may believe, Umbrella applications aren't a perfect solution for every problem.
Before we `mix new` we should pause and consider some simple rules to help guide our decision making process:

- Is this a simple Application or is it likely to grow in scope and size quickly?
- Are there multiple  distinct and separate internal components to our application?
- Will multiple people be working on different parts of the code at once?

If we've answered YES to the above then an Umbrella might be the right choice for your project.

We'd love to hear your thoughts!
Are you using Umbrellas applications?
Have you found them helpful or a hinderance?
