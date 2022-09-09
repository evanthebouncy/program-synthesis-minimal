---
layout: post
title:  "generating programs with language models"
permalink: /generating-programs/
---

With infinite compute, program synthesis is trivial -- just look through all programs for one that works. In practice, we need to find a working program with as few samples as possible. This post covers how to _generate_ programs using language modeling.

Let's start with a short re-cap of last post.

## the problem
The `spec` is: rectangle that keeps the grass coordinates inside, and mushrooms outside.
![Image with caption](/program-synthesis-primer/assets/generating-programs/spec.png "icons created by Freepik - Flaticon, monkik")

## the dream and the reality

Given this `spec`, let's look up its corresponding row `M[spec,:]` in the meaning matrix.

![Image with caption](/program-synthesis-primer/assets/generating-programs/a-row.png)

The **ground-truth distribution** for program synthesis can be constructed using `M[spec,:]`, with probability proportional to whether the entry `M[spec,prog]` is correct. This distribution gives uniform probability over correct programs, and 0 probability over incorrect programs.

![Image with caption](/program-synthesis-primer/assets/generating-programs/ground-truth.png ){: width="70%" }

The _dream_ is that we can easily sample from the ground-truth distribution. <ins>The _reality_ is that sampling from the ground-truth distribution is _hard_</ins> -- While we can easily check whether a given program is correct with respect to the spec, coming up with such programs is difficult. 

## the synthesis algorithm

In practice, we resort to _rejection sampling_ from this ground-truth distribution using synthesis.

![Image with caption](/program-synthesis-primer/assets/generating-programs/approximate.png ){: width="75%" }

This distribution is implemented by the synthesis algorithm -- a program writer _proposes_ somewhat plausible programs, then a checker _filters_ for correctness. 

![Image with caption](/program-synthesis-primer/assets/synthesis-problem/synthesizer-gut.png )

Thus, <ins>the better the writer (closer to ground-truth), the fewer programs the checker has to check</ins>. To implement a reasonable program writer, we turn to language models.

# language models

 A **language** is a set of strings (a sequence of characters). A **language model** assigns probabilies over the strings within a language. For a language model to be a good program writer, it needs to put high probabilities over strings that _resemble_ correct programs. We start with **unconditional language models** -- models that generates programs unconditioned on specifications. 

### all strings
We can generate a sequence by uniformly picking a random character a number of times. This language contains all possible strings of a certain length.
{% highlight python %}
def writer1():
    return ''.join(random.choice(string.printable) for i in range(9))
print(writer1()) # you should see something like "=1Vi![Au37"
{% endhighlight %}
A typical program string such as `[1,3,1,4]` has 9 characters, and each character has a probability of `1 / len(string.printable)`. Thus this writer has a `(1/100)^9` chance of stumbling across it.

### writable programs
We can restrict the generation to "strings that look like programs". This is done with a **domain specific language** (DSL), which identifies a "reasonable" subset of all-strings as programs. The DSL is specified by a **generator** -- a process capable of generating strings within the DSL.
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
By giving structure to the language, we increased our chance to `(1/6)^4` to encounter the program `[1,3,1,4]`, despite our DSL containing uninterpretable programs such as `[5,2,1,0]`.

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
What's our chances of stumbling across `[1,3,1,4]` now? I'll leave the combinatorics to you -- is `writer3` uniform across all legal programs? 

### unconditional generation is prior distribution of programs
The language models covered so-far are un-conditional language models, with no way to "steer" it into different distributions. This is a **prior distribution** -- how programs are naturally distributed, in the absence of specifications. One can use the prior `P(prog)` as a program writer as is.

![Image with caption](/program-synthesis-primer/assets/generating-programs/prior-synthesis.png ){: width="75%" }

This often resulting in a "good enough" synthesis algorithm for a small space of programs, and is among the first things a synthesis practictioner will try. We now turn to conditional generation.

