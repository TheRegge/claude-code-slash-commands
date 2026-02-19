# Custom Claude Code Slash Commands Repository
# Version: 1.0.0

This repository contains custom slash commands for Claude Code, stored in ~/.claude/commands.

## USE AT YOUR OWN RISK

These are custom commands that extend Claude Code functionality. While designed to be helpful,
they may have unintended side effects or interact unexpectedly with your system. Review the code
before use and test in non-critical environments first.


## Commands

### /gitco - Git Commit Command

A comprehensive git commit helper that analyzes staged changes and creates intelligent commit
messages.

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
- `/gitco --amend "Updated message"` - Amend the last commit

**Smart Splitting:**
The command can intelligently group related changes into separate commits, such as:
- Separating feature implementation from tests and documentation
- Grouping related files and dependencies
- Maintaining logical commit order

---

### /semver - Semantic Version Management

Manages semantic versioning across multiple project types with auto-detection, consistency
checking, and git integration.

**Supported Project Types:**
- React / Next.js (package.json)
- WordPress Plugin (main PHP file + readme.txt)
- WordPress Theme (style.css)
- Swift iOS/macOS App (project.pbxproj)
- Swift Package (git tags only)
- Custom projects via .semverrc configuration

**Key Features:**
- Auto-detection of project type on first run
- Version consistency checking across all version files
- Prerelease support (alpha, beta, rc)
- Dry-run mode to preview changes
- Git commit and tag creation
- iOS build number management (--build-number, --increment-build, --reset-build)
- Custom configuration via .semverrc
- Generated file support (skip reading, still update on write)

**Usage Examples:**
- `/semver --current` - Display current version and locations
- `/semver patch` - Bug fix bump: 1.2.3 -> 1.2.4
- `/semver minor --git-tag` - Feature bump with git tag: 1.2.4 -> 1.3.0
- `/semver major` - Breaking change bump: 1.3.0 -> 2.0.0
- `/semver prerelease alpha` - Prerelease: 1.2.3 -> 1.2.4-alpha.1
- `/semver --set 2.1.0` - Set explicit version
- `/semver minor --dry-run` - Preview without modifying files
- `/semver --init` - Initialize .semverrc configuration

**Configuration Files:**
- `.semverrc` - Custom version file locations and patterns (user-created, commit to repo)
- `.semver-cache.json` - Auto-generated detection cache (safe to delete, add to .gitignore)

---

### /document - Intelligent Documentation Generator

Creates and updates documentation using git history to identify what needs documenting.
Produces output optimized for both developers and LLMs.

**Key Features:**
- Git-driven analysis to identify what needs documentation
- Documents current state of code (not historical changes, unless --changes flag)
- Finds and updates existing related documentation
- LLM-optimized output format
- CLAUDE.md integration (updates references if the file exists)
- Tracks documented items via .aidoc file

**Core Modes:**
- `feature [name]` - Document a specific feature
- `fix [issue]` - Document a bug fix or corrected behavior
- `api [endpoint]` - Document API endpoints
- `architecture [component]` - Document architectural decisions
- `update [topic]` - Update existing documentation
- `verify` - Check if documentation is current

**Flags:**
- `--changes` - Include historical context about what changed
- `--full` - Ignore tracking, document entire codebase
- `--reset` - Clear tracking state and start fresh
- `--since [commit]` - Document code modified since a specific commit

**Usage Examples:**
- `/document` - Document current state based on recent changes
- `/document feature "user authentication"` - Document auth system
- `/document api "/api/v2/users"` - Document API endpoints
- `/document --changes` - Include historical context
- `/document update "database schema"` - Update existing docs
- `/document verify` - Check documentation freshness

---

### /timetrack - Human Development Time Estimation

Analyzes projects and creates realistic time tracking estimates based on actual human
development effort, not AI-accelerated time.

**Key Features:**
- Realistic estimates accounting for planning, implementation, testing, debugging
- Factors in learning curves and research time for new technologies
- Includes code review and iteration time
- Categorization by type (Feature, Bug Fix, Refactoring, Testing, Documentation, Setup)
- Proportional scaling to fit budget constraints
- Rounding options for cleaner reports

