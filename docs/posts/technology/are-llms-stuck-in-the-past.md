---
date:
  created: 2025-01-22
authors:
  - bruce
categories:
  - Technology
tags:
  - llm
  - chatgpt
  - rag
---

# Are LLMs Stuck in the Past?
Large Language Models (LLMs) like ChatGPT and LLaMA represent groundbreaking advancements in technology. But as impressive as they are, are they living in the past?

![Living in the Past](living-in-the-past.png)
image by DALL-E
<!-- more -->

## The Rise of LLMs
ChatGPT has revolutionized how we perform research and search for information. For many tasks, it has replaced the traditional search engines. While I still use Google, the conversational, iterative model of LLMs offers an experience I prefer. 

## The Problem: Outdated Information

Despite their strengths, LLMs have a critical limitation: outdated knowledge. This was especially evident in earlier iterations, such as ChatGPT-3.5, which lacked **web search** capabilities. Even now, users relying on the free version of ChatGPT face this challenge.


For example, previously when I asked what was the current long-term support (LTS) version of Node.js, the older ChatGTP-3.5 model confidently gave me outdated information (v14.x instead of the correct v18.x). While not a **hallucination**, this inability to access current data highlighted a significant gap.

## Solutions: Web Search (RAG)

For Plus subscribers, **web search** capabilities can partially address this issue. By allowing the model to augment its answers with real-time search (retrieval-augmented generation or RAG), it can provide more accurate and current responses. However, this approach has its downsides:

 - Cost: **Web search** requires the Plus subscription, which costs $20 per month.
 - Longer Response Times: Searching the web takes extra time.
 - Data Misinterpretation: The model may occasionally misrepresent search sourced information.

## The Importance of Knowledge Cutoff Dates

Understanding the knowledge cutoff dates for different models is essential. Here is a quick reference:

| Model | Knowledge Cutoff |
| --- | --- | 
| ChatGPT 3.5 | 2021 |
| ChatGPT 3.5 +/Turbo | January 2022 | 
| ChatGPT 4 | October 2023 |
| ChatGPT 4o | October 2023 |
| o1-mini | October 2023 |
| o1 | December 2023 |
| LLaMA 2 | 2023 |

(Source: Wikipedia, OpenAI Community Forum)

## Asking for Sources and Citations

When seeking time-sensitive or critical information, consider asking your LLM to include references or citations. This helps assess the reliability and how current the data is. You can also curate the results by specifying which sources to include or exclude, ensuring more authoritative and relevant results.

## Summary

LLMs are transformative tools, but their limitations—such as outdated training data must be understood. Leveraging features like **web search** and adopting better prompting strategies can help mitigate these issues, unlocking their full potential.