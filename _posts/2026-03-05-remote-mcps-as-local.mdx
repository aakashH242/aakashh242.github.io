---
layout: post
title: "Remote MCPs as local"
date: March 05, 2026
categories: [blog]
---

> **TLDR;** Check out [remote-mcp-adapter](htts://github.com/aakashh242/remote-mcp-adapter) which provides stateful proxies for upstream MCP servers and handles the file exchange interaction with "file-touching" tools.

Recently, a use-case came along where we were tasked with hosting a central MCP platform. The idea was to bring all MCP
servers under a common umbrella, apply guardrails and governance on them and make it the go-to place for all things
MCP in the org. And it made sense, given the dangers unverified MCP servers in the wild pose. If we could provide
most of the tools teams needed and enabled a secure self-service model to add more servers,
we could push teams to use the org-approved MCP platform.

## The problem

Although mainly built for local usage, most MCP servers did implement the Streamable HTTP protocol, allowing them to
be hosted remotely. We did not run into much hiccups till we got around to adding servers that work with files -
either consume or produce them. An example is the [Playwright MCP](https://github.com/microsoft/playwright-mcp) server
that can produce artifacts in the form of console logs, screenshots, saved PDFs and consume local files for browser
uploads.

While other tools worked as expected, problems arose with the file-touching tools. Since the server and the agent
did not share a filesystem, artifacts generated would never reach the agent and whenever the agent needed to upload
files, the server would not find it.

## MCP constructs to the rescue

The MCP specs define a construct called [resources](https://modelcontextprotocol.io/specification/2025-06-18/server/resources)
to share data that provides context to language models. They are perfect for sharing the server generated artifacts with
the agents. However, file uploads require special handling too as the MCP specification
[does not support](https://github.com/modelcontextprotocol/modelcontextprotocol/issues/1306) file uploads via [elicitation](https://modelcontextprotocol.io/specification/draft/client/elicitation) yet.

I initially wrote a wrapper that acted as a proxy between the agent and the Playwright MCP server but pretty soon, need
arose to host many more of these types of "file-touching" MCP servers centrally. As a result, instead of writing separate
proxies for each, I wrote the [remote-mcp-adapter](htts://github.com/aakashh242/remote-mcp-adapter) which provides
stateful proxies for upstream MCP servers and handles the file exchange interaction with "file-touching" tools.

Being a consumer of opensource, I am hopeful it will be beneficial to the community.
