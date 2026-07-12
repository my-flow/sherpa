---
name: sherpa
description: Fetch and summarize unread email threads
homepage: https://github.com/my-flow/sherpa
metadata: {"openclaw":{"requires":{"bins":["awk","cut","grep","himalaya","jq","mktemp","parallel","sed","shasum","sherpa","timeout","tr"]}}}
---

# Sherpa – Unread Email Assistant
Use `sherpa` to retrieve the user's unread email threads across all folders.

# Workflow
## Step 1 – Check for new mail
Run the lightweight checksum query:
```sh
sherpa | jq '{checksum} + ([.data[] | {id: .ids[], folder: .folders[]}] | group_by(.folder) | map({(.[0].folder): [.[].id] | unique}) | add)'
```
Compare the `checksum` field against `memory/email-state.json`. If the file does not exist yet, treat every email as new.

## Step 2 – Update email state
Extend `memory/email-state.json`: replace `checksum` and store the `folders` with their new `ids`.

## Step 3 – Fetch full content (only when checksum changed)
If the checksum differs from the last known value, run:
```sh
sherpa
```
Parse the full JSON output. Each entry contains `from`, `subject`, `date`, `has_attachment`, `folders`, and `thread` fields.

## Step 4 – Report
**Report only NEW unread emails** — messages whose ids were not present in the previous state.
**Report only RELEVANT emails** — see relevance rules below.
Present a concise human-readable summary of new threads grouped by sender or topic. Quote the relevant `thread` context. Never expose internal/technical details, such as `checksum`, `ids`, and state changes.

# Email state tracking
Read/write `memory/email-state.json` to track which IDs were already reported.
Format: `{ "checksum": checksum, "FolderName": [id1, id2, ...] }` — last checksum and list of IDs already seen/reported.

# Relevance rules: pure feedback-based — no assumptions
**Start by reporting ALL new unread emails.**
Only suppress an email type after the user explicitly says to ignore it.
Log each feedback decision in `memory/email-relevance.md`.
