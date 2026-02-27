---
layout: post
title: "The curse of context windows"
date: February 20, 2026
categories: [blog]
---

> **TL;DR**: Large-document extraction with LLMs fails less from “bad reasoning” and more from hard output limits. JSON structured outputs waste tokens on repeated keys and still truncate on big PDFs. Switching to CSV reduces overhead but doesn’t fix truncation—your output can still cut off silently. The reliable fix is chunking the document into page batches, processing chunks asynchronously with strict concurrency limits (semaphores), and stitching results back in order; run summarization as a separate pass.

I was working on a problem to extract structured information from large documents on very lean infrastructure.
The input documents were either PDF, CSV or Excel. For CSV and Excel, extraction was pretty straightforward but PDFs
posed a separate challenge of their own.

The PDFs we ingested were mostly a mix of digital and scanned. The moment scanned PDFs come in to the picture, one can
imagine the various edge cases that come with them - image quality, noise, orientation, spillovers etc. 

OCR was the obvious option and with so many opensource libraries available, we were spoilt for choices. I 
wanted to use [Docling](https://docling-project.github.io/docling/) as my prior experience with it has been good so
far (I shall write a separate blog on those use-cases) but we were constrained by the infra.

Docling uses deep learning to extract structure - something very costly on just CPU. I ran a few tests and it was
evident that we needed another approach - we just cannot wait hours for a document to process.

## Iteration 1 - LLM with Structured Outputs

Latest LLMs are now capable of processing documents, why not use them? After evaluating the major providers and models,
we chose **Gemini 2.5 Flash**. It ticked all the checkboxes we had - 

- [x] Large context window to support huge PDFs (1 million tokens)
- [x] Fast output speed
- [x] Controllable reasoning
- [x] Affordable at scale

Our first approach was using structured outputs. The data to extract had to follow a certain schema and also have
a kind of "summary". Seemed a perfect use-case for outputs following a strict JSON schema.

Well, it worked well for smaller documents. As soon as documents grew beyond a certain size, the output parsing would
fail. Why? Because the output exceeded the model's maximum allowed output tokens. Setting this to max did not help
either.

## Iteration 2 - Why waste tokens on repeated extra characters?

We realized that a lot of the output tokens were being consumed as JSON field names and characters.
Rows of data, same schema = same identifiers repeated for every row. A real waste, in my view. 
So we took a different route.

Since the extracted data had to follow a certain schema, we took a two-step approach. 
Make the LLM output raw CSV in one pass and make it generate the summary in another parallel pass.
CSV is much condensed than JSON and should solve the parsing problem, right?

Well, it kind of did eliminate the parsing problems and the CSV output would parse successfully too. But on a closer
inspection, it revealed that the problem had not been solved - just masked by the CSV parser. The LLM would return
complete rows but for large documents, rows would suddenly stop. The output was still truncated - we just didn't catch
it this time as there were no parsing errors.

## Iteration 3 - Divide and conquer!

Taking a leaf from the old-school world of batch automations, we thought - instead of the entire document,
why not pass batches of pages to the LLM? This could enable us to target two birds with one stone:

- the issue of truncation
- really slow speeds when processing large documents (the more the LLM has to output, the longer the wait time)
 
So we implemented a pipeline that chunks documents into batches, process
them asynchronously, collect the results and perform order-aware stitching to produce the final output. The separate
summarization pass still stayed. 

We tested this on large documents and found that the truncation problem was gone! Now we were able to process large
documents at respectable speeds without the fear of losing data. 

## What I Learned

- Structured outputs is good but only when constrained
- For large-scale data extraction using LLMs, not the reasoning capability but the output token limit is your enemy
- Asynchronous processing for I/O heavy tasks can go out of bounds and cause all sorts of issues - from rate-limits to memory spikes. Always have semaphores
- Thinking tokens matter - especially for large documents:
  - For one-to-one copy-paste extraction, with a few prompt engineering tricks, you can get the LLM to follow order and output the copy exactly as the source without using thinking tokens
  - For extractions which involve copy-paste, transformations and data inference, with vs without thinking tokens produce noticeable differences in the transformed and inferred data