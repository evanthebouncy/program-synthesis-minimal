---
layout: post
title:  "a typical program synthesis problem"
permalink: /typical-synthesis-problem/
---

As [Leslie](https://people.csail.mit.edu/lpk/) puts it, you must _define_ the problem first before _solving_ it. This post is about how to define and analyize a typical synthesis problem, along with a naive solution.

## grass turtle and mushrooms
Imagine you're building a rectangular turtle enclosure on a field of 6x6 grid. You want to keep the grass inside the enclosure, and keep the mushrooms outside.

![Image with caption](/program-synthesis-primer/assets/synthesis-problem/comic2.png "icons created by Freepik - Flaticon, monkik")

Given a field, how do you quickly come up with a working rectangle?

## modeling programming
Program synthesis starts with **programming**. Programming starts with an **interpreter**.

![Image with caption](/program-synthesis-primer/assets/synthesis-problem/interpreter.png "the interpreter")

Here's how to model an interpreter for our task.

### program
The program is a data-structure. Here, a program is a rectangle, defined by top, down, left, right.

{% highlight python %}
# the rectangle in our figure
rectangle = [1,3,1,4]
{% endhighlight %}

### input
The input is a data-structure. Here, the input is a coordinate, defined by (row, col).
{% highlight python %}
# the mushrooms and grass in our figure
shroom1, shroom2 = (0,4), (4,1)
grass1, grass2 = (1,1), (3,3)
{% endhighlight %}

### output
The output is a data-structure. In our case, it simply indicates inside or outside
{% highlight python %}
inside = True
outside = False
{% endhighlight %}

### interpreter
The interpreter is a function (program, input) → output. In our case, given a rectangle and a coordinate, our interpreter checks if the coordinate is inside the rectangle.
{% highlight python %}
def is_inside(program, inputt):
    T, D, L, R = program
    i, j = inputt
    return i >= T and i <= D and j >= L and j <= R
{% endhighlight %}

### putting it together
Does our rectangle include the grass, and exclude the mushrooms?
{% highlight python %}
assert is_inside(rectangle, grass1) == inside
assert is_inside(rectangle, grass2) == inside
assert is_inside(rectangle, shroom1) == outside
assert is_inside(rectangle, shroom2) == outside
print ("working as intended !")
{% endhighlight %}

## the programming problem
How does a programmer write a program, so that the interpreter does the right thing for a task?

![Image with caption](/program-synthesis-primer/assets/synthesis-problem/programming-problem1.png "the programming problem")

There are many ways to specify a task -- imagine the different ways you can ask a developer to build an App. There are many ways to check if a program is "correct" -- imagine the different ways to test the App they built. In program synthesis, task and correctness are typically given as follows.

### task
A task is an input-output specification (think of test-cases). Consider a different grid of mushrooms and grass. 
{% highlight python %}
shroom3 = (2,2)
grass3 = (4,3)
shroom4 = (5,4)
spec = [(shroom3, outside), (grass3, inside), (shroom4, outside)]
{% endhighlight %}

### correctness
The advantage of giving the task as input-output is that we can check for correctness automatically -- if executing the program on the inputs produces the corresponding outputs.
{% highlight python %}
def is_correct(program, spec):
    for inputt, output in spec:
        if is_inside(program, inputt) != output:
            return False
    return True
{% endhighlight %}

### solve a few by hand !
It is good (I cannot recommend this enough) to solving a few programming problems by hand. It gives insights on how to build a machine synthesizer capable of doing the same.
{% highlight python %}
# let's try to make a rectangle that satisfy this new spec
rect2 = [1,3,1,4]
print (is_correct(rectangle, spec)) # didn't work
rect3 = [3,4,1,3]
print (is_correct(rect3, spec)) # worked okay
{% endhighlight %}

Our programming environment, `rectangle.py` [can be found here](https://gist.github.com/evanthebouncy/25114aaf0be20df21468735aa7103bef). You'll need it for later.

<br>
<hr>
[Let's take a break](https://www.youtube.com/watch?v=p5UuFfx85FY&ab_channel=Ensk). no seriously follow along and rest your eyes.
<hr>
<br>

## a typical synthesis problem

With the synthesizer, a programmer can program in program++, which is simply the task itself.

![Image with caption](/program-synthesis-primer/assets/synthesis-problem/synthesis.png "the synthesis set up")

### the meaning matrix M

Imagine you have infinite computation, and construct the **meaning matrix** `M = spec x prog`. Each row is a specification, each col is a program, and each entry relates the two using the interpreter -- `M[spec,prog] = is_correct(prog,spec)`. M _completely_ characterizes the synthesis problem. Assuming M can be built, the synthesis problem becomes : Given a spec (row), look up a prog (col) such that the matrix entry of `M[spec,prog] = True`.

![Image with caption](/program-synthesis-primer/assets/synthesis-problem/hardness1.png "the synthesis hardness")

_Some quick maff_ : Number of programs/rectangles is roughly `6^4`, inputs/coordinates is `6^2`, outputs/booleans is `2`. There are `(6^2)*2` input-output pairs. Assuming up to 10 grass and mushrooms on the field, there are roughly `((6^2)*2)^10` specs. Thus, we need to pre-compute a matrix of `((6^2)*2)^10 x 6^4 = 4.852e+21`. [Oh no](https://youtu.be/JqXg20sm_i4). 

Synthesis starts with writing down M -- the horrific combinatorial monster. Understanding and approximating the structure of M is how we fight the monster with limited computations.

## a typical synthesis algorithm

Rather than pre-computing the entirety of M, a typical program synthesis algorithm samples elements from a specific row `M[spec,:]`. This algorithm has a **program writer** that proposes different programs based on tasks, and a **program checker** that uses the interpreter and the specification to check if the proposed programs are correct. 

![Image with caption](/program-synthesis-primer/assets/synthesis-problem/synthesizer-gut1.png "the synthesizer")

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
This is much better than using the random writer on eyeball value.

## exercise
How do the writers compare on a variety of different specs? Can you come up with a even better program writer? Explain why a writer might be performing better than another using `M`. Explore these questions by [using the synthesizer code here](https://gist.github.com/evanthebouncy/ffa855eac2caa38716b3bc8d8b62645a).

## up next
Next time we will fine-tune a language model for the program writer.

-- evan 2022-08-12