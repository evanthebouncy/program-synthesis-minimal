---
layout: post
title:  "a typical program synthesis problem"
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
The input is a data-structure. Here, the input is a coordinate, defined by (col, row).
{% highlight python %}
# the mushrooms and grass in our figure
shroom1, shroom2 = (0,4), (4,1)
grass1, grass2 = (1,1), (3,3)
{% endhighlight %}

### output
The output, y, is a data-structure. In our case, it simply indicates inside or outside
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
The programming problem is : given an interpreter, how does a programmer turn a task into a program, so that the interpreter does the right thing?

![Image with caption](/program-synthesis-primer/assets/synthesis-problem/programming-problem1.png "the programming problem")

There are many ways to specify a task -- imagine all the different ways you might communicate to a developer on building an App. There are many ways to check if a program "does the right thing" -- imagine all the user testings with the App to make sure it's usable. We will explain how these are typically framed for a program synthesis problem.

### task
A task is typically given as an input-output specification (think of test-cases). Let's consider a different grid of mushrooms and grass. 
{% highlight python %}
shroom3 = (2,2)
grass3 = (4,3)
shroom4 = (5,4)
spec = [(shroom3, outside), (grass3, inside), (shroom4, outside)]
{% endhighlight %}

### correctness
The advantage of giving the task as input-output is that we can check for correctness automatically: if executing the program on the inputs produces the corresponding outputs.
{% highlight python %}
def is_correct(program, spec):
    for inputt, output in spec:
        if is_inside(program, inputt) != output:
            return False
    return True
{% endhighlight %}

## do a few by hand !
After implementing the programming problem, it is good practice (I cannot recommend this enough) to do a few by hand. Doing so turns you into a human program synthesizer, and gives insights on how to build a machine synthesizer capable of taking your place.
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
[Let's take a break](https://www.youtube.com/watch?v=p5UuFfx85FY&ab_channel=Ensk)
<hr>
<br>

## the synthesis problem

TODO

define the synthesis problem, make a point about sandbox-simulation, i.e. the synthesizer has blackbox access to the execution, you also need to tell the synthesizer about the space of programs, etc . . . all that.