**Core Modes:**
- `init` - Initialize new time tracking file
- `update` - Add recent changes to existing tracking
- `report` - Generate summary report from existing data
- `scale [hours]` - Scale existing report to fit within hour limit

**Flags:**
- `--since [date]` - Track time since a specific date
- `--category [name]` - Track time for a specific category
- `--human-pace` - Emphasize realistic human development speed
- `--round-full` - Round all hours up to nearest full hour
- `--round-half` - Round all hours up to nearest 0.5 increment

**Usage Examples:**
- `/timetrack` - Analyze entire project
- `/timetrack update` - Add time for recent changes
- `/timetrack report` - Show summary by category
- `/timetrack scale 40` - Rescale report to 40 hours
- `/timetrack scale 40 --round-full` - Rescale and round to full hours
- `/timetrack --since 2024-01-01` - Track since a specific date

**Output:** Creates/updates `.timetrack.json` at project root.

---

### /aidoc - Documentation Tracking File Manager

Manages the `.aidoc` JSON tracking file that records which documentation has been created or
updated and from which git commits.

**Key Features:**
- Records git commit SHA for documentation ranges
- Preserves documentation history
- Enables incremental documentation workflows

**Usage Examples:**
- `/aidoc` - Update tracking file with current commit
- `/aidoc --init` - Create fresh .aidoc file, clearing history
- `/aidoc --docs docs/api.md docs/features.md` - Record specific doc files
- `/aidoc --from-commit a1b2c3d4` - Specify starting commit for range

**Relationship to /document:**
The `/document` command creates the actual documentation and automatically calls `/aidoc`
to track what was documented. Use `/aidoc` directly only when you need manual control over
the tracking file.


## Custom Agents

The slash commands above use specialized subagents to perform their work. These agents are
defined in the Claude Code Task tool configuration and are invoked programmatically by the
commands. They are listed here for reference.

### aidoc-tracker (Internal)

**Used by:** /document, /aidoc
**User-facing:** No - invoked internally by slash commands

Specialized agent for creating and maintaining `.aidoc` tracking files in JSON format. Ensures
consistent, predictable tracking of documentation runs with proper git commit tracking and file
history. Users should interact with this through the `/aidoc` or `/document` commands rather
than invoking the agent directly.

### semver-manager (Internal)

**Used by:** /semver
**User-facing:** No - invoked internally by the /semver command

Specialized agent for managing semantic versioning across different project types. Auto-detects
project type, updates version numbers in all relevant locations, and maintains the project
metadata cache (`.semver-cache.json`). Users should use the `/semver` command instead of
invoking this agent directly.

### time-tracker (Internal)

**Used by:** /timetrack
**User-facing:** No - invoked internally by the /timetrack command

Specialized agent for analyzing projects and creating realistic human-effort time tracking
estimates. Evaluates code complexity, git history, and project scope to estimate senior
developer hours, emphasizing realistic human pace over AI-accelerated development times.
Users should use the `/timetrack` command instead of invoking this agent directly.

### superpowers:code-reviewer (Automatic)

**Used by:** Claude Code (automatically after major steps)
**User-facing:** Indirectly - Claude may invoke this automatically during a session

Performs code review after a major project step has been completed. Validates the implementation
against the original plan and coding standards. This agent is triggered by Claude when it
determines a logical chunk of work is complete, not invoked directly by the user via a command.


## How Commands and Agents Work Together

```
User invokes          Command delegates to        Agent performs work
--------------        ----------------------      -------------------
/document       -->   aidoc-tracker agent    -->   Updates .aidoc file
/aidoc          -->   aidoc-tracker agent    -->   Updates .aidoc file
/semver         -->   semver-manager agent   -->   Updates version files
/timetrack      -->   time-tracker agent     -->   Creates .timetrack.json
/gitco          -->   (no dedicated agent)   -->   Direct git operations
```

The code-reviewer agent operates independently, invoked by Claude Code itself when it
detects that a significant implementation step has been completed.


## Installation

These commands are automatically loaded by Claude Code when placed in the ~/.claude/commands
directory. No additional setup is required.

## Contributing

Feel free to add new commands or improve existing ones, but remember to test thoroughly
before committing changes.