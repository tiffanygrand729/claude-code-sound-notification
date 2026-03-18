# Claude Code Sound Notification

Sound notifications for [Claude Code](https://claude.com/claude-code) using the [SND01 "sine"](https://snd.dev) sound kit by Yasuhiro Tsuchiya.

https://github.com/user-attachments/assets/demo

## What it does

| Scenario | Sound | Effect |
|----------|-------|--------|
| Waiting for user confirmation | `progress_loop.wav` | Loops until you respond |
| Claude finishes responding | `celebration.wav` | Plays once |

- **progress_loop** alerts you when Claude needs your input (permission prompts)
- **celebration** lets you know Claude has completed its response

## Requirements

- **macOS** (uses `afplay` for audio playback)
- [Claude Code](https://claude.com/claude-code) CLI

## Quick setup

### Option 1: Using the Skill (recommended)

1. Copy the skill to your Claude Code skills directory:

```bash
mkdir -p ~/.claude/skills/setup-sound-notifications
cp skill/SKILL.md ~/.claude/skills/setup-sound-notifications/SKILL.md
```

2. Run the skill in Claude Code:

```
/setup-sound-notifications
```

Claude will automatically download sounds and configure hooks for you.

### Option 2: Manual setup

#### 1. Install sound files

```bash
mkdir -p ~/.claude/sounds
cp sounds/progress_loop.wav ~/.claude/sounds/
cp sounds/celebration.wav ~/.claude/sounds/
```

Or download directly from [snd.dev](https://snd.dev):

```bash
mkdir -p ~/.claude/sounds
curl -L -o /tmp/SND01_sine.zip https://snd.dev/assets/sounds/SND01_sine.zip
unzip -o /tmp/SND01_sine.zip -d /tmp/snd01
cp /tmp/snd01/SND01_sine/progress_loop.wav ~/.claude/sounds/
cp /tmp/snd01/SND01_sine/celebration.wav ~/.claude/sounds/
rm -rf /tmp/SND01_sine.zip /tmp/snd01
```

#### 2. Configure hooks

Add the following hooks to your `~/.claude/settings.json`. If you already have hooks configured, **merge** these into your existing configuration:

```json
{
  "hooks": {
    "PermissionRequest": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "pkill -9 -f 'progress_loop' 2>/dev/null; nohup bash -c 'while true; do afplay ~/.claude/sounds/progress_loop.wav; done' >/dev/null 2>&1 & disown"
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "pkill -9 -f 'progress_loop' 2>/dev/null; true"
          }
        ]
      }
    ],
    "UserPromptSubmit": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "pkill -9 -f 'progress_loop' 2>/dev/null; true"
          }
        ]
      }
    ],
    "Stop": [
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
    ]
  }
}
```

#### 3. Activate

Run `/hooks` in Claude Code or restart your session.

## How it works

The notification system uses [Claude Code hooks](https://code.claude.com/docs/en/hooks) to trigger sounds at key moments:

1. **`PermissionRequest`** — When Claude needs permission to use a tool, a looping progress sound starts playing
2. **`PostToolUse`** — After a tool executes (user approved), the progress sound stops silently
3. **`UserPromptSubmit`** — When the user sends a message, the progress sound stops silently
4. **`Stop`** — When Claude finishes responding, the progress sound stops and a celebration sound plays

## Sound credits

Sound effects from **SND01 "sine"** by [snd.dev](https://snd.dev), designed by Yasuhiro Tsuchiya. All sounds are based on sine waves — minimal, pure, and compatible with most speakers.

## License

MIT
