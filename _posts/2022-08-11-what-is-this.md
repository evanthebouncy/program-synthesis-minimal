---
layout: post
title:  "what you will find here"
sticky: true
hidden: true
permalink: /what-is-synthesis/
---

# so, you want to know about program synthesis?

Recently (as of 2022), the topic of "building AI to write code" has become fashionable in both industry and academia, with [copilot](https://github.com/features/copilot) making its way to production (if you have not used it, stop reading and play with it first). However, program synthesis is more than generating few lines of code from comments. Programs are broadly useful in modeling both computation and communication, and whenever programs exist, program synthesis is sure to follow. 

<ins>This blog series is a primer to the **modern**, **practical** aspects of program synthesis.</ins> I intend to keep the write-ups concise, at a cost of accuracy. The examples will be pedagogical, introducing you to a concept and coding it up using off-the-shelf technologies, at a cost of fidelity. The goal is 入门 (to get your foot through the door), so you can read and replicate academic literatures, and have the confidence to apply program synthesis to your own works. 

This blog is intented for undergraduate/graduate level researchers with knowledge of machine-learning, who want to "get into" program synthesis. Some level of mathematical maturity will be assumed, but not essential as the algorithms and applications are self-contained.

Let us begin.

## programming
Before computers, when a person has a task to do, they manually does stuff. This is tiring.
![Image with caption](/program-synthesis-minimal/assets/what-is-this/doing.png "doing")

Programming is the act of a person asking the computer to do something. Hacking the linux kernel is programming. Creating an email filter is programming. Me setting up this blog is programming. One must always think of programming with these three: a programmer, a program, and an interpreter. A **programmer** turns a task into a program. A **program** is the thing that programmer gives, which the interpreter understands. An **interpreter** takes in the program and does stuff.

<!-- 
Programming consists of a **programmer**, who writes a **program**, which is executed on an **interpreter**. One must think about programming with all three in mind! -->
![Image with caption](/program-synthesis-minimal/assets/what-is-this/programming1.png "programming")

We program because interpreters (computers) can help us with difficult tasks. A human cannot multiplex different apps at 4GHz. A human cannot sort through 1000s of emails manually. I cannot deliver the blog to all of you in person.

## program synthesis
Program synthesis is a system that makes programming easier. gcc frees you from having to write assembly. Using "filter emails like this" makes creating a filter easier. Jekyll generating the website makes creating this blog easier. With program synthesis, the programmer can now program using "program++", which is easier to use than "program". The **synthesizer** is a middle-man programmer, which turns program++ into a program.

![Image with caption](/program-synthesis-minimal/assets/what-is-this/prog_plus.png "synthesis")

 Together, the synthesizer and the interpreter becomes a better interpreter, the "interpreter++", which takes in program++ (commonly called a **spec** for specification) and does stuff. Note how this process can be recursive, where we build layers upon layers of synthesizers/interpreters, making it easier and easier for a human to program. Thus, program synthesis is simply "easier programming" -- one step of this recursive process -- and must be stated in relationship with the original "harder programming" context. Let's see some examples.

## a brief history of program synthesis
The first synthesizers were compilers, which automatically generated code (assembly) from high-level specifications (FORTRAN). In **deductive synthesis**[^deductive], one transforms a specification (the array should become sorted) using mathematical rules, resulting in a provably correct program (the merge-sort algorithm).

Most modern synthesis are **inductive synthesis** -- where the algorithm performs a _search_ to find a solution that meets the specification. In neuro-symbolic[^neurosym] program synthesis, the search is performed with a neural-network, which proposes plausible (symbolic) programs. Recent (2020+) advances [^alphacode],[^codex] frequently leverages natural language (such as code comments) as additional context to guide the search (to generate python code). The subject of transforming natural language to executable programs (think siri or alexa) traditionally falls under **semantic-parsing**[^sem-parse],[^sem-tutorial]. 

Lastly, as programs are flexible in representing symbolic knowledge, they have been used to model cognition [^joshrule],[^probmods], where learning can be thought of as a form of "program induction".

As we cannot afford the space to go into it further, kindly refer to [^1],[^2] for more history.

## where do we stand now?
Program synthesis had a fairly big 门槛 (entry barrier). You needed expertise in programming language design, compilers, constraint-solving, semantic-parsing, and designing specific neural-network architectures tailored toward code generation. However, with recent advances of foundational models, I believe (could be wrong) most of these expertise are no longer required to get started. With this blog, I hope you can replicate the workflow of a typical program synthesis practitioner with a lighter toolbox.

## what is the end game?
We can take a [Joshian](https://youtu.be/RB78vRUO6X8) point of view (gesturing wildly at humans), and let the goal of program synthesis be building a system that can subsitute for a human developer (programmer) -- who turns fully naturalistic human-human interactions into working programs. The art of program synthesis is making program++/specification as humane as possible, while keeping the mapping between program++ and program tractable.

![Image with caption](/program-synthesis-minimal/assets/what-is-this/synthesis-ultimate1.png "human-program")

Of course, we don't have to be limited by human programming capabilities. We aim to build a synthesizer that can to go even further beyond!! AAaaaaaAaaA[AAaaaAA](https://youtu.be/3FM2kbvYljw?t=18)AAaaAaahhHHhHHhH (okay I'll stop).

-- evan  2022-08-11

## up next
The next post covers how to set-up a typical synthesis problem. [let's go for it](/program-synthesis-minimal/typical-synthesis-problem/)

### notes
[^1]: [Armando's lecture notes on definition and history of program synthesis](https://people.csail.mit.edu/asolar/SynthesisCourse/Lecture1.htm)

[^2]: [A 2017 survey on program synthesis by microsoft folks](https://www.microsoft.com/en-us/research/wp-content/uploads/2017/10/program_synthesis_now.pdf)

[^deductive]: [Armando's lecture notes on deductive synthesis](https://people.csail.mit.edu/asolar/SynthesisCourse/Lecture17.htm)

[^neurosym]: [A 2021 survey on neuro-symbolic program synthesis](https://www.cs.utexas.edu/~swarat/pubs/PGL-049-Plain.pdf)

[^alphacode]: [Competitive programming with AlphaCode](https://www.deepmind.com/blog/competitive-programming-with-alphacode)

[^codex]: [Evaluating Large Language Models Trained on Code](https://arxiv.org/abs/2107.03374)

[^sem-parse]: [Bringing machine learning and compositional semantics together by Percy Liang](https://web.stanford.edu/~cgpotts/manuscripts/liang-potts-semantics.pdf)

[^sem-tutorial]: [ACL 2018 tutorial on neural semantic parsing](https://github.com/allenai/acl2018-semantic-parsing-tutorial)

[^joshrule]: [The Child as Hacker](http://colala.berkeley.edu/papers/rule2020child.pdf)

[^probmods]: [Probabilistic Models of Cognition](http://probmods.org/)