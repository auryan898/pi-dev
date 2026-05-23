# Next.js Runtime and API Architecture

## 1. Server-side ownership model

The browser should not own the canonical agent state. The server should.

The clean server model is:

- one server-side `AgentSession` per active browser workspace, or
- one server-side `AgentSessionRuntime` per workspace when session replacement is needed

The browser receives events and sends user intents.

Sources:

- `packages/coding-agent/docs/sdk.md`
- `packages/coding-agent/src/core/agent-session-runtime.ts`

## 2. When to use `createAgentSession()`

Use `createAgentSession()` when the web app only needs one active session object and can recreate it explicitly when needed.

This fits:

- simple embedded chat pages
- ephemeral workspaces
- server-managed one-session-per-tab experiences

It gives the web backend:

- prompting
- event subscriptions
- model/tool access
- compaction
- queueing
- direct state access

Sources:

- `packages/coding-agent/docs/sdk.md`
- `packages/coding-agent/src/core/sdk.ts`

## 3. When to use `createAgentSessionRuntime()`

Use `createAgentSessionRuntime()` when the web app wants to support pi's session-replacement flows in place:

- new session
- switch session
- fork
- import

This matters for a browser app trying to mirror pi closely. The runtime abstraction exists specifically because the active session object changes after these operations.

Important consequence: after replacement, the host must rebind subscriptions and extension bindings to `runtime.session`.

Sources:

- `packages/coding-agent/docs/sdk.md`
- `packages/coding-agent/examples/sdk/13-session-runtime.ts`
- `packages/coding-agent/src/core/agent-session-runtime.ts`

## 4. Transport layer

A Next.js agent workspace needs a push channel from server to browser.

The minimum useful design is:

- HTTP POST for user actions
- SSE or WebSocket for event streaming back to the browser

This is needed because `session.subscribe()` emits a live event stream during prompting, tool execution, retries, and compaction.

A plain request/response API would lose the main interactive behavior of pi.

Sources:

- `packages/coding-agent/docs/sdk.md`
- `packages/coding-agent/src/core/agent-session.ts`

## 5. Backend endpoint responsibilities

The backend layer would expose operations roughly like:

- prompt
- steer
- follow-up
- abort
- get current state
- create new session
- switch session
- fork session/tree entry
- compact
- set model
- set thinking level
- list sessions
- read session tree

These are not guesses; they map directly to the session and runtime APIs, and they align with the behaviors already formalized in RPC mode.

Sources:

- `packages/coding-agent/docs/sdk.md`
- `packages/coding-agent/docs/rpc.md`
- `packages/coding-agent/src/core/agent-session.ts`
- `packages/coding-agent/src/core/agent-session-runtime.ts`

## 6. Resource and configuration bootstrap

The server bootstrap should create or inject:

- `AuthStorage`
- `ModelRegistry`
- `SettingsManager`
- `SessionManager`
- `ResourceLoader`

For a multi-user web product, those should not default blindly to one shared `~/.pi/agent` directory. They should be mapped to tenant- or user-scoped storage.

That is one of the biggest differences between a local CLI and a hosted Next.js product.

Sources:

- `packages/coding-agent/docs/sdk.md`
- `packages/coding-agent/examples/sdk/09-api-keys-and-oauth.ts`
- `packages/coding-agent/examples/sdk/11-sessions.ts`
- `packages/coding-agent/examples/sdk/12-full-control.ts`

## 7. Suggested decomposition inside a Next.js app

A practical split would be:

- **Next.js app routes** for the web shell and authenticated pages
- **server-side agent service** wrapping SDK sessions and runtimes
- **event bridge** converting session events into browser-consumable messages
- **storage layer** mapping users/workspaces to sessions, settings, auth, and uploads

This keeps the SDK-facing code concentrated and prevents UI code from depending on local CLI assumptions.
