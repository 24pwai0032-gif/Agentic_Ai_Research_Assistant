markdown# 🤖 Agentic AI Research Assistant

> Built with LangChain · LangGraph · Groq (LLaMA 3.3) · Gradio

![Python](https://img.shields.io/badge/Python-3.9+-blue?style=flat-square&logo=python)
![LangChain](https://img.shields.io/badge/LangChain-latest-green?style=flat-square)
![LangGraph](https://img.shields.io/badge/LangGraph-latest-orange?style=flat-square)
![Gradio](https://img.shields.io/badge/Gradio-latest-yellow?style=flat-square)
![License](https://img.shields.io/badge/License-MIT-purple?style=flat-square)

---

## 📌 Project Title

**Agentic AI Research & Productivity Assistant** — A conversational AI agent that reasons, selects tools dynamically, remembers context across turns, and presents results through an interactive Gradio interface.

---

## 🎯 Selected Use Case

A **multi-tool research and productivity assistant** that helps users get instant answers across four domains without switching between apps:

- 📚 Look up factual information on any topic (Wikipedia)
- 🌤️ Check live weather for any city in the world
- 🧮 Solve math problems and expressions
- 😄 Lighten the mood with a random joke

The agent decides *on its own* which tool to use based on what the user asks — the user just types naturally.

---

## 🛠️ Tools Used

| Tool | Type | Purpose |
|------|------|---------|
| `wikipedia_search` | External API | Fetch factual summaries on any topic |
| `get_weather` | External API | Get live weather data for any city |
| `get_joke` | External API | Fetch a random joke by category |
| `calculator` | Custom Python | Evaluate math expressions safely |

---

## 🔌 APIs Integrated

| API | Endpoint / Library | Auth Required | Cost |
|-----|--------------------|---------------|------|
| **Groq LLM** | `api.groq.com` via `langchain-groq` | API Key (free) | Free tier |
| **Wikipedia** | `wikipedia` Python library | None | Free |
| **wttr.in Weather** | `https://wttr.in/{city}?format=j1` | None | Free |
| **Official Joke API** | `https://official-joke-api.appspot.com` | None | Free |

> ✅ All APIs used are **completely free**. Get your Groq API key at [console.groq.com](https://console.groq.com) — no credit card required.

---

## 🔀 LangGraph Workflow Explanation

The agent uses a **ReAct-style (Reason + Act)** loop implemented as a LangGraph `StateGraph`.

### Graph Architecture
User Input
│
▼
┌─────────────┐
│  agent_node │  ← LLM receives message + full conversation history
└─────────────┘
│
├── tool_calls present? ──► YES ──► ┌───────────┐
│                                   │ tool_node │  ← executes the chosen tool
│                                   └───────────┘
│                                         │
│◄────────────────────────────────────────┘
│                                 (result injected back into state)
│
└── No tool calls ──────────────────► END → Final response to user

### Graph Components

| Component | Description |
|-----------|-------------|
| **`AgentState`** | `TypedDict` holding `messages: Annotated[list, add_messages]` — the full conversation history |
| **`agent_node`** | Calls the LLM with current message history; LLM decides whether to respond or call a tool |
| **`tool_node`** | LangGraph's built-in `ToolNode` — automatically executes whichever tool the LLM requested |
| **`should_use_tools`** | Conditional edge — checks if the last AI message contains `tool_calls`; routes to `tools` or `END` |
| **`MemorySaver`** | Checkpointer attached at `compile()` — persists state between `graph.invoke()` calls |

### Conditional Routing Logic

```python
def should_use_tools(state):
    last_message = state["messages"][-1]
    if hasattr(last_message, "tool_calls") and last_message.tool_calls:
        return "use_tools"   # → routes to ToolNode
    return "end"             # → routes to END
```

---

## 🧠 Memory Implementation

Memory is implemented using LangGraph's built-in **`MemorySaver`** checkpointer.

### How It Works

```python
memory = MemorySaver()
graph = graph_builder.compile(checkpointer=memory)
```

Every call to `graph.invoke()` passes a `config` dict with a `thread_id`:

```python
config = {"configurable": {"thread_id": "user-session-abc123"}}
graph.invoke({"messages": [HumanMessage(content=user_input)]}, config=config)
```

LangGraph uses `thread_id` as a key to **save and restore the full message history** between calls.

### Graph State vs Memory

| Concept | Scope | Lives In |
|---------|-------|----------|
| **Graph State** (`AgentState`) | Single invocation — one request/response cycle | RAM during execution |
| **Memory** (`MemorySaver`) | Across multiple invocations — entire conversation | Checkpoint store (in-memory) |

### Multi-Turn Memory Demo
Turn 1 → "My name is Ahmed"          → stored in checkpoint
Turn 2 → "What is 15 * 24?"          → calculator tool called → 360
Turn 3 → "Do you remember my name?"  → agent recalls "Ahmed" ✅

Each Gradio session generates a unique `thread_id` via `uuid4()`, so different users have completely isolated conversation memory.

---

## 🚀 How to Run the Application

### Prerequisites

- Python 3.9+
- A free Groq API key → [console.groq.com](https://console.groq.com)

### 1. Clone the Repository

```bash
git clone https://github.com/your-username/agentic-ai-assistant.git
cd agentic-ai-assistant
```

### 2. Install Dependencies

```bash
pip install langchain langgraph langchain-groq gradio wikipedia requests
```

### 3. Add Your API Key

Open `agentic_ai_assistant.ipynb` and in **Step 2**, replace:

```python
GROQ_API_KEY = "your_groq_api_key_here"
```

with your actual key from [console.groq.com](https://console.groq.com).

### 4. Launch the Notebook

```bash
jupyter notebook agentic_ai_assistant.ipynb
```

Run all cells from top to bottom. The Gradio app launches automatically in the final cell:
Running on local URL:  http://127.0.0.1:7860
Running on public URL: https://xxxx.gradio.live

Open either URL in your browser to start chatting.

---

## 💬 Example Prompts

| Category | Prompt |
|----------|--------|
| 🌤️ Weather | `What's the weather in Tokyo right now?` |
| 🌤️ Weather | `Is it cold in Karachi today?` |
| 📚 Wikipedia | `Tell me about the theory of relativity` |
| 📚 Wikipedia | `Who was Ada Lovelace?` |
| 📚 Wikipedia | `What is quantum entanglement?` |
| 🧮 Calculator | `What is sqrt(1764) + 15 * 8?` |
| 🧮 Calculator | `Calculate log(1000) times pi` |
| 🧮 Calculator | `What is 2 to the power of 16?` |
| 😄 Jokes | `Tell me a programming joke` |
| 😄 Jokes | `Give me a knock-knock joke` |
| 🧠 Memory | `My name is Sara` → later → `Do you remember my name?` |
| 🔀 Multi-tool | `What's the weather in Paris and tell me about the Eiffel Tower?` |

---

## ⚠️ Challenges Faced

**1. Decommissioned Groq Model**
The original notebook used `llama3-8b-8192` which was deprecated by Groq mid-project. Had to update to `llama-3.3-70b-versatile`, the current recommended free model.

**2. Safe Calculator Evaluation**
Using Python's `eval()` directly is a security risk. Built a restricted namespace with only whitelisted math functions (`sqrt`, `sin`, `log`, `pi`, etc.) to prevent arbitrary code execution while still supporting rich expressions.

**3. Duplicate System Messages**
LangGraph's `add_messages` reducer appends messages on every turn. Naively prepending a `SystemMessage` each time caused duplicates in the history. Fixed by checking `if not isinstance(messages[0], SystemMessage)` before prepending.

**4. Wikipedia Disambiguation Errors**
Many search terms trigger disambiguation pages instead of a direct article. Handled with a `try/except DisambiguationError` block that automatically retries with the first suggested option.

**5. Gradio Session Memory Isolation**
Without isolation, all users sharing one Gradio app would share the same `thread_id`, causing memory to bleed between sessions. Solved using `gr.State` to store a unique `uuid4()` per browser session.

---

## 🔮 Future Improvements

- **RAG Integration** — Add a document upload feature so users can query their own PDFs using vector search (FAISS + HuggingFace embeddings)
- **Multi-Agent System** — Split the assistant into specialist sub-agents (Research Agent, Math Agent, Weather Agent) coordinated by a Supervisor Agent
- **Human-in-the-Loop** — Add a confirmation step before executing sensitive tool calls using LangGraph's interrupt mechanism
- **LangSmith Tracing** — Integrate LangSmith for full observability — trace every LLM call, tool use, latency, and token cost
- **Streaming Responses** — Use `graph.astream()` with Gradio's streaming output for real-time token-by-token responses
- **Persistent Memory** — Replace `MemorySaver` with `SqliteSaver` or `RedisSaver` so conversations survive app restarts
- **Hugging Face Deployment** — Deploy the Gradio app permanently to HF Spaces with secrets management for the API key
- **More Tools** — News search, currency converter, dictionary/thesaurus, code executor

---

## 📁 Project Structure
agentic-ai-assistant/
│
├── agentic_ai_assistant.ipynb   # Main notebook (all code)
├── requirements.txt             # Python dependencies
└── README.md                    # This file

---

## 📦 Requirements
langchain
langgraph
langchain-groq
gradio
wikipedia
requests

---

## 📄 License

This project is licensed under the MIT License.

---

## 🙏 Acknowledgements

- [LangChain](https://www.langchain.com/) — LLM orchestration framework
- [LangGraph](https://langchain-ai.github.io/langgraph/) — Stateful agent workflow engine
- [Groq](https://groq.com/) — Ultra-fast free LLM inference
- [Gradio](https://www.gradio.app/) — Interactive UI framework
- [wttr.in](https://wttr.in/) — Free weather API
- [Official Joke API](https://official-joke-api.appspot.com/) — Free joke API
