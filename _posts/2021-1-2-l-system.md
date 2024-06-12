---
layout: post
title: Growing fractal plants in Go 
---

Recently I've been programming simulations of *continuous* [dynamical systems](https://en.wikipedia.org/wiki/Dynamical_systems_theory) at work, and *discrete* dynamical systems as part of many [solutions](https://github.com/acjensen/advent-of-code/tree/main/day11) to the [Advent of Code 2020](https://adventofcode.com/2020). In this post I'll first compare continuous vs discrete system simulation methods, and then "grow" a plant by modelling it as a discrete dynamical system.

## Dyanamical systems

A time-invariant, first-order, *continuous* dynamical system can be described with a simple equation \\( \frac{dx}{dt}=f(x(t)) \\), where \\( x(t) \\) is the state of the system at time \\( t \\), and \\( f \\) is some function describing how to get to the next state in terms of the current state \\( x(t) \\).

How can we simulate a continous-time system with a discrete computer? A computer can simulate a dynamical system evolving over time by applying the function \\( f \\) over many infinitely small timesteps \\( dt \\). In other words, at each subsequent timestep \\( dt \\), we can calculate the next state with \\( x_{next}=x_{now} + dx \\). From our first equation, we know that \\( dx = f(x(t))dt \\), and we arrive at the following computation:

\\[ x_{next} = x_{now} + f(x_{now})dt \\]

It is difficult to compute \\( x_{next} \\) accurately because \\( dt \\) is infintesimally small, and computers cannot directly compute boundless quantities (read: [Forward Euler Method](https://en.wikipedia.org/wiki/Euler_method)).

In *discrete* dynamical systems however, the evolution of the state \\( x \\) does not depend on time: \\( x_{next} \\) is a direct function of the current state \\( x_{now} \\).

\\[ x_{next} = f(x_{now}) \\]

[Lindenmayer systems](https://en.wikipedia.org/wiki/L-system) are interesting discrete dynamical systems to study because they demonstrate how a simple state mutation function applied many times can generate beautifully complex results.

## Growing a fractal plant

A particularly interesting Lindenmayer system is the fractal plant. A fractal plant is grown by repeatedly applying a set of "growth" rules that describe how to get from the current plant state to the next. Here's the rules I used:

- X --> F+[[X]-X]-F[-FX]+X
- F --> FF

The plant's state is represented as an ordered list of symbols that evolve according to the rules above. Each symbol can be interpreted as an instruction for a 2D renderer, resulting in a plant-like image.

- F --> "draw forward"
- \- --> "turn right"
- \+ --> "turn left"
- X --> "ignore"
- [ --> "start a new branch"
- ] --> "finish the current branch"

Here's a plant with 6 iterations starting with a state of `X`.

<img src="../images/fractal_plant.gif" width="400">

You can view the source code on [GitHub](https://github.com/acjensen/meristem).