<br>
<hr>
[Let's take a break](https://youtu.be/jQJVHjlvuVE). no seriously follow along and stretch! the next part will be intense.
<hr>
<br>

# approximating ground-truth via training

Rather than approximating ground-truth synthesis distribution by hand, it is often simpler to _train_ a parameterized program-writer `P_theta(prog|spec)`. The objective is this:

<br>
![Image with caption](/program-synthesis-primer/assets/generating-programs/approx-goal.png ){: width="90%" }
<br>
Which is to say, over some distribution of specs, minimize the KL distance between the ground-truth and the program-writer. 

To turn this objective into an algorithm, we leverage a remarkable property of programs: while it is difficult to generate correct programs from specifications, <ins>it is trivial to generate correct specifications from programs</ins>.

## generating specifications from programs
Given a program, we can generate a specification for it by: (1) sampling some random inputs, (2) executing the program to obtain their corresponding outputs.

![Image with caption](/program-synthesis-primer/assets/generating-programs/sampling-spec.png ){: width="90%" }

This is what it looks like in code:
{% highlight python %}
def sample_input():
    return random.randint(0,W-1), random.randint(0,W-1)

def sample_spec(prog):
    # pick a number of inputs, up to 10 patches of mushroom / grass total
    n_inputs = random.randint(1,10)
    inputs = [sample_input() for i in range(n_inputs)]
    prog = eval(prog) # needed since for this post progs are modelled as strings
    outputs = [is_inside(prog, input) for input in inputs]
    return list(zip(inputs, outputs))

r_spec = sample_spec("[1,3,1,4]")
print (r_spec) # [((5, 2), False), ((4, 4), False), ((0, 1), False), ((3, 3), True)]
{% endhighlight %}

## procedurally generating a dataset **D**

We can generate a **dataset** D by first generating a program, then generate its specification.

![Image with caption](/program-synthesis-primer/assets/generating-programs/generate-D.png ){: width="90%" }

Each point of the dataset follows this distribution:

![Image with caption](/program-synthesis-primer/assets/generating-programs/joint-sample.png ){: width="50%" }

This is what it looks like in code:

{% highlight python %}
def sample_D(n_samples):
    D = []
    for i in range(n_samples):
        prog = writer3()
        spec = sample_spec(prog)
        D.append((prog, spec))
    return D

D = sample_D(10000)
print (D[4542]) # should see something like ('[3,5,4,6]', [((1, 0), False), ((5, 0), False)])
{% endhighlight %}

## D is a sample of M

One should view <ins>the dataset D as a *sampled summary* of the meaning matrix M</ins>. While it is impossible to enumerate all entries of M, we can nonetheless take samples from it.

![Image with caption](/program-synthesis-primer/assets/generating-programs/sample_D.png ){: width="80%" }

## the approximation theorem of program synthesis

<br>
<ins>**theorem**: For a parameterized program-writer, we can use supervised learning on D to approximate the ground-truth synthesis distribution.</ins>

![Image with caption](/program-synthesis-primer/assets/generating-programs/approx-kl.png ){: width="90%" }
<br>

[Full proof typical in the style of variational inference, (X=spec, Y=prog)](/program-synthesis-primer/assets/generating-programs/proof.jpeg) with [kevin's approval](/program-synthesis-primer/assets/generating-programs/proof.png).

This is a typical result from most amortized inference set-ups, where the "forward" probability is simple and the "backward" can be learned. Nonetheless, this theorem has practical implications. 

1. (scaling) training the program-writer will approach the ground-truth distribution in the limit
    - given a big enough sample of D
    - given a flexible enough model class of program-writers
2. (procedural) generating D requires no human annotations
    - the entirety of M is defined by progs, specs, and the interpreter that relates them
    - attractive for self-play style of trainings

<br>
<hr>
[Let's take another break](https://www.youtube.com/watch?v=c--etqIJcow&ab_channel=RapidLiquid). We're almost done
<hr>
<br>

# conditional generation using unigrams
Let's put the theory into practice using our rectangle example. One of the simplest ways to generate programs is a **unigram distribution** where, conditioned on the `spec`, samples each attributes of the program _independently_. Note that we're accepting this flawed assumption (the parameters of the rectangles are definitely not independent) for computational simplicity. 

![Image with caption](/program-synthesis-primer/assets/generating-programs/factored.png ){: width="90%" }

Each of the T,D,L,R factors can then be treated as a multi-way classification problem. For no particular reason, we'll model it using `sklearn.linear_model.LogisticRegression()`.

### encoding the spec
We choose to encode the spec in the most naive way possible, as a linearized (WxWx2) grid, where the last "channel" indicate whether a coordinate in WxW is inside or outside the rectangle.

{% highlight python %}
def spec_to_bitvec(spec):
    bitvec = np.zeros((W,W,2))
    for coord,bool in spec:
        # turn bool into a number 0 or 1
        bool_num = 1 if bool else 0
        bitvec[coord[0],coord[1],bool_num] = 1
    # flatten the bitvec into a 1D array
    return bitvec.flatten()
{% endhighlight %}

### fitting the factors independently

{% highlight python %}
import sklearn.linear_model
# train the unigram distribution
def train_unigram(D):
    spec_bitvec, Ts, Ds, Ls, Rs = [], [], [], [], []
    for prog, spec in D:
        T, D, L, R = eval(prog)
        spec_bitvec.append(spec_to_bitvec(spec))
        Ts.append(T)
        Ds.append(D)
        Ls.append(L)
        Rs.append(R)
    # convert to numpy arrays
    spec_bitvec = np.array(spec_bitvec)
    Ts = np.array(Ts)
    Ds = np.array(Ds)
    Ls = np.array(Ls)
    Rs = np.array(Rs)
    model_T = sklearn.linear_model.LogisticRegression()
    model_T.fit(spec_bitvec, Ts)
    model_D = sklearn.linear_model.LogisticRegression()
    model_D.fit(spec_bitvec, Ds)
    model_L = sklearn.linear_model.LogisticRegression()
    model_L.fit(spec_bitvec, Ls)
    model_R = sklearn.linear_model.LogisticRegression()
    model_R.fit(spec_bitvec, Rs)
    return model_T, model_D, model_L, model_R
{% endhighlight %}

### making a program-writer using the fitted unigrams
{% highlight python %}
def get_writer4(model_T, model_D, model_L, model_R):
    def writer4(spec):
        spec_bitvec = spec_to_bitvec(spec)
        model_T_prob = model_T.predict_proba([spec_bitvec])[0]
        model_T_sample = np.random.choice(range(len(model_T_prob)), p=model_T_prob)
        model_D_prob = model_D.predict_proba([spec_bitvec])[0]
        model_D_sample = np.random.choice(range(len(model_D_prob)), p=model_D_prob)
        model_L_prob = model_L.predict_proba([spec_bitvec])[0]
        model_L_sample = np.random.choice(range(len(model_L_prob)), p=model_L_prob)
        model_R_prob = model_R.predict_proba([spec_bitvec])[0]
        model_R_sample = np.random.choice(range(len(model_R_prob)), p=model_R_prob)
        return '[{},{},{},{}]'.format(model_T_sample, model_D_sample, model_L_sample, model_R_sample)
    return writer4
{% endhighlight %}

## compare different language models

We can now create different program synthesizers by using different writers, including the manual `better_writer` algorithm from last post:

{% highlight python %}
synthesizer1 = get_synthesizer(lambda spec : writer1(), is_correct, 100)
synthesizer2 = get_synthesizer(lambda spec : writer2(), is_correct, 100)
synthesizer3 = get_synthesizer(lambda spec : writer3(), is_correct, 100)
synthesizer4 = get_synthesizer(get_writer4(*train_unigram(D_train)), is_correct, 100)
synthesizer5 = get_synthesizer(manual_writer, is_correct, 100)
{% endhighlight %}

The customary plot comparing different synthesis algorithms shows **search budget** on the x axis, and **fraction of tasks solved** on the y axis.

![Image with caption](/program-synthesis-primer/assets/generating-programs/synth-performance2.png ){: width="90%" }

As we can see, our fitted unigram writer performs even better than the manual solution.

## code

[All code for this post can be found here](https://gist.github.com/evanthebouncy/1703d3e9aee71ba9124405fdb30bd967). It depends on [rectangle.py](https://gist.github.com/evanthebouncy/25114aaf0be20df21468735aa7103bef)

## conclusion
In this post we covered how to obtain several reasonable program writers, including the generating from the prior, and by training on synthetically generated data.

-- evan 2022-08-29

## up next
The next post cover we'll cover how to fine-tune a large language model for the same task, which offers additional flexibilities. [let's go for it](/program-synthesis-primer/generation-with-llm/)