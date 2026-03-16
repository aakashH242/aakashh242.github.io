---
layout: post
title: "Dev Blog - Securing the Remote MCP Adapter"
date: March 16, 2026
categories: [blog]
---

> **TL;DR:** I built Remote MCP Adapter to solve remote file and artifact handling. Then I realized the same adapter could also be abused unless it actively defends against poisoned tool metadata and session misuse. So v0.3.0 is about turning that middleware into a safer boundary, not just a convenient one.

After finishing off the core work for the [Remote MCP Adapter](https://github.com/aakashh242/remote-mcp-adapter), I took
a step back and started sharing it in forums to see how others are solving the same issue. You can read more about the
inspiration behind it in [this blog](2026-03-05-remote-mcps-as-local.md).

I read this [blog on dev.to](https://dev.to/luckypipewrench/your-mcp-servers-tool-descriptions-are-an-attack-surface-37pj)
that talks about how the MCP protocol has an attack layer via tool descriptions. This issue might not affect folks 
running MCP servers locally but becomes a headache for teams and organizations wanting to host them centrally. An
attacker can essentially poison tool descriptions or manipulate tool arguments to make the Agents perform sinister
stuff! The author of that post has built a tool, [Pipelock](https://github.com/luckyPipewrench/pipelock) that acts as
a firewall for AI agents. Do show some love to his work!

## The realization

While pondering over the article, I realized that I have created a monster that could easily be used to infiltrate 
systems. And it's out in the wild! So I'm taking the next logical step - learn from the blog, explore MCP attack
surfaces and provide built-in defense against these attack surfaces.

While writing the v0.3.0 release, I am focusing on addressing four issues - 

1. [Tool definition pinning and drift detection](https://github.com/aakashH242/remote-mcp-adapter/issues/22) - when enabled, the adapter will baseline tools during the first `list_tools` call for a session. Any drifts or changes in tool titles, schemas and descriptions in subsequent calls will be detected and either warned or blocked entirely.
2. [Normalize and sanitize tool schemas before forwarding](https://github.com/aakashH242/remote-mcp-adapter/issues/23) - add a metadata preprocessing feature be able to apply a conservative sanitization step to model-visible tool metadata before that metadata reaches the client or model.
3. [Tool description minimization/stripping](https://github.com/aakashH242/remote-mcp-adapter/issues/24) - allow users to minimize or remove tool descriptions altogether - for those extra-secure environments.
4. [Harden adapter-managed session semantics for stateful HTTP/SSE flows](https://github.com/aakashH242/remote-mcp-adapter/issues/25) - once the adapter stores uploads, artifacts, cancellation state, or other per-session data, session integrity is no longer just an MCP transport concern. It becomes part of the product's own security posture. The goal is to make sure the adapter's own stateful features cannot be misused just because a session ID is known.

I can see a long night up ahead as I gear up to write and test these guardrails out. I shall publish my test results here
in a new blog once I have them ready.
