# Phase 1: Sequential Foundations -- Results

## Environment

- **Date:** 2026-04-07
- **Claude model:** claude-opus-4-6 (1M context)
- **Platform:** macOS Darwin 25.4.0
- **Claude Code version:** With skills, teams, and agent tools available

## Critical Finding: OpenProse Skill Not Registered

The `prose run` command could not be executed. The OpenProse skill (`SKILL.md`) exists in the repository but is **not registered** as an available Claude Code skill in this environment. Attempts to invoke it:

```
Skill "prose" -> Unknown skill: prose
Skill "open-prose" -> Unknown skill: open-prose
```

The SKILL.md defines activation triggers and command routing, but the skill registration mechanism requires the skill to be installed in Claude Code's skill system (likely via a plugin or `.claude/` configuration). Without this registration, the VM spec (`prose.md`) cannot be automatically loaded, and the `prose run` command has no handler.

### Secondary Finding: No Task/Subagent Tool

Even if the skill were registered, the OpenProse VM requires a "Task" tool to spawn subagent sessions (each `session` statement = one Task tool call). The available tools in this environment include:
- `TaskCreate` / `TaskUpdate` -- for task tracking, NOT subagent spawning
- `TeamCreate` / `SendMessage` -- for multi-agent teams, different paradigm
- No direct "spawn a subagent with this prompt and return its output" tool

The VM spec maps "Task tool" to "OpenClaw `sessions_spawn`", suggesting OpenProse was designed for a specific runtime (OpenClaw) that provides this primitive. Standard Claude Code does not expose this tool.

## Built-in Examples

| Example | Ran successfully? | Fidelity notes |
|---------|-------------------|----------------|
| `01-hello-world.prose` | NO -- skill not registered | Cannot invoke `prose run`. Manual VM simulation possible but defeats the purpose of evaluating the tool. |
| `02-research-and-summarize.prose` | NO -- skill not registered | Same issue. Two sequential sessions would require Task tool to spawn subagents. |
| `04-write-and-refine.prose` | NO -- skill not registered | Same issue. Four sequential sessions, each improving on the last. |

## Custom .prose Programs

| Program | Ran successfully? | Output quality (1-10) | Notes |
|---------|-------------------|----------------------|-------|
| `atp-architecture-summary.prose` | NO | N/A | Written but cannot be executed -- skill not registered |
| `atp-adapters-patterns.prose` | NO | N/A | Written but cannot be executed -- skill not registered |

## Baseline Comparison

The baseline tasks were executed directly as plain Claude Code prompts. Since the .prose programs could not run, the comparison is asymmetric.

| Task | .prose quality | Baseline quality | .prose cost (approx) | Baseline cost (approx) | Winner |
|------|---------------|-----------------|---------------------|----------------------|--------|
| Architecture Summary | N/A (could not run) | 8/10 | N/A | ~2K tokens in, ~500 tokens out | Baseline (by default) |
| Adapters Patterns | N/A (could not run) | 8/10 | N/A | ~3K tokens in, ~800 tokens out | Baseline (by default) |

### Baseline Results Summary

**Task 1 (Architecture Summary):** Direct prompting produced a clear, structured summary covering purpose, tech stack, package structure, and design decisions. Quality: 8/10 -- comprehensive and accurate.

**Task 2 (Adapters Patterns):** Direct prompting identified all 12 adapters, their file organization (single-file vs multi-file), the shared `AgentAdapter` ABC, typed config pattern, common exception hierarchy, and an inconsistency in directory structure conventions. Quality: 8/10.

## Key Findings

1. **OpenProse is not operational in standard Claude Code.** The skill registration mechanism is missing. The repository contains the full specification but no way to activate it as a Claude Code skill in the current environment.

2. **The "Task" tool dependency is a critical gap.** OpenProse's execution model fundamentally depends on a "Task" tool that spawns subagent sessions. This tool does not exist in standard Claude Code. The spec references "OpenClaw `sessions_spawn`" suggesting a specific runtime requirement.

3. **The specification is well-written but untestable.** The layered architecture (SKILL.md -> prose.md -> state backends) is thoughtfully designed. The 51 examples demonstrate a rich language. But without the runtime infrastructure, none of it can execute.

4. **"Simulation with sufficient fidelity is implementation" has limits.** The core thesis of OpenProse is that an LLM reading the VM spec "becomes" the VM. But the VM requires external tools (Task/subagent spawning) that the LLM cannot simulate -- it needs actual tool infrastructure.

5. **Direct prompting is strictly superior for the tasks tested.** Without a working runtime, plain Claude Code prompts accomplish the same tasks immediately, with lower latency and no overhead.

## Issues / Surprises

- **Skill discovery gap:** The `SKILL.md` file exists but there is no documented mechanism for registering it as a Claude Code skill. The `.claude/` directory configuration for skills is not explained in the repository.

- **OpenClaw dependency:** The spec references "OpenClaw" runtime primitives (sessions_spawn, read, write, web_fetch) but OpenClaw is not documented or available. This creates a hard dependency on an unavailable platform.

- **Manual VM simulation is theoretically possible but impractical.** An LLM could read prose.md and manually perform each session's work inline (without spawning subagents), but this defeats the entire architecture -- sessions would share context, there would be no isolation, no state management, and no parallelism capability.

- **The `.prose/runs/` state management is well-designed** for inspection and debugging. The file-based state with `state.md`, `bindings/`, and `agents/` directories would provide excellent observability if it worked.

## Files Created

- `evaluation/phase1/atp-architecture-summary.prose` -- Custom .prose program (written, not executed)
- `evaluation/phase1/atp-adapters-patterns.prose` -- Custom .prose program (written, not executed)
- `evaluation/phase1/baseline-prompts.md` -- Plain prompts and results for baseline comparison
- `evaluation/results/phase-1.md` -- This results file
