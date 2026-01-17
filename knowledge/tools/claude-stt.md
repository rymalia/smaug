---
title: Claude STT
author: jarrodwatts
url: https://github.com/jarrodwatts/claude-stt
stars: 165
language: Python
tags: [accessibility, claude-code, claude-code-plugin, dictation, speech-to-text, stt, voice]
bookmarked: Thursday, January 15, 2026
---

# Claude STT

Speech-to-text input for Claude Code with live streaming dictation.

## Overview

Claude STT gives you voice input directly into Claude Code. Hold a hotkey, speak, and your words appear in the input field — all processed locally without any external services.

## Key Features

| Feature | Details |
|---------|---------|
| **Local Processing** | All audio processed on-device using Moonshine STT |
| **Low Latency** | ~400ms transcription time |
| **Push-to-Talk** | Hold hotkey to record, release to transcribe |
| **Cross-Platform** | macOS, Linux, Windows |
| **Privacy-First** | No audio or text sent to external services |

## Installation

Inside a Claude Code instance, run:

```bash
# Step 1: Add the marketplace
/plugin marketplace add jarrodwatts/claude-stt

# Step 2: Install the plugin
/plugin install claude-stt

# Step 3: Run setup
/claude-stt:setup
```

Setup installs dependencies (uv if available, otherwise a local `.venv`), downloads the Moonshine model (~200MB), and checks microphone permissions.

Default hotkey: **Ctrl+Shift+Space** to start recording, press again to stop and transcribe.

## How It Works

```
Press Ctrl+Shift+Space → start recording
        ↓
Audio captured from microphone
        ↓
Press Ctrl+Shift+Space → stop recording
        ↓
Moonshine STT processes locally (~400ms)
        ↓
Text inserted into Claude Code input
```

## Configuration

Customize settings anytime with:

```bash
/claude-stt:config
```

| Option | Values | Default | Description |
|--------|--------|---------|-------------|
| `hotkey` | Key combo | `ctrl+shift+space` | Trigger recording |
| `mode` | `toggle`, `push-to-talk` | `toggle` | Press to toggle vs hold to record |
| `engine` | `moonshine`, `whisper` | `moonshine` | STT engine |
| `moonshine_model` | `moonshine/tiny`, `moonshine/base` | `moonshine/base` | Model size |
| `output_mode` | `auto`, `injection`, `clipboard` | `auto` | How text is inserted |
| `sound_effects` | `true`, `false` | `true` | Play audio feedback |
| `max_recording_seconds` | 1-600 | 300 | Maximum recording duration |

Settings stored in `~/.claude/plugins/claude-stt/config.toml`.

## Requirements

- **Python 3.10-3.13**
- **~200MB disk space** for STT model
- **Microphone access**

### Platform-Specific Requirements

| Platform | Requirements |
|----------|-------------|
| **macOS** | Accessibility permissions (System Settings > Privacy & Security) |
| **Linux** | xdotool for window management; X11 recommended (Wayland has limitations) |
| **Windows** | pywin32 for window tracking |

## Commands

| Command | Description |
|---------|-------------|
| `/claude-stt:setup` | First-time setup: check environment, install deps, download model |
| `/claude-stt:start` | Start the STT daemon |
| `/claude-stt:stop` | Stop the STT daemon |
| `/claude-stt:status` | Show daemon status and readiness checks |
| `/claude-stt:config` | Change settings |

## Privacy

All processing is local:
- Audio captured from microphone is processed entirely on-device
- Uses Moonshine ONNX for fast local inference
- Audio is never sent anywhere, never stored (processed in memory, discarded)
- Transcribed text only goes to Claude Code input or clipboard
- No telemetry or analytics

## Troubleshooting

| Issue | Solution |
|-------|----------|
| No audio input | Check microphone permissions in system settings |
| Keyboard injection not working | **macOS**: Grant Accessibility permissions. **Linux**: Ensure xdotool installed |
| Model not loading | Run `/claude-stt:setup` to download. Check disk space (~200MB) |
| Hotkey test fails during setup | Fix permissions or rerun `/claude-stt:setup --skip-hotkey-test` |
| Whisper dependencies missing | Run `/claude-stt:setup --with-whisper` |
| Hotkey not triggering | Check for conflicts with other apps. Try `/claude-stt:config` to change hotkey |
| Text going to wrong window | Plugin tracks original window — ensure Claude Code was focused when recording started |

Set `CLAUDE_STT_LOG_LEVEL=DEBUG` for verbose logs when starting daemon.

## Related

- **Bird CLI** - Twitter bookmark fetching tool (similar local-first approach)
- **Claude Code** - The IDE that runs this plugin

