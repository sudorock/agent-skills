---
name: claude-recall
description: Search and extract data from Claude Code session history stored on disk under ~/.claude/projects and ~/.claude/file-history. Use whenever the user asks to find a past conversation, search their Claude history, recover an old prompt or assistant message, audit which Bash or tool calls ran in a prior session, locate a session by cwd or date, extract a transcript, or recover a historical version of a file Claude previously edited (including an older CLAUDE.md or config). Trigger even when the user phrases it loosely ("what did I ask Claude yesterday about X", "pull the commands from that session last week", "find the old version of this file before Claude changed it").
---

# Claude Recall

Claude Code records every session to disk as append-only JSONL. This skill covers how to locate sessions, filter them, and extract data from them.

## Layout

```
~/.claude/projects/<slug>/<session-uuid>.jsonl                    # main session
~/.claude/projects/<slug>/<session-uuid>/subagents/agent-*.jsonl  # spawned subagents
~/.claude/file-history/<session-uuid>/<path-hash>@v<N>            # file snapshots
```

- `<slug>` is the session's `cwd` with `/` replaced by `-` (leading `-`). A single project directory may have many slugs if the project lives under multiple paths (worktrees, symlinks).
- The real `cwd` is inside each JSONL line, not just the slug. Do not trust the slug for filtering by project.

## JSONL Line Shape

Each line is one JSON object. Top-level `.type` identifies the kind of record. Observed values:

| `.type` | Meaning |
|---|---|
| `system` | bridge/remote-control status, hooks output, meta |
| `user` | user turn: typed prompt, slash command, tool_result, local-command stdout, task notifications |
| `assistant` | assistant turn: thinking, text, tool_use |
| `attachment` | pasted/attached content |
| `file-history-snapshot` | pointer into `~/.claude/file-history/` |
| `permission-mode` | permission mode change |
| `last-prompt` | marker for last prompt in session |

Common fields on most lines: `timestamp`, `sessionId`, `cwd`, `gitBranch`, `version`, `uuid`, `parentUuid`.

### User lines

`.message.content` is either a string (typed prompt, slash command, local-command output, task notification) or an array of content blocks (`text`, `tool_result`).

Not every `user` line is something the user typed. Filter these tags out when you want real prompts:

```
<task-notification>    <local-command-stdout>  <local-command-stderr>
<command-message>      <command-name>          <command-args>
<user-prompt-submit-hook>
```

Also check `.isMeta != true`.

### Assistant lines

`.message.content` is always an array of content blocks. Observed block types: `text`, `thinking`, `tool_use`.

### Tool results

Live inside `user` lines as `tool_result` content blocks. `.content` is a string or an array of `text` / `image` / `tool_reference` blocks.

## Search Strategy

Two-stage is fastest:

1. **Prefilter with `rg`** across all JSONL files (millisecond scan over thousands of files).
2. **Refine with `jq`** only on the files that matched, to filter by message type and ignore noise.

`rg` is a regex search. Escape dots if you care about literal dots. For literal phrases use `-F`.

## Recipes

Set these once:

```bash
ROOT=~/.claude/projects
```

### Find sessions by substring anywhere in history

```bash
rg -l --no-messages -i "<pattern>" "$ROOT"
```

### Find sessions whose real cwd matches a path

The slug is lossy. Use the `cwd` field.

```bash
CWD=/Users/indy/dev/nanyar
fd -e jsonl . "$ROOT" -d 2 | while read f; do
  head -20 "$f" | jq -rc --arg cwd "$CWD" \
    'select(.cwd == $cwd) | [.timestamp, input_filename] | @tsv' 2>/dev/null | head -1
done | sort
```

### Find sessions in a date range (by file mtime)

```bash
FROM=2026-04-01
TO=2026-04-10
fd -e jsonl . "$ROOT" --changed-after "$FROM" --changed-before "$TO"
```

### List every user prompt the user actually typed (excluding noise)

```bash
FILE=<session.jsonl>
jq -rc '
  select(.type == "user" and (.message.content | type) == "string") |
  select(.isMeta != true) |
  select(.message.content | test("^<(task-notification|local-command-stdout|local-command-stderr|command-message|command-name|command-args|user-prompt-submit-hook)"; "") | not) |
  [.timestamp, .sessionId, .message.content] | @tsv
' "$FILE"
```

### Find a user-typed phrase across all sessions (two-stage)

