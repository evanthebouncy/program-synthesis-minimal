---
layout: post
title:  "language models for programs"
permalink: /generating-programs/
---

A good program synthesis algorithm boils down to the simple question -- Given a task, how do you generate a correct program with as few samples as possible? This post covers a few reasonable strategies that are easy to implement.

This post has dependencies on [the previous post](/program-synthesis-primer/typical-synthesis-problem/).

## the dream and the reality

Let's revisit the meaning matrix M. Given a `spec`, let's look at its corresponding row `M[spec,:]`.

![Image with caption](/program-synthesis-primer/assets/generating-programs/a-row.png "icons created by Freepik - Flaticon, monkik")

Using this row, construct the following distribution, with probability proportional to whether the entry `M[spec,prog]` is True or False. This distribution gives uniform probability over correct programs, and 0 probability over incorrect programs.

![Image with caption](/program-synthesis-primer/assets/generating-programs/ground-truth.png ){: width="50%" }

The dream is that we can easily sample from this distribution. The reality is that sampling from this distribution is _hard_ -- While we can easily check whether a given program is correct with respect to the spec, coming up with such programs is difficult. Instead, given a spec, we must _approximate_ this ground-truth distribution using synthesis.

![Image with caption](/program-synthesis-primer/assets/generating-programs/approximate.png ){: width="75%" }

This distribution is implemented by the synthesis algorithm from the previous post -- a program _writer_ to propose somewhat plausible programs, then use a _checker_ to ensure correctness. 

To implement a program writer, we turn to language modeling.

## language modeling

 **language models** gives probabilies over sequences of characters. For a language model to be a good program writer, it needs to put high probabilities over sequences that resemble correct programs. Given the `spec` from earlier, let's suppose we want to generate the correct program `[1,3,1,4]`. Consider the following language models of various fidelity. 

![Image with caption](/program-synthesis-primer/assets/generating-programs/language-modeling.png )

### all strings
We can generate a sequence by uniformly picking a random character a number of times. This language contains all possible strings.
{% highlight python %}
def writer1():
    return ''.join(random.choice(string.printable) for i in range(9))
print(writer1()) # you should see something like "=1Vi![Au37"
{% endhighlight %}
The string `[1,3,1,4]` has 9 characters, and each character has a probability of `1 / len(string.printable)`. Thus this writer has a `(1/100)^9` chance of stumbling across `[1,3,1,4]`.

### writable programs
Instead of generating free-form sequences, we can restrict the generation to "reasonably looking sequences". In programming, this is done with a **domain specific language** (DSL), which identifies a "reasonable" subset of all-strings. The DSL is specified by a **grammar**: a process capable of generating strings within the DSL.
{% highlight python %}
def writer2():
    # W is globally defined to be 6
    U = random.randint(0,W)
    D = random.randint(0,W)
    L = random.randint(0,W)
    R = random.randint(0,W)
    return '['+str(U)+','+str(D)+','+str(L)+','+str(R)+']'
print(writer2()) # you should see something like "[5,2,1,0]"
{% endhighlight %}
By giving structure to the language, we increased our chance to `(1/6)^4`, despite our DSL containing uninterpretable programs such as `[5,2,1,0]`.

### legal programs
We can further refine our DSL to be "tighter".
{% highlight python %}
def writer3():
    U = random.randint(0,W-1)
    L = random.randint(0,W-1)
    height = random.randint(1,W-U)
    width = random.randint(1,W-L)
    D, R = U+height, L+width
    return '['+str(U)+','+str(D)+','+str(L)+','+str(R)+']'
print (writer3()) # you should see something like "[0,2,4,6]"
{% endhighlight %}
What's our chances of stumbling across `[1,3,1,4]` now? I'll leave the combinatorics to you.

### plausible programs
We can be creative and try to use the specification to generate more plausible programs. For instance, we can make it more likely to put the upper-left corner of the rectangle on an inside coordinate.
{% highlight python %}
def writer4(spec):
    row_weights = [1,1,1,1,1,1]
    col_weights = [1,1,1,1,1,1]
    # get the coordinates of spec that are inside and outside
    inside_coords = [coord for coord,bool in spec if bool]
    # up-weight the inside coords
    for coord in inside_coords:
        row_weights[coord[0]] += 3.1415
        col_weights[coord[1]] += 3.1415
    # sample U and D from the row weights
    U = random.choices(range(W), weights=row_weights, k=1)[0]
    L = random.choices(range(W), weights=col_weights, k=1)[0]
    height = random.randint(1,W-U)
    width = random.randint(1,W-L)
    D, R = U+height, L+width    
    return '['+str(U)+','+str(D)+','+str(L)+','+str(R)+']'

spec = [( (0,4), outside), ( (4,1), outside), ( (1,1), inside), ( (3,3), inside)]
print (writer4(spec)) # you should see something like [1,5,5,6]
{% endhighlight %}
Does this really help? I have no idea.

### correct programs
As it turns out, you _can_ write an algorithm that writes a correct rectangle every time. But in general, one can only get to the point of plausible programs.

<br>
<hr>
[Let's take a break](https://youtu.be/jQJVHjlvuVE). no seriously follow along and stretch!
<hr>
<br>

## training a language model

TODO.