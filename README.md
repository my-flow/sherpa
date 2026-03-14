<div align="center">
  <img src="./logo.png" alt="Logo" width="123" height="128" />
  <h1>📫 Sherpa</h1>
  <p>Helps your AI assistant to summarize your unread email threads</p>
</div>

```
Mailbox
├── INBOX
├── Friends (1 unseen)
├── Receipts
├── Travel (2 unseen)
└── Work (1 unseen)
```

```
$ sherpa
{
  "data": [
    {
      "from": "Alice <alice@example.org>",
      "subject": "Important meeting",
      "date": "2026-02-04 02:10+01:00",
      "thread": "-------- Message 3144 --------\nFrom: Alice <alice@example.com>\nTo: Bob <bob@example.com>\nSubject:"
    }
  ],
  "checksum": "a09a2743cf8a80009d99d84f50f12ca0f576251b2f7c0f88fbb6f638b2d63061"
}
```

# Features
- Automatically fetches all unread emails from all folders in parallel.
- Does the heavy lifting for AI assistants such as [OpenClaw](https://openclaw.ai) by providing a thread preview in JSON.
- Includes a checksum for downstream change detection. Wrap the script in a [cronjob](https://docs.openclaw.ai/automation/cron-jobs) or [heartbeat](https://docs.openclaw.ai/gateway/heartbeat) to handle state.

# Installation
## Dependencies
### macOS with Homebrew
```sh
brew install bash coreutils himalaya jq parallel
```
[Configure](https://github.com/pimalaya/himalaya?tab=readme-ov-file#configuration) Himalaya.

### Debian/Ubuntu
```sh
sudo apt-get install bash coreutils jq parallel
```
[Install](https://github.com/pimalaya/himalaya?tab=readme-ov-file#installation) and [configure](https://github.com/pimalaya/himalaya?tab=readme-ov-file#configuration) Himalaya.

### OpenClaw's [locally built Docker image](https://docs.openclaw.ai/install/docker)
```sh
export OPENCLAW_DOCKER_APT_PACKAGES="jq parallel"
./docker-setup.sh
```

## Sherpa
```sh
curl -sSL https://raw.githubusercontent.com/my-flow/sherpa/main/sherpa -o sherpa
sudo mv sherpa /usr/local/bin/sherpa
sudo chmod +x /usr/local/bin/sherpa
```

# Usage
Pass `--verbose` to print progress information to stderr:
```sh
$ sherpa --verbose
[sherpa] Fetching folder list
[sherpa] Fetching unseen mails across 5 folders in parallel
[sherpa] Finished fetching unseen mails from 5 folders
[sherpa] Fetching threads for 4 mails in parallel
[sherpa] Finished fetching 4 threads
```

```json
{
  "checksum":"a09a…",
  "data": [
    …
  ]
}
```

### Change detection with OpenClaw
[OpenClaw](https://openclaw.ai) can use Sherpa as a lightweight polling mechanism. Save tokens and let Sherpa do the heavy lifting by extracting unread emails, so that your AI model can focus on their interpretation.

A first pass extracts the checksum and a compact folder-to-IDs index:
```sh
sherpa | jq '{checksum} + ([.data[] | {id: .ids[], folder: .folders[]}] | group_by(.folder)
 | map({(.[0].folder): [.[].id] | unique}) | add)'
```

```json
{
  "checksum": "a09a…",
  "Friends": ["5678"],
  "Travel": ["6012", "6013"],
  "Work": ["9051"]
}
```
Instruct OpenClaw to compare the checksum against its last known state. If the checksum has changed, let it run a full `sherpa` to fetch the complete thread data for new messages only.

# Configuration
Sherpa can read its configuration from `~/.sherpa/sherpa.conf` ([documentation](sherpa.example.conf)).
