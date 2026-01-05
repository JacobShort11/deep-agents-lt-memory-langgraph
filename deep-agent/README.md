# Deep Research Agent

A LangGraph Deep Agents project for markets research with dedicated sub-agents for analysis, web research, and credibility checking.

> **Note:** This demo uses synthetically generated financial data rather than real Bloomberg terminal data. All market data, charts, and visualizations are based on mock datasets for demonstration purposes.

## Features
- Three specialized sub-agents: Analysis (Daytona Python), Web Research (Tavily search), Credibility (fact-checking).
- Cloud-hosted long-term memory and checkpointer via LangSmith.
- Terminal file navigation and editing tools for each agent.
- Organized scratchpad (in state) for intermediate notes and reports.

## Quick Start
1. **Install**
   ```bash
   cd deep-agent
   pip install -r ../requirements.txt
   ```
2. **Configure `.env`**
   - `OPENAI_API_KEY`
   - `TAVILY_API_KEY`
   - `DAYTONA_API_KEY`
   - `LANGSMITH_API_KEY`
   - `LANGSMITH_PROJECT`
   - `LANGSMITH_TRACING` (true/false)
   - Plot uploads (Cloudinary): `CLOUDINARY_CLOUD_NAME`, `CLOUDINARY_API_KEY`, `CLOUDINARY_API_SECRET`, `CLOUDINARY_PUBLIC_ID_PREFIX`, `CLOUDINARY_URL`
3. **Run the LangGraph server** (from `deep-agent/`)
   ```bash
   langgraph dev
   ```
4. **Use LangGraph Studio** — When the server starts, a LangGraph Studio popup will appear. Select the agent you want to run and chat with it directly in the Studio interface.

### Experiment Notebooks
The notebooks in `experiments/` are for testing individual agents, tools, and memory:
- `analysis_agent.ipynb`
- `web_research_agent.ipynb`
- `credibility_agent.ipynb`
- `memory_management.ipynb`
- `cloudinary_upload_smoke_test.ipynb`
- `daytona_package_test.ipynb`

## Agent Architecture

### Shared Capabilities (All Agents)

Every agent in the system has access to:

| Category | Tools |
|----------|-------|
| **File Navigation** | `ls`, `glob`, `grep` |
| **File Operations** | `read_file`, `write_file`, `edit_file` |
| **Short-Term Memory** | Shared `/scratchpad` folder in state |
| **Long-Term Memory** | Persistent database storing lessons learned, high-quality sources, effective research patterns, and prior failures—continuously trimmed and reused for self-improvement |

All agents operate under constraints for tool limits, context compaction, and memory forgetting to manage resources effectively.

---

### Main Orchestrator Agent

The main agent plans, coordinates, and synthesizes the final output. It can dynamically spawn **10+ sub-agents in parallel**, choosing from:

| Sub-Agent | Purpose |
|-----------|---------|
| **Analysis** | Runs sandboxed Python for quantitative analysis and data visualization |
| **Research** | Performs 100+ web searches per task for market, macro, and price-relevant data |
| **Credibility** | Cross-references claims, evaluates source quality, flags weaknesses |
| **General Purpose** | Handles general reasoning tasks as needed |

**Responsibilities:**
- Plans and maintains a to-do list, updating tasks as "completed", "in progress", or "to do"
- Outputs a final report with plots, images, and recommendations

---

### Research Sub-Agent

- Conducts **100+ web queries** per task using the `web-search` tool
- Gathers market intelligence, extracts datasets, and tracks source quality with citations
- Self-improves by learning about source reliability and effective search strategies

---

### Analysis Sub-Agent

- Executes Python code in a **remote sandbox** via the `code-execution` tool
- Performs time-series analysis, trend detection, correlations, and statistical tests
- Returns analysis summaries and saves generated plots to urls
- Learns from historical code execution outcomes

---

### Credibility Sub-Agent

- Delivers a **truthfulness verdict** for each claim:
  - Claim status and source assessment
  - Answer quality and recommendations
  - Final trustworthiness rating
- Focuses on recency, bias/conflicts, and contradictions
- Builds institutional knowledge about trustworthy sources and common pitfalls

---

## Memory Model
- Persistent store is provided by LangGraph (LangSmith cloud), namespaced per `assistant_id`.
- Files under `/memories/` include:
  - `website_quality.txt` — Source ratings
  - `research_lessons.txt` — What works
  - `source_notes.txt` — Source notes
  - `coding.txt` — Code mistakes and lessons

## Layout
```
agents/           # Main agent and sub-agents
tools/            # Code execution (Daytona) and web search tools
middleware/       # Memory backend configuration
experiments/      # Jupyter notebooks for testing agents and memory
langgraph.json    # LangGraph graph definition
```

## Troubleshooting
- Connection errors: ensure `langgraph dev` is running from `deep-agent/` and reachable at `http://localhost:2024`.
- Empty assistant list: start `langgraph dev` once to register the graph with LangSmith.

## Architecture Overview
LangSmith provides the cloud infrastructure for this project:
- **Checkpointer**: Persists conversation state, enabling pause/resume and time-travel debugging.
- **Store**: Shared key-value storage where agents read and write memories (e.g., source ratings, research lessons).
- **Shared Memory**: All agents (main orchestrator and sub-agents) access the same memory store, allowing them to learn from each other and build on previous research.

## Known Limitations & Future Improvements

### Current Limitations

- **File System Concurrency**: Multiple agents write to the shared file system, creating potential race conditions where one agent could overwrite another's changes. This won't crash the database but may cause data loss in edge cases.
- **Image Handling**: LangGraph Studio and the LangGraph UI do not natively support non-text files (images, PDFs). Currently working around this by uploading generated plots to Cloudinary public URLs.
- **LangGraph State Constraints**: The framework's state does not currently support binary data (images, data files), requiring workarounds to persist visual outputs to the scratchpad.
- **Mock Financial Data**: No Bloomberg terminal access for real market data, so plot generation relies on synthetic/mock datasets.
- **Sandbox Environment**: The Daytona sandbox container has a limited set of Python libraries, restricting the quality of time series visualizations.

### Cost & Performance

- **API Costs**: Running the full agent system incurs non-trivial costs, requiring throttling to stay within budget rather than letting agents run to full potential.
- **Optimization Opportunities**: Could reduce costs through prompt caching, using smaller models for simpler sub-tasks, and batching API calls where possible.

### Future Improvements

- **Append-Only Memory**: For production, implement append-only memory writes to prevent accidental overwrites between agents.
- **Custom Sandbox Image**: Build a custom Daytona container image with required libraries (e.g., Plotly for interactive visualizations, additional time series packages).
- **Enhanced Long-Term Memory**:
  - Current implementation uses basic file navigation for memory read/write.
  - Future: Semantic search for retrieving similar past research, memory decay/forgetting mechanisms, and a more scalable architecture for hundreds of memories.
  - Store structured semantic information (user preferences, domain facts, source reputation scores).
- **Evaluation Framework**: Create a LangSmith evaluation dataset with representative test cases to benchmark agent performance. This would enable quantified measurement of improvements or regressions with each change, removing guesswork from development iterations.
