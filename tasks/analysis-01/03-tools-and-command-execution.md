# Tools and Command Execution

## 1. Tool philosophy

The agent is not hardcoded to only chat. It is designed as a tool-using runtime where the model reasons in natural language and delegates concrete work to typed tools.

Built-in tools are split between:

- read-only exploration
- code/file mutation
- shell execution

Source: `packages/coding-agent/src/core/tools/index.ts`

## 2. Built-in tool set

The default built-in tool families are:

- `read`
- `bash`
- `edit`
- `write`
- `grep`
- `find`
- `ls`

The runtime can expose all tools, a coding subset, a read-only subset, or a custom allowlist.

Source: `packages/coding-agent/src/core/tools/index.ts`, `packages/coding-agent/src/core/agent-session.ts`

## 3. Tool definition shape

A tool definition contains:

- stable name
- UI label
- model-facing description
- typed parameter schema
- optional argument preprocessor
- execution mode preference
- executor function
- optional custom rendering hooks for the UI
- optional prompt snippets and guidelines for system-prompt assembly

This is broader than a simple function registry. It treats tools as both execution units and UX/prompting units.

Source: `packages/coding-agent/src/core/extensions/types.ts`

## 4. Tool execution lifecycle

For each tool call:

1. parse and validate arguments
2. emit a start event
3. run `beforeToolCall` interception
4. execute the tool
5. stream intermediate updates if available
6. run `afterToolCall` interception
7. emit completion
8. synthesize a tool-result message for the transcript

Errors are represented as failed tool results, not as crashes in the main loop.

Source: `packages/agent/src/agent-loop.ts`, `packages/coding-agent/src/core/agent-session.ts`

## 5. Shell execution model

Shell execution is treated as just another tool, but with extra behavior:

- streamed output chunks
- truncation limits
- cancellation support
- detached child tracking for cleanup
- abstract operations so the shell backend can be local or replaced

The important portable design choice is the abstraction of shell operations behind an interface rather than binding the tool directly to local process spawning.

Source: `packages/coding-agent/src/core/bash-executor.ts`, `packages/coding-agent/src/core/tools/bash.ts`, `packages/coding-agent/src/modes/print-mode.ts`, `packages/coding-agent/src/modes/rpc/rpc-mode.ts`

## 6. Mutation boundaries

File changes are performed by dedicated file tools rather than by ad hoc shell commands. That gives the runtime:

- better promptability
- predictable argument shapes
- more structured UI rendering
- cleaner logging and replay

A recreation should preserve the distinction between structured file tools and unrestricted shell access.

Source: `packages/coding-agent/src/core/tools/index.ts`, `packages/coding-agent/src/core/extensions/types.ts`

## 7. Extension tools

Extensions can register tools with the same lifecycle shape as built-ins. This means:

- the model can call extension tools the same way it calls native tools
- extensions can add domain-specific actions without patching core runtime code
- prompt content can change based on which tools are currently active

Source: `packages/coding-agent/src/core/extensions/types.ts`, `packages/coding-agent/src/core/agent-session.ts`

## 8. Recreation guidance

To recreate this behavior:

- use typed schemas for tool contracts
- normalize all tool outputs into a single transcript format
- make tools observable through lifecycle events
- separate shell execution from file editing tools
- allow per-tool concurrency policy
- allow interception before and after tool execution

That reproduces the main operational semantics of `pi`'s tool system.
