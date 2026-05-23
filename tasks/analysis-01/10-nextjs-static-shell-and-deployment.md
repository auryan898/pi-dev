# Next.js Static Shell and Deployment

## 1. Correct use of SSG

SSG still has a useful role in this architecture, just not for the live agent runtime.

Good SSG candidates:

- marketing pages
- product docs
- help/reference pages
- settings scaffolds
- shell routes that render the frame around the agent workspace
- pre-rendered session index pages that are refreshed dynamically after load

The interactive agent pane itself should be treated as dynamic.

## 2. Runtime boundary

The full pi feature set depends on a live Node runtime with:

- filesystem access for sessions and tools
- process execution for bash
- long-lived memory for active session objects
- streaming responses to the client

That means the agent backend should run in a Node environment, not an edge-only or static-only target.

Sources:

- `packages/coding-agent/src/core/sdk.ts`
- `packages/coding-agent/src/core/agent-session.ts`
- `packages/coding-agent/src/core/session-manager.ts`

## 3. Multi-user storage boundary

A hosted web app must separate storage domains that the local CLI collapses into one machine-local layout.

At minimum, the product would need per-user or per-workspace handling for:

- auth data
- sessions
- uploaded images/files
- settings
- extension packages or extension config

Without that separation, a direct reuse of local defaults would leak state across users.

Sources:

- `packages/coding-agent/examples/sdk/09-api-keys-and-oauth.ts`
- `packages/coding-agent/examples/sdk/11-sessions.ts`
- `packages/coding-agent/examples/sdk/12-full-control.ts`

## 4. Security boundary

Pi's full feature set includes powerful local tools:

- shell execution
- file edits
- filesystem reads

A hosted Next.js product must decide whether it really means "entire feature set" literally. If yes, those tools need to run inside a sandbox or per-workspace execution environment, not on the shared web server host.

So the deployment model should distinguish:

- **web server** for browser/API/auth
- **workspace runtime** for agent execution and tool access

That could be separate processes, containers, or remote workers.

Sources:

- `packages/coding-agent/src/core/tools/index.ts`
- `packages/coding-agent/src/core/bash-executor.ts`
- `packages/coding-agent/docs/extensions.md`

## 5. Recommended product shape

The most faithful product design would be:

1. a statically generated Next.js shell
2. authenticated dynamic workspace routes
3. a Node-based session/runtime service using the SDK directly
4. streaming transport from runtime to browser
5. isolated execution environments for tools
6. a browser-side extension UI adapter

That keeps the parts Next.js is good at while preserving pi's real runtime semantics.

## 6. Bottom line

If the goal is a true browser UI on top of the pi SDK, the project should be described as:

- **a Next.js application with static outer pages and a dynamic agent workspace**

not as:

- **a purely static Next.js SSG application**

That distinction is necessary to preserve the behavior described across the rest of `tasks/analysis-01/`.
