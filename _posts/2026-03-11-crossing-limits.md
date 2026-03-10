---
layout: post
title: "Crossing limits"
date: March 11, 2026
categories: [blog]
---

> **TL;DR:** Too many MCP tools in the context window slow agents down and worsen tool selection. GitHub tackles this with clustering and embedding-guided routing. In `remote-mcp-adapter`, Code-Mode avoids the problem by letting agents discover tools progressively instead of loading them all upfront.

The [Model Context Protocol](https://modelcontextprotocol.io/docs/getting-started/intro) has been both - a boon and a
curse for Agentic workflows. Yes, it allows you to connect your agent with diverse systems without needing to write
custom integrations for every one of them. But as your tasks grow in breadth and complexity, the more integrations you 
need hence, the more MCP servers you run. This eventually results in large of number of tools that get shoved into
your agent's context window. Maybe the agent needs just 4-5 tools to perform the task, but you still end up paying
the price for those extra tokens in the context window.

## Why is it harmful

If you look at it from the Agent's perspective, it sees

- the system instructions
- task specific instructions
- previous tool call results (if any)
- tool descriptions and schemas
- and finally, your task or the next task at hand

Do you see the problem? It sees the task at the end and, a lot of unrelated information beforehand can confuse even the
best of the LLMs. You'll notice an increased latency, a tendency to choose the wrong tools, losing context in between
and claiming a half-finished task as complete and an overall decrease in quality and consistency.

A simple way to restrict this issue is by limiting the number of tools you can have active in a request. GitHub Copilot
used to restrict to [128 tools](https://github.com/microsoft/vscode/issues) per request but after a lot of flak from
the community, they decided to remove that limit. So how did they solve the too-many tools problem? You can read about
it [here](https://github.blog/ai-and-ml/github-copilot/how-were-making-github-copilot-smarter-with-fewer-tools/).
In a nutshell, GitHub improved tool selection in Copilot by grouping tools into clusters and using 
embeddings to pre-select the most relevant ones, so the model doesn’t have to reason over hundreds of tools every time.

## A conscious trade-off

While working on my [remote-mcp-adapter](https://github.com/aakashh242/remote-mcp-adapter), I came to the realization
that my adapter will contribute to increased token usage. 

One of the adapter's functionality is to override tools that required file uploads from clients. While
overriding the tool, it also appends instructions on how to perform a staged-upload to the original description.
This is done so that the model knows about the original semantics and constraints but also aligns with the staged-upload 
procedure. This means for every upload-type tool configured, more tokens are sent in a `list_tools` call.
Although each upstream has its own MCP mount path, clients configured to connect to all upstreams would eventually get
hit by context bloat.

I did not want to replace the tool description entirely with the upload-staging instruction and risk losing
semantics so, I went for keeping only the first sentence of the upstream tool description trimmed to 50 token. I figured
the savings in tokens should make up for sacrificing a bit of semantics.

## Enter Code-Mode

After reading that [FastMCP 3.1.0 brought support for Code-Mode](https://www.jlowin.dev/blog/fastmcp-3-1-code-mode), I
was ecstatic. My adapter uses FastMCP so I could just implement a config toggle to enable code-mode.

Code-Mode allows the Agent to progressively discover the tools it needs without having to bloat the context window with
all tool definitions. Instead of all your 1000 tools, it surfaces 5 tools - `search`, `tags`, `list_tools`, `get_schema`
and execute. The Agent searches for certain keywords, discovers tools matching those, decides which ones to use, get 
their schemas then triggers an execute call. The video below will demonstrate Agent behavior without vs with code-mode.

<video controls>
  <source src="/assets/videos/code-mode-demo.mp4" type="video/mp4">
</video>

## Limits bypassed

With Code-Mode's progressive discovery integrated into **remote-mcp-adapter**, teams can configure as many upstreams
as they want (within their infra limits, of course). The latest 
[v0.2.0](https://github.com/aakashH242/remote-mcp-adapter/releases/tag/v0.2.0) release of 
[remote-mcp-adapter](https://github.com/aakashH242/remote-mcp-adapter) now includes Code-Mode. Let's see how it fares.

