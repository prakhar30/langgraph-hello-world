# langgraph-hello-world

A ReAct agent built by hand in LangGraph instead of with a prebuilt helper: one node reasons, one node runs tools, and a conditional edge loops between them until the model stops asking for tools. The sample question forces both a live web search and a local tool call in a single run.

## What it does
- **`react.py`** — defines the tools (Tavily web search + a `triple` function) and binds them to the model
- **`nodes.py`** — the reasoning node and a prebuilt `ToolNode` for execution
- **`main.py`** — wires the two nodes into a cyclic graph, compiles it, and invokes it

## Notes & details
- **The whole ReAct loop is four lines of wiring.** An entry point at the reasoning node, a conditional edge out of it, and an unconditional edge from the tool node back to reasoning. That back-edge is the cycle — and cycles are the reason you reach for LangGraph instead of a plain LCEL chain.
- **Routing is one check**: `should_continue` looks at whether the last message has `tool_calls`. If yes, go execute them; if no, the model is done, so hit `END`. No parser, no state machine of your own.
- **`MessagesState` does the accumulation for you.** Nodes return only the *new* messages; the built-in reducer appends them to the running list rather than overwriting it.
- **`bind_tools` is what makes it work** — it attaches the tool schemas to the model so it can emit structured `tool_calls` instead of describing what it wishes it could do.
- **`ToolNode` handles the boring half**: reading the tool calls off the last message, dispatching them, and wrapping the results as `ToolMessage`s.
- **The graph draws itself.** `draw_mermaid_png` writes `flow.png` on every run, so the committed diagram can't drift out of sync with the code. Note it's a module-level side effect, so it fires on import, not just on `main()`.
- **The sample prompt is chosen to force a multi-hop run** — look up today's temperature in Toronto, then triple it. That's a search tool and a local tool in sequence, which means at least two trips around the loop.
- **Requires** `OPENAI_API_KEY` and `TAVILY_API_KEY` in `.env`.

## Run
```bash
uv sync
uv run main.py
```
