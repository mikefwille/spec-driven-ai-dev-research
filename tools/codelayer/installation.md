# CodeLayer Installation Guide

CodeLayer is a desktop application for human-in-the-loop AI development, distributed as part of the HumanLayer project.

## Installation on macOS (Homebrew)

### 1. Add the HumanLayer tap

```bash
brew tap humanlayer/humanlayer
```

This adds the HumanLayer repository to Homebrew, making CodeLayer available for installation.

### 2. Install CodeLayer

```bash
brew install --cask codelayer
```

This will:
- Download the CodeLayer app for your architecture (e.g., `CodeLayer-darwin-arm64.dmg`)
- Install `CodeLayer.app` to `/Applications/CodeLayer.app`
- Create symlinks for CLI commands:
	- `humanlayer` → `/opt/homebrew/bin/humanlayer`
	- `hld` → `/opt/homebrew/bin/hld`

### 3. Launch the application

**Important:** The command-line tools (`humanlayer`, `hld`) require the CodeLayer desktop app to be running.

Launch CodeLayer from:
- **Spotlight:** Press `Cmd+Space` and type "CodeLayer"
- **Applications folder:** Open Finder → Applications → CodeLayer.app
- **Launchpad:** Find CodeLayer in your apps

## Troubleshooting

### "command not found: codelayer"

The cask installs as `humanlayer` and `hld`, not `codelayer`. Use:

```bash
humanlayer
# or
hld
```

### "zsh: killed humanlayer"

This indicates the CLI tool tried to run but the desktop app isn't running. The CLI commands are just interfaces to the desktop application.

**Solution:** Launch the CodeLayer desktop app first, then use the CLI commands.

### First-time macOS security prompt

If macOS blocks the app from opening, go to:
1. **System Settings** → **Privacy & Security**
2. Scroll down to find the blocked app message
3. Click **Open Anyway**

## Available Commands

Once CodeLayer.app is running:

- **`humanlayer`** - Main CLI interface for HumanLayer/CodeLayer
- **`hld`** - Shortened alias for humanlayer

## Links

- GitHub: https://github.com/humanlayer/humanlayer
- Documentation: https://docs.humanlayer.dev
