# takopi

**Repository:** https://github.com/banteg/takopi
**Stars:** 223
**Language:** Python
**Topics:** bridge, codex, telegram

## Description

🐙 *he just wants to help-pi*

Telegram bridge for codex, claude code, opencode, pi, and other agents. Manage multiple projects and worktrees, stream progress, and resume sessions anywhere.

## Features

- **Projects and worktrees:** Register repos with `takopi init`, target them via `/project`, route to branches with `@branch`
- **Stateless resume:** Continue a thread in the chat or pick up in the terminal
- **Progress updates:** Stream progress while agent runs (commands, tools, notes, file changes, elapsed time)
- **Robust markdown rendering:** Quality-of-life tweaks for better output formatting
- **Parallel runs:** Across threads with per-thread queue support
- **Cancel support:** Use `/cancel` to stop a running task
- **Voice note transcription:** Optional voice note transcription for Telegram (routes transcript like typed text)
- **Per-project chat routing:** Assign different Telegram chats to different projects

## Requirements

- `uv` for installation (`curl -LsSf https://astral.sh/uv/install.sh | sh`)
- Python 3.14+ (`uv python install 3.14`)
- At least one engine on PATH:
  - `codex` (`npm install -g @openai/codex` or `brew install codex`)
  - `claude` (`npm install -g @anthropic-ai/claude-code`)
  - `opencode` (`npm install -g opencode-ai@latest`)
  - `pi` (`npm install -g @mariozechner/pi-coding-agent`)

## Installation

```bash
uv tool install -U takopi
```

## Setup

Run `takopi` and follow the interactive prompts:
1. Create a bot token via [@BotFather](https://t.me/BotFather)
2. Capture your `chat_id` from the most recent message you send to the bot
3. Set a default engine

To re-run onboarding (and overwrite config), use `takopi --onboard`.

Run your agent CLI once interactively in the repo to trust the directory.

## Configuration

Global config at `~/.takopi/takopi.toml`

```toml
default_engine = "codex"
# optional: reload config changes without restarting
watch_config = true

# optional, defaults to "telegram"
transport = "telegram"

[transports.telegram]
bot_token = "123456789:ABCdefGHIjklMNOpqrsTUVwxyz"
chat_id = 123456789
voice_transcription = true

[codex]
# optional: profile from ~/.codex/config.toml
profile = "takopi"
# optional: extra codex CLI args (exec flags are managed by Takopi)
# extra_args = ["-c", "notify=[]"]

[claude]
model = "sonnet"
# optional: defaults to ["Bash", "Read", "Edit", "Write"]
allowed_tools = ["Bash", "Read", "Edit", "Write", "WebSearch"]
dangerously_skip_permissions = false
# uses subscription by default, override to use api billing
use_api_billing = false

[opencode]
model = "claude-sonnet-4-20250514"

[pi]
model = "gpt-4.1"
provider = "openai"
# optional: additional CLI arguments
extra_args = ["--no-color"]
```

## Usage

Start takopi in the repo you want to work on:

```bash
cd ~/dev/your-repo
takopi
# or override the default engine for new threads:
takopi claude
takopi opencode
takopi pi
```

List available plugins (engines/transports/commands):

```bash
takopi plugins
takopi --transport telegram
```

Send a message to the bot. Start a new thread with a specific engine by prefixing your message with `/codex`, `/claude`, `/opencode`, or `/pi`.

To continue a thread, reply to a bot message containing a resume line.

To stop a run, reply to the progress message with `/cancel`.

## Plugins

Takopi supports entrypoint-based plugins for engines, transports, and command backends.

See `docs/plugins.md` and `docs/public-api.md`.

## Development

See `docs/specification.md` and `docs/developing.md`.
