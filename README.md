# Top Agent - Terminal Challenge Checkpoint 4 Winner

High-performance autonomous coding agent that achieved **top ranking** at [Terminal Challenge](https://term.challenge) Checkpoint 4. Built with Claude Opus 4.5 via OpenRouter, featuring advanced context management, prompt caching, and self-verification.

## Architecture Overview

```mermaid
flowchart TB
    subgraph Entry["Entry Point (agent.py)"]
        A[agent.py] --> B[AgentContext]
        A --> C[LiteLLMClient]
        A --> D[ToolRegistry]
    end

    subgraph MainLoop["Main Agent Loop (loop.py)"]
        E[Initialize Messages] --> F[Get Initial State]
        F --> G{Context Over 60%?}
        G -->|Yes| H[manage_context]
        G -->|No| I[Apply Caching]
        H --> I
        I --> J[Call LLM]
        J --> K{Has Tool Calls?}
        K -->|Yes| L[Execute Tools]
        L --> M[Add Results to Messages]
        M --> G
        K -->|No| N{Verification Phase}
        N -->|None| O[Request Verification]
        O --> G
        N -->|First| P[Request Confirmation]
        P --> G
        N -->|Confirmed| Q[Task Complete]
    end

    Entry --> MainLoop
```

## Context Management Flow

```mermaid
flowchart TD
    subgraph ContextMgmt["manage_context()"]
        H1[Prune Old Images] --> H2[Estimate Tokens]
        H2 --> H3{Over 60%?}
        H3 -->|Yes| H4[Prune Tool Outputs]
        H4 --> H5{Still Over?}
        H5 -->|Yes| H6[AI Compaction]
        H6 --> H7[Return Messages]
        H5 -->|No| H7
        H3 -->|No| H7
    end
```

## Prompt Caching Flow

```mermaid
flowchart LR
    subgraph Caching["apply_caching()"]
        I1[Get Messages] --> I2[Mark System Messages]
        I2 --> I3[Mark Last 2 Messages]
        I3 --> I4[Return Cached Messages]
    end
```

## Tool Registry

```mermaid
flowchart LR
    subgraph Tools["ToolRegistry.execute()"]
        L[Tool Call] --> T1[shell_command]
        L --> T2[read_file]
        L --> T3[write_file]
        L --> T4[apply_patch]
        L --> T5[grep_files]
        L --> T6[list_dir]
        L --> T7[view_image]
        L --> T8[web_search]
        L --> T9[spawn_process]
        L --> T10[kill_process]
    end
```

## Detailed Flow Diagram

```mermaid
sequenceDiagram
    participant User
    participant Agent as agent.py
    participant Core as loop.py
    participant LLM as LiteLLMClient
    participant Context as compaction.py
    participant Tools as ToolRegistry

    User->>Agent: --instruction task
    Agent->>Agent: Initialize Context, LLM, Tools
    Agent->>Core: run_agent_loop()
    
    Core->>Core: Build initial messages
    Core->>Tools: shell pwd and ls
    Tools-->>Core: Initial state
    
    rect rgb(240, 240, 240)
        Note over Core,Tools: Main Loop - max 200 iterations
        Core->>Context: manage_context messages
        Context->>Context: Prune images if over 10
        Context->>Context: Estimate tokens
        alt Over 60 percent threshold
            Context->>Context: Prune old tool outputs
            alt Still over threshold
                Context->>LLM: AI Compaction summarize
                LLM-->>Context: Summary
            end
        end
        Context-->>Core: Managed messages
        
        Core->>Core: Apply prompt caching
        Core->>LLM: chat messages tools
        LLM-->>Core: Response
        
        alt Has tool calls
            Core->>Tools: execute tool_name args
            Tools-->>Core: ToolResult
            Core->>Core: Add results to messages
        else No tool calls
            alt No verification yet
                Core->>Core: Inject verification prompt
            else First verification done
                Core->>Core: Inject confirmation prompt
            else Confirmed
                Core->>Agent: Task complete
            end
        end
    end
    
    Agent-->>User: Done
```

## Tool Execution Flow

```mermaid
flowchart LR
    subgraph Input
        A[Tool Call] --> B{Check Cache}
    end
    
    subgraph Execution
        B -->|Hit| C[Return Cached]
        B -->|Miss| D[Execute Tool]
        D --> E{Success?}
        E -->|Yes| F[Cache Result]
        E -->|No| G[Add Guidance]
        F --> H[Return Result]
        G --> H
    end
    
    subgraph Output
        C --> I[ToolResult]
        H --> I
        I --> J[Truncate if needed]
        J --> K[Add to Messages]
    end
```

## Available Tools

| Tool | Description | Parameters | Cacheable |
|------|-------------|------------|-----------|
| `shell_command` | Execute shell commands | `command`, `workdir?`, `timeout_ms?` | No |
| `read_file` | Read file with line numbers | `file_path`, `offset?`, `limit?` | Yes |
| `write_file` | Create/overwrite files | `file_path`, `content` | No |
| `apply_patch` | Apply unified diff patches | `patch` | No |
| `grep_files` | Search with ripgrep | `pattern`, `include?`, `path?`, `limit?` | Yes |
| `list_dir` | List directory recursively | `dir_path?`, `depth?`, `limit?` | Yes |
| `view_image` | Analyze images (returns base64) | `path` | Yes |
| `web_search` | Search the web | `query`, `num_results?`, `search_type?` | Yes |
| `update_plan` | Update task plan | `steps`, `explanation?` | No |
| `transcript` | Analyze video with Gemini 3 | `url`, `instruction` | No |
| `spawn_process` | Start background process | `command`, `cwd?`, `stdout_path?` | No |
| `kill_process` | Terminate process by PID | `pid`, `signal?` | No |
| `wait_for_port` | Wait for TCP port | `port`, `host?`, `timeout_sec?` | No |
| `wait_for_file` | Wait for file existence | `path`, `timeout_sec?`, `min_size_bytes?` | No |
| `run_until_file` | Run command until file exists | `command`, `file_path`, `timeout_sec?` | No |

## Core Components

| Component | File | Purpose |
|-----------|------|---------|
| Entry Point | `agent.py` | CLI interface, initializes all components |
| Agent Loop | `src/core/loop.py` | Main agentic loop with verification |
| Context Management | `src/core/compaction.py` | Token management, pruning, AI compaction |
| LLM Client | `src/llm/client.py` | LiteLLM wrapper with cost tracking |
| Tool Registry | `src/tools/registry.py` | Tool dispatch, caching, statistics |
| System Prompt | `src/prompts/system.py` | Codex-inspired autonomous prompt |
| Output Events | `src/output/jsonl.py` | JSONL event emission for SDK |

## Key Features

### 1. Prompt Caching (90% Cost Reduction)
```mermaid
flowchart LR
    A[Messages] --> B[Mark System Messages]
    B --> C[Mark Last 2 Messages]
    C --> D[Send to API]
    D --> E{Cache Hit?}
    E -->|Yes| F[Use Cached Prefix]
    E -->|No| G[Full Processing]
    F --> H[Only Process New Tokens]
    G --> H
```

### 2. Context Management Strategy
```mermaid
flowchart TD
    A[Check Token Usage] --> B{< 60% of 200K?}
    B -->|Yes| C[No Action]
    B -->|No| D[Prune Old Tool Outputs]
    D --> E{Still Over?}
    E -->|No| F[Done]
    E -->|Yes| G[AI Compaction]
    G --> H[Summarize Old Messages]
    H --> I[Keep Summary + Recent]
    I --> F
```

### 3. Self-Verification System
```mermaid
stateDiagram-v2
    [*] --> Working
    Working --> FirstVerification: No more tool calls
    FirstVerification --> Confirmation: Verification complete
    Confirmation --> Complete: All requirements met
    Confirmation --> Working: Issues found
    Complete --> [*]
```

## Configuration

| Parameter | Default | Description |
|-----------|---------|-------------|
| `model` | `openrouter/anthropic/claude-opus-4.5` | LLM model |
| `max_tokens` | `16384` | Max output tokens per request |
| `max_iterations` | `200` | Max loop iterations |
| `cost_limit` | `100.0` | Max cost in USD |
| `auto_compact_threshold` | `0.6` | Trigger compaction at 60% |
| `cache_enabled` | `true` | Enable prompt caching |
| `prune_protect` | `40000` | Protect last N tokens from pruning |

## Installation

```bash
pip install -e .
# or
pip install -r requirements.txt
```

## Usage

```bash
export OPENROUTER_API_KEY="your-key"
python agent.py --instruction "Your task here..."
```

## Project Structure

```
top-agent/
├── agent.py                 # Entry point
├── src/
│   ├── core/
│   │   ├── loop.py          # Main agent loop
│   │   ├── compaction.py    # Context management
│   │   └── session.py       # Session handling
│   ├── llm/
│   │   └── client.py        # LiteLLM client
│   ├── tools/
│   │   ├── registry.py      # Tool dispatcher
│   │   ├── shell.py         # Shell execution
│   │   ├── apply_patch.py   # Patch application
│   │   └── ...              # Other tools
│   ├── prompts/
│   │   ├── system.py        # System prompt
│   │   └── templates.py     # Verification prompts
│   └── output/
│       └── jsonl.py         # Event emission
├── rules/                   # Agent development guidelines
└── astuces/                 # Practical techniques
```

## License

MIT License
