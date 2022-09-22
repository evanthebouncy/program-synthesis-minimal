---
layout: post
title:  "a typical synthesis problem"
permalink: /typical-synthesis-problem/
---

As [Leslie](https://people.csail.mit.edu/lpk/) puts it, you must _define_ the problem first before _solving_ it. This post is about how to define and analyze a typical synthesis problem, along with a naive solution.

## grass turtle and mushrooms
Imagine you're building a rectangular turtle enclosure on a field of 6x6 grid. You want to keep the grass inside the enclosure, and keep the mushrooms outside.

![Image with caption](/program-synthesis-primer/assets/synthesis-problem/comic4.png "icons created by Freepik - Flaticon, monkik"){: width="75%" }

Given a field, how do you quickly come up with a working rectangle?

## modeling programming
Program synthesis starts with **programming**. Programming starts with an **interpreter**.

![Image with caption](/program-synthesis-primer/assets/synthesis-problem/interpreter1.png "the interpreter"){: width="65%" }

Here's how to model an interpreter for our task.

### program
The program is a data-structure. Here, a program is a rectangle, defined by top, down, left, right.

{% highlight python %}
# the rectangle in our figure, [T,D,L,R]
rectangle = [1,3,1,4]
{% endhighlight %}

### input
The input is a data-structure. Here, the input is a coordinate, defined by (row, col).
{% highlight python %}
# the mushrooms and grass coordinates in our figure
shroom1 = (0,4)
shroom2 = (4,1)
grass1 = (1,1)
grass2 = (3,3)
{% endhighlight %}

### output
The output is a data-structure. In our case, it simply indicates inside or outside
{% highlight python %}
inside = True
outside = False
{% endhighlight %}

### interpreter
The interpreter is a function (program, input) → output. In our case, given a program (a rectangle, i.e. `[1,3,1,4]`) and an input (a coordinate, i.e. `(3,3)`), our interpreter interprets, or *executes* the program on the input by returning if the coordinate is inside or outside the rectangle.
{% highlight python %}
def interpret(program, inputt):
    T, D, L, R = program
    i, j = inputt
    return i >= T and i <= D and j >= L and j <= R
{% endhighlight %}

The interpreter gives _semantics_ to a program -- without the interpreter, the program is but a lifeless pile of syntax (`[1,3,1,4]` isn't a rectangle on its own).

### putting it together
Does our rectangle include the grass, and exclude the mushrooms?
{% highlight python %}
assert interpret(rectangle, grass1) == inside
assert interpret(rectangle, grass2) == inside
assert interpret(rectangle, shroom1) == outside
assert interpret(rectangle, shroom2) == outside
print ("working as intended !")
{% endhighlight %}

## programming
How does a programmer write a program, so that the interpreter does the right thing for a task?

![Image with caption](/program-synthesis-primer/assets/synthesis-problem/does_right_stuff.png "the programming problem")

There are many ways to specify a task -- imagine the different ways you can ask a developer to build an App. There are many ways to check if a program is "correct" -- imagine the different ways to test the App they built. People's real-life tasks can rarely be understood by computers. 

### task as specifications

A **specification** or `spec` is a way of stating the task so that _both human and computer can agree_ on what needs to be done. In program synthesis, a `spec` is typically given as a list of input-outputs. <ins>Input-output is one of the rare, special form of communication that is readily understood by both humans and computers</ins>. 

There are other forms of specifications, for instance, it could be an objective function[^gend] to be maximized, a set of constraints on the computer's behaviours[^markov-lim], or a natural language sentence[^dan]. I hope to cover these topics at a different time!

Here are some specs.
{% highlight python %}
spec = [(grass1, inside), (grass2, inside), (shroom1, outside), (shroom2, outside)]
shroom3 = (2,2)
grass3 = (4,3)
shroom4 = (5,4)
spec2 = [(shroom3, outside), (grass3, inside), (shroom4, outside)]
{% endhighlight %}

### correctness
With a `spec`, the computer can automatically decide if a program "did the right stuff" by checking if executing the program on the inputs produces the corresponding outputs.
{% highlight python %}
def is_correct(program, spec):
    for inputt, output in spec:
        if interpret(program, inputt) != output:
            return False
    return True
{% endhighlight %}

### programming problem 
A typical programming problem consists of a human writing both the specification and the program, and having the computer to check if the program meets the specification.

![Image with caption](/program-synthesis-primer/assets/synthesis-problem/pre-synthesis.png "the programming problem")

<ins>It is crucial to solve a few programming problems by hand before attempting program synthesis</ins>. It gives insights on how to build a machine synthesizer capable of doing the same.
{% highlight python %}
# let's try to make a rectangle that satisfy spec2
rect2 = [1,3,1,4]
print (is_correct(rectangle, spec2)) # didn't work
rect3 = [3,4,1,3]
print (is_correct(rect3, spec2)) # worked okay
{% endhighlight %}

Our programming environment, `rectangle.py` [can be found here](https://gist.github.com/evanthebouncy/25114aaf0be20df21468735aa7103bef).

### the asymmetry

<ins>It is often easier to write the specification than it is to program</ins>. This asymmetry alone motivates program synthesis. Do _not_ attempt program synthesis if this asymmetry is not present[^deductive]. 

<br>
<hr>
[Let's take a break](https://www.youtube.com/watch?v=p5UuFfx85FY&ab_channel=Ensk). no seriously follow along and rest your eyes.
<hr>
<br>

## a typical synthesis problem

With the synthesizer, a programmer writes the specification (a form of easier programming), and leaves the harder programming to the synthesizer.

![Image with caption](/program-synthesis-primer/assets/synthesis-problem/synthesis1.png "the synthesis set up"){: width="75%" }

The synthesis problem is turning specs into programs. It is formalized using the meaning matrix.

## characterize the synthesis problem with meaning matrix M

Imagine you have infinite computation, and construct the **meaning matrix M** `M = spec x prog`. Each row is a specification, each col is a program, and each entry relates the two using the interpreter -- `M[spec,prog] = is_correct(prog,spec)`. 

<ins>M completely characterizes the synthesis problem</ins>. Assuming M can be built, the synthesis problem becomes trivial: Given a spec (row), look up a prog (col) such that the matrix entry `M[spec,prog] = True`.

![Image with caption](/program-synthesis-primer/assets/synthesis-problem/hardness1.png "the synthesis hardness")

### using M to quantify difficulty
How difficult is our synthesis problem? Some _quick maffs_ : Number of programs/rectangles is roughly `6^4`, inputs/coordinates is `6^2`, outputs/booleans is `2`. There are `(6^2)*2` input-output pairs. Assuming up to 10 grass and mushrooms on the field, there are roughly `((6^2)*2)^10` specs. Thus, we need to pre-compute a matrix of `((6^2)*2)^10 x 6^4 = 4.852e+21`. [Oh no](https://youtu.be/JqXg20sm_i4). 

Synthesis starts with writing down M -- the horrific combinatorial monster. Understanding and approximating the structure of M is how we fight the monster with limited computations.

## a typical synthesis algorithm

Rather than pre-computing the entirety of M, a typical program synthesis algorithm samples elements from a specific row `M[spec,:]`. This algorithm has a **program writer** that proposes different programs based on tasks, and a **program checker** that uses the interpreter and the specification to check if the proposed programs are correct. 

![Image with caption](/program-synthesis-primer/assets/synthesis-problem/synthesizer-gut2.png "the synthesizer")

### program writer
The writer proposes programs given specs. Consider the most naive program writer that simply writes a random program.

{% highlight python %}
def random_writer(spec):
    # ignores the spec, W=6 globally
    T, D, L, R = random.randint(0,W), random.randint(0,W), random.randint(0,W), random.randint(0,W)
    return [T, D, L, R]
{% endhighlight %}

### program checker
The `is_correct` function we wrote is the checker we need -- it internally uses the interpreter.
{% highlight python %}
def program_cheker(prog, spec):
    return is_correct(prog, spec)
{% endhighlight %}

### putting it together
We will use `budget` to denote how many programs our writer can propose before giving up.
{% highlight python %}
# a synthesizer that returns both a working program (if it finds any)
# and the number of samples it took to find it
def get_synthesizer(writer, checker, budget):
    def synthesizer(spec):
        prog = writer(spec)
        for i in range(budget):
            prog = writer(spec)
            if checker(prog, spec):
                return (i, prog)
        return None
    return synthesizer
{% endhighlight %}

Let's try it!

{% highlight python %}
synthesizer = get_synthesizer(random_writer, program_cheker, 1000)
spec1 = [( (0,4), outside), ( (4,1), outside), ( (1,1), inside), ( (3,3), inside)]
n_tries, prog = synthesizer(spec1)
print (n_tries, prog) 
# results vary, but I got 146 [1, 3, 0, 3], 31 [1, 3, 0, 5], 56 [1, 3, 1, 6], etc
{% endhighlight %}
It is not terribly efficient.

### making the writer better
Instead of ignoring the spec, maybe it is a good idea to only use the coordinates of the grass (inside) as the parameters of the rectangle. 
{% highlight python %}
def better_writer(spec):
    # get the coordinates of spec that are inside
    inside_coords = [coord for coord,bool in spec if bool]
    if inside_coords == []:
        # if there are no inside coordinates, default to a random
        return random_writer(spec)
    # otherwise, use the inside coords to suggest parameters of the rectangle
    row_coords = [coord[0] for coord in inside_coords]
    col_coords = [coord[1] for coord in inside_coords]
    T, D = random.choice(row_coords), random.choice(row_coords)
    L, R = random.choice(col_coords), random.choice(col_coords)
    return [T, D, L, R]
{% endhighlight %}

Let's try it!

{% highlight python %}
synthesizer2 = get_synthesizer(better_writer, program_cheker, 1000)
n_tries, prog = synthesizer2(spec1)
print (n_tries, prog)
# results vary, but I got 1 [1, 3, 1, 3], 23 [1, 3, 1, 3], 5 [1, 3, 1, 3], etc
{% endhighlight %}
This is much better than using the random writer on first glance.

## exercise
How do the writers compare on a variety of different specs? Can you come up with a even better program writer? Explore these questions by [using the synthesizer code here](https://gist.github.com/evanthebouncy/ffa855eac2caa38716b3bc8d8b62645a).

-- evan 2022-08-12

## up next
The next post cover how to systematically generate programs with language models. [let's go for it](/program-synthesis-primer/generating-programs/)

### notes
[^deductive]: [Read page 117-119 on how to reverse a list using deductive synthesis](https://dl.acm.org/doi/pdf/10.1145/357084.357090)

[^gend]: [Generative design](https://www.autodesk.com/solutions/generative-design)

[^markov-lim]: [On the Expressivity of Markov Reward](https://arxiv.org/abs/2111.00876)

[^dan]: [Task-Oriented Dialogue as Dataflow Synthesis](https://youtu.be/jFCiYlK6Rb8)