```bash
PAT='claude\.md'
rg -l --no-messages -i "$PAT" "$ROOT" | \
  xargs -I{} jq -rc --arg p "$PAT" '
    select(.type == "user" and (.message.content | type) == "string") |
    select(.isMeta != true) |
    select(.message.content | test("^<(task-notification|local-command|command-message|command-name|command-args|user-prompt-submit-hook)"; "") | not) |
    select(.message.content | test($p; "i")) |
    [.timestamp, .sessionId, (.message.content | .[0:160])] | @tsv
  ' {}
```

### Extract plain-text conversation transcript

```bash
FILE=<session.jsonl>
jq -r '
  select(.type == "user" or .type == "assistant") |
  if .type == "user" then
    if (.message.content | type) == "string"
    then "USER: " + .message.content
    else (.message.content[] | select(.type == "text") | "USER: " + .text)
    end
  else
    (.message.content[] | select(.type == "text") | "ASSISTANT: " + .text)
  end
' "$FILE"
```

### Extract every Bash command the assistant ran

```bash
FILE=<session.jsonl>
jq -rc '
  select(.type == "assistant") |
  .message.content[] |
  select(.type == "tool_use" and .name == "Bash") |
  .input.command
' "$FILE"
```

### Extract every tool invocation of any kind

```bash
FILE=<session.jsonl>
jq -rc '
  select(.type == "assistant") |
  .message.content[] |
  select(.type == "tool_use") |
  [.name, (.input | tostring | .[0:200])] | @tsv
' "$FILE"
```

### Extract assistant thinking

```bash
FILE=<session.jsonl>
jq -r '
  select(.type == "assistant") |
  .message.content[] |
  select(.type == "thinking") |
  .thinking
' "$FILE"
```

### Find sessions that ran a specific shell command

```bash
PAT='git push'
rg -lF --no-messages "$PAT" "$ROOT" | \
  xargs -I{} jq -rc --arg p "$PAT" '
    select(.type == "assistant") |
    .message.content[]? |
    select(.type == "tool_use" and .name == "Bash") |
    select(.input.command | contains($p)) |
    [input_filename, .input.command] | @tsv
  ' {}
```

### Count message types in a session

```bash
FILE=<session.jsonl>
jq -r '.type' "$FILE" | sort | uniq -c
```

### Resolve session-uuid to its file

```bash
SID=<session-uuid>
fd "^${SID}\.jsonl$" "$ROOT"
```

### Locate subagent transcripts for a parent session

```bash
SID=<session-uuid>
fd -e jsonl . "$ROOT"/*/"$SID"/subagents 2>/dev/null
```

### Recover a prior version of an edited file

`~/.claude/file-history/` stores versioned snapshots of files the assistant modified. Each session directory contains `<path-hash>@v<N>` plain files.

```bash
# Find every historical snapshot whose content matches a phrase
rg -l --no-messages "<phrase>" ~/.claude/file-history/ | \
  xargs -I{} stat -f '%Sm  %z  %N' {} | sort
```

The content of a `<path-hash>@v<N>` file is the full file contents at that revision. The hash is not reversible to the original path; identify the right snapshot by mtime, size, and content.

## Performance Notes

- `rg` across `~/.claude/projects/` (thousands of files) is sub-second.
- `jq` per-file is the bottleneck. Always prefilter with `rg` first when searching by phrase.
- `head -20 | jq` is faster than `jq` on a whole file when you only need metadata from the first few lines (e.g. `cwd`, first `timestamp`).
- Two-stage recipes spawn one `jq` per matched file via `xargs -I{}`. For large match sets, add `-P 8` (or similar) to run jq invocations in parallel: `xargs -I{} -P 8 jq ...`. Scales near-linearly on multi-core machines.
- JSONL is line-delimited JSON: one record per line. Never load the whole file into a single `jq` invocation without `-c`/streaming for large sessions.

## Gotchas

- The slug under `~/.claude/projects/` is derived from the cwd at session start. Worktrees, symlinks, and renamed directories produce separate slugs for the same logical project. Always filter on the `cwd` field when correctness matters.
- User-typed prompts and system-generated user turns both have `.type == "user"`. Filter tag-prefixed content to get real prompts.
- `tool_result.content` is sometimes a string and sometimes an array. Handle both: `if type == "string" then . else .[].text? // empty end`.
- Subagent transcripts are separate JSONL files under `.../<session-uuid>/subagents/`. A `rg -l` over `$ROOT` includes them; a `fd -d 2` does not.
- Session JSONL does not store the loaded `CLAUDE.md` content. Only the user prompts, assistant output, and tool I/O are persisted. To recover historical `CLAUDE.md` content, use `~/.claude/file-history/` or external backups.
- `timestamp` is ISO-8601 UTC. File mtime reflects the last append, not session start.
