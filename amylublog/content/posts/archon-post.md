+++
date = 2025-07-29
title = "Archon"
author = "Amy Lu"
showToc = true
TocOpen = false
draft = false
hidemeta = false
comments = false
canonicalURL = ""
disableHLJS = false
disableShare = false
hideSummary = false
searchHidden = false
ShowReadingTime = true
ShowBreadCrumbs = true
ShowPostNavLinks = true
ShowWordCount = true
ShowRssButtonInSectionTermList = true
UseHugoToc = true

[cover]
    image = ""
    alt = ""
    caption = ""
    relative = false
    hidden = false

[editPost]
    URL = "https://github.com/amylu0828/blog/blob/main/amylublog/content/posts/archon-post.md"
    Text = "Suggest Changes"
    appendFilePath = true
+++

My first learning experience with a full stack project with AI agents is with this project  https://github.com/coleam00/Archon.

This project is aimed at building an agent with Pydantic AI and Langraph that builds other agents.

This is the hands on best project for beginners who just had some encounterance with Agentic workflows, but has only built small-scaled demos and is wondering how these workflows can be turned into actual products/applications.

The best part of this project to me is that it includes 6 iterations. This gradual process of adding features and optimizations make it easier for me to follow - if you were to throw at me the final version at the very beginning I would not have understand. So, let’s get started.

---
# Version 1: Simple Pydantic Agent

Version 1 is a simple Pydantic AI Agent with RAG. The major point here is the RAG, as, to let your agent be able to generate code with Pydantic_ai, it must know the syntax of Pydantic_ai and perhaps some examples in building with that framework. This is provided with RAG, by retrieving crawled pydantic_ai documentation from a database. 

So, the workflow is as such:

1. Crawl pydantic_ai docs into Supabase
    1. Get urls from pydantic ai sitemap
    2. For each url…
        1. Fetch the content in the webpage
        2. Process and store the content
            1. Split the text into reasonable chunks
            2. Process the chunks - extracting title and summary, generating embeddings
            3. Insert into the database
2. Streamlit Frontend
    1. Collect message history
    2. With the user’s input
        1. Start streaming with pydantic_ai_coder and collect the result
        2. Output the result in real time
        3. Add the result to the message history
3. Pydantic AI Coder
    1. Tools:
        1. retrieve relevant documentation - find the relevant documentation by embedding the user’s query
        2. list documentation pages - list all the available pages
        3. get page content - get the content from an url

---
# Version 2: Orchestration with Langgraph

