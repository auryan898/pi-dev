# Modes and User Experience

## 1. Same session, different hosts

`pi` does not create separate agent implementations for TUI, print, and RPC. Each mode is a host around the same `AgentSession`.

That design is worth preserving because it keeps behavior consistent across interfaces.

Source: `packages/coding-agent/src/modes/interactive/interactive-mode.ts`, `packages/coding-agent/src/modes/print-mode.ts`, `packages/coding-agent/src/modes/rpc/rpc-mode.ts`

## 2. Interactive mode

Interactive mode is a terminal application that subscribes to session events and renders:

- user messages
- streamed assistant output
- tool execution rows
- queue state
- footer/status information
- selectors, dialogs, editor overlays, and login flows

Its responsibilities are primarily presentation and user interaction. The underlying session owns the behavior.

Source: `packages/coding-agent/src/modes/interactive/interactive-mode.ts`

## 3. Print mode

Print mode is single-shot execution:

- accepts an initial prompt and optional additional prompts
- runs the same session logic
- either prints only final assistant text or emits every event as JSON
- exits when done

This makes `pi` usable as a batch tool without changing the agent model.

Source: `packages/coding-agent/src/modes/print-mode.ts`

## 4. RPC mode

RPC mode exposes the session as a long-lived JSON protocol over stdin/stdout.

The host can send commands such as:

- prompt
- steer
- follow-up
- abort
- set model
- set thinking level
- create/switch sessions

The runtime answers with:

- command responses
- streamed agent/session events
- extension UI requests that the host may satisfy

This is the clearest evidence that the core behavior is transport-agnostic.

Source: `packages/coding-agent/src/modes/rpc/rpc-mode.ts`, `packages/coding-agent/src/modes/rpc/rpc-types.ts`

## 5. Host responsibilities

A generic host for this architecture needs to provide:

- event rendering or forwarding
- input capture
- interruption controls
- extension UI primitives
- cleanup on shutdown
- rebind logic when the current session is replaced

The last point matters because `pi` allows session switching and forking without restarting the process.

Source: `packages/coding-agent/src/core/agent-session-runtime.ts`, `packages/coding-agent/src/modes/print-mode.ts`, `packages/coding-agent/src/modes/rpc/rpc-mode.ts`

## 6. Queue-oriented UX

The user experience is designed around the idea that the agent may already be busy. Instead of rejecting input outright, the runtime can:

- steer the next turn
- enqueue follow-up work
- show queue state to the user

That interaction model is more agent-like than a traditional request/response chat.

Source: `packages/coding-agent/src/core/agent-session.ts`, `packages/coding-agent/src/modes/interactive/interactive-mode.ts`

## 7. Recreation guidance

In another framework, keep mode adapters thin:

- let the session own behavior
- let the host own I/O and visuals
- make mode-specific UI capabilities optional
- preserve a common event protocol across all hosts

That makes it easy to add future hosts such as web, desktop, or remote-control bridges.
