---
name: setup-sound-notifications
description: Set up sound notifications for Claude Code using SND01 sine sound kit from snd.dev. Plays a progress loop while waiting for user confirmation, and a celebration sound when Claude finishes responding.
disable-model-invocation: true
allowed-tools: Bash, Read, Edit, Write, WebFetch, Glob
---

# Setup Sound Notifications for Claude Code

Set up a sound notification system that provides audio feedback during Claude Code sessions.

## Sound Effects

This skill uses the **SND01 "sine"** sound kit from [snd.dev](https://snd.dev), designed by Yasuhiro Tsuchiya. All sounds are based on sine waves — minimal, clean, and compatible with most speakers.

| Scenario | Sound | Description |
|----------|-------|-------------|
| Waiting for user confirmation | `progress_loop.wav` (looping) | Alerts the user that Claude needs input |
| Task complete (Claude stops) | `celebration.wav` | Celebrates task completion |

## Setup Steps

### Step 1: Download and extract SND01 sounds

Download the SND01 sound kit from `https://snd.dev/assets/sounds/SND01_sine.zip`. Use `WebFetch` to download, then extract with `unzip`. Copy these files to `~/.claude/sounds/`:

- `progress_loop.wav`
- `celebration.wav`

```bash
mkdir -p ~/.claude/sounds
# After downloading and extracting:
cp SND01_sine/progress_loop.wav ~/.claude/sounds/
cp SND01_sine/celebration.wav ~/.claude/sounds/
```

### Step 2: Configure hooks in `~/.claude/settings.json`

Read the existing `~/.claude/settings.json` first, then **merge** these hooks with any existing hooks (do NOT replace existing hooks):

**PermissionRequest** — Start looping `progress_loop.wav` when Claude needs user confirmation:

```json
{
  "matcher": "",
  "hooks": [
    {
      "type": "command",
      "command": "pkill -9 -f 'progress_loop' 2>/dev/null; nohup bash -c 'while true; do afplay ~/.claude/sounds/progress_loop.wav; done' >/dev/null 2>&1 & disown"
    }
  ]
}
```

**PostToolUse** — Silently stop `progress_loop` after the user confirms and the tool executes:

```json
{
  "matcher": "",
  "hooks": [
    {
      "type": "command",
      "command": "pkill -9 -f 'progress_loop' 2>/dev/null; true"
    }
  ]
}
```

**UserPromptSubmit** — Silently stop `progress_loop` when the user submits a message:

```json
{
  "matcher": "",
  "hooks": [
    {
      "type": "command",
      "command": "pkill -9 -f 'progress_loop' 2>/dev/null; true"
    }
  ]
}
```

**Stop** — Stop any `progress_loop` and play `celebration.wav` when Claude finishes responding:

```json
{
  "matcher": "",
  "hooks": [
    {
      "type": "command",
      "command": "pkill -9 -f 'progress_loop' 2>/dev/null; afplay ~/.claude/sounds/celebration.wav",
      "timeout": 10
    }
  ]
}
```

### Step 3: Verify

1. Validate JSON syntax with `jq -e '.' ~/.claude/settings.json`
2. Tell the user to run `/hooks` or restart Claude Code to activate the hooks

## Important Notes

- This skill is for **macOS only** (uses `afplay` for audio playback)
- Uses `pkill -9` (SIGKILL) for immediate process termination
- The `progress_loop` detection via `pgrep`/`pkill` uses the filename pattern, which is specific enough to avoid false positives
- Always **merge** with existing hooks — never overwrite the user's existing hook configurations
- If the user already has hooks on the same events, append the sound hook commands to the existing hooks array
