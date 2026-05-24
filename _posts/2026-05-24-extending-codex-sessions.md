---
title: Extending Codex sessions with CtxSift
description: The problem CtxSift is built to solve in long-running coding-agent sessions.
---

> TLDR: I didn't like Codex usages hitting limits in the middle of a session. Was looking for a way to save tokens and extend my usage. Wanted to target the two places where token usage occurs most - command outputs and state recollection. Other opensource solutions either targeted just one problem or was too heavy and complex for my use-case. Inspired by `Distill`, CtxSift came into being, extending upon those ideas.

With AI providers introducing stricter usage limits, I often found myself burning through the usage windows quickly.
Quite often, they would abruptly end in the middle of some work and I would be left writing it out myself while I waited
for the limit to reset.

I use Codex as my primary agent harness and, what I noticed was inconsistent session lengths. Sometimes, a few prompts
would blow through the 5 hour window while at other times, I could get more turns out of it before the limit hit. Upon a bit of
analysis, I found that the amount of tokens the agent sees and outputs contribute to how fast the usage gets over. The
sessions where it had to constantly refresh its recollection and state led to shorter usages while, the ones where it had
didn't need to perform many recollections went on longer. This recollection loop is noticeable especially after context
compaction events after which, agents mostly start re-reading files and re-running commands to get back to where they were.

## What I Was Looking For

Modern LLMs often do not need to view full command outputs to get what they need for a task or to reason.
It became evident that I'd have to control what the agent sees and how much time it spends re-exploring during state recollection.
I started looking at what was already available in opensource and came across a few brilliant projects -
[RTK](https://github.com/rtk-ai/rtk), [context-mode](https://github.com/mksglu/context-mode), [squeez](https://github.com/claudioemmanuel/squeez),
[LeanCTX](https://leanctx.com/) and [Distill](https://github.com/samuelfaj/distill) to name a few. They all had good approaches
to address the token wastage problem but, I felt that some added more complexity to the agents' workflow.

I wanted something simple and lightweight - minimal addition to the agents' workflow, no extra MCP server dependency with multiple tools,
run locally, no complex knowledge graphs and not just heuristics-based compression. I was not looking for a semi or full-fledged memory
system as Codex internally handles its own memory. What I looked for was something to complement this. `Distill` was the
closest to what I wanted - a command exposed as a skill - the agent runs a command and, pipes its output to distill
along with an instruction of what it wants from the output. Distill uses a language model to compress the output to only
what was asked for and returns it to the agent.

Distill example:

```bash frame="none"
$ pytest -q | distill "Return only the failing tests. No explanations."
tests/api/test_users.py::test_create_user_requires_email
tests/jobs/test_reconcile.py::TestReconcile::test_retries_deadlock
```

## What Was Missing

Distill worked well for output compression and I could see limits draining slowly...till a context compression event or when I asked something
unrelated to the current task flow. As the agent was "distilling" command outputs, its working memory did not store raw
details - which is a double-edged sword. On one hand, it keeps the memory small and reduces how much tokens it uses but,
on the other hand, the agent now has to re-read files and re-run commands to recollect and get back to a state from where
it can continue the given task. It shifted the token tax to recollection events! What I saved during distillation got used
during state recollection.

What was remaining was a simple way for the agent to recall its actions and recollect state faster and more efficiently.
I didn't want a complex knowledge management system or a full-fledged memory layer - they can get noisy fast. Instead, I
opted for a simple, local, workspace-scoped caching system which the agent could access with just one command - no multiple
commands, no chained tool calls - just one command with 3 optional flags. That became by vision for CtxSift - distill
commands when necessary, save those distilled results and let the agent recollect them when it needs context.

## Building CtxSift

My goal was to introduce an agent skill that plugs in two simple steps in an agents' workflow - `compress` and `recall`.
I wanted it to be local and have a simple caching system. I started building CtxSift as an extension of `Distill`'s capabilities
and, after a few iterations, here it is, ready for public use.

In its current state, CtxSift can use local and remote LLMs for compression and works with both CPU and GPU. For caching,
it uses SQLite with FTS5 and SQLite-Vec to store records. A hybrid retrieval pipeline with deterministic scoring and filtering
ensures fresh, grounded context is made available to the agent. It also maintains record freshness and superseded, older
records get marked stale as context updates.

I am really excited to share this with the broader community. Hope it helps you squeeze that extra bit out of your daily
sessions. Happy Sifting!



