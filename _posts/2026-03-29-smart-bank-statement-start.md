---
layout: post
title: "How I Got Into Building Smart Bank Statement"
date: March 29, 2026
categories: [blog]
---

> TL;DR: This was not my idea originally. Rupert had already started exploring the space when I got involved. Once I joined, we looked harder at the market, the actual workflow pain and ([aakashh242.github.io](https://aakashh242.github.io/blog/2026/03/05/remote-mcps-as-local.html)). What started as a broader finance direction became a much narrower product: take messy bank statement PDFs and turn them into structured, usable data. From there, we worked together to get the MVP out.

## How I came into it

An old college friend and roommate of mine, Rupert, had already started thinking in this space before I joined. So this was not one of those stories where two people sit down on day one with a blank page and magically arrive at the final product. The motion had already started. I entered after that, and once I did, my role became less about “coming up with the idea” and more about pressure-testing it, sharpening it and helping move it toward something that could become a real product.

At the time, the idea-space was wider. Like many things around finance, it is very easy to drift toward the flashy layer first: dashboards, summaries, personal finance views, spending insights, charts and all the things that look good in a demo. On paper, that feels like the obvious direction. People do want visibility into their money, after all.

The problem is that this part of the market is crowded and, more importantly, the pain is softer. There is a difference between a problem people find interesting and a problem they are willing to pay to make disappear. The more we looked at it, the more it felt like the “analyzer” route sat closer to the first category. Useful, maybe. Attractive, sure. But harder to build a serious business around unless there is a very strong edge.

## Where the idea started to tighten

So we kept looking.

The more practical side of the workflow started standing out. Not the part where somebody wants prettier insights. The part where somebody already has the data locked inside a bank statement PDF and needs it in a usable format for real work.

That was more interesting.

Because on the surface, converting bank statements to Excel sounds solved. It sounds like one of those dull utility problems the internet has already handled ten times over. But once you look at the actual inputs people deal with, the ugliness shows up quickly. Scanned statements, inconsistent layouts, broken table structure, different debit-credit conventions, weird balance columns, low-quality images, multi-page files, sometimes even multiple accounts in the same document. Suddenly this “simple conversion” problem stops being simple.

And that is before the downstream pain even begins.

Getting rows out of a PDF is not the same as getting reliable data. That distinction matters a lot more in accounting and bookkeeping workflows than it does in casual consumer use-cases. If the extraction is only mostly correct, somebody still has to sit there and verify the output line by line. One shifted row, one wrong amount, one broken balance trail and the time savings start collapsing. In these workflows, “almost correct” is not a nice middle ground. It is often just another form of manual work.

That was the point where the product started becoming more real to me.

## What changed once I joined

Once we teamed up, the conversation changed from “what can we build in finance?” to “what painful workflow exists here that people actually need solved?” That is a much better question, because it forces you to stop thinking in vague product language and start looking at where time is genuinely being lost.

We looked at existing players too. There were already tools in the market, obviously. Some looked dated. Some were too broad or enterprise-heavy. Some could handle easy statements but would struggle as soon as the documents became messy. Some could extract data, but still left enough cleanup and checking on the user that the problem was not really solved.

That gap mattered.

To me, the opportunity was never “nobody is doing this.” That is usually the wrong lens anyway. The real opportunity was that the problem was still painful enough, despite existing tools, that there was room for a more focused and more accurate product.

So the idea got narrower.

Not a personal finance dashboard. Not a generic document AI platform. Not a bloated accounting suite. Just a focused workflow: upload a bank statement PDF and get back structured output that is usable enough to save real time.

That kind of narrowing is easy to say and much harder to do.

## The build and the MVP

Once the direction became sharper, the implementation questions also became sharper. You stop thinking only in terms of OCR and start thinking about statement variance, normalization, row structure, balances, reconciliation, scanned versus digital PDFs, error detection and the difference between extraction that merely looks plausible and extraction that can actually be trusted.

That distinction shaped how we approached the product.

The goal could not just be “convert PDF to Excel.” There are too many ways to technically do that while still dumping the messy part back onto the user. The output had to be clean enough that it reduced work, not just moved work to a different stage.

That is what we built the MVP around.

Rupert had the initial seed. I joined once things were already underway. From there, together, we took it through the more difficult but more valuable phase: questioning the original direction, tightening the scope, understanding the market better and actually getting a usable MVP built instead of staying stuck in idea-land.

## Why I like this story more than the polished version

A lot of startup stories get rewritten after the fact to sound cleaner than they were. Two founders see a giant market, spot a perfect gap, align instantly and start executing with full clarity. Real life is usually more uneven than that.

This one certainly was.

The idea was already in motion before I came in. The initial space was broader than where we ended up. The clearer version of the product only emerged after spending more time with the pain, the market and the workflow details.

But honestly, I prefer that kind of story.

It feels more real. Better products often come out of that process: not from trying to sound ambitious from the beginning, but from being honest enough to keep narrowing until the pain becomes sharp and the value becomes obvious.

That is how I got into building Smart Bank Statement.

Not by inventing the idea from zero, but by joining an old friend, helping pressure-test it and then building with Rupert toward something much more grounded than where it began.
