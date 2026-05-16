# LangGraph: Stateful Conversational Agents

A 10-section progression from a minimal single-node graph to a memory-aware conversational agent — built with [LangGraph](https://github.com/langchain-ai/langgraph) and LangChain.

## 📌 Project Overview

This notebook explores **LangGraph**, a framework for building stateful, multi-actor applications with LLMs. Starting from the simplest possible graph structure, each section incrementally introduces a new concept — conditional routing, reducers, memory persistence, and summarization — culminating in a full conversational agent with both short-term and long-term memory.

The progression is designed to build intuition from the ground up: every new concept is introduced only after the motivation for it becomes clear.

## 🛠️ Libraries Used

- **LLM / Graph:** `langgraph`, `langchain-openai`, `langchain-core`
- **Persistence:** `sqlite3` (via `SqliteSaver`)
- **Environment:** `python-dotenv`

## 🚀 Key Implementation Steps

**1. Simple Graph — `START → chatbot → END`**

- Defined a `State` TypedDict holding a list of `BaseMessage` objects.
- Built a single `chatbot` node that invokes GPT-4o and returns the response.
- Compiled a `StateGraph` into a `Runnable` using `.compile()`.

**2. Conditional Edges — Interactive Q&A Loop**

- Added `ask_question` and `ask_another_question` nodes for user input.
- Introduced a **routing function** that inspects state and returns the name of the next node.
- Used `add_conditional_edges` to branch back into the loop or exit to `END`.

**3. Reducer Functions — Persistent Conversation Memory**

- Replaced raw list assignment with `Annotated[Sequence[BaseMessage], add_messages]`.
- The `add_messages` reducer **appends** incoming messages rather than overwriting — preserving full history across loop iterations.

**4. `MessagesState` — Built-in Convenience Class**

- Replaced the hand-rolled `TypedDict` with LangGraph's built-in `MessagesState`, which ships with the `add_messages` reducer pre-wired.
- Demonstrated subclassing `MessagesState` to add custom state fields.

**5. `RemoveMessage` — Selective Memory Pruning (Proactive)**

- Introduced `RemoveMessage` sentinels passed to `add_messages` to delete specific messages by ID.
- Added a `trim_history` node after every chatbot turn, keeping the last 5 messages unconditionally.

**6. Deferred Pruning — Trim on Loop-Back**

- Moved the pruning node to the routing layer: trimming only occurs when the user continues (`"yes"`), not on the final turn.
- Demonstrated how **graph topology** shapes behavior independently of node logic.

**7. Summarizing Messages — Long-Term Memory with Compression**

- Subclassed `MessagesState` to add a `summary: str` field.
- Added a `summarize_and_delete_messages` node that compresses conversation history into a rolling summary and deletes all raw messages.
- The `chatbot` node injects the summary as a `SystemMessage` before each LLM call, preserving context without consuming the full context window.

**8. Short-Term Memory — `InMemorySaver`**

- Attached an `InMemorySaver` checkpointer to the compiled graph.
- State is keyed by `thread_id`; different threads share the same graph without interfering.
- Demonstrated resuming a conversation across multiple `.invoke()` calls.

**9. `StateSnapshot` — Auditing Graph History**

- Used `get_state_history(config)` to retrieve all checkpointed snapshots for a thread.
- Each snapshot exposes `values`, `next`, and `metadata["step"]` for replay and debugging.

**10. Long-Term Memory — `SqliteSaver`**

- Swapped `InMemorySaver` for `SqliteSaver` backed by a local SQLite database.
- Conversation history now survives process restarts — the agent picks up exactly where it left off after a kernel restart.

## 📊 Concepts Covered

| Section | Concept |
|---|---|
| 1 | StateGraph, nodes, edges, `.compile()` |
| 2 | Conditional edges, routing functions, loops |
| 3 | Reducer functions, `add_messages` |
| 4 | `MessagesState`, subclassing |
| 5 | `RemoveMessage`, proactive pruning |
| 6 | Deferred pruning, topology-driven behavior |
| 7 | Summarization, rolling memory compression |
| 8 | `InMemorySaver`, thread-keyed checkpoints |
| 9 | `StateSnapshot`, `get_state_history` |
| 10 | `SqliteSaver`, cross-restart persistence |

## 📂 Repository Structure

```
├── LangGraph.ipynb   # Main notebook — all 10 sections
├── .env              # Your OpenAI API key (not committed)
├── requirements.txt  # Python dependencies
└── README.md         # Project documentation
```

## 👩‍💻 How to Run

1. Clone this repository
2. Install requirements: `pip install -r requirements.txt`
3. Create a `.env` file in the root directory and add your OpenAI API key:
   ```
   OPENAI_API_KEY=your_key_here
   ```
4. Open `LangGraph.ipynb` in Jupyter
5. Run all cells from top to bottom

> **Note:** Sections 2–7 include interactive `input()` prompts — run the notebook cell-by-cell when testing those sections.

---

**Developed by:**  
Nicole Kaye A. Cardel  
[nkcardel@gmail.com](mailto:nkcardel@gmail.com)  
*Software Designer & Developer*
