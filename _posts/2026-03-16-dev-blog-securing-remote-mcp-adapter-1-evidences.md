---
layout: post
title: "Dev Blog - Proving the Remote MCP Adapter's Security Guardrails part 1"
date: March 16, 2026
categories: [blog]
---

> **TL;DR:** In the last post, I said v0.3.0 would harden the Remote MCP Adapter against poisoned tool metadata and weak session semantics. This post is the proof. I built a mutable mock MCP server, ran live adapter instances against it, and captured evidence for four security controls: tool-definition pinning, metadata sanitization, description minimization, and session-integrity binding.

In the [last post](2026-03-16-dev-blog-securing-remote-mcp-adapter-1.md), I wrote about four security issues I wanted to tackle in the [Remote MCP Adapter](https://github.com/aakashh242/remote-mcp-adapter).

That post was about design and intent.

This one is about evidence.

I did not want to stop at unit tests and say "trust me, it works." So I built a local test harness around a mutable FastMCP server and used it to exercise the adapter end to end. The result is a set of reproducible evidence artifacts showing what the adapter actually does when tool metadata changes, when descriptions are too verbose, and when a session is reused under the wrong authenticated context.

## What I tested

I ran four live security scenarios:

1. Tool-definition pinning and drift detection
2. Tool metadata sanitization
3. Tool description truncation and stripping
4. Session integrity hardening for stateful flows

Each scenario produced:

- the exact adapter config used for the run
- raw upstream tool snapshots
- raw adapter tool snapshots
- error payloads
- process logs
- SQLite state snapshots where relevant
- a per-scenario summary

So this was not a mocked "assert function returned true" setup. It was a real adapter process talking to a real mutable MCP server.

## The test setup

The setup was simple on purpose:

- a mutable FastMCP upstream server running locally
- one or more local adapter instances with scenario-specific config
- a runner script that switched upstream revisions, called the adapter, and saved the results

The upstream could change its catalog on demand. That made it possible to test:

- a benign first tool catalog
- a changed description later in the same session
- dirty metadata
- very long descriptions
- reused sessions after an auth-context change

This was exactly the kind of thing I wanted to prove before claiming the adapter had become a safer boundary.

## 1. Tool-definition pinning

This was the most important one.

The adapter was configured to:

- pin the first visible tool catalog for a session
- block mid-session drift
- invalidate the session when drift is detected

### What happened

The first `tools/list` call established the session baseline.

Then I changed the upstream tool description.

The next `tools/list` in the same session failed with a drift message. The adapter invalidated that session and refused to keep using it. Reusing the same session again resulted in `409 Conflict`. Starting a fresh session succeeded and picked up the upgraded catalog.

### What that proves

This closes the "rug pull" path where an upstream server can look safe during the initial review and then quietly mutate the tool surface after trust has already been established.

The trust boundary becomes:

- first catalog exposure pins trust for that session
- mid-session tool drift is not silently accepted
- a new session is required to accept upstream changes

That is exactly the behavior I wanted.

## 2. Tool metadata sanitization

The second scenario targeted dirty model-visible metadata.

The mock upstream exposed tool metadata with:

- decomposed Unicode
- zero-width characters
- dirty schema descriptions

I ran the adapter twice:

- once with sanitization enabled
- once with sanitization set to block

### What happened

With sanitization enabled, the adapter cleaned the visible metadata before forwarding it.

The differences were visible in the captured tool snapshots:

- dirty title -> normalized title
- dirty description -> normalized description
- dirty schema property description -> normalized schema property description

With `block` mode enabled, the dirty tool disappeared from the adapter-visible catalog entirely.

### What that proves

The adapter no longer has to behave like a naive tunnel for model-visible metadata.

It can:

- clean suspicious text conservatively
- or refuse to forward a tool whose metadata had to be changed

That gives operators a real first layer of defense against poisoned tool metadata.

## 3. Tool description truncation and stripping

The third scenario was about description surface minimization.

This is different from metadata sanitization.

Sanitization cleans obviously suspicious text. Description policy answers a different question:

> How much tool prose should the model see at all?

The mock upstream exposed very long tool descriptions and very long nested schema descriptions.

I ran the adapter in two modes:

- `truncate`
- `strip`

### What happened

In truncate mode:

- the top-level tool description was shortened to the configured limit
- the nested schema description was also shortened

In strip mode:

- the top-level tool description was removed
- the nested schema description was removed too

### What that proves

This is not just a UI tweak on the top-level tool description.

The policy applies to the model-visible description surface more broadly, including nested schema prose. That matters because otherwise an upstream could just move the same persuasive or poisoned text from the tool description into schema field descriptions.

So this control now works the way it should.

## 4. Session integrity hardening

The fourth scenario focused on session integrity.

This one matters because the adapter is stateful. It stores uploads, artifacts, tombstones, and other per-session state. That means session handling is part of the product's security posture, not just a transport detail.

For this test I:

- enabled adapter auth
- used disk-backed state persistence
- established one session with token A
- restarted the adapter against the same persisted state
- tried to reuse the old session with token B

### What happened

The old session had already been bound to the first authenticated context.

When I tried to reuse it under the rotated token, the adapter rejected it with `409 Conflict`.

Then I started a fresh session under token B, and that worked.

The persisted SQLite state showed separate trust-context fingerprints for the old and new sessions.

### What that proves

Knowing or reusing an `Mcp-Session-Id` is not enough on its own.

When auth is enabled, the adapter now binds the session to the authenticated context that created it. A stale session cannot just be picked up under a different auth context and treated as valid.

That is the right direction for a stateful gateway.

## What the evidence bundle contains

I saved the full evidence bundle and will attach it as a ZIP artifact.

> **Evidence bundle:** [Download the full ZIP artifact]({{ '/assets/evidence-pack-remote-mcp-adapter-v0.3.0.zip' | relative_url }})

It includes:

- a top-level report per scenario
- a machine-readable summary
- per-scenario configs
- per-scenario logs
- raw upstream and adapter snapshots
- persisted SQLite state snapshots

So anyone interested can inspect the actual evidence instead of relying on screenshots or paraphrases.

## One honest note

There is one small wrinkle in the captured client behavior.

When a blocked session is retried, the FastMCP client sometimes surfaces the failure as a generic `409 Conflict` instead of preserving the full response body each time.

That does not weaken the result, because the evidence still shows:

- the initial detailed block message
- the repeated `409` response
- the persisted invalidation or trust-binding state
- the fresh-session success path

Still, it is worth calling out plainly.

## Why this release matters

The Remote MCP Adapter started as a way to make remote MCP servers more practical by handling uploads, artifacts, and stateful mediation.

That is still true.

But once a gateway starts mediating tool metadata and storing session state, it can no longer pretend it is just a dumb transport wrapper. It is part of the security boundary whether it wants to be or not.

That is what v0.3.0 is really about.

Not security theater.
Not vague "hardened mode" marketing.

Actual controls.
Actual live tests.
Actual evidence.

## What comes next

I am not done with the security work yet.

But this release crosses an important line: the adapter is now starting to defend the boundary it creates, instead of just expanding it.

That was the goal.
