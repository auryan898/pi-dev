# Pi Agent Behavior Analysis

This directory contains a framework-agnostic analysis of how the `pi` coding agent behaves so the same design could be recreated in another language or runtime.

## Files

- `01-runtime-lifecycle.md` — startup, session selection, runtime assembly, and request lifecycle
- `02-agent-loop-and-events.md` — the core turn loop, streaming model, tool-call handling, and event contract
- `03-tools-and-command-execution.md` — tool architecture, shell execution, mutation model, and safety boundaries
- `04-session-state-and-context.md` — persisted session format, branching, compaction, retries, and context reconstruction
- `05-modes-and-user-experience.md` — interactive, print, and RPC modes, plus their host responsibilities
- `06-configuration-models-and-extensibility.md` — settings, model/auth resolution, system prompt construction, skills, and extensions
- `07-nextjs-sdk-feasibility.md` — how the pi SDK maps to a Next.js web product, including the SSG constraint
- `08-nextjs-runtime-and-api-architecture.md` — backend session/runtime patterns for a Next.js app using the SDK
- `09-nextjs-feature-mapping.md` — how to expose pi features in a browser UI
- `10-nextjs-static-shell-and-deployment.md` — what can be static, what must stay dynamic, and deployment/security boundaries

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
- `packages/coding-agent/docs/sdk.md`
- `packages/coding-agent/docs/rpc.md`
- `packages/coding-agent/docs/sessions.md`
- `packages/coding-agent/docs/extensions.md`
