# agent-skills

Agent skills for Claude Code, Codex, and other AI coding agents.

## Install

All skills:

```
npx skills add sudorock/agent-skills
```

Single skill:

```
npx skills add sudorock/agent-skills --skill terse
```

## Skills

| Skill | Description |
|---|---|
| [terse](terse/) | Structured, information-dense output with coordinate addressing. ~75% token reduction without information loss. |
| [claude-recall](claude-recall/) | Search and extract data from Claude Code session history on disk. Find past conversations, recover prompts, audit tool calls, or restore file versions. |
| [pulumi-aws-naming](pulumi-aws-naming/) | Consistent naming conventions for Pulumi-managed AWS resources. Logical and physical names, resource type abbreviations, formatting exceptions. |
| [interphase](interphase/) | Reactive state management for React with `@aktopia/interphase`. Subscriptions, events, and hooks built on Zustand, TanStack Query, and Immer. |

## License

[MIT](LICENSE)
