# Pi Agent Behavior Analysis

This directory contains a framework-agnostic analysis of how the `pi` coding agent behaves so the same design could be recreated in another language or runtime.

## Files

- `01-runtime-lifecycle.md` — startup, session selection, runtime assembly, and request lifecycle
- `02-agent-loop-and-events.md` — the core turn loop, streaming model, tool-call handling, and event contract
- `03-tools-and-command-execution.md` — tool architecture, shell execution, mutation model, and safety boundaries
- `04-session-state-and-context.md` — persisted session format, branching, compaction, retries, and context reconstruction
- `05-modes-and-user-experience.md` — interactive, print, and RPC modes, plus their host responsibilities
- `06-configuration-models-and-extensibility.md` — settings, model/auth resolution, system prompt construction, skills, and extensions

## Primary source areas

- `packages/coding-agent/src/main.ts`
- `packages/coding-agent/src/core/agent-session.ts`
- `packages/coding-agent/src/core/session-manager.ts`
- `packages/coding-agent/src/core/system-prompt.ts`
- `packages/coding-agent/src/core/extensions/types.ts`
- `packages/coding-agent/src/modes/interactive/interactive-mode.ts`
- `packages/coding-agent/src/modes/print-mode.ts`
- `packages/coding-agent/src/modes/rpc/rpc-mode.ts`
- `packages/agent/src/agent-loop.ts`
- `packages/agent/src/agent.ts`
- `packages/agent/src/types.ts`
