# Deep Research Agent

A LangGraph Deep Agents project for markets research with dedicated sub-agents for analysis, web research, and credibility checking.

## Features
- Three specialized sub-agents: Analysis (Daytona Python), Web Research (Tavily search), Credibility (fact-checking).
- Cloud-hosted memory and checkpointer via LangSmith.
- Automatic context compaction (~170k tokens) built into the framework.
- Organized scratchpad for notes and reports.

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
   - Plot uploads (Cloudinary): `CLOUDINARY_CLOUD_NAME`, `CLOUDINARY_API_KEY`, `CLOUDINARY_API_SECRET` (or `CLOUDINARY_UPLOAD_PRESET` for unsigned), optional `CLOUDINARY_PUBLIC_ID_PREFIX` (default plots/). Alternatively, set `CLOUDINARY_URL` (e.g., `cloudinary://<api_key>:<api_secret>@<cloud_name>`).
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
- `md_to_pdf.ipynb`

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
scratchpad/       # Notes and reports written by agents
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
