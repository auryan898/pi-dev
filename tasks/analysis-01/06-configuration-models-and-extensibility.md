# Configuration, Models, and Extensibility

## 1. Settings cascade

Settings are merged from:

1. global settings
2. project settings
3. CLI overrides

The merge is deep for nested objects and value-replacing for primitives and arrays.

Source: `packages/coding-agent/src/core/settings-manager.ts`

## 2. What configuration controls

Settings influence:

- default provider and model
- default thinking level
- steering and follow-up queue modes
- compaction and retry policies
- transport behavior
- UI/theme behavior
- enabled models and package sources
- extension, skill, prompt-template, and theme discovery
- session storage location

This is broader than model configuration; it is runtime-policy configuration.

Source: `packages/coding-agent/src/core/settings-manager.ts`

## 3. Model resolution

Model selection supports:

- explicit provider/model references
- model IDs without provider when unambiguous
- scoped model sets for cycling
- saved defaults from previous use

Thinking level is resolved together with model selection and clamped to the chosen model's capabilities.

Source: `packages/coding-agent/src/core/model-resolver.ts`, `packages/coding-agent/src/core/agent-session.ts`, `packages/coding-agent/src/main.ts`

## 4. Authentication model

Authentication is resolved as part of session behavior:

- API keys or OAuth credentials are looked up by provider
- prompts are blocked before execution if required auth is unavailable
- compaction and normal prompting each resolve auth through the model registry

This is important because the runtime supports provider switching during the life of a session.

Source: `packages/coding-agent/src/core/agent-session.ts`, `packages/coding-agent/src/core/auth-storage.ts`, `packages/coding-agent/src/core/model-registry.ts`

## 5. System prompt construction

The system prompt is assembled dynamically from:

- a default or custom base prompt
- the currently active tools
- tool-specific prompt snippets and guidelines
- project context files
- available skills
- current date
- current working directory

Extensions can further modify the prompt on a per-turn basis before the agent starts.

Source: `packages/coding-agent/src/core/system-prompt.ts`, `packages/coding-agent/src/core/agent-session.ts`

## 6. Skills and prompt templates

`pi` includes two lightweight composition features:

- **skills**: reusable instruction documents that can be expanded inline from command syntax
- **prompt templates**: reusable text templates expanded before submission

These are not separate execution engines. They are preprocessing layers over normal user input.

Source: `packages/coding-agent/src/core/agent-session.ts`, `packages/coding-agent/src/core/prompt-templates.ts`, `packages/coding-agent/src/core/skills.ts`

## 7. Extension model

Extensions are a major part of the architecture. They can:

- handle lifecycle events
- register tools
- register commands and shortcuts
- inject UI
- discover more resources
- modify tool behavior
- participate in compaction and tree navigation flows
- persist custom session data

In effect, extensions can change both product behavior and host UX without rewriting the core agent loop.

Source: `packages/coding-agent/src/core/extensions/types.ts`, `packages/coding-agent/src/core/extensions/runner.ts`, `packages/coding-agent/src/core/agent-session.ts`

## 8. Extension context design

Extension APIs are split by safety and intent:

- general extension context for observation and lightweight control
- command context for explicit session-changing actions
- replacement-session context for post-switch work after new session binding

This separation prevents stale references and clarifies which operations are safe in which lifecycle stage.

Source: `packages/coding-agent/src/core/extensions/types.ts`, `packages/coding-agent/src/core/agent-session-runtime.ts`

## 9. Recreation guidance

To reproduce the extensibility model:

- expose a rich event bus around the session layer
- let extensions register tools, commands, and UI hooks
- give extensions a read/write session context with clear safety boundaries
- treat prompt construction as composable data, not a hardcoded string
- allow extension-defined persistence entries in session history

This is what makes `pi` feel like a harness rather than a single fixed application.
