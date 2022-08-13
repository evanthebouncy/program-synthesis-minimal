---
layout: post
title:  "a typical synthesis problem"
---

As [Leslie](https://people.csail.mit.edu/lpk/) puts it, you must define the problem before any attempts at solving it. This post is about how to construct and analyize a typical synthesis problem, along with a naive solution.

## grass turtle and mushrooms
Imagine you're building a rectangular turtle enclosure on a 6x6 grid field of grass and mushrooms. You want to keep grass inside the enclosure, and keep mushrooms outside.

![Image with caption](/program-synthesis-primer/assets/synthesis-problem/comic2.png "icons created by Freepik - Flaticon, monkik")

Given a field, how do you quickly come up with a working rectangle?

## the programming problem
One always defines the programming problem before the synthesis problem. A programming problem consists of: the program, the input, the output, the interpreter, and the task.

### program
The program, f, is a datastructure. In our case, f represents a rectangle, consists of 4 values denoting the boundaries, top, down, left, right. f = [1,4,1,5] in the figure.

### input
The input, x, is a datastructure. In our case, x represents a coordinate, consists of 2 values (i,j). There is a grass at (1,1), and there is a mushroom at (4,0).

### output
The output, y, is a datastructure. In our case, it is just a boolean True or False.

### interpreter
The interpreter, exe, is a function. It maps (f,x) → y. In our case, exe takes a rectangle, and a coordinate, and returns whether the coordinate is inside or outside the rectangle. For example: exe([1,4,1,5], (1,1)) → True, exe([1,4,1,5], (4,0)) → False.

### task
The task, spec, is a set of input-output pairs. It is a set of "test cases" indicating what does it mean for a program to be correct. In our case, it specifies which coordinates (of grass) should be inside, and which coordinates (of mushroom) should be outside. In the figure, spec = { (1,1)→True, (4,0)→False, (1,4)→False, (3,3)→True }.

### do a few programming problems by hand !
Once you've defined (and implemented) the programming problem, it is good practice (I cannot recommend this enough) to do a few programming problems by hand. So go ahead and try it [here](https://gist.github.com/evanthebouncy/a23f4918077b7537081a437888d46317). You will be building a synthesizer that will take your spot later down the line, and by doing these problems by hand gives vital insights on how to build it.