Version 2 optimized on version 1 by breaking up the agentic task into a workflow. Karpathy mentioned in [https://www.youtube.com/watch?v=LCEmiRjPEtQ&t=1373](https://www.youtube.com/watch?v=LCEmiRjPEtQ&t=1373s)  that currently, the best agents are those that support an effective AI-Human coorperation. AI excels at generation but is not yet ready to take full control and should always be supervised by humans. Therefore, the best approach is to “keep AI on a leash” by incorporating human verification throughout the process.

Version 1 allowed too much autonomy to the AI—it decided entirely what to do first, which tools to use, and so on, without any guidance or human oversight. However, for AI coding tasks, certain steps are clearly necessary and should be made mandatory. Additionally, human supervision must be integrated into the workflow.

This leads to two possibilities for optimization:

1. **Implement restrictions on AI** by using built-in workflows that break down tasks, reducing its autonomy and guiding its actions.
2. **Incorporate human-in-the-loop supervision** so that users can monitor the workflow and make decisions at critical points.

Thus, version 2 built, upon version 1, a workflow with langgraph…

![Archon Langgraph Workflow](/images/posts/archon-post/archon-langgraph-workflow.png)

**Scope Reasoner:** has a view of the use query and all the documents collected about Pydantic AI, and with these, it generates a scope including the architecture, core compoennts, external dependencies, and lists the urls it finds useful for the coder agent to use.

**Get Next User Message:** uses langgraph’s human in the loop feature interrupt() to catch the user input from the frontend, and then resume the thread.

---
# Version 3: Adding MCP

Previously, I did not fully understand the purpose of MCP. Like the question in this post, https://www.reddit.com/r/ClaudeAI/comments/1h0w1z6/model_context_protocol_vs_function_calling_whats/, I didn’t really see the difference between MCP and Function Calling. I’ve also heard that MCP does not have much use for small personal projects, so I was spectulative in what it can do.

However, I think v3 got me a lot closer to understanding what MCP is.

Suppose I want to deploy this Pydantic AI generator to an IDE like Cursor, allowing me to use the generator within the Cursor chat and run the generated code directly. Previously, implementing this integration would have been difficult. However, since Cursor supports the Model Context Protocol (MCP), the process is now much simpler. All I need to do is build an MCP server for my agent, expose the agent’s calling function, and Cursor can then directly communicate with my agent to request and retrieve results.

Thus, while the Streamlit UI exposes my agent directly to users through a chat interface, the MCP exposes my agent to AI IDEs.

<div align="center">
  <img src="/images/posts/archon-post/archon-v3.png" alt="Archon MCP" width="500"/>
</div>

---
# Version 4: Complete Streamlit UI

With all the features listed above, v4 constructs a complete frontend with the following tabs: introduction, enviornment, chat, database, documentation, agent service, mcp.

<div align="center">
  <img src="/images/posts/archon-post/archon-v4-graph.png" alt="Archon V4 Graph">
</div>

One major concept that is added:

1. **Real-Time Progress Tracking for Long-Running Crawling Tasks**

When building a web crawler or any long-running asynchronous task, it's crucial to provide **real-time feedback** to users. Since crawling can take a significant amount of time, users naturally want to see the current progress instead of waiting blindly for completion.

To address this, we use a **Progress Tracker** that acts as a bridge between the backend crawling process and the frontend user interface (UI). The main idea is:

- The **UI passes a callback function** to the crawling thread.
- The **crawling thread creates a Progress Tracker instance**.
- The tracker exposes methods like `.log()`, `.start()`, `.complete()`, and `.get_status()`.
- Each time one of these methods is called during crawling, the tracker uses the callback to **send updated status information back to the UI**.
- The frontend stores this status and updates the display in real time, showing progress bars, logs, counts, and other useful metrics.

```python
#documentation_tab.py

def update_progress(status):
	st.session_state.crawl_status = status

st.session_state.crawl_tracker = start_crawl_with_request(update_progress)
st.session_state.crawl_status = st.session_state.crawl_tracker.get_status()
```

```python
#crawl_pydantic_ai_docs.py

class CrawlProgressTracker:
    def __init__(self, progress_callback: Optional[Callable[[Dict[str, Any]], None]] = None): #Callable[[input_type], output_type]
        self.progress_callback = progress_callback
        self.urls_found = 0
        self.urls_processed = 0
        self.urls_failed = 0
        self.urls_succeeded = 0
        self.chunks_stored = 0
        self.logs = []
        self.is_running = False
        self.start_time = None
        self.end_time = None
    
    def log(self, message: str):
        current_time = datetime.now().strftime("%H:%M:%S")
        self.logs.append(f"[{current_time}] {message}")
        print(message)

        #call progress_callback if provided
        if self.progress_callback:
            self.progress_callback(self.get_status())
    
    def start(self):
        ...

        if self.progress_callback:
            self.progress_callback(self.get_status())
    
    def complete(self):
        ...

    def get_status(self) -> Dict[str, Any]:
        return {
            "urls_found": self.urls_found,
            "urls_processed": self.urls_processed,
            "urls_succeeded": self.urls_succeeded,
            "urls_failed": self.urls_failed,
            "chunks_stored": self.chunks_stored,
            "progress_percentage": self.urls_processed / self.urls_found * 100 if self.urls_found > 0 else 0,
            "logs": self.logs,
            "start_time": self.start_time,
            "end_time": self.end_time
        }
    @property #used to define a method as a "getter" for a class attribute, allowing you to access it like a property (without parentheses)
    def is_completed(self) -> bool: 
        return not self.is_running and self.end_time is not None
    
    @property
    def is_successful(self) -> bool:
        return self.is_completed and self.urls_failed == 0 and self.urls_succeeded > 0
        
...

def start_crawl_with_requests(progress_callback: Optional[Callable[[Dict[str, Any]], None]] = None):
    tracker = CrawlProgressTracker(progress_callback)

    def run_crawl():
        try:
            asyncio.run(main_with_requests(tracker))
        except Exception as e:
            print(f"Error starting crawling: {str(e)}")
            tracker.log(f"Error starting crawling: {str(e)}")
            tracker.complete()
    
    thread = threading.Thread(target=run_crawl)
    thread.daemon = True #allows the thread to exit when the main program exits
    thread.start()
```
---
# Version 5: **Optimizing Archon: Smarter Crawling and Streamlined Agent Coordination**

While the previous setup looked solid in theory, when I actually ran Archon, its performance was underwhelming. The code it generated for building Pydantic AI agents didn’t really seem like it had *read* the documentation we gave it. For example, check out this snippet — it defines a Pydantic AI agent in a way that’s completely different from the docs, and it doesn’t even import `pydantic_ai`.

<div align="center">
  <img src="/images/posts/archon-post/archon-v5-p1.png" alt="Archon Performance">
</div>

That made me suspect something was off with the **RAG** part. Since RAG is supposed to be the heart of this project — the part that helps the agent actually *use* the docs — it seemed like the agent wasn’t really accessing or understanding the documentation properly.

---

### **Problem 1: Missing Important Docs**

Here’s a quick refresher on the tools the coder agent has:

1. `retrieve_relevant_documentation` — looks up docs based on how similar they are to the user’s query.
2. `list_documentation_pages` — lists all the documentation pages available.
3. `get_page_content` — fetches the content of a specific page.

`retrieve_relevant_documentation` is the one used most often. But the problem is, it doesn’t always bring back *all* the important docs. For example, if a user says:

```markdown
User: Build me an Weather Agent that can tell me the weather of a certain place at any time.
```

The system, with embeddings, might pull up docs about “Weather Agent” or “weather at a place,” but it could easily miss key pages like `/agent`, `/messages`, or `/tools` — which are crucial for building the agent correctly.

However, remember that the Scope Reasoner already knows which docs are important and picks out the URLs it thinks are needed. The problem was that these URLs weren’t always actually crawled and passed to the coder agent.

To fix this, I changed how the Scope Reasoner talks to the coder agent:

- Instead of just sending a string, it now sends a JSON object like this:

```json
{"scope": "...", "selected_urls": [...]}
```

- And then I made sure the coder agent *has* to crawl all those URLs before it starts coding. Here’s a snippet from the system prompt that enforces this:

```python
@ai_coder.system_prompt
def add_info(ctx: RunContext[AIDeps]) -> str:
    print(ctx.deps.selected_urls)
    return f"""
    \n\n
    !!!IMPORTANT!!! YOU MUST BUILD YOUR AGENT IN THE {ctx.deps.framework} FRAMEWORK.
    !!!IMPORTANT!!! YOU MUST GET THE CONTENT FROM ALL THESE URLS {ctx.deps.selected_urls}.
    [WORKFLOW]
    1. Research
        - 1.1: get the contents from all the urls listed above with get_page_content tool
        - 1.2: RAG search for relevant documentation based on the query
        - 1.3: If you feel like you need anything else, use list_documentation_pages tool to get a list of all the documentation pages for this framework
    2. Implementation
        - 2.1 Provide complete, working code following framework best practices
        - 2.2 YOU MUST SPLIT UP THE FILES, do not have one single file with all the code. Learn how to split up your code with the documentation.
        - 2.2 Include necessary comments and documentation

    [FRAMEWORK ADAPTATION]
    Your code structure and approach should adapt based on the framework:

    \n\nAdditional Instructions from the reasoner LLM:
    {ctx.deps.scope}
    """
```

This way, the agent *has* to include all the important documentation.

| Problem 2: Too Much Noise in the Crawled Content | |
|---|---|
| Once I fixed that, another problem popped up. The content the agent crawled was full of junk — things like navigation menus, line numbers in code blocks, page margins, and other stuff that doesn't help at all.<br><br>If you take the first 20,000 characters of that content, more than half of it is just noise. That wastes tokens and makes it harder for the agent to focus on the useful parts.<br><br>So, to really improve Archon, I realized we need to make the crawling smarter — extracting only the meaningful documentation and ignoring all the clutter. | ![Archon Problem 2](/images/posts/archon-post/archon-v5-p2.png) |
