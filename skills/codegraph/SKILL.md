---
name: codegraph
description: |
  CodeGraph is a local-first code intelligence tool that indexes any codebase into a queryable knowledge graph.
  This skill MUST be loaded when the user asks to search, explore, analyze, or understand any codebase:
  including symbol lookup, call-graph traversal (callers/callees), impact analysis, file structure exploration,
  tracing execution paths between symbols, and any task that requires understanding how code is connected.
  Also triggers when the user mentions "codegraph", "code graph", "代码图谱", "代码搜索", "调用关系",
  "代码分析", "影响分析", or requests to init/install codegraph on a project.
  CodeGraph is the DEFAULT and PREFERRED code search tool: use it instead of Grep/Glob for all code exploration.
  This skill covers: install, init, indexing, MCP tool selection guide, CLI commands, and best practices.
agent_created: true
---

# CodeGraph

## Overview

CodeGraph is a local-first, pre-indexed code knowledge graph engine for AI coding agents.
It parses source code into a typed graph (symbols, call edges, imports, framework routes)
stored in a local SQLite database, exposed via MCP tools. Using CodeGraph instead of
Grepping/Globbing reduces token consumption by ~47%, tool calls by ~58%, and response time by ~22%.

Project: https://github.com/colbymchenry/codegraph (MIT, 39K+ stars)

## When to Use CodeGraph

CodeGraph is the **default code exploration tool**. Use it for ANY of the following:

| Trigger | Preferred Tool |
|---------|----------------|
| "How does X work?" / architecture questions | `codegraph_context` |
| Find a symbol by name | `codegraph_search` |
| See source of several related symbols | `codegraph_explore` |
| "Who calls X?" / upstream dependencies | `codegraph_callers` |
| "What does X call?" / downstream dependencies | `codegraph_callees` |
| "What would break if I change X?" | `codegraph_impact` |
| Browse project file structure | `codegraph_files` |
| Trace path from A to B in code | `codegraph_trace` |
| Drill into one symbol's details + trail | `codegraph_node` |
| Check index health | `codegraph_status` |

**Golden rule**: Prefer `codegraph_context` as the FIRST call for any "how does X work" question.
It composes search + node + callers + callees into a single call — usually enough to answer
without further tool calls. Only fall back to individual tools when `codegraph_context` results
are incomplete.

## Tool Selection Guide

### Primary Tools (use first)

1. **`codegraph_context`** — THE primary tool. Use for any code comprehension task.
   Combines search + node + callers + callees in one call.
   ```json
   { "task": "how does user login work", "maxNodes": 20, "includeCode": true }
   ```

2. **`codegraph_explore`** — Get source for SEVERAL related symbols grouped by file.
   Prefer this over chaining individual `codegraph_node` calls.
   ```json
   { "query": "AuthService loginUser session-manager", "maxFiles": 12 }
   ```
   Important: query with symbol/file/code terms, NOT natural-language sentences.
   Run `codegraph_search` first to find names if unsure.

3. **`codegraph_search`** — Quick symbol lookup by name. Returns locations only.
   ```json
   { "query": "signIn", "kind": "function", "limit": 10 }
   ```

### Call-Graph Tools

4. **`codegraph_callers`** — Who calls symbol X?
   ```json
   { "symbol": "UserService.createUser", "limit": 20 }
   ```

5. **`codegraph_callees`** — What does symbol X call?
   ```json
   { "symbol": "UserService.createUser", "limit": 20 }
   ```

6. **`codegraph_impact`** — What code would be affected by changing X?
   ```json
   { "symbol": "UserService", "depth": 2 }
   ```

7. **`codegraph_trace`** — Find the call PATH from symbol A to symbol B.
   ```json
   { "from": "handleHttpRequest", "to": "executeQuery" }
   ```
   This is structurally impossible with grep. Use it for flow questions like
   "how does an HTTP request reach the database?".

### Structural Tools

8. **`codegraph_node`** — Get ONE symbol's details, source (opt), and trail (callers+callees).
   Use to walk the call graph hop-by-hop from a starting point.
   ```json
   { "symbol": "main", "includeCode": true }
   ```

9. **`codegraph_files`** — Get indexed file tree. Much faster than Glob/filesystem.
   ```json
   { "path": "src/components", "format": "tree", "pattern": "*.tsx" }
   ```

10. **`codegraph_status`** — Check index health/stats. Use when results seem stale.
    ```json
    {}
    ```

### IMPORTANT: `projectPath` Parameter

ALL codegraph MCP tools accept an optional `projectPath` parameter to query
a DIFFERENT initialized project. Omit to use the current workspace.

```json
{ "task": "...", "projectPath": "/path/to/other/project" }
```

## Workflow Patterns

### Pattern 1: Understand a Feature

```
1. codegraph_context(task="how does feature X work")
2. If results incomplete → codegraph_explore(query="key symbols from step 1")
3. If specific flow unclear → codegraph_trace(from="entry", to="target")
```

### Pattern 2: Impact Analysis Before Refactoring

```
1. codegraph_impact(symbol="TargetClass", depth=2)
2. codegraph_callers(symbol="TargetClass.methodToChange")
3. Review affected files, plan changes
```

### Pattern 3: Explore Unknown Codebase

```
1. codegraph_files(format="grouped")          → understand structure
2. codegraph_context(task="main entry point") → find where to start
3. codegraph_node(symbol="main", includeCode=true) → drill into entry
```

### Pattern 4: Debug a Bug

```
1. codegraph_context(task="bug: description of the issue")
2. codegraph_trace(from="triggerPoint", to="failureSite")
3. codegraph_impact(symbol="suspectedRootCause")
```

## Installation & Setup

### One-Line Install

**macOS / Linux:**
```bash
curl -fsSL https://raw.githubusercontent.com/colbymchenry/codegraph/main/install.sh | sh
```

**Windows (PowerShell):**
```powershell
irm https://raw.githubusercontent.com/colbymchenry/codegraph/main/install.ps1 | iex
```

**npm (requires Node.js 18.17+):**
```bash
npm i -g @colbymchenry/codegraph
```

### Connect to AI Agents

```bash
codegraph install
```
Auto-detects: Claude Code, Cursor, Codex CLI, opencode, Hermes, Gemini CLI,
Antigravity IDE, Kiro. This configures the MCP server in the agent's config.

Non-interactive mode:
```bash
codegraph install --yes                              # auto-detect, global
codegraph install --target=claude,cursor --yes       # specific targets
```

### Initialize a Project

```bash
cd your-project
codegraph init -i    # -i builds the initial index immediately
```

### Manual MCP Config (if auto-install fails)

Add to `~/.claude.json`:
```json
{
  "mcpServers": {
    "codegraph": {
      "type": "stdio",
      "command": "codegraph",
      "args": ["serve", "--mcp"]
    }
  }
}
```

## CLI Quick Reference

| Command | Purpose |
|---------|---------|
| `codegraph init -i` | Initialize project + build index |
| `codegraph index --force` | Full re-index |
| `codegraph sync` | Incremental update |
| `codegraph status` | Index statistics |
| `codegraph query <name>` | Search symbols |
| `codegraph files` | Show file structure |
| `codegraph callers <symbol>` | Who calls this? |
| `codegraph callees <symbol>` | What does this call? |
| `codegraph impact <symbol>` | Impact radius |
| `codegraph affected src/auth.ts` | Find affected tests |
| `codegraph serve --mcp` | Start MCP server manually |
| `codegraph uninit` | Remove from project |
| `codegraph uninstall` | Remove from all agents |

All commands accept `[path]` to work on a different project.

For full CLI reference, see `references/cli_commands.md`.
For detailed MCP tool schemas, see `references/mcp_tools.md`.

## Best Practices

1. **Always init first.** Before any code exploration on a new project, run `codegraph init -i`.
   Without this, all MCP tools will fail.

2. **Prefer `codegraph_context` over chaining.** One `codegraph_context` call replaces
   3-4 individual tool calls (search + node + callers + callees). Less tokens, less latency.

3. **Use `codegraph_explore` not multiple `codegraph_node`.** Each `codegraph_node` re-reads
   the full context. `codegraph_explore` batches related symbols efficiently.

4. **Query with symbol names, not natural language.** For `codegraph_explore` and
   `codegraph_search`, use symbol/file/code terms like `"AuthService loginUser"`,
   not sentences like `"how does authentication work"`.

5. **Re-index after major changes.** Run `codegraph sync` (or `codegraph index --force`)
   after pulling, merging, or making substantial code changes.

6. **Verify with `codegraph_status`.** If results seem stale or incomplete, check the
   index status first.

7. **Use `projectPath` for cross-project queries.** All tools accept `projectPath`
   to query a different initialized project without changing directories.

8. **Fall back to Read for in-flight changes.** CodeGraph is an index — it has a slight
   lag behind live files due to file-watch debounce. If a file was just modified, prefer
   reading it directly.

## Supported Languages

TypeScript, JavaScript, Python, Go, Rust, Java, C#, PHP, Ruby, C, C++, Objective-C,
Swift, Kotlin, Scala, Dart, Svelte, Vue, Liquid, Pascal/Delphi, Lua, Luau

## Framework Route Detection

Django, Flask, FastAPI, Express, NestJS, Laravel, Drupal, Rails, Spring, Gin/chi/gorilla/mux,
Axum/actix/Rocket, ASP.NET, Vapor, React Router/SvelteKit
