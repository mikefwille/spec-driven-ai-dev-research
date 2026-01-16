# Claude Code Status Line Setup for Team

This guide shows how to install the standardized status line configuration globally on your machine.

## Quick Setup

### Step 1: Copy the Status Line Script

Copy the `statusline.sh` file from this project to your global Claude Code configuration:

```bash
# Create the .claude directory if it doesn't exist
mkdir -p ~/.claude

# Copy the statusline script
cp .claude/statusline.sh ~/.claude/statusline.sh

# Make it executable (if needed)
chmod +x ~/.claude/statusline.sh
```

### Step 2: Update Your Global Settings

Add the following to your global `~/.claude/settings.json` file:

```json
{
  "statusLine": {
    "type": "command",
    "command": "~/.claude/statusline.sh",
    "padding": 0
  }
}
```

**Note**: If your `~/.claude/settings.json` already has other settings, just add the `statusLine` section to the existing JSON object.

### Step 3: Restart Claude Code

The status line should now appear in all your Claude Code sessions across all projects.

## What This Status Line Shows

- **Line 1**: Directory, Git branch, Model name, Version info
- **Line 2**: Context remaining with progress bar
- **Line 3**: Usage costs, token statistics, burn rates

## Features Included

- =Á Working Directory
- <? Git Branch
- > Model Name & Version
- >à Context Remaining (with progress bar)
- =µ Usage & Cost
-  Session Time Remaining
- =Ê Token Statistics
- ¡ Burn Rate (tokens/min)

## Requirements

- **Node.js 16+** (already required for Claude Code)
- **jq** (recommended for full functionality) - Install with:
  ```bash
  # macOS
  brew install jq

  # Linux
  apt-get install jq  # or yum install jq
  ```

## Troubleshooting

If the status line doesn't appear:

1. Check that the script exists and is executable:
   ```bash
   ls -la ~/.claude/statusline.sh
   ```

2. Verify your settings.json syntax:
   ```bash
   cat ~/.claude/settings.json
   ```

3. Check the log file for errors:
   ```bash
   cat ~/.claude/statusline.log
   ```

## References

- **Source**: https://github.com/chongdashu/cc-statusline
- **Tutorial Video**: https://youtube.com/watch?v=oWsjmNSxoLQ
