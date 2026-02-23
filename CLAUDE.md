# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

OpenProse is a programming language for AI sessions — zero-dependency, pure-specification. There is no runtime binary, no package.json, no build system. The entire project is markdown (`.md`) and prose program (`.prose`) files that an LLM reads to become the OpenProse VM. "Simulation with sufficient fidelity is implementation."

**License:** MIT

## Architecture

The project has a layered documentation architecture where each layer builds on the previous:

| Layer | Files | Purpose |
|-------|-------|---------|
| **Skill entry** | `SKILL.md` | Activation triggers, command routing, file locations |
| **VM spec** | `prose.md` (36KB) | Execution semantics — how to run programs |
| **Language spec** | `compiler.md` (83KB) | Full grammar, validation rules, compilation |
| **State backends** | `state/filesystem.md`, `state/in-context.md`, `state/sqlite.md`, `state/postgres.md` | Four state management strategies |
| **Session primitives** | `primitives/session.md` | Subagent context management, compaction guidelines |
| **Authoring guidance** | `guidance/patterns.md`, `guidance/antipatterns.md` | Best practices for writing .prose programs |
| **Standard library** | `lib/*.prose` (9 programs) | Inspector, profiler, cost-analyzer, memory, etc. |
| **Examples** | `examples/*.prose` (51 programs) | From hello-world to "build a browser from scratch" |
| **Alternative syntaxes** | `alts/*.md` (5 files) | Borges, Folk, Arabian Nights, Homer, Kafka keyword skins |

### Key Concept: Specification-as-VM

The LLM reads `prose.md` and becomes the VM. Each `session` statement triggers a real `Task` tool call spawning a real subagent. The VM never holds full binding values — only pointers to where outputs are stored (filesystem paths or DB coordinates).

### Command Routing (via SKILL.md)

| Command | What loads |
|---------|-----------|
| `prose run <file>` | `prose.md` + `state/filesystem.md` (default) |
| `prose compile <file>` | `compiler.md` only |
| `prose help` | `help.md` |
| `prose examples` | `examples/` listing |
| `prose update` | Migration logic (in SKILL.md) |

### Context Budget Warning

`compiler.md` is 83KB (~2971 lines). Only load it when the user explicitly requests compilation/validation. After compiling, recommend `/compact` before running — don't keep both `compiler.md` and `prose.md` in context simultaneously.

## Working With This Codebase

### Editing Specifications

Changes to `prose.md` or `compiler.md` affect ALL `.prose` programs. These are the core VM and language specs — edit with care. Verify changes against examples.

### Writing New .prose Programs

Load `guidance/patterns.md` and `guidance/antipatterns.md` before authoring. Key patterns: captain's chair (examples 29-31), fan-out-fan-in, RLM recursive processing (examples 40-43).

### Standard Library (`lib/`)

The stdlib forms a self-improvement loop:
```
Run Program -> Inspector -> VM Improver -> PR
                         -> Program Improver -> PR
                         -> Cost Analyzer -> optimizations
```

### State Backends

Default is filesystem (`state/filesystem.md`). State lives in `.prose/runs/{YYYYMMDD}-{HHMMSS}-{random6}/`. Four backends available — filesystem, in-context, sqlite (experimental), postgres (experimental). Each is a separate `.md` spec the VM loads.

### Alternative Syntaxes (`alts/`)

These map the same semantics to different keyword sets (e.g., `agent` -> `dreamer`, `session` -> `dream` in Borges register). Used for learnability research, not production.

## File Conventions

- `.prose` — executable programs (Python-like indentation, no actual Python)
- `.md` — specifications, documentation, state files
- `.prose/` directory (in user projects) — runtime state, agent memory, config
- `bindings/{name}.md` — subagent outputs (written by subagents, not the VM)
- `agents/{name}/memory.md` — persistent agent state with segment files `{name}-NNN.md`

## Validation

There are no unit tests, CI, or build steps. Validation is:
1. `prose compile <file>` — validates syntax and semantics against `compiler.md`
2. The 51 examples serve as an implicit test suite — if the VM can interpret them, the spec works
3. `lib/inspector.prose` — post-run evaluation of execution fidelity
