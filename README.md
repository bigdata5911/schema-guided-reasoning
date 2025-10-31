# Schema-Guided Reasoning (SGR) – Business Agent Demo

This project demonstrates Schema-Guided Reasoning (SGR) to drive a simple business agent that can plan, decide, and call tools (issue invoices, send emails, create rules, etc.) against an in-memory CRM.

Two runnable examples are provided:

- OpenAI API based: `schema-guided-reasoning.py`
- Local llama.cpp based (Qwen3-4B): `sgr_assistant.py`

Both run a sequence of tasks and let the model plan steps and call tools via structured outputs validated by Pydantic schemas.

---

## Features

- Reasoning-driven agent with explicit, validated schema (SGR)
- Tool calling without special SDKs: pure schema + simple dispatch
- In-memory CRM: products, invoices, rules, emails
- Two backends:
  - OpenAI API (uses `openai-gpt-4o` in the code)
  - Local llama.cpp server with Qwen3-4B GGUF
- Pretty console output with `rich`

---

## Requirements

- Python 3.10+
- Packages:
  - `pydantic`
  - `annotated-types`
  - `rich`
  - For OpenAI example: `openai`
  - For llama.cpp example: `requests`

Install dependencies:

```bash
pip install pydantic annotated-types rich openai requests
```

---

## Project Structure

- `schema-guided-reasoning.py`: OpenAI-based SGR agent
- `sgr_assistant.py`: llama.cpp-based SGR agent using Qwen3-4B

Both files define:
- An in-memory `DB` with `products`, `invoices`, `emails`, and `rules`
- Tool schemas (`SendEmail`, `GetCustomerData`, `IssueInvoice`, `VoidInvoice`, rules)
- A dispatcher (`dispatch`) to simulate side effects and state updates
- A task list (`TASKS`) the agent executes sequentially
- An `execute_tasks()` entry point

---

## OpenAI variant

File: `schema-guided-reasoning.py`

### Configure

- Set your OpenAI credentials via environment variables (standard OpenAI SDK usage), e.g. on Windows PowerShell:

```powershell
$env:OPENAI_API_KEY = "YOUR_API_KEY"
```

- The code references model `openai-gpt-4o`. You can change it inside the script if needed.

### Run

```bash
python schema-guided-reasoning.py
```

The agent will:
- Print each task
- Plan remaining steps
- Call a tool by emitting a structured object (validated by Pydantic)
- Dispatch tool execution and append results back into the conversation
- Stop when it reports completion

---

## Local llama.cpp variant (Qwen3-4B)

File: `sgr_assistant.py`

This variant talks to a local llama.cpp HTTP server and uses a small Qwen3-4B model with constrained JSON outputs.

### Model and server

- Example GGUF: `Qwen3-4B-Instruct-2507-Q8_0.gguf`
- Start llama.cpp server (example from the script comments):

```bash
./llama-server \
  -m /path/to/Qwen3-4B-Instruct-2507-Q8_0.gguf \
  -ngl 999 \
  --port 12345 \
  --threads -1 \
  --host 127.0.0.1 \
  --ctx-size 20000
```

The script expects the completion endpoint at `http://127.0.0.1:12345/completion` by default.

### Run

```bash
python sgr_assistant.py
```

Notes:
- `sgr_assistant.py` includes light post-processing for Qwen outputs (e.g., removing `<think>...</think>`, stripping fenced blocks) to recover the JSON payload for schema validation.
- You can tweak temperature, top-k/top-p, penalties in the payload.

---

## Customizing tasks and tools

- Tasks: edit the `TASKS` list in either file to change agent objectives.
- Tools: extend or modify Pydantic models (`SendEmail`, `IssueInvoice`, etc.) and update the `dispatch` function to implement behavior.
- System prompt: both variants build a `system_prompt` with rules and product catalog; you can edit or augment it.

---

## Troubleshooting

- Missing packages: install with `pip install pydantic annotated-types rich openai requests`.
- OpenAI auth errors: ensure `OPENAI_API_KEY` is set in your shell.
- llama.cpp server connection errors: verify the server is running at the expected URL and port; adjust `LocalLLM(url=...)` in `sgr_assistant.py` if needed.
- JSON parsing errors: models must return JSON conforming to the schema; reduce temperature or check the output cleaning logic in the llama.cpp variant.

---