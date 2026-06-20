# HappyClaude

HappyClaude is a personal local Agent project for coding, task orchestration, and remote operation from a mobile browser.

The core runtime runs on your own computer. It can call an LLM, execute local tools, manage tasks, run background commands, schedule reminders, spawn teammate agents, and isolate parallel work with Git worktrees. The optional client layer lets a phone connect to the local Agent through a browser, either on the same LAN or through a Cloudflare Worker relay.

## Features

- ReAct-style Agent loop with model tool calls, local tool execution, tool result feedback, and multi-turn reasoning.
- Unified tool registry for file operations, shell commands, task management, background jobs, cron jobs, team communication, and worktree management.
- Permission and hook system for risky actions such as file writes and shell commands.
- Context engineering with system prompt assembly, memory injection, skills loading, compaction, and recovery compaction.
- Persistent memory stored under `.memory/`, with memory index, relevant-memory loading, extraction, and consolidation.
- Project task board stored under `.tasks/`, with task creation, claiming, completion, dependencies, and worktree binding.
- Background task runtime for long-running commands, with status query, waiting, cancellation request, and completion notifications.
- Cron scheduler for time-based Agent prompts.
- Multi-agent team runtime with teammate agents, mailbox communication, protocol requests, plan approval, shutdown, and autonomous task claiming.
- Git worktree isolation for task-level branches and independent work directories.
- LLM request gateway with queueing and task-based model routing.
- Optional LangGraph entrypoint that maps the Agent loop into a `StateGraph`.
- Mobile browser client for remote operation through local WebSocket or Cloudflare Worker relay.

## Architecture

```text
Mobile Browser / Desktop CLI
  |
  |-- local LAN WebSocket
  |-- Cloudflare Worker relay
  v
HappyClaude Client
  |
  v
Python Bridge
  |
  v
MainLoop / MainLoopLangGraph
  |
  |-- LlmGateway          request queue and model routing
  |-- SystemPrompt        prompt assembly
  |-- Tools               tool interface and registry
  |-- Hooks               lifecycle hooks
  |-- Permissions         path and command guard
  |-- ContextCompact      context compression
  |-- Memory              long-term memory
  |-- Skills              skill catalog and lazy loading
  |-- TaskSystem          persistent task board
  |-- BackgroundTasks     async tool execution
  |-- CronScheduler       time-based prompt queue
  |-- AgentTeams          multi-agent collaboration
  |-- WorktreeManager     git worktree isolation
  |
  v
LLM API
```

## Project Structure

```text
HappyClaude/
  Main/
    MainLoop.py              default CLI entrypoint
    MainLoopLangGraph.py     LangGraph-based entrypoint
    LlmGateway.py            queued LLM gateway and model routing
    Tools.py                 tool interface and registry
    ToolResult.py            structured tool result format
    Permissions.py           permission checks
    Hooks.py                 hook system
    ContextCompact.py        context compression
    Memory.py                memory extraction and injection
    Skills.py                skill discovery and loading
    SystemPrompt.py          system prompt assembly
    TaskSystem.py            persistent task board
    BackgroundTasks.py       background tool execution
    CronScheduler.py         cron scheduler
    AgentTeams.py            multi-agent team runtime
    WorktreeManager.py       git worktree management
    ErrorHandler.py          user-facing error handling
    ErrorRecovery.py         model/API retry and recovery
    StructuredOutput.py      model output validation and repair
    llm_http/                Anthropic-compatible HTTP client
    requirements.txt
  client/
    src/                     local TypeScript server
    public/                  browser UI
    python/                  Python bridge to the Agent runtime
    worker/                  Cloudflare Worker relay
  skills/                    local skill files
```

Runtime-generated directories:

```text
.tasks/            persistent task JSON files
.memory/           memory index and memory files
.mailboxes/        teammate inbox files
.task_outputs/     saved large tool outputs
.worktrees/        git worktree directories
.transcripts/      compact/recovery transcripts
```

## Requirements

- Python 3.11
- Node.js 20+
- Git
- Conda or virtualenv
- Anthropic API key or an Anthropic-compatible API endpoint
- Cloudflare account, only if remote relay mode is needed

## Configure Agent

Create Python environment and install dependencies:

```powershell
conda create -n Claude python=3.11
conda activate Claude
cd C:\Users\24021\Desktop\java\learnclaudecode\HappyClaude\Main
pip install -r requirements.txt
```

Create local environment file:

```powershell
Copy-Item .env.example .env
```

Edit `Main/.env`:

```env
ANTHROPIC_API_KEY=your_api_key_here
MODEL_ID=your_model_id_here
ANTHROPIC_BASE_URL=https://api.anthropic.com
```

Optional LLM gateway configuration:

```env
LLM_MAX_CONCURRENT=1
TEAM_MODEL_ID=your_team_model
MEMORY_MODEL_ID=your_small_memory_model
COMPACT_MODEL_ID=your_compact_model
REPAIR_MODEL_ID=your_repair_model
MEMORY_MATCH_MODE=keyword
```

## Run CLI

Default runtime:

```powershell
cd C:\Users\24021\Desktop\java\learnclaudecode\HappyClaude\Main
python MainLoop.py
```

LangGraph runtime:

```powershell
cd C:\Users\24021\Desktop\java\learnclaudecode\HappyClaude\Main
python MainLoopLangGraph.py
```

## Run Mobile Client

Install and build the local client:

```powershell
cd C:\Users\24021\Desktop\java\learnclaudecode\HappyClaude\client
npm install
npm run build
npm start
```

The console prints a local URL such as:

```text
http://192.168.x.x:9527
```

Open that URL from a phone on the same LAN to operate HappyClaude remotely.

The client starts a Python bridge process and keeps one Agent history inside that process. The browser sends prompts to the bridge and receives Agent logs, tool output, final assistant text, and permission prompts.

## Remote Relay Mode

Remote relay mode lets the phone connect when it is not on the same LAN. The computer opens an outbound WebSocket connection to a Cloudflare Worker, and the phone connects to the same Worker session.

Deploy the Worker:

```powershell
cd C:\Users\24021\Desktop\java\learnclaudecode\HappyClaude\client
npm install
npm run build
cd worker
npm install
Copy-Item wrangler.example.jsonc wrangler.jsonc
npm run deploy
```

Start local client with the Worker URL:

```powershell
cd C:\Users\24021\Desktop\java\learnclaudecode\HappyClaude\client
$env:HAPPYCLAUDE_WORKER_URL="https://happyclaude-client.your-subdomain.workers.dev"
$env:HAPPYCLAUDE_SESSION_ID="replace-with-a-long-random-session"
npm start
```

The console prints a remote URL:

```text
https://happyclaude-client.your-subdomain.workers.dev/?session=your-session
```

Open that URL from the phone. The Cloudflare Worker only relays WebSocket messages and serves static assets; Agent execution and data stay on the local computer.

## Client Configuration

Use a custom Python executable:

```powershell
$env:HAPPYCLAUDE_PYTHON="C:\path\to\python.exe"
npm start
```

Use a custom local port:

```powershell
$env:HAPPYCLAUDE_CLIENT_PORT="9530"
npm start
```

Use a fixed remote session:

```powershell
$env:HAPPYCLAUDE_SESSION_ID="my-fixed-session"
npm start
```

## Example Prompts

```text
Inspect the project structure and explain the Agent runtime modules.
```

```text
Create two persistent tasks, make the second depend on the first, and show the task board.
```

```text
Run the test command in the background and continue reading the README while waiting.
```

```text
Create a worktree for a refactor task, bind the task to it, and spawn a teammate to inspect the target files.
```

```text
Remember that I prefer concise technical explanations.
```

## Security

Do not commit local credentials. Keep API keys in `Main/.env`; this file is ignored by Git.

Before making the repository public, run:

```powershell
rg -n --hidden --glob '!/.git/**' --glob '!**/__pycache__/**' --glob '!**/.venv/**' "sk-[A-Za-z0-9_-]{20,}|API_KEY|SECRET|TOKEN" .
git grep -n "sk-[A-Za-z0-9_-]\{20,\}" HEAD -- .
```

Remote relay sessions should use a long random `HAPPYCLAUDE_SESSION_ID`. The current client is designed for personal use and should not be exposed as a public multi-user service without authentication, authorization, audit logs, and stronger sandboxing.

## Notes

- Shell execution and file access are powerful local capabilities. Use the permission prompts carefully.
- Background task cancellation is cooperative. Python threads cannot be force-killed safely, so cancellation is recorded and applied when the underlying tool returns.
- Cron jobs run only while the Agent process is running.
- Worktree changes should be reviewed before removal or merge.

## License

No license has been specified yet.
