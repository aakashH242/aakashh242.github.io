---
layout: post
title: "A skill issue"
date: February 27, 2026
categories: [blog]
---

> TL;DR: LLMs can accelerate development dramatically, but they also widen the ownership gap if you don’t understand your system deeply. Without discipline, agents tend to duplicate logic, bloat control files, and create subtle technical debt. Tooling like persistent “skills” helps, but clean architecture is still a developer responsibility.

Do you remember the old days when there were no LLMs and you wrote the entire code by hand? It took time to deliver but,
you had the bragging rights of knowing the entire system inside out. When something broke, you didn't just troubleshoot
blindly, you already had a suspicion of where the bug could be.

With the advent of the AI bubble, the pace at which code can be spewed out increased drastically! Now, more people can
*produce code*. It indeed was a revolution - now, non-tech folks could get a taste of the dopamine hits of seeing your
code work and the frustrations when it didn't. I mean, it got the general person to a terminal, what more can you ask for? 😄

I'm not going to lie - I use LLMs and coding agents in my workflow. While it has helped clean gaps in my architecture
and ship faster, it has also led to increased time in code review and reading through code and refactoring. As this 
[article](https://dev.to/ismail9k/once-upon-a-time-writing-code-was-fun-62) puts it,
**The Ownership Gap in Production** becomes a real problem if you do not understand entirely what your codebase does
and I want to avoid it at all costs. I need to know how my systems work.

## Repeats and Monoliths

My experience with AI-assisted development has been mixed. Yes, I enjoy the depth I can go to when hardening architecture
and the pace at which I can implement once I have locked on a spec. The rush of laying out the groundwork then, seeing
your feature get implemented is great. Tests work too, awesome! But now, review the additions and changes and compare
with specs - that's the boring part. 

Even if the implementation matches the spec, what I observed was the changes would be limited to a few files and a lot
of logic would be duplicated across them. Be it improving/fixing an existing feature or adding a new one, the few major
files containing control logic would grow into monoliths with huge functions all over, spaghetti-like call stacks and
repetition of same logic that could be generalized. The time I saved in writing the whole thing will now be spent in
cleaning up. I tried prompting to stay DRY and modular but thanks to context compression, in a few turns, the behavior
returned.

## [Agent Skills](https://agentskills.io/home)

Since prompts didn't help much, next step was to look at constructs that stay active in the agent's working context
window. I looked at Agent Skills and found the [clean-code-skills](https://github.com/ertugrul-dmr/clean-code-skills/)
that enforce Robert C. Martin's *Clean Code* principles. Unlike prompts that get compressed out of context, 
skills remain active in the agent’s working memory, influencing structure across turns.

I found it really useful and experienced the difference first-hand on the code quality and technical debt when using 
it for a fresh project. However, at times, I found that on existing or larger code bases, agents would tend to 
duplicate functionality and turn existing files into giants by cramming as much as possible into them. 

I added an extra skill to target specifically this problem - called it `clean-features` - and I found it useful. 
I thought of contributing it back and I have put in a pull request with the owner. 🤞

Meanwhile, if you are interested, you can use my [fork](https://github.com/aakashH242/clean-dry-code-skills). 
I shall keep it synced with the owner's upstream as I use this skill in my workflow.

## Parting Thoughts

In the end, this isn’t about AI. It’s about skills. Agents will focus on finishing tasks, not on making sense. 
If you don’t understand modularity, separation of concerns, and long-term maintenance, you will just create technical 
debt more quickly. AI doesn’t replace discipline; it highlights the problems that come from not having it.

**PS** - I have also started using [desloppify](https://github.com/peteromallet/desloppify) to monitor and maintain
code quality. 