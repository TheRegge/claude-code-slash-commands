# Custom Claude Code Slash Commands Repository

This repository contains custom slash commands for Claude Code, stored in ~/.claude/commands.

## ⚠️ USE AT YOUR OWN RISK ⚠️

These are custom commands that extend Claude Code functionality. While designed to be helpful, they may have unintended side effects or interact unexpectedly with your system. Review the code before use and test in non-critical environments first.

## Current Commands

### /gitco - Git Commit Command

A comprehensive git commit helper that analyzes staged changes and creates intelligent commit messages.

**Key Features:**
- Automatic commit message generation based on staged changes
- Smart commit splitting (--split) to create multiple logical commits
- Interactive mode (--interactive/-i) to review commits before execution
- Conventional commit format support (--type)
- Stage all changes option (--all/-a)
- Amend last commit functionality (--amend)

**Usage Examples:**
- `/gitco "Fix authentication bug"` - Simple commit with custom message
- `/gitco --split --interactive` - Analyze changes and create multiple commits with review
- `/gitco --type feat --all` - Stage all changes and create conventional commit

**Smart Splitting:**
The command can intelligently group related changes into separate commits, such as:
- Separating feature implementation from tests and documentation
- Grouping related files and dependencies
- Maintaining logical commit order

## Installation

These commands are automatically loaded by Claude Code when placed in ~/.claude/commands directory.

## Contributing

Feel free to add new commands or improve existing ones, but remember to test thoroughly before committing changes.