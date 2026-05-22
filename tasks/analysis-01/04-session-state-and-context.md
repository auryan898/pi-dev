# Session State and Context

## 1. Session as append-only history

`pi` persists sessions as JSONL histories with typed entries. A session is not just a raw chat transcript. It includes:

- messages
- model changes
- thinking-level changes
- compaction summaries
- branch summaries
- custom extension entries
- labels and session metadata

Source: `packages/coding-agent/src/core/session-manager.ts`

## 2. Tree structure instead of flat history

Each entry has an id and parent id, so a session behaves like a navigable tree path rather than a single immutable line.

This enables:

- forking from prior points
- navigating to different leaves
- summarizing skipped branches
- rebuilding any leaf-specific context

A portable implementation should store explicit parent links, not just ordered timestamps.

Source: `packages/coding-agent/src/core/session-manager.ts`, `packages/coding-agent/src/core/agent-session-runtime.ts`

## 3. Reconstructing active context

The active transcript is rebuilt by walking from the current leaf to the root and interpreting entry types along that path.

During reconstruction, the runtime also derives:

- current thinking level
- current model
- synthetic summary messages from compaction or branch summarization

This means persisted history and active model context are related but not identical.

Source: `packages/coding-agent/src/core/session-manager.ts`

## 4. Compaction behavior

Compaction exists to reduce context size without losing the session narrative.

The process is:

1. inspect the current branch
2. compute what can be summarized while keeping recent context
3. ask a model to summarize the dropped region, or accept an extension-provided summary
4. persist a compaction entry with summary and resume boundary
5. rebuild in-memory context so the summary replaces the dropped span

Compaction can be:

- manual
- threshold-triggered
- overflow-recovery triggered

Source: `packages/coding-agent/src/core/agent-session.ts`, `packages/coding-agent/src/core/compaction/compaction.ts`

## 5. Overflow recovery

If a provider returns a context-overflow error:

1. the failing assistant error message is removed from active context
2. compaction runs automatically
3. the agent retries once

This is important behavior to preserve: context overflow is treated as a recoverable state-management problem, not only as a user-visible error.

Source: `packages/coding-agent/src/core/agent-session.ts`

## 6. Branch summarization

When the user forks or navigates through the session tree, `pi` can summarize the work done on the abandoned path and inject that summary into the new branch context.

This keeps branch context small while preserving previous outcomes.

Source: `packages/coding-agent/src/core/compaction/branch-summarization.ts`, `packages/coding-agent/src/core/session-manager.ts`, `packages/coding-agent/src/core/agent-session-runtime.ts`

## 7. Retry behavior

Retry handling is session policy, not provider policy alone.

After a run ends, the session layer checks whether the last assistant message represents a retryable failure. If so, it performs bounded exponential backoff and continues the agent.

Notably:

- retryable transport/provider failures are retried
- context overflow is not handled as normal retry; it goes through compaction

Source: `packages/coding-agent/src/core/agent-session.ts`

## 8. Recreation guidance

To recreate the same behavior:

- persist transcript and control-plane events together
- model the session as a branchable tree
- rebuild live context from persistent history instead of trusting memory alone
- support summary entries that replace long spans of old context
- separate retry policy from provider SDK internals

This is one of the main reasons `pi` can resume and reshape long-lived work reliably.
