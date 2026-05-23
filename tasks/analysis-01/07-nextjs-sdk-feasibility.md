# Next.js SDK Feasibility

## 1. Short answer

Yes, the pi SDK can back a Next.js web product, but not as a purely static site.

The SDK is explicitly intended for building custom UIs, including web interfaces. The recommended path for a Node.js or TypeScript application is to use `AgentSession` directly rather than spawning the RPC subprocess.

Sources:

- `packages/coding-agent/docs/sdk.md`
- `packages/coding-agent/docs/rpc.md`

## 2. Why the SDK fits a web product

The SDK already exposes the main pieces a web host needs:

- `createAgentSession()` for one active session
- `createAgentSessionRuntime()` for session replacement flows
- event subscriptions for streaming updates
- queueing APIs for steer and follow-up behavior
- session persistence and tree navigation
- model, auth, and tool configuration
- extension loading and extension-driven UI requests

That means a Next.js app does not need to reimplement pi's agent logic. It mainly needs to host it and provide browser-facing transport plus UI.

Sources:

- `packages/coding-agent/docs/sdk.md`
- `packages/coding-agent/src/core/sdk.ts`
- `packages/coding-agent/src/core/agent-session-runtime.ts`

## 3. The main architectural constraint

The problem is the phrase "NextJS SSG web project" combined with "entire feature set."

Pure SSG is not enough for the full pi feature set because pi is a long-lived, stateful runtime with:

- mutable session state
- persisted session files
- streaming model output
- tool execution
- session replacement
- compaction and retries
- extension UI request/response flows

Those behaviors require a live server-side runtime.

So the correct architecture is:

- **SSG for static shell pages** such as landing pages, docs, settings scaffolding, or pre-rendered session browsers
- **dynamic Node runtime** for the actual agent workspace

Sources:

- `packages/coding-agent/docs/sdk.md`
- `packages/coding-agent/src/core/agent-session.ts`
- `packages/coding-agent/src/core/agent-session-runtime.ts`
- `packages/coding-agent/src/core/session-manager.ts`

## 4. What "full feature set" means in web terms

To match pi's behavior closely, the web product must support:

- prompt submission with streaming output
- steering and follow-up queues while a run is active
- session resume and session creation
- tree navigation, branching, fork, and clone-like flows
- compaction and branch summaries
- model selection and thinking-level changes
- built-in tools and extension tools
- extension commands and extension UI interactions
- auth and provider configuration

That is not a static page problem. It is a stateful application problem with a static front-end shell.

Sources:

- `packages/coding-agent/docs/sdk.md`
- `packages/coding-agent/docs/sessions.md`
- `packages/coding-agent/docs/extensions.md`
- `packages/coding-agent/docs/rpc.md`

## 5. Recommended interpretation for a Next.js project

If this were recreated in Next.js, the clean interpretation would be:

- use Next.js SSG for public and mostly read-only pages
- use App Router server routes or a dedicated Node service for the interactive agent runtime
- use the SDK directly on the server
- treat the browser as a rich client on top of the server-owned `AgentSession` or `AgentSessionRuntime`

That preserves pi's behavior without forcing the agent into a browser-only or build-time model it does not support.
