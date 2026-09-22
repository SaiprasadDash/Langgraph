# 🕸️ LangGraph Playground

A collection of small, focused [LangGraph](https://langchain-ai.github.io/langgraph/) examples exploring core agentic patterns — sequential flows, parallel reducers, conditional RAG, iterative tool use, and **human-in-the-loop (HITL)** approval loops.

The flagship example is **`HITL_Agent.py`** — a LinkedIn post generator that writes a draft, pauses for a real human to approve or reject it, and loops back to rewrite based on feedback, using LangGraph's `interrupt` / `Command(resume=...)` mechanism.

## 🔎 Overview

Most agent demos either run fully autonomously or fully manually — this repo is about the middle ground: **agents that know when to stop and ask**.

`HITL_Agent.py` builds a small stateful graph with two nodes:

1. **`writer`** — an LLM (`gpt-4o-mini` via `langchain-openai`) drafts a LinkedIn post on a topic you give it. On the first pass it writes from scratch; on every retry, it rewrites specifically to address the feedback from the previous round.
2. **`human_review`** — the graph *interrupts itself* here and hands control back to you in the terminal. You either type `approved` to accept the draft, or type free-form feedback to send it back for a rewrite.

A conditional edge (`should_stop_looping`) decides what happens next:
- ✅ Approved → the graph ends
- 🔁 Rejected → loops back to `writer` with your feedback, up to **3 attempts**
- ⏹️ Max attempts reached → ends anyway with the last draft

State (topic, draft, feedback, approval status, attempt count) is tracked in a `TypedDict` and persisted across the interrupt/resume cycle using LangGraph's `MemorySaver` checkpointer, so the graph can pause indefinitely and pick up exactly where it left off once you respond.

## ✨ What's in this repo

| File | Pattern demonstrated |
|---|---|
| `HITL_Agent.py` | **Human-in-the-loop** — `interrupt()` + `Command(resume=...)`, approval/feedback loop with a retry cap |
| `sequential_base.py` | A basic linear (sequential) LangGraph flow |
| `parallel_reducers.py` | Fan-out/fan-in nodes with state reducers |
| `iterative_tools.py` | An agent that iteratively calls tools in a loop |
| `conditional_RAG.py` | Retrieval-augmented generation with conditional routing (uses `academics_handbook.pdf` / `fee_structure.pdf` as sample documents) |
| `states.py` | Shared `TypedDict` state schema(s) used across the examples |

## 🛠️ Tech Stack

- **[LangGraph](https://github.com/langchain-ai/langgraph)** — stateful graph orchestration, checkpointing, interrupts
- **[LangChain](https://github.com/langchain-ai/langchain)** — LLM/prompt orchestration, community integrations
- **OpenAI** (`langchain-openai`, `gpt-4o-mini`) — used by `HITL_Agent.py`
- **Groq** (`langchain-groq`) — used by other examples in the repo
- **HuggingFace sentence-transformers** + **FAISS** — embeddings and vector search for the RAG example
- **pypdf** — PDF loading for `conditional_RAG.py`

## 🚀 Getting Started

### Prerequisites

- Python ≥ 3.10
- An [OpenAI API key](https://platform.openai.com/api-keys) (for `HITL_Agent.py`)
- A [Groq API key](https://console.groq.com/keys) (for the other examples that use `langchain-groq`)

### Installation

```bash
git clone https://github.com/SaiprasadDash/Langgraph.git
cd Langgraph
python -m venv venv
source venv/bin/activate   # On Windows: venv\Scripts\activate
pip install -r requirements.txt
```

### Configuration

Create a `.env` file in the project root:

```env
OPENAI_API_KEY=your_openai_api_key_here
GROQ_API_KEY=your_groq_api_key_here
```

## ▶️ Running the HITL Agent

```bash
python HITL_Agent.py
```

You'll be prompted for a topic, then walked through the approval loop:

```
=======================================================
Welcome to the LinkedIn Post Generator (HITL Edition)
=======================================================

This tool will draft a LinkedIn post for you, show it to
YOU for review, and rewrite based on your feedback.
=======================================================

What topic do you want a LinkedIn post about?
> The future of AI agents

Starting generation...

[Attempt 1] Writer is drafting the post...

[Draft ready]
-------------------------------------------------------
...draft text...
-------------------------------------------------------

=======================================================
DRAFT FOR YOUR REVIEW (Attempt 1)
=======================================================
...draft text...
=======================================================

Type 'approved' to accept, or type your feedback to request a rewrite.

Your response: make the hook punchier and add a question at the end
```

Keep giving feedback (or type `approved`) until you're happy with the result or the 3-attempt limit is hit. The final post and a short run summary (total attempts, whether it was human-approved) are printed at the end.

### Running the other examples

Each script is standalone:

```bash
python sequential_base.py
python parallel_reducers.py
python iterative_tools.py
python conditional_RAG.py
```

## 🗺️ Roadmap

- [ ] Add a `requirements-dev.txt` / lockfile for reproducible installs
- [ ] Swap `MemorySaver` for a persistent checkpointer (SQLite/Postgres) so HITL sessions survive process restarts
- [ ] Add a simple web UI for the human-review step instead of the terminal prompt

## 🤝 Contributing

Issues and pull requests are welcome — this repo is meant as a learning reference for LangGraph patterns, so cleanups, new examples, and corrections are all useful.

## 📄 License

No license specified yet — consider adding one (e.g. MIT) to clarify usage terms.