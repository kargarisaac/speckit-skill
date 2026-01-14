# Installation Guide

## Option 1: Install from GitHub (Recommended)

### Step 1: Publish to Your Private GitHub

```bash
cd /tmp/speckit-plugin

# Initialize git
git init
git add .
git commit -m "Initial commit: Speckit v2.0.0"

# Create repo on GitHub (private)
# Go to https://github.com/new
# Repository name: speckit-plugin
# Visibility: Private
# Don't initialize with README (we already have one)

# Add remote and push
git remote add origin https://github.com/YOUR_USERNAME/speckit-plugin.git
git branch -M main
git push -u origin main
```

### Step 2: Install in Claude Code

```bash
# Start Claude Code in any project
claude

# Install the plugin
/install https://github.com/YOUR_USERNAME/speckit-plugin
```

### Step 3: Verify Installation

```bash
# Check installed plugins
/help

# You should see:
# - /sdd-flow
# - /speckit.specify
# - /speckit.plan
# - etc.
```

## Option 2: Install Locally (Development)

For testing before publishing:

```bash
# Copy to plugins directory
cp -r /tmp/speckit-plugin ~/.claude/plugins/repos/speckit-plugin

# Or use Claude's local install
cd /tmp/speckit-plugin
# Then in Claude Code:
/install-local /tmp/speckit-plugin
```

## Post-Installation Setup

### For Each Project Using Speckit

1. **Create constitution file**:
```bash
mkdir -p .specify/memory
```

2. **Edit `.specify/memory/constitution.md`** with your project's rules:
```markdown
# Project Constitution

## Core Principles
[Your rules here]

## Tech Stack
[Your stack here]
```

3. **Create `CLAUDE.md`** with code locations:
```markdown
# CLAUDE.md

## Code Locations
| What | Where |
|------|-------|
| Source | src/ |
| Tests | tests/ |
```

4. **Start building**:
```bash
/sdd-flow "Your first feature description"
```

## Updating the Plugin

### If Installed from GitHub

```bash
# In Claude Code
/update speckit
```

### If Installed Locally

```bash
# Pull latest
cd /path/to/speckit-plugin
git pull

# Reinstall
/install-local /path/to/speckit-plugin
```

## Troubleshooting

### Commands not showing

```bash
# Restart Claude Code
exit
claude

# Or reinstall
/uninstall speckit
/install https://github.com/YOUR_USERNAME/speckit-plugin
```

### Permission errors on scripts

```bash
# Make scripts executable
chmod +x ~/.claude/plugins/cache/*/speckit/*/scripts/*.sh
```

### Template not found

Check that `.specify/memory/constitution.md` exists in your project.

## Sharing with Team

### Private GitHub (Recommended)

1. Add team members as collaborators to your repo
2. They install with: `/install https://github.com/YOUR_USERNAME/speckit-plugin`

### Self-Hosted

1. Clone to shared location (network drive, internal GitLab, etc.)
2. Each person installs locally: `/install-local /path/to/speckit-plugin`

## Support

For issues or questions:
- GitHub Issues: https://github.com/YOUR_USERNAME/speckit-plugin/issues
- Email: isaac@nazmito.com
