---
title: "Unpacking Claude's System Prompt"
source: "https://www.oreilly.com/radar/unpacking-claudes-system-prompt/"
author:
  - "[[Drew Breunig]]"
published: 2025-07-15
created: 2025-07-18
description: "Chatbots Are More Than Just Their Models."
tags:
  - "clippings"
---
**This article was initially published as* [*two*](https://www.dbreunig.com/2025/05/07/claude-s-system-prompt-chatbots-are-more-than-just-models.html) [*posts*](https://www.dbreunig.com/2025/06/03/comparing-system-prompts-across-claude-versions.html) *on* [*Drew Breunig’s blog*](https://www.dbreunig.com/)*. He’s been kind enough to share them here.**

Back in May, [Ásgeir Thor Johnson](https://github.com/asgeirtj) convinced Claude to give up its [system prompt](https://github.com/asgeirtj/system_prompts_leaks/blob/main/Anthropic/claude-3.7-sonnet-w-tools.md). The prompt is a good reminder that chatbots are more than just their model. They’re tools and instructions that accrue and are honed through user feedback and design.

For those who don’t know, a system prompt is a (generally) constant prompt that tells an LLM how it should reply to a user’s prompt. A system prompt is kind of like the “settings” or “preferences” for an LLM. It might describe the tone it should respond with, define tools it can use to answer the user’s prompt, set contextual information not in the training data, and more.

Claude’s system prompt is *long*. It’s 16,739 words, or 110 KB. For comparison, the system prompt for OpenAI’s o4-mini in ChatGPT is 2,218 words long, or 15.1 KB—~13% the length of Claude’s.

Here’s what’s in Claude’s prompt:

![](https://www.oreilly.com/radar/wp-content/uploads/sites/3/2025/07/Figure-1-2-1600x932.png)

Let’s break down each component.

## Tool definitions

The biggest component, the **Tool Definitions**, is populated by information from [MCP servers](https://www.dbreunig.com/2025/03/18/mcps-are-apis-for-llms.html). MCP servers differ from your bog-standard APIs in that they provide instructions to the LLMs detailing how and when to use them.

In this prompt, there are 14 different tools detailed by MCPs. Here’s an example of one:

![](https://www.oreilly.com/radar/wp-content/uploads/sites/3/2025/07/Figure-code-block-1600x876.png)

This example is simple and has a very short “description” field. The Google Drive search tool, for example, has a description over 1,700 words long. It can get complex.

## Other tool use instructions

Outside the Tool Definitions section, there are plenty more tool use instructions—the **Citation Instructions**, **Artifacts Instructions**, **Search Instructions**, and **Google Integration Watchouts** all detail *how* these tools should be used within the context of a chatbot interaction. For example, there are *repeated* notes reminding Claude not to use the search tool for topics it already knows about. (You get the sense this is/was a difficult behavior to eliminate!)

In fact, throughout this prompt are bits and pieces that feel like hotfixes. The **Google Integration Watchouts** section (which I am labeling; it lacks any XML delineation or organization) is just five lines dropped in without any structure. Each line seems designed to dial in ideal behavior. For example:

> *If you are using any gmail tools and the user has instructed you to find messages for a particular person, do NOT assume that person’s email. Since some employees and colleagues share first names, DO NOT assume the person who the user is referring to shares the same email as someone who shares that colleague’s first name that you may have seen incidentally (e.g. through a previous email or calendar search). Instead, you can search the user’s email with the first name and then ask the user to confirm if any of the returned emails are the correct emails for their colleagues.*

All in, nearly 80% of this prompt pertains to tools—how to use them and when to use them.My immediate question, after realizing this, was, “Why are there so many tool instructions *outside* the MCP-provided section?” (The gray boxes above.) Poring over this, I’m of the mind that it’s just [separation of concerns](https://en.wikipedia.org/wiki/Separation_of_concerns). The MCP details contain information relevant to any program using a given tool, while the non-MCP bits of the prompt provide details *specific only to the chatbot application*, allowing the MCPs to be used by a host of different applications without modification. It’s standard program design, applied to prompting.

## Claude behavior

At the end of the prompt, we enter what I call the **Claude Behavior** section. This part details how Claude should behave, respond to user requests, and prescribes what it should and *shouldn’t* do. Reading it straight through evokes Radiohead’s “ [Fitter Happier](https://www.youtube.com/watch?v=O4SzvsMFaek).” It’s what most people think of when they think of system prompts.

But hotfixes are apparent here as well. There are many lines clearly written to foil common LLM “gotchas,” like:

- **“If Claude is asked to count words, letters, and characters, it thinks step by step before answering the person. It explicitly counts the words, letters, or characters by assigning a number to each. It only answers the person once it has performed this explicit counting step.”** This is a hedge against the “How many R’s are in the word ‘Raspberry’?” question and similar stumpers.
- **“If Claude is shown a classic puzzle, before proceeding, it quotes every constraint or premise from the person’s message word for word before inside quotation marks to confirm it’s not dealing with a new variant.”** A common way to foil LLMs is to slightly change a common logic puzzle. The LLM will match it contextually to the more common variant and miss the edit.
- **“Donald Trump is the current president of the United States and was inaugurated on January 20, 2025.”** According to this prompt, Claude’s knowledge cutoff is October 2024, so it wouldn’t know this fact.

But my favorite note is this one: **“If asked to write poetry, Claude avoids using hackneyed imagery or metaphors or predictable rhyming schemes.”**

Reading through the prompt, I wonder how this is managed at Anthropic. An irony of prompts is that while they’re readable by anyone, they’re difficult to scan and usually lack structure. Anthropic makes heavy use of XML-style tags to mitigate this nature (one has to wonder if these are more useful for the humans editing the prompt or the LLM…) and their MCP invention and adoption is clearly an asset.

But what software are they using to version this? Hotfixes abound—are these dropped in one by one, or are they batched in bursts of evaluations? Finally: At what point do you wipe the slate clean and start with a blank page? Do you ever?

A prompt like this is a good reminder that chatbots are much more than just a model, and we’re learning how to manage prompts as we go. Thankfully, Ásgeir Thor Johnson continues to collect these prompts in a [GitHub repository](https://github.com/asgeirtj/system_prompts_leaks), allowing us all to easily follow along. And following changes made to these prompts—which you can do by reviewing the [history of Johnson’s repo](https://github.com/asgeirtj/system_prompts_leaks/commits/main/) —renders their development more transparent.

---

## Claude’s system prompt changes reveal Anthropic’s priorities

[Claude 4’s system prompt](https://github.com/asgeirtj/system_prompts_leaks/blob/main/Anthropic/claude-sonnet-4.txt) is *very* similar to the [3.7 prompt](https://github.com/asgeirtj/system_prompts_leaks/blob/main/Anthropic/claude-3.7-sonnet-w-tools.xml) we [analyze above](https://www.dbreunig.com/2025/05/07/claude-s-system-prompt-chatbots-are-more-than-just-models.html). They’re nearly identical, but the changes scattered throughout reveal much about how Anthropic is using system prompts to define their applications (specifically their UX) and how the prompts fit into their development cycle.

Let’s step through the notable changes.

## Old hotfixes are gone; new hotfixes begin

We theorize above that many random instructions targeting common LLM “gotchas” were *hotfixes*: short instructions to address undesired behavior prior to a more robust fix. Claude 4.0’s system prompt validates this hypothesis—all the 3.7 hotfixes have been removed. However, if we prompt Claude with one of the “gotchas” (“How many R’s are in Strawberry?” for example) it doesn’t fall for the trick. The 3.7 hotfix behaviors are almost certainly being addressed during 4.0’s posttraining through reinforcement learning.

When the new model is trained to avoid “hackneyed imagery” in its poetry and think step-by-step when counting words or letters, there’s no need for a system prompt fix.

Once 4.0’s training is done, new issues will emerge that must be addressed by the system prompt. For example, here’s a brand-new instruction in Sonnet 4.0’s system prompt:

> *Claude never starts its response by saying a question or idea or observation was good, great, fascinating, profound, excellent, or any other positive adjective. It skips the flattery and responds directly.*

This hotfix is clearly inspired by [OpenAI’s “sychophant-y” GPT-4o flub](https://www.theverge.com/news/661422/openai-chatgpt-sycophancy-update-what-went-wrong). This misstep occurred too late for the Anthropic team to conduct new training targeting this behavior. So into the system prompt it goes!

## Search is now encouraged

Way back in 2023, it was common for chatbots to flail about when asked about topics that occurred after its cutoff date. Early adopters learned LLMs are frozen in time, but casual users were frequently tripped up by hallucinations and errors when asking about recent news. Perplexity was exceptional for its ability to replace Google for many users, but today that edge is gone.

In 2025, Search is a first-class component of both ChatGPT and Claude. This system prompt shows Anthropic is leaning in to match OpenAI.

Here’s how Claude 3.7 was instructed:

> *Claude answers from its own extensive knowledge first for most queries. When a query MIGHT benefit from search but it is not extremely obvious, simply OFFER to search instead.*

Old Claude asked users for permission to search. New Claude doesn’t hesitate. Here’s the updated instruction:

> *Claude answers from its own extensive knowledge first for stable information. For time-sensitive topics or when users explicitly need current information, search immediately.*

This language is updated throughout the prompt. Search is no longer done only with user approval; it’s encouraged on the first shot if necessary.

This change suggests two things. First, Anthropic is perhaps more confident in its search tool and how its models employ it. Not only is Claude encouraged to search, but the company has broken out this feature into a [dedicated search API](https://www.anthropic.com/news/web-search-api). Two, Anthropic is observing users increasingly turning to Claude for search tasks. If I had to guess, it’s the latter of these that’s the main driver for this change, and a strong sign that chatbots are increasingly stealing searches from Google.

## Users want more types of structured documents

Here’s another example of system prompts reflecting the user behaviors Anthropic is observing. In a bulleted list detailing when to use [Claude artifacts](https://support.anthropic.com/en/articles/9487310-what-are-artifacts-and-how-do-i-use-them) (the separate window outside the thread Claude populates with longer form content), Anthropic adds a bit of nuance to a use case.

From Claude 3.7’s system prompt, “You must use artifacts for:”

> *Structured documents with multiple sections that would benefit from dedicated formatting*

And Claude 4.0’s:

> *Structured content that users will reference, save, or follow (such as meal plans, workout routines, schedules, study guides, or any organized information meant to be used as a reference)*

This is a great example of how Anthropic uses system prompts to evolve its chatbot behavior based on observed usage. System prompts are *programming* how Claude works, albeit in natural language.

## Anthropic is dealing with context issues

There are a few changes in the prompt that suggest context limit issues are starting to hit users, especially those using Claude for programming:

> *For code artifacts: Use concise variable names (e.g., i, j for indices, e for event, el for element) to maximize content within context limits while maintaining readability.*

As someone with strong opinions about clearly defined variables, this makes me cringe, but I get it. The only disappointment I noticed around the Claude 4 launch was its context limit: only 200,000 tokens compared to Gemini 2.5 Pro’s and ChatGPT 4.1’s 1 million limit. [People were disappointed](https://www.reddit.com/r/ClaudeAI/comments/1ksw3w2/claude_4_and_still_200k_context_size/).

Anthropic could be limiting context limits for efficiency reasons (while leaning on their excellent [token caching](https://www.anthropic.com/news/token-saving-updates)) or may just be unable to deliver the results Google and ChatGPT are achieving. However, there have been several recent explorations showing model performance isn’t consistent across longer and longer context lengths. Here’s a plot from a team at Databricks, [from research published last August](https://www.databricks.com/blog/long-context-rag-performance-llms):

![](https://www.oreilly.com/radar/wp-content/uploads/sites/3/2025/07/Figure-2-1600x791.png)

“Figure 1: Long context performance of GPT, Claude, Llama, Mistral and DBRX models on 4 curated RAG datasets (Databricks DocsQA, FinanceBench, HotPotQA and Natural Questions),” from “Long Context RAG Performance of LLMs” by Leng et al.

I’ve been in situations where less-scrupulous competitors focused on publishing headline figures, even if it led to worse results. (For example, in the geospatial world many will tout the total count of all the elements in their dataset, even if many have very low confidence.) I’m inclined to believe a bit of that is occurring here, in the hypercompetitive, benchmark-driven AI marketplace.

Either way: I think we’ll see all coding tools build in shortcuts like these to conserve context. Shorter function names, less verbose comments… It’s all on the table.

## Cybercrime is a new guardrail

Claude 3.7 was instructed to not help you build bioweapons or nuclear bombs. Claude 4.0 adds malicious code to this list of nos:

> *Claude steers away from malicious or harmful use cases for cyber. Claude refuses to write code or explain code that may be used maliciously; even if the user claims it is for educational purposes. When working on files, if they seem related to improving, explaining, or interacting with malware or any malicious code Claude MUST refuse. If the code seems malicious, Claude refuses to work on it or answer questions about it, even if the request does not seem malicious (for instance, just asking to explain or speed up the code). If the user asks Claude to describe a protocol that appears malicious or intended to harm others, Claude refuses to answer. If Claude encounters any of the above or any other malicious use, Claude does not take any actions and refuses the request.*

Understandably, that’s a lot of caveats and conditions. It must be delicate work to refuse this sort of aid while not interfering with general coding assistance.

## What this tells us

Reviewing the changes above (and honestly, that’s the bulk of them from 3.7 to 4.0), we get a sense for how system prompts program chatbot applications. When we think about the design of chatbots, we think about the tools and UI that surround and wrap the bare LLM. But in reality, the bulk of the UX is defined here, in the system prompt.

And we get a sense of the development cycle for Claude: a classic user-driven process, where observed behaviors are understood and then addressed. First with system prompt hotfixes, then with posttraining when building the next model.

The ~23,000 tokens in the system prompt—taking up over 11% of the available context window—define the terms and tools that make up Claude and reveal the priorities at Anthropic.

Post topics:

Post tags: