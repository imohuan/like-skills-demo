---
name: zoom-out
description: Tell the agent to zoom out and give broader context or a
  higher-level perspective. Use when you're unfamiliar with a section of code or
  need to understand how it fits into the bigger picture.
disable-model-invocation: true
agent_created: true
disable: false
---

# Zoom Out — 项目全景视角

## Overview

When invoked, zoom out from the current code context and produce a structured, high-level map of the project. The goal is to answer "What is this project, what are the parts, how do they connect, and where does this piece I'm looking at fit in?"

## When to Use

Invoke this skill manually whenever:

- Entering an unfamiliar project or code area for the first time
- Needing to understand how a specific module/component fits into the larger architecture
- Preparing to make changes and wanting to see the impact radius before touching code
- Onboarding — getting the "lay of the land" before diving into details

## Workflow

### Step 1: Gather Project-Level Context

Start broad. Read these files if they exist (parallel reads preferred):

- `README.md`, `package.json`, `tsconfig.json` — tech stack, scripts, dependencies
- Project root config files (`vite.config.ts`, `next.config.js`, etc.) — build tooling
- Any `docs/` directory or architecture documentation
- Directory listing of top-level `src/` to identify module boundaries

### Step 2: Build the Module Map

Use CodeGraph to trace the architecture. Key commands:

```
codegraph explore <project-root>         # Get the dependency graph summary
codegraph search <key-symbol>            # Find where a specific concept lives
codegraph callers <symbol>              # Who depends on this?
codegraph callees <symbol>              # What does this depend on?
codegraph context <symbol>              # One-shot: node + neighbors + callers + callees
codegraph trace <source> <target>       # Trace the path between two symbols
codegraph impact <symbol>               # What would break if this changes?
```

Prioritize `codegraph_context` — it gives the richest signal in a single call.

### Step 3: Extract Domain Glossary

While exploring, collect key domain terms. Look for:

- Interface/type names that represent business concepts
- Directory names that reflect domain boundaries (e.g., `orders/`, `auth/`, `billing/`)
- Comments and README references to domain-specific language
- Route names, API endpoint paths, Vue/React component names

Build a glossary mapping each term to its meaning **in this project's context**.

### Step 4: Synthesize and Present

Produce a structured output with these sections:

1. **Project Summary** — One paragraph: what this project does, tech stack, and architectural style (SPA, SSR, microservices, monorepo, etc.)

2. **Module Map** — Hierarchical list of top-level module groups, each with:
   - Purpose (one sentence)
   - Key entry points (files/symbols)
   - Dependencies (what it imports from, what imports it)

3. **Caller/Callee Heatmap** — For the area the user is investigating, show:
   - Direct callers (who consumes this)
   - Direct callees (what this consumes)
   - Indirect impact radius (2-hop dependencies)

4. **Domain Glossary** — Key business terms and their project-specific meanings

5. **Navigation Tips** — "To understand X, start at file Y; to modify Z, look at files A, B, C"

## Presentation Style

- Use bullet points and short, scannable text. No walls of prose.
- Use CodeGraph results directly — cite file paths and symbol names precisely.
- When ambiguity exists (two modules with the same name, unclear responsibility), flag it explicitly rather than guessing.
- If the project is too large to map fully, map the top 2-3 layers and note where to drill deeper.
