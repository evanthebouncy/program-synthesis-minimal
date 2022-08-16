---
layout: post
title:  "generating plausible programs"
permalink: /generating-programs/
---

A good program synthesis algorithm boils down to the simple question -- Given a task, how do you generate a correct program with as few samples as possible? 

This post assumes you are familiar with the language and examples of [the previous post](/program-synthesis-primer/typical-synthesis-problem/).

## the dream

Let's revisit **M**. Given a spec, let's look at its corresponding row `M[spec,:]`.

![Image with caption](/program-synthesis-primer/assets/generating-programs/a-row.png "icons created by Freepik - Flaticon, monkik")

The dream is to be able to model this row exactly.

![Image with caption](/program-synthesis-primer/assets/generating-programs/ground-truth.png ){: width="50%" }
