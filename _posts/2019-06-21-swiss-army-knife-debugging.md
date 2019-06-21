---
layout: post
title: "Swiss Army Knives: Debugging in QEMU"
tags: [debugging][qemu][getting stuck in open source]
author: Aarushi Mehta
redirect_from: "/2019/06/15/swiss-army-knife-debugging"
---

### How to get stuck & all the times I did it
#### You don't understand the system you're writing code in
When you're new to a project, you don't often understand where code goes and what it does. Overcoming this is hard and takes patience.
When it comes to QEMU, my preference of places to look are documentation, mailing lists (discussions about code when it was written can be immensely helpful), google-fu of QEMU maintainer blogs and Stack Overflow, reading trace events and then asking.

I spent a lot of time being unable to figure out how exactly QEMU options were generated -- so I asked my mentor who directed me to the relevant files. I still didn't understand generation though, before I realised I could go through the commit history of how other features added their options.

#### You haven't walked through your code
My first review made me realise that I didn't understand how the engine I was implementing really worked. I was initially working from scratch, but had abandoned that effort after realising that I could mirror code from a similar engine. Except -- porting other code is often much harder than doing it yourself, you have to be sure you understand it how it works AND you are limited by its design choices

A significant majority of errors from that patchset came from this so please, walk through and rubber duck debug. Use some paper and sketch program execution.

#### You wrote code without designing it first (so hard not to do)
It becomes exceptionally dangerous to debug programs that were written without design plans in mind because you cannot DEBUG if you are not sure why the components do what they do -- it results in lots of wasted time because the design choices were random, so the temptation to make more random design choices is even higher.

### How to be less stuck
#### Stepping out of the problem
There's often a tendency to start wielding fancy tools on problems without understanding them in detail. I tried to read memory dumps of a guest that didn't exist because QEMU had bugged out. All I had to was breathe instead of panic google searching. Recognsing the problem and then setting out to fix it. I would have recognised immediately that QEMU had to be debugged, and the right tool for the job was not memory dumps.

#### Tracing & Debugging tools
I love gdb. May paeans be written to gdb.
You can drop fprintf's through your code but nothing beats the sweet sweet joy of single stepping through erroring code. It is very useful to use a debugging tool and use it well, the tools more uncommon features come in handy. LEARN DEBUGGING TOOLS!

#### Asking
Recognising when you are stuck beyond your own capacity to handle is very hard. I often just, don't. It's really hard to ask for help.

This section is a TODO, I don't yet have very good strategies for knowing when to ask and *simultaneously* avoiding the hot flush of shame of a stupid question.
Kudos to my mentors Julia Suvovra and Stefan Hajnoczi for helping me through those.

### Developing systems that prevent stuckiness
Currently, come back to this blog post and listen to your own advice. 
I am learning that it is exceptionally important to record the things you learn -- I have a growing swapfile of useful snippets. One weekend I'll write some personal documentation. The same is true for stuckiness.