# Next.js Feature Mapping

## 1. Prompting and streaming

Pi's core browser experience would map naturally to:

- editor input in the browser
- server call to `session.prompt()`
- live event stream rendered into assistant text, thinking blocks, and tool activity rows

`message_update` events become the browser's incremental render path. `message_end` closes the row. `agent_end` clears the active-run state.

Sources:

- `packages/coding-agent/docs/sdk.md`
- `packages/agent/src/agent-loop.ts`
- `packages/coding-agent/src/core/agent-session.ts`

## 2. Steering and follow-up

The web UI should expose the same two queueing concepts pi uses:

- **steer**: interrupt the next model turn after current tool work
- **follow-up**: queue work for when the agent naturally stops

This should not be collapsed into one generic "send later" button, because the semantics are different and are important to how pi behaves.

The browser should also show queue state from `queue_update` events.

Sources:

- `packages/coding-agent/docs/sdk.md`
- `packages/coding-agent/src/core/agent-session.ts`

## 3. Sessions and tree navigation

The web app can map pi's session model to:

- a session list page
- a session detail/workspace page
- a tree viewer for branch navigation
- branch/fork controls on prior messages

The active transcript should come from the current session path, not from a flat message log.

When the user moves in the tree, the web UI should follow pi's semantics:

- selecting a user message prepares a new branch from before that message
- selecting a non-user entry resumes from that point

Sources:

- `packages/coding-agent/docs/sessions.md`
- `packages/coding-agent/src/core/session-manager.ts`

## 4. Compaction and branch summaries

The browser should expose:

- a manual compact action
- compaction progress and errors
- visual markers for compaction summary entries
- branch-summary entries when navigating between branches

These are core pi features, not background implementation details.

Sources:

- `packages/coding-agent/src/core/agent-session.ts`
- `packages/coding-agent/src/core/session-manager.ts`
- `packages/coding-agent/src/core/compaction/branch-summarization.ts`

## 5. Tools

The browser should represent tools as structured activity, not just text blobs.

Feature mapping:

- tool start/end rows from tool execution events
- expandable details per tool call
- separate rendering for read, bash, edit, write, grep, find, and ls
- optional custom rendering for extension tools

The backend should keep tool execution authoritative. The browser only renders the emitted state.

Sources:

- `packages/coding-agent/src/core/tools/index.ts`
- `packages/coding-agent/src/core/extensions/types.ts`
- `packages/coding-agent/src/modes/interactive/components/tool-execution.ts`

## 6. Models, providers, and auth

A full-featured web UI would also need:

- provider/model picker
- thinking-level selector
- configured-model visibility per user or workspace
- auth/login management for provider credentials

The model and auth system already exist in the SDK. The web app's job is to wrap them in browser-safe flows and admin/user boundaries.

Sources:

- `packages/coding-agent/docs/sdk.md`
- `packages/coding-agent/src/core/model-resolver.ts`
- `packages/coding-agent/src/core/settings-manager.ts`
- `packages/coding-agent/examples/sdk/09-api-keys-and-oauth.ts`

## 7. Extensions, commands, and custom UI

To cover pi's full feature set, the browser app must handle extension-driven behaviors too.

That includes:

- custom slash/extension commands
- custom tools
- extension notifications
- confirm/select/input/editor-style UI requests
- persistent extension status areas and widgets

In other words, the browser app needs an extension UI adapter the same way the TUI and RPC mode do.

Sources:

- `packages/coding-agent/docs/extensions.md`
- `packages/coding-agent/docs/rpc.md`
- `packages/coding-agent/src/core/extensions/types.ts`

## 8. What the browser can pre-render

Even in a dynamic workspace, the browser can still pre-render:

- session lists
- workspace chrome
- settings screens
- documentation/help panels
- empty-state pages

But the active transcript and live tool stream must be hydrated from server state.
