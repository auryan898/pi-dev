# Agent Loop and Events

## 1. Core mental model

The low-level agent is a turn engine over a mutable context:

- input messages are appended to context
- the model streams an assistant message
- tool calls inside that message are executed
- tool results are appended to context
- the loop either continues or stops

Source: `packages/agent/src/agent-loop.ts`

## 2. Message model

The runtime distinguishes between:

- **agent messages**: the richer internal transcript
- **LLM messages**: the reduced provider-facing representation

Internal messages can include:

- user messages
- assistant messages
- tool result messages
- custom application messages
- summary-style synthetic messages

Before each provider call, the internal transcript is transformed and then converted into provider-compatible messages. This is a key portability point: keep an internal message model richer than the provider schema.

Source: `packages/agent/src/types.ts`, `packages/agent/src/agent-loop.ts`

## 3. Event contract

The core loop emits an explicit event stream:

- `agent_start`
- `turn_start`
- `message_start`
- `message_update`
- `message_end`
- `tool_execution_start`
- `tool_execution_update`
- `tool_execution_end`
- `turn_end`
- `agent_end`

This event stream is the boundary between behavior and presentation. UIs, loggers, RPC bridges, and persistence subscribers all consume the same sequence.

Source: `packages/agent/src/types.ts`, `packages/agent/src/agent.ts`

## 4. Assistant streaming behavior

When the provider starts streaming:

1. a partial assistant message is inserted into state
2. incremental provider events update that partial message in place
3. `message_update` events are emitted for each delta
4. the partial message is replaced by the finalized assistant message
5. `message_end` is emitted once the full result is known

A portable implementation should preserve this distinction between partial and finalized assistant state.

Source: `packages/agent/src/agent-loop.ts`

## 5. Tool-call loop

After an assistant message completes:

1. inspect assistant content for tool calls
2. preflight and validate arguments
3. emit tool start events
4. execute tools
5. emit streaming updates if the tool reports progress
6. finalize each result
7. append tool-result messages to context
8. emit `turn_end`
9. decide whether another model turn is needed

Tool execution can be globally parallel or sequential, with per-tool overrides.

Source: `packages/agent/src/agent-loop.ts`

## 6. Steering and follow-up queues

`pi` supports two distinct queued-input concepts:

- **steering**: injected before the next assistant call after the current turn finishes
- **follow-up**: injected only when the agent would otherwise stop

Each queue supports either:

- one-at-a-time delivery
- all-at-once delivery

This is a useful generic design because it separates interruption from backlog.

Source: `packages/agent/src/types.ts`, `packages/agent/src/agent.ts`, `packages/coding-agent/src/core/agent-session.ts`

## 7. Session-level preflight around prompting

Before a prompt reaches the low-level loop, the session layer may:

- intercept slash-style extension commands
- let extensions transform or fully handle the input
- expand skill references
- expand prompt templates
- validate model selection and credentials
- inject pending custom context for the next turn
- let extensions alter the system prompt for that turn

In another framework, keep this preflight in a session/policy layer, not in the provider loop itself.

Source: `packages/coding-agent/src/core/agent-session.ts`

## 8. Recreated algorithm

The simplest generic reconstruction is:

1. maintain a persistent transcript
2. derive provider context from transcript before each turn
3. stream assistant output into mutable state
4. execute model-requested tools through a tool runner
5. append normalized tool results back to transcript
6. poll steering and follow-up queues between turns
7. emit events for every state transition

That reproduces the core behavior without tying it to TypeScript or the existing provider SDK.
