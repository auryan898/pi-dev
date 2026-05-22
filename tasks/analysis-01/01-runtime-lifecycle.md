# Runtime Lifecycle

## 1. Purpose

`pi` is structured as a layered coding-agent runtime:

1. CLI startup decides how the process should run.
2. A session runtime is assembled around a working directory and a persisted session file.
3. A stateful agent session wraps the lower-level agent loop.
4. A host mode exposes that session through a TUI, plain stdout, or JSON RPC.

This separation is the first property worth preserving in another implementation.

## 2. Startup flow

At startup, the process:

1. Parses arguments.
2. Detects whether stdin is interactive or piped.
3. Resolves the application mode: interactive, print/text, print/json, or RPC.
4. Loads migrations and settings.
5. Resolves the effective session location.
6. Creates services bound to the target working directory.
7. Creates an `AgentSession`.
8. Hands that session to the selected host mode.

Source: `packages/coding-agent/src/main.ts`

## 3. Session selection model

The runtime treats session selection as a first-class concern, not a side effect.

Supported behaviors include:

- create a new session
- continue the most recent session in the current project
- open a specific session
- resume via an interactive picker
- fork an existing session into the current project
- run without persistence using an in-memory session

Important recreation detail: the selected session can imply a different working directory than the current shell, so configuration and resource discovery happen only after the final session cwd is known.

Source: `packages/coding-agent/src/main.ts`, `packages/coding-agent/src/core/session-manager.ts`, `packages/coding-agent/src/core/agent-session-runtime.ts`

## 4. Runtime assembly

The runtime host owns more than the agent object. It owns:

- the current `AgentSession`
- cwd-bound services such as settings, auth, model registry, and resources
- diagnostics gathered during construction
- logic for replacing the current session at runtime

That means session switching, forking, and reloading are runtime operations, not process restarts.

Source: `packages/coding-agent/src/core/agent-session-runtime.ts`, `packages/coding-agent/src/core/agent-session-services.ts`

## 5. Core session abstraction

`AgentSession` is the main behavioral unit shared by all modes. It centralizes:

- prompt submission
- queueing of steering and follow-up messages
- session persistence
- compaction and retry logic
- model and thinking-level changes
- tool registration and extension integration
- session tree navigation and branching

The host mode should stay thin and delegate policy to this layer.

Source: `packages/coding-agent/src/core/agent-session.ts`

## 6. Recreated architecture recommendation

To recreate `pi` in another stack, keep these layers separate:

- **launcher layer**: argument parsing, stdin inspection, process mode selection
- **runtime host layer**: owns current session and handles session replacement
- **agent-session layer**: policy, persistence, configuration, retries, context shaping
- **agent-core layer**: generic turn loop with streaming and tool execution
- **mode adapters**: TUI, non-interactive text/JSON, or RPC

This keeps the core loop portable while allowing different transports and UIs.
