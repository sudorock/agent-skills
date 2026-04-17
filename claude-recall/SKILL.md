---
name: claude-recall
description: Search and extract data from Claude Code session history stored on disk under ~/.claude/projects and ~/.claude/file-history. Use whenever the user asks to find a past conversation, search their Claude history, recover an old prompt or assistant message, audit which Bash or tool calls ran in a prior session, locate a session by cwd or date, extract a transcript, or recover a historical version of a file Claude previously edited (including an older CLAUDE.md or config). Trigger even when the user phrases it loosely ("what did I ask Claude yesterday about X", "pull the commands from that session last week", "find the old version of this file before Claude changed it").
---

# 1 Claude Recall

Locate, filter, extract data from Claude Code session JSONL on disk.

## 1.1 Tool Availability

Recipes use `rg` (ripgrep), `fd`. IF unavailable, substitute:
1. `rg` -> `grep -r`
2. `fd` -> `find`

## 1.2 Layout

1.
```
~/.claude/projects/<slug>/<session-uuid>.jsonl                    # main session
~/.claude/projects/<slug>/<session-uuid>/subagents/agent-*.jsonl  # subagents
~/.claude/file-history/<session-uuid>/<path-hash>@v<N>            # file snapshots
```

2. `<slug>` = cwd with `/` -> `-` (leading `-`); lossy; worktrees, symlinks produce separate slugs
3. real `cwd` inside each JSONL line; do not trust slug for project filtering

## 1.3 JSONL Line Shape

each line = one JSON object; `.type` identifies record kind

1.
| `.type` | Meaning |
|---|---|
| `system` | bridge/remote-control status, hooks, meta |
| `user` | typed prompt, slash command, tool_result, local-command stdout, task notification |
| `assistant` | thinking, text, tool_use |
| `attachment` | pasted/attached content |
| `file-history-snapshot` | pointer into `~/.claude/file-history/` |
| `permission-mode` | permission mode change |
| `last-prompt` | last prompt marker |

2. common fields: `timestamp`, `sessionId`, `cwd`, `gitBranch`, `version`, `uuid`, `parentUuid`

### 1.3.1 User Lines

1. `.message.content` = string (prompt, slash cmd, local-command output) | array of content blocks (`text`, `tool_result`)
2. not every `user` line is user-typed; filter these tags for real prompts:
3.
```
<task-notification>    <local-command-stdout>  <local-command-stderr>
<command-message>      <command-name>          <command-args>
<user-prompt-submit-hook>
```

4. also filter `.isMeta != true`

### 1.3.2 Assistant Lines

`.message.content` = array of content blocks: `text`, `thinking`, `tool_use`

### 1.3.3 Tool Results

1. inside `user` lines as `tool_result` content blocks
2. `.content` = string | array of `text` / `image` / `tool_reference` blocks

## 1.4 Search Strategy

two-stage; fastest approach:
1. prefilter: `rg` across all JSONL (sub-second over thousands of files)
2. refine: `jq` on matched files only; filter by type, ignore noise
3. `rg` = regex; escape dots for literal match; `-F` for literal phrases

## 1.5 Recipes

```bash
ROOT=~/.claude/projects
```

### 1.5.1 Find Sessions by Substring

```bash
rg -l --no-messages -i "<pattern>" "$ROOT"
```

### 1.5.2 Find Sessions by Real cwd

slug is lossy; use `cwd` field:

```bash
CWD=/Users/indy/dev/nanyar
fd -e jsonl . "$ROOT" -d 2 | while read f; do
  head -20 "$f" | jq -rc --arg cwd "$CWD" \
    'select(.cwd == $cwd) | [.timestamp, input_filename] | @tsv' 2>/dev/null | head -1
done | sort
```

### 1.5.3 Find Sessions by Date Range (file mtime)

```bash
FROM=2026-04-01
TO=2026-04-10
fd -e jsonl . "$ROOT" --changed-after "$FROM" --changed-before "$TO"
```

### 1.5.4 Extract Real User Prompts (exclude noise)

```bash
FILE=<session.jsonl>
jq -rc '
  select(.type == "user" and (.message.content | type) == "string") |
  select(.isMeta != true) |
  select(.message.content | test("^<(task-notification|local-command-stdout|local-command-stderr|command-message|command-name|command-args|user-prompt-submit-hook)"; "") | not) |
  [.timestamp, .sessionId, .message.content] | @tsv
' "$FILE"
```

### 1.5.5 Find User-Typed Phrase Across All Sessions (two-stage)

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

### 1.5.6 Extract Conversation Transcript

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

### 1.5.7 Extract Bash Commands

```bash
FILE=<session.jsonl>
jq -rc '
  select(.type == "assistant") |
  .message.content[] |
  select(.type == "tool_use" and .name == "Bash") |
  .input.command
' "$FILE"
```

### 1.5.8 Extract All Tool Invocations

```bash
FILE=<session.jsonl>
jq -rc '
  select(.type == "assistant") |
  .message.content[] |
  select(.type == "tool_use") |
  [.name, (.input | tostring | .[0:200])] | @tsv
' "$FILE"
```

### 1.5.9 Extract Assistant Thinking

```bash
FILE=<session.jsonl>
jq -r '
  select(.type == "assistant") |
  .message.content[] |
  select(.type == "thinking") |
  .thinking
' "$FILE"
```

### 1.5.10 Find Sessions by Shell Command

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

### 1.5.11 Count Message Types

```bash
FILE=<session.jsonl>
jq -r '.type' "$FILE" | sort | uniq -c
```

### 1.5.12 Resolve session-uuid to File

```bash
SID=<session-uuid>
fd "^${SID}\.jsonl$" "$ROOT"
```

### 1.5.13 Locate Subagent Transcripts

```bash
SID=<session-uuid>
fd -e jsonl . "$ROOT"/*/"$SID"/subagents 2>/dev/null
```

### 1.5.14 Recover Prior File Version

`~/.claude/file-history/` stores versioned snapshots; `<path-hash>@v<N>` = full file contents at that revision
1.
```bash
rg -l --no-messages "<phrase>" ~/.claude/file-history/ | \
  xargs -I{} stat -f '%Sm  %z  %N' {} | sort
```

2. hash not reversible to original path; identify snapshot by mtime, size, content

## 1.6 Performance

1. `rg` across `~/.claude/projects/` = sub-second (thousands of files)
2. `jq` per-file = bottleneck; always prefilter with `rg` first
3. `head -20 | jq` faster than full-file `jq` for metadata (cwd, first timestamp)
4. large match sets: `xargs -I{} -P 8 jq ...` for parallel jq; scales near-linearly
5. JSONL = one record per line; always use `-c`/streaming for large sessions

## 1.7 Gotchas

1. slug derived from cwd at session start; worktrees, symlinks, renames -> separate slugs; filter on `cwd` field when correctness matters
2. `.type == "user"` includes system-generated turns; filter tag-prefixed content for real prompts
3. `tool_result.content` = string | array; handle both: `if type == "string" then . else .[].text? // empty end`
4. subagent transcripts = separate JSONL under `.../<session-uuid>/subagents/`; `rg -l` over `$ROOT` includes them; `fd -d 2` does not
5. session JSONL does not store loaded `CLAUDE.md`; recover via `~/.claude/file-history/` or external backups
6. `timestamp` = ISO-8601 UTC; file mtime = last append, not session start
