# Claude Recall

Search and extract data from Claude Code session history stored on disk.

Find past conversations, recover prompts, audit tool invocations, extract transcripts, or restore prior file versions — all from the JSONL session logs Claude Code writes to `~/.claude/projects/`.

## Install

```
npx skills add sudorock/agent-skills --skill claude-recall
```

## What It Does

Claude Code records every session as append-only JSONL. Claude Recall teaches the agent how to navigate this data:

- **Find sessions** by keyword, date range, or working directory
- **Extract prompts** you actually typed (filtering out system noise)
- **List tool calls** — every Bash command, file read, or edit the agent ran
- **Pull transcripts** — full user/assistant conversation in plain text
- **Recover file versions** from `~/.claude/file-history/` snapshots
- **Search across all sessions** with two-stage `rg` + `jq` recipes

## Example

```
> what did I ask Claude about migrations last week?
```

The agent uses `rg` to scan all session files for "migration", then `jq` to extract the matching user prompts with timestamps.

## System Requirements

| Tool | Purpose | Install |
|---|---|---|
| [ripgrep](https://github.com/BurntSushi/ripgrep) (`rg`) | Fast regex search across session files | `brew install ripgrep` |
| [jq](https://jqlang.github.io/jq/) | Parse and filter JSONL records | `brew install jq` |
| [fd](https://github.com/sharkdp/fd) | Find session files by date/name | `brew install fd` |
