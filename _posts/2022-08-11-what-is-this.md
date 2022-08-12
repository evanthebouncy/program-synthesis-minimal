---
layout: post
title:  "what you will find here"
sticky: true
hidden: true
---

# so, you want to know about program synthesis?

Recently (as of aug 2022), the topic of "building AI to write code" have become fashionable in both industry and academia, with [copilot](https://github.com/features/copilot) making its way to production (if you have not used it, stop reading and play with it first). However, program synthesis is much broader than generating few lines of code from comments. Programs are broadly useful in modeling both computation and communication, and whenever programs exist, program synthesis is sure to follow. 

<ins>This blog series is a primer to the **modern**, **practical** aspects of program synthesis.</ins> I intend to keep the write-ups concise, at a cost of accuracy. The coding examples will be pedagogical, introducing you to a concept and realizing it using off-the-shelf technologies, at a cost of fidelity. The goal is 入门 (to get your foot through the door), so you can read and replicate academic literatures, and have the confidence to apply program synthesis to your own works. 

Let us begin.

## programming
Before computers, when a person has a task to do, they manually does stuff. This is tiring.
![Image with caption](/program-synthesis-primer/assets/what-is-this/doing.png "doing")

Programming is the act of a person asking the computer to do something. Hacking the linux kernel is programming. Creating an email filter is programming. Me setting up this blog is programming. One must always think of programming with these three: a programmer, a program, and an interpreter. A **programmer** turns a task into a program. A **program** is the thing that programmer gives, which the interpreter understands. An **interpreter** takes in the program and does stuff.

<!-- 
Programming consists of a **programmer**, who writes a **program**, which is executed on an **interpreter**. One must think about programming with all three in mind! -->
![Image with caption](/program-synthesis-primer/assets/what-is-this/programming1.png "programming")

We program because interpreters (computers) can help us with difficult tasks. A human cannot multiplex different programs at 4GHz. A human cannot sort through 1000s of emails manually. I cannot deliver the blog to all of you in person.

<!-- ![Image with caption](/assets/what-is-this/programming.png "programming") -->

## program synthesis
Program synthesis is a system that makes programming easier. gcc frees you from having to write assembly (in the 70s and 80s, program synthesis literally used to mean compilers). Using "filter emails like this" makes creating a filter easier. Jekyll generating the website makes creating this blog easier. With program synthesis, the programmer can now program using program++, which is easier to use. The **synthesizer** is a "middle-man" programmer, which turns a task (program++) into a program.

![Image with caption](/program-synthesis-primer/assets/what-is-this/prog_plus.png "synthesis")

 The synthesizer and the interpreter together is the interpreter++, which takes in program++ and does stuff. Thus, program synthesis is simply "easier programming" and must be stated in relationship with the original "harder programming" context.

## what is the end game?
The end game is to make programming as easy as possible. To make it more concrete, we can take a [Joshian](https://youtu.be/RB78vRUO6X8) point of view, and look at human programmers as the ultimate synthesizer.

![Image with caption](/program-synthesis-primer/assets/what-is-this/synthesis-ultimate.png "human-program")

Of course, we are not to be limited by human capabilities. The goal is to go even further beyond!! AAaaaaaAaaA[AAaaaAA](https://youtu.be/3FM2kbvYljw?t=18)AAaaAaahhHHhHHhH (okay I'll stop).

-- evan  2022-08-11

[continue to a working example](future link here)

