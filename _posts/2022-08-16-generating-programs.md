---
layout: post
title:  "generating programs"
permalink: /generating-programs/
---

A good program synthesis algorithm boils down to this -- Given a task, how do you generate a correct program with as few samples as possible? This post covers a few standard strategies.

This post has dependencies on [the previous post](/program-synthesis-primer/typical-synthesis-problem/).

## the problem
Let's re-use the same `spec` as before, the goal is to find a rectangle that meets the specification.
![Image with caption](/program-synthesis-primer/assets/generating-programs/spec.png "icons created by Freepik - Flaticon, monkik")


## the dream and the reality

Given this `spec`, let's look up its corresponding row `M[spec,:]` in the meaning matrix.

![Image with caption](/program-synthesis-primer/assets/generating-programs/a-row.png)

The **ground-truth** distribution for program synthesis can be constructed using `M[spec,:]`, with probability proportional to whether the entry `M[spec,prog]` is correct. This distribution gives uniform probability over correct programs, and 0 probability over incorrect programs.

![Image with caption](/program-synthesis-primer/assets/generating-programs/ground-truth.png ){: width="70%" }

The _dream_ is that we can easily sample from the ground-truth distribution. The _reality_ is that sampling from this distribution is _hard_ -- While we can easily check whether a given program is correct with respect to the spec, coming up with such programs is difficult. 

## the synthesis algorithm

In practice, we resort to _approximating_ this ground-truth distribution using synthesis.

![Image with caption](/program-synthesis-primer/assets/generating-programs/approximate.png ){: width="75%" }

This distribution is implemented by the synthesis algorithm -- a program writer proposes somewhat plausible programs, then a checker filters for correctness. 

![Image with caption](/program-synthesis-primer/assets/synthesis-problem/synthesizer-gut.png )

Thus, the better the writer (closer to ground-truth), the fewer programs the checker has to check. To implement a reasonable program writer, we turn to language modeling.

## language modeling

 **language models** gives probabilies over sequences of characters. For a language model to be a good program writer, it needs to put high probabilities over sequences that _resemble_ correct programs. Let's suppose we want to generate a correct program `[1,3,1,4]` for our `spec`. Consider the following language models of various fidelity. 

![Image with caption](/program-synthesis-primer/assets/generating-programs/language-modeling.png )

Let's start with **unconditional language models**. These models do not take `spec` into account.

### all strings
We can generate a sequence by uniformly picking a random character a number of times. This language contains all possible strings.
{% highlight python %}
def writer1():
    return ''.join(random.choice(string.printable) for i in range(9))
print(writer1()) # you should see something like "=1Vi![Au37"
{% endhighlight %}
The string `[1,3,1,4]` has 9 characters, and each character has a probability of `1 / len(string.printable)`. Thus this writer has a `(1/100)^9` chance of stumbling across `[1,3,1,4]`.

### writable programs
We can restrict the generation to "reasonably looking sequences". In programming, this is done with a **domain specific language** (DSL), which identifies a "reasonable" subset of all-strings. The DSL is specified by a **grammar** -- a process capable of generating strings within the DSL.
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
We can further refine our DSL to be "tighter" to exclude uninterpretable programs. 
{% highlight python %}
def writer3():
    U = random.randint(0,W-2)
    L = random.randint(0,W-2)
    height = random.randint(1,W-U)
    width = random.randint(1,W-L)
    D, R = U+height, L+width
    return '['+str(U)+','+str(D)+','+str(L)+','+str(R)+']'
print (writer3()) # you should see something like "[0,2,4,5]"
{% endhighlight %}
What's our chances of stumbling across `[1,3,1,4]` now? I'll leave the combinatorics to you. 

### unconditional generation is important !
In program synthesis, it is _crucial_ that you have a good enough program generator. This generator samples programs from a **prior distribution**, P(prog), unconditioned on the spec. One can use unconditional generation as a program writer as is.

![Image with caption](/program-synthesis-primer/assets/generating-programs/prior-synthesis.png ){: width="75%" }

This often resulting in a "good enough" synthesis algorithm for a small space of programs.

<br>
<hr>
[Let's take a break](https://youtu.be/jQJVHjlvuVE). no seriously follow along and stretch! the next part will be kind of intense.
<hr>
<br>

# conditional generation

We can be creative and manually approximate the ground-truth distribution, for example, the `better_writer` from the last post. This strategy underpinned most classical program-synthesis algorithms. However, it is often simpler to _learn_ this approximation. 

To do so, notice this remarkable property of program synthesis: While it is difficult to generate correct programs from specifications, _it is easy to generate correct specifications from programs_.

## generating specifications from programs
Given a program, we can generate a specification for it by: (1) sampling some random inputs, (2) executing the program to obtain their corresponding outputs.

![Image with caption](/program-synthesis-primer/assets/generating-programs/sampling-spec.png ){: width="90%" }

## generating artifical dataset **D**

We can generate an **dataset** `D` by first generating a program, then generate its specification.

![Image with caption](/program-synthesis-primer/assets/generating-programs/generate-d.png ){: width="90%" }
