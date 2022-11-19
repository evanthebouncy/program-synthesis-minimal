---
layout: post
title:  "这里有什么"
hidden: true
permalink: /what-is-synthesis-cn/
---

翻译：黄迪，濮烨文

# 你想了解程序综合么？
# so, you want to know about program synthesis?
最近（截至 2022 年），程序综合（“用AI来编程”）在工业界和学术界都逐渐开始流行起来，其中的佼佼者[copilot](https://github.com/features/copilot) 正在从学术原型走向实际生产（如果你从未接触过copilot，十分建议你试用它一下，这会让你对程序综合能干什么有更深刻的理解）。不过，程序综合的作用不仅仅是从注释中生成几行代码。程序（program）既可以很好的模拟各种类型的运算，也可以很好的作为一个人与机器沟通的媒介－它无处不在。而只要程序存在，对程序综合（也就是“用AI来编程序”）的需求就会随之存在。

Recently (as of 2022), the topic of "building AI to write code" has become fashionable in both industry and academia, with [copilot](https://github.com/features/copilot) making its way to production (if you have not used it, stop reading and play with it first). However, program synthesis is more than generating few lines of code from comments. Programs are broadly useful in modeling both computation and communication, and whenever programs exist, program synthesis is sure to follow. 

<ins>这个博客系列主要从程序综合的**实用**、**现代**方面切入。</ins>我刻意的牺牲准确性以使文章保持简洁易懂。博客中的案例牺牲了真实度，以此更快地向你介绍相关背景并使用现成的技术进行编程。博客的目标是让你对程序综合入门，使你可以阅读和复现学术文献，并将程序综合应用得有底气。

<ins>This blog series serves as a entry-point to the **practical**, **modern** aspects of program synthesis.</ins> I intend to keep the write-ups concise, at a cost of accuracy. The examples will be pedagogical, introducing you to a concept and coding it up using off-the-shelf technologies, at a cost of fidelity. The goal is 入门 (to get your foot through the door), so you can read and replicate academic literatures, and have the confidence to apply program synthesis to your own works. 

这些博客面向具有机器学习知识的本科/研究生水平，且希望了解程序综合的研究人员。我将假定读者具有一定的数学功底——但这并非必需，我会把算法和应用分开来讲，使得它们不互相干扰。

This blog is intented for undergraduate/graduate level researchers with knowledge of machine-learning, who want to "get into" program synthesis. Some level of mathematical maturity will be assumed, but not essential as the algorithms and applications are self-contained.

旅程开始~~

Let us begin.

## 何谓“编程”
## programming

在计算机（computer）问世之前，人们只能通过人力堆积的方式来完成任务（task）。苦也。

Before computers, when a person has a task to do, they manually does stuff. This is tiring.
![Image with caption](/program-synthesis-minimal/assets/what-is-this/doing.png "完成任务")

编程，就是人为了让计算机来完成某个任务而做出的一种操作。举几个例子：编辑linux系统是编程，创建一个Email过滤器是编程，我写这个博客同样也是编程。总体而言，编程这个操作必定由三个元素组成：程序员、程序和解释器。 **程序员（programmer）**将任务变成程序。**程序（program）**是一种东西，它能被程序员生成，也能被解释器理解。**解释器（interpreter）**接受并且运行程序，以此完成任务。

Programming is the act of a person asking the computer to do something. Hacking the linux kernel is programming. Creating an email filter is programming. Me setting up this blog is programming. One must always think of programming with these three: a programmer, a program, and an interpreter. A **programmer** turns a task into a program. A **program** is the thing that programmer gives, which the interpreter understands. An **interpreter** takes in the program and does stuff.

<!-- 
Programming consists of a **programmer**, who writes a **program**, which is executed on an **interpreter**. One must think about programming with all three in mind! -->
![Image with caption](/program-synthesis-minimal/assets/what-is-this/programming1.png "编程")

我们需要编程，是因为解释器（计算机）可以帮助我们完成艰巨的任务—人类无法以4GHz的频率同时执行不同的应用程序，人类无法手动整理几千封电子邮件，我无法亲手将博客打印出来并送到每个人手里…

We program because interpreters (computers) can help us with difficult tasks. A human cannot multiplex different apps at 4GHz. A human cannot sort through 1000s of emails manually. I cannot deliver the blog to all of you in person.

## 何谓“程序综合”
## program synthesis
程序综合是一个系统——这个系统让人们更容易编程。举几个例子：gcc使你不必从汇编开始编写程序，Jekyll生成网站使创建这个博客更容易。程序综合给予你一个更简易的编程环境，“程序++（program++）”（比如自然语言）。例如你只要说出“像这样过滤Email”这句话，就可以轻松地创建一个Email过滤器。在这一过程中，**综合器（synthesizer）**是一个中介程序员，它将program++转换成了program。

Program synthesis is a system that makes programming easier. gcc frees you from having to write assembly. Using "filter emails like this" makes creating a filter easier. Jekyll generating the website makes creating this blog easier. With program synthesis, the programmer can now program using "program++", which is easier to use than "program". The **synthesizer** is a middle-man programmer, which turns program++ into a program.

![Image with caption](/program-synthesis-minimal/assets/what-is-this/prog_plus.png "综合")

将综合器和解释器组合到一起，我们可以得到一个更好的解释器：“解释器++（interpreter++）”。它接受program++（通常被称为**样例（spec）**）并完成一些事情。这个过程是递归的：我们构建一个综合器/解释器，并得到了易于理解的program++语言；之后，我们在这一基础的上再构建新的综合器/解释器，得到了在program++的基础上更容易理解和编程的program++++语言。可以看到，程序综合实际上就是“让编程更简单”—也就是这个递归过程中的一个子步骤（从 program 到 program++）。因此，程序综合必须被放在“编程困难”这一语境下才有意义。现在让我们看一些例子。

 Together, the synthesizer and the interpreter becomes a better interpreter, the "interpreter++", which takes in program++ (commonly called a **spec** for specification) and does stuff. Note how this process can be recursive, where we build layers upon layers of synthesizers/interpreters, making it easier and easier for a human to program. Thus, program synthesis is simply "easier programming" -- one step of this recursive process -- and must be stated in relationship with the original "harder programming" context. Let's see some examples.

## 程序综合简史
## a brief history of program synthesis
严格来说，第一个综合器(synthesizer)是一个编译器(compiler)，它从高级语言样例（譬如FORTRAN）自动生成（汇编）代码。 在**演绎综合（deductive synthesis）**[^deductive]中，研究者使用数学规则来转换样例（“对数组进行排序”），从而得到一个可被证明是正确的程序（合并排序算法）。

The first synthesizers were compilers, which automatically generated code (assembly) from high-level specifications (FORTRAN). In **deductive synthesis**[^deductive], one transforms a specification (the array should become sorted) using mathematical rules, resulting in a provably correct program (the merge-sort algorithm).

大多数现代综合器是**归纳综合（inductive synthesis）**——即使用 _搜索_ 的方法来找到满足样例的程序。在神经/符号（neuro-symbolic）[^neurosym]程序综合中，我们使用神经网络进行搜索，神经网络网络会给出一些还算像那么回事的（符号 "symbolic"）程序。 最新（2020+）的进展 [^alphacode],[^codex] 经常利用自然语言（例如代码注释）作为额外的上下文来指导搜索（来生成 python 代码）。将自然语言转换为可执行的程序（如语音助手siri或者alexa）的主题一般属于**语意解析（semantic-parsing）**[^sem-parse],[^sem-tutorial]的范畴。

Most modern synthesis are **inductive synthesis** -- where the algorithm performs a _search_ to find a solution that meets the specification. In neuro-symbolic[^neurosym] program synthesis, the search is performed with a neural-network, which proposes plausible (symbolic) programs. Recent (2020+) advances [^alphacode],[^codex] frequently leverages natural language (such as code comments) as additional context to guide the search (to generate python code). The subject of transforming natural language to executable programs (think siri or alexa) traditionally falls under **semantic-parsing**[^sem-parse],[^sem-tutorial]. 

最后，由于程序在表示符号知识方面很灵活，认知科学中会使用程序进行认知的建模[^joshrule],[^probmods]。在这些理论中，“学习”可以被认为是“程序归纳（program induction）”的一种形式。

Lastly, as programs are flexible in representing symbolic knowledge, they have been used to model cognition [^joshrule],[^probmods], where learning can be thought of as a form of "program induction".

空间有限。更多的内容请参阅[^1],[^2]。

As we cannot afford the space to go into it further, kindly refer to [^1],[^2] for more history.

## 我们身处何处？
## where do we stand now?
成为程序综合领域的高手曾经具有相当的门槛——你需要是编程语言设计、编译器、约束求解、语义解析和针对代码生成的神经网络结构设计的专家。不过，伴随着基础模型(foundational model)的最新进展，我相信对于初学者来说这里面大多数的专业知识都不再那么要紧（这直觉也可能是错的）。通过阅读这个博客，我希望你可以学会如何用一个更加轻量级的工具箱来复现一个典型的程序综合从业者的工作流程。

Program synthesis had a fairly big 门槛 (entry barrier). You needed expertise in programming language design, compilers, constraint-solving, semantic-parsing, and designing specific neural-network architectures tailored toward code generation. However, with recent advances of foundational models, I believe (could be wrong) most of these expertise are no longer required to get started. With this blog, I hope you can replicate the workflow of a typical program synthesis practitioner with a lighter toolbox.

## 我们的目的地在哪？
## what is the end game?

我们可以采取[Josh](https://youtu.be/RB78vRUO6X8)（Joshua Tenenbaum，贝叶斯学派认知科学专家）的观点（既“你瞧人类思维多么🐂🍺”），让程序综合的终极目标为替代人类开发者（程序员）。这个系统可以将完全自然的、人与人的沟通转化为程序。程序综合的精髓是，通过设计，使样例(specification/program++)尽可能的人性化（比如自然语言），同时保证program++编辑成program的可行性（易于生成程序）。

We can take a [Joshian](https://youtu.be/RB78vRUO6X8) point of view (gesturing wildly at humans), and let the goal of program synthesis be building a system that can subsitute for a human developer (programmer) -- who turns fully naturalistic human-human interactions into working programs. The art of program synthesis is making program++/specification as humane as possible, while keeping the mapping between program++ and program tractable.

![Image with caption](/program-synthesis-minimal/assets/what-is-this/synthesis-ultimate1.png "人类-程序")

当然，我们不必受限于人类的编程能力。人类的能力是有限的……我不做人了，JOJO！！！……好吧，其实我们的目标只是打造一个比人更厉害的综合器（当然这也很强了）。

Of course, we don't have to be limited by human programming capabilities. We aim to build a synthesizer that can to go even further beyond!! AAaaaaaAaaA[AAaaaAA](https://youtu.be/3FM2kbvYljw?t=18)AAaaAaahhHHhHHhH (okay I'll stop).

## 练习
## exercise
在程序综合中，_用户意图（intent）_ 这个词总是出现，但很少被正式定义。选择几个不同的编程场景，并尝试用你的方式定义用户意图这个概念。考虑下这样两个问题：你定义下的用户意图传达给人类同胞容易吗？你定义下的用户意图传达给机器容易吗？

In program synthesis, the word _intent_ always shows up, and yet is rarely formally defined. Pick several different programming scenarios and attempt to formalize the notion of programmer's intent in your own words. How easy would it be to convey this formalized intent to a fellow human? How easy would it be to convey it to a machine?

-- evan  2022-08-11

## 接下来
## up next
接下来的博客介绍了如何构建一个经典的程序综合问题。[让我们开始吧~](/program-synthesis-minimal/typical-synthesis-problem/)

The next post covers how to set-up a typical synthesis problem. [let's go for it](/program-synthesis-minimal/typical-synthesis-problem/)

### 注释
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