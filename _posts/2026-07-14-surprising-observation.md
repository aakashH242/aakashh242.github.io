---
title: A Surprising Observation
description: While benchmarking Trimwise, we found that a smaller, query-aware context could match and sometimes, outperform, a full prompt without losing source provenance.
---

> **TL;DR:** We expected sending the full prompt to win every time. It did not. On our controlled benchmark, Trimwise Hybrid at 512 tokens matched the full-prompt answer-pass baseline on GPT-5.4 Mini and GPT-5.6 Luna, while beating it slightly on Nano. Trimwise Lexical got very close too, but at roughly 8 ms
instead of Hybrid’s ~52 ms. The bigger finding was not just score: token-level compressors can leave behind broken Markdown, clipped identifiers, damaged JSON, and context fragments that look relevant but are no longer safe to use. Smaller context is useful only
when it is still intact, traceable, and actually usable.

At [Tenwrite](https://github.com/tenwritehq), we are hard at work writing an internal agent that manages our SEO footprint
by creating and managing content. Naturally, for such a system to be able to learn constantly and stay updated with changing
SEO trends, it will have a lot of blogs to analyze. Now, we just cannot shove full blog content into every prompt. It works
for smaller blogs but, for larger ones, they result in excessive token usage and longer run times. We initially tried
the cursed `first N truncation` but the results came back to haunt us so bad we had to revert and think of other solutions.

Our problem was simple - too many huge blogs to process so, condense them into a token budget each with minimal reduction of information
density to hurt performance. The usual methods that we analyzed were LLMLingua, LongLLMLingua and RECOMP. Although their
main use-case is context compression, they still allow query-aware compression. But a few trials revealed a deeper problem - 
these methods remove a lot of tokens which affect information density quite significantly. This was especially visible when
the source texts had institutional knowledge the models had not trained on.

So, we did what any team would do after seeing this mess - we went digging.

The first thing we realized was that "context compression" is a very overloaded phrase. In one place, it means "make this prompt shorter". In another place, it means "extract the useful passages". In another place, it effectively
means "delete tokens until the model is okay with the input length". These are very different things, even though all of them get marketed under roughly the same name.

For our case, we did not want a summary. A summary is useful when a human asks, "what is this blog about?". But our agent may ask something far more specific later: what was the old recommendation for canonical tags in this particular
article? What example did the author give for internal linking? Was a caveat mentioned near the end? Did the blog say something that contradicts a newer internal document? These are not summary questions. These are retrieval and
evidence questions.

And this is where the usual "compress aggressively and hope" approach becomes dangerous.

A lot of blog content is repetitive. Great, remove repetition. A lot of it is filler. Great, remove filler. But the same blog can have one weird paragraph containing a piece of institutional knowledge, a client-specific exception, an
experiment result, or an old SEO decision that no foundation model has ever seen. Remove that one paragraph and the output can still look very clean, very short, and very wrong. This is the annoying part. The failure does not always
look like a failure. Sometimes the context is still grammatically fine. The agent simply loses the only thing that mattered.

We initially tried the usual family of solutions: LLMLingua, LongLLMLingua, RECOMP, and a few simpler baselines. We also had the classic first N truncation baseline because, well, everyone has to make that mistake at least once before
moving on with their life.

first N is not ideal for obvious reasons. It works suspiciously well when the answer happens to be near the top, which makes you think your benchmark is fine. Then the useful thing is near the end, or split between the beginning and the
middle, and suddenly your agent starts confidently answering a question with half the evidence missing. It is not really a compression strategy. It is a positional assumption pretending to be one.

The existing compressors were more interesting, but they came with a different class of problem.

They can be very aggressive at token removal. On normal prose, that can look okay at first. But on real working material - headings, Markdown, lists, code snippets, JSON, identifiers, links, technical instructions, weird internal
terminology - token-level removal can damage the actual shape of the source. We saw LLMLingua-family outputs where headings became glued fragments, identifiers got clipped, Markdown fences got damaged, and JSON started looking like
punctuation soup. Not "the model missed a sentence" bad. More like "the context is now technically present but no longer a thing you can safely hand to another system" bad.

For example, this kind of output is not useful provenance:

```
### 3aching reduces and. latency##
```

Neither is this:
```
`{-3 "_ ],
```

This matters a lot more than a generic question-answer benchmark will tell you. If your context is only prose, you may get away with it. If it contains rules, code, configuration, exact claims, links, citations, or source material
that needs to be quoted back later, you really cannot.

So we made a list of the things we actually cared about.

We wanted query-aware compression because the agent normally knows what it is trying to do. We wanted a hard output budget because "roughly shorter" is not a useful systems contract. We wanted the retained text to stay source-backed
instead of being quietly rewritten. We wanted the output to preserve source order. We wanted omissions to be visible instead of pretending two distant paragraphs were originally adjacent. And, because this was going into an agent
pipeline, we wanted to know exactly where every retained piece came from.

That last one was important enough that it shaped the API.

Trimwise returns source spans for the retained content. These are Python-string offsets into the original input: inclusive start, exclusive end. If Trimwise keeps two regions from a long blog, the caller gets two source spans in
source order. If Trimwise inserts an omission marker between them, that marker is intentionally not part of either source span.

Why do we care? Because now we can take a trimmed excerpt and still map it back to the original blog, the original document section, or the original repository file. We can keep paths and line references accurate. We can cite the
actual source instead of citing a synthetic compressed blob. And if we want to split a retained block further during final prompt assembly, we can do that without losing provenance.

That is the basic design of Trimwise: do the ranking work, but keep the source relationship intact.

Under the hood, the library starts by segmenting the input into structural units. It tries to respect real boundaries: headings, paragraphs, lists, fenced code blocks, source lines, and smaller fallback units when needed. Then it can
rank those pieces in a few different ways.

There is a structural mode when there is no useful query. There is a lexical mode for query-aware retrieval using term relevance. There is a semantic mode when the caller provides embeddings. And there is a hybrid mode which combines
lexical and semantic signals. We deliberately made embeddings caller-owned. We did not want the library to quietly decide which embedding model gets downloaded, loaded, cached, billed, or trusted inside someone else’s application. If
you already have an embedding stack, use it. If you want a small local model, use that. If you want a stronger remote one, that is your decision too.

The output is still source text. The ranking passages can have extra context to help the scorer understand where a paragraph sits in a document, but the final composition only uses the original slices. This sounds small, but it avoids
a very weird class of bug where a ranking helper accidentally leaks enriched context into the final prompt.

Then, naturally, we benchmarked it. And then naturally, the benchmark became its own project.

At first, we had a dataset that looked reasonable on paper and was absolutely not reasonable once we inspected where the evidence lived. Most answers were near the beginning. This made first N look much better than it deserved to
look. The benchmark was basically rewarding a method for preserving the part of the document we had accidentally made most important. That is not a result. That is us measuring our own dataset bias.

So we fixed it.

We built a position-controlled set of 160 cases:

- 40 where required evidence appears near the beginning
- 40 where it appears in the middle
- 40 where it appears near the end
- 40 where the answer needs multiple separate regions

The cases cover answerable content, instructions, procedures, structured/code-heavy material, adversarial material, and source-backed real content. We kept these tracks separate because one score cannot honestly describe all of them.

A short answer question should be measured differently from "did the required instruction survive?", which should be measured differently from "did these ordered procedure steps remain in order?", which should be measured differently
from "is this JSON/code/config block still exact?". Trying to force all of those through one fuzzy "quality" score is how benchmark dashboards become very pretty and very useless.

We tested 128-token and 512-token output budgets. We compared Trimwise Lexical and Trimwise Hybrid against LLMLingua, LongLLMLingua, and RECOMP. We then took the full contexts and compressed contexts and ran them through three
downstream evaluators: GPT-5.4 Nano, GPT-5.4 Mini, and GPT-5.6 Luna. The full prompt was kept as a reference line, not as a compressor, because obviously it has no compression latency or output budget to compare fairly.

The first result was pleasantly boring: Trimwise Lexical is fast.

On our controlled set, Lexical was around 8 ms median compression latency. Hybrid was around 52 ms because embedding work is not free. At 128 tokens, both were around the same source-retention level: roughly 50% macro case pass across
the different task tracks. At 512 tokens, Hybrid pulled ahead more clearly: about 62.8% macro source pass compared to about 58.4% for Lexical.

So there is a real trade-off here, not magic.

If you need low-latency query-aware compression and your source is mostly normal prose, Lexical is a very serious option. It gets you most of the way there at a fraction of the time. If you can afford the embedding step and care about
semantic matching across larger contexts, Hybrid starts earning its cost as the budget grows.

The downstream answer results were the more interesting part.

At 128 tokens, Trimwise Lexical was already very close to the full-prompt answer-pass baseline for all three evaluators. At 512 tokens, Trimwise Hybrid matched the full-prompt baseline on GPT-5.4 Mini and GPT-5.6 Luna, and slightly
exceeded it on GPT-5.4 Nano in this controlled setup.

That does not mean compression somehow makes a model universally smarter. It means that for these tasks, the compressed context often removed enough irrelevant material that the smaller context was at least as usable as the full one.
This is exactly the type of thing we wanted to know before wiring a compressor into an agent system.

RECOMP was interesting too. It preserves cleaner source pieces than the token-deletion approaches because it is selecting passages rather than aggressively shaving text within them. But in our run it was much slower, around 220 ms,
and it did not retain as much usable source material as Trimwise on the controlled set. This is not a dunk on RECOMP. It is trained around particular retrieval/compression objectives, and that matters. A compressor tuned for one kind
of QA corpus is not automatically the best fit for long SEO material, mixed Markdown, internal documentation, or repository excerpts.

LLMLingua and LongLLMLingua had another issue beyond score: budget compliance.

At the 128-token target, LLMLingua exceeded the budget on around 70% of cases. LongLLMLingua exceeded it on around 86.9% of cases. At 512 tokens, LLMLingua improved a lot, but LongLLMLingua was still over budget on a large chunk of
cases. This makes comparison awkward. If one method gets to use more context than the others, it may get an unfair quality advantage. In our run, it still did not perform well, but the point remains - a token budget should be a
contract, not a suggestion.

We also kept LLMLingua2 out of the main head-to-head chart. Not because it is unimportant, but because it is queryless. Our main question was: if every method receives the same source and the same query, which one gives us the best
context? Giving LLMLingua2 less information and then presenting it beside query-aware methods would not be fair.

We did run it separately. It performed poorly on the diagnostic set, but that result belongs in a queryless comparison, where it can be compared fairly with structural compression, Selective Context, and positional baselines. Mixing
all of those together would make a nice crowded graph and a bad conclusion.

There is another caveat worth saying out loud: the absolute answer-pass baseline was not amazing. Full prompt scored around 41.6% to 44.4% macro answer pass across the three evaluators. That does not automatically mean the models are
bad or that the compressor is bad. It means the benchmark is strict, the tasks are varied, and not every source task is naturally reducible to "produce one short answer matching this gold string". We therefore keep answer metrics
separate from instruction survival, procedure ordering, and exact structured-source preservation.

This is also why we are not going to publish a graph saying "Trimwise is 2x better". That would be silly. The interesting result is more specific:

For query-aware context assembly, Trimwise Lexical is extremely fast and preserves a lot of useful source. Trimwise Hybrid costs more but preserves more useful context at larger budgets. Both preserve source shape and provenance. In
our controlled set, they were substantially more usable than the token-level compression baselines we tested, especially when source boundaries mattered.

And source boundaries do matter.

If you are compressing generic English prose before asking a generic question, a slightly damaged sentence may not hurt you. If you are building an agent that works with institutional knowledge, SEO policies, old blog decisions, code,
URLs, configuration, or materials that need to be cited accurately later, you need a stronger guarantee than "the output kind of looks relevant".

You need to know what stayed, what got omitted, what got damaged, and where the retained text came from.

That is the actual reason we built Trimwise.

Not because long prompts are bad. Long prompts are sometimes exactly what you need. But because when context needs to get smaller, we wanted the process to be explicit, query-aware, budgeted, source-backed, and honest about what it
removed.

No haunted first N truncation. No clean-looking token graveyards. Just a smaller context that still knows where it came from.

