+++
date = 2025-07-29  # Removed quotes for proper TOML date
title = "Getting Started with AI Agents"
author = "Me"
showToc = true
TocOpen = false
draft = false
hidemeta = false
comments = false
description = "Desc Text."
canonicalURL = "https://canonical.url/to/page"
disableHLJS = true
disableShare = false
hideSummary = false
searchHidden = true
ShowReadingTime = true
ShowBreadCrumbs = true
ShowPostNavLinks = true
ShowWordCount = true
ShowRssButtonInSectionTermList = true
UseHugoToc = true

[cover]
    image = "<image path/url>"
    alt = "<alt text>"
    caption = "<text>"
    relative = false
    hidden = true

[editPost]
    URL = "https://github.com/<path_to_repo>/content"
    Text = "Suggest Changes"
    appendFilePath = true
+++

## Introductory Resources

Anthropic has this great introduction to **AI Agents** - from definition to basic patterns. This is the first post I read, and it gave me a general idea of what AI Agents are, and very importantly, the difference between AI Agents and workflows.  
[Building Effective Agents](https://www.anthropic.com/engineering/building-effective-agents)

Now, to actually start learning, I find this blog by Galileo a good guideline for the different stages (and complexities) in building AI Agents, and my learning is also pretty much down this path.  
[A Field Guide to AI Agents](https://galileo.ai/blog/a-field-guide-to-ai-agents?utm_source=reddit&utm_content=ai_agents)

---

## Step 1: Workflows

[YouTube Tutorial](https://www.youtube.com/watch?v=bZzyPscbtI8)  
I started with this YouTube tutorial (as I saw other Reddit posts recommending to start with this).

### Why this is a good project at this step:
1. It starts from the very basics of API calling, function calling, structured output and then explores different workflows (from Anthropic's article)
2. It uses pure Python and does not use any frameworks
3. Uses an easy example of [your example here]

### What I learned:
1. A workflow is essentially breaking down a problem into several steps, with each step taken care of by an LLM call and then chaining these calls together. What's crucial here is to specify the connections between different calls, and this is where structured output comes in.
2. Logging information is very helpful - it shows you in time which step the AI Agent is at, and is helpful for debugging
3. Note: Not all AI models support structured output. I started with this tutorial using Groq - as it's free and pretty good - but it does not support structured output, so I eventually switched to OpenAI.

### What I wonder:
What is the actual difference between workflow and agents?

---

## Step 2 - Route 1: Basic Agents (ReAct)

Obviously, Step 1 is not good enough, and although it is briefly introduced in Anthropic's article, it makes me wonder: what is the actual difference between a workflow and an agent. I think I found the answer to this question through this project:  
[Text2SQL Project (Chinese)](https://www.bilibili.com/video/BV1pu5KzrEc4/?spm_id_from=333.337.search-card.all.click&vd_source=c83eb829a20620fd4b1c0fb727b2dc88)  
It's a text2sql project with simple coding and agent structure. It's in Chinese though, but I'm sure it is possible to find similar tutorials in English.

### Why this is a good project at this step:
1. It's implemented in two ways - a basic way (workflow), and an optimized way (agent + other optimizations). This makes the difference between a workflow and agent very clear.
2. A great introduction to the ReAct (reasoning+action) framework
3. All data are preprocessed - there are no tedious data cleaning required
4. It uses a bit of Langchain, but I understood the code without any previous acquaintance with Langchain

### What I learned:
1. **The actual difference between workflows and agents**:
   - **Workflow**: A manually designed structured sequence of steps, calling the LLM at each step
   - **Agent**: An autonomous entity that can interpret inputs, generate and examine its own outputs, make decisions, and invoke actions dynamically

   **Example**:
   - *Workflow*:
     1. Extract schema from data
     2. Generate SQL (with user query and schema)
     3. Run SQL  
     (The steps and connections between steps are predefined by us)
   
   - *Agent*:
     1. Extract schema from data
     2. Generate SQL and wrap into a json request
     3. Decides whether to use tool run_sql_query
     4. Tool use
     5. If execution fails, goes back to (2); if it worked continues to (5)
     6. Decides whether to use tool file_saver (print the retrieved data into a file)  
     (Steps 2-5 are all entirely controlled by the LLM)

2. The **ReAct framework** - Thought (Reasoning) + Action. Seeing the output of the agent in the end makes it very clear how this framework is useful.
3. **Side knowledge #1**: Basics of SQL, SQLite, Schema - I previously had no knowledge of any of these, but now have a vague sense of how these things work and how to interact with databases.
4. **Side knowledge #2**: The project also optimizes schema with a more effective m_schema presented by Alibaba's Xi-yan. Although I didn't go into the details, it is interesting to see how the optimization works.

### What I wonder:
Now, I'm interested in Langchain. The ReAct framework (which is the reason that this process is automated) is coded in Langchain.
