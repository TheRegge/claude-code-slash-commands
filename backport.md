# /backport

Compare a clone application against its base project and backport improvements. This command helps maintain a "base" Next.js (or any) application by identifying improvements made in clone projects that should flow back upstream.

**IMPORTANT**: If the user provides `--help` or `-h` as an argument, display the complete usage information below instead of executing any operations.

## Arguments:

**Commands:**
- `apply`: Apply accepted backports to the base project
- `--init`: First-time setup (configure base project path, create state files)
- `--status`: Show dashboard of pending, applied, and deferred decisions
- `--deferred`: Re-review previously deferred items
- `--reset`: Clear all decision history, learned patterns, and start fresh

**Options:**
- `--dry-run` or `-d`: Preview what would be applied (use with `apply`)
- `--ignore <pattern>`: Add a glob pattern to the ignore list
- `--path <glob>`: Focus review on a specific directory or file pattern
- `-h`, `--help`: Show detailed usage information

## Behavior:

### First Run / Init
On first run (or with `--init`), the command will:
1. Ask the user for the base project path
2. Validate the path exists and is a valid project directory
3. Create `.backport.json` in the clone project root
4. Add `.backport.json` to `.gitignore`
5. Scan both projects and report file counts

### Review Mode (default)
Running `/backport` without arguments starts a review session:
1. Load `.backport.json` configuration
2. Use git to find files changed since last review session
3. For each changed file, compare against the base project version
4. Present each difference with an AI assessment:
   - **Generic improvement** - likely backportable
   - **App-specific customization** - suggest skip
   - **Mixed** - contains both generic and app-specific changes
   - **Ambiguous** - needs user judgment
5. Collect user decisions: Backport, Skip, or Defer
6. Update `.backport.json` with session results and learned patterns

The AI assessment uses file path analysis, content analysis, import dependency checks, and learned patterns from past decisions - but **never auto-skips**. Every change is always presented for the user to decide.

### Apply Mode
Running `/backport apply` pushes accepted changes to the base:
1. Validate base project has a clean git working tree
2. Apply each accepted change file by file
3. Handle merge conflicts with AI-assisted resolution
4. Update `.backport-registry.json` in the base project
5. Mark decisions as applied in clone's `.backport.json`
6. Suggest next steps (review, test, commit in base)

### Status Mode
Running `/backport --status` shows a dashboard:
- Pending backports (accepted but not yet applied)
- Recently applied backports
- Deferred items awaiting re-review
- Learned patterns summary

### Decision Learning
The command learns from your decisions over time:
- Tracks directories/files you consistently skip or backport
- After 2+ consistent decisions, strengthens future AI suggestions
- Shows notes like: *"You've skipped files in src/app/dashboard/ 4 times before"*
- Patterns never cause auto-skipping - only influence recommendations
- Use `--reset` to clear learned patterns

### State Files

**`.backport.json`** (clone project root):
- Stores base project path
- Tracks review sessions and decisions
- Contains learned patterns
- Contains custom ignore patterns
- **Should be gitignored** (local decisions)

**`.backport-registry.json`** (base project root):
- Tracks all backported changes and their source projects
- Records which clone each change came from
- **Should be committed to git** (provenance tracking)

## Examples:

```bash
# First-time setup
/backport --init

# Review changes since last session
/backport

# Review only changes in src/lib/
/backport --path "src/lib/**"

# Show status dashboard
/backport --status

# Re-review deferred items
/backport --deferred

# Apply accepted backports to base
/backport apply

# Preview what would be applied
/backport apply --dry-run

# Add an ignore pattern
/backport --ignore "src/app/custom-page/**"

# Clear history and start fresh
/backport --reset
```

## Workflow Examples:

### Initial Setup
```bash
# In your clone project
cd ~/projects/my-client-app

# Configure the base project
/backport --init
# → Enter base path: ~/projects/nextjs-base
# → Scanning both projects...
# → Configuration saved. Run /backport to start reviewing.
```

### Regular Review Cycle
```bash
# After making improvements in your clone
/backport
# → Reviews each changed file with AI assessment
# → You decide: Backport, Skip, or Defer for each

# When ready to push changes to base
/backport apply
# → Applies accepted changes to base project

# Then go test in the base
cd ~/projects/nextjs-base
npm test
git diff
git add . && git commit -m "feat: backport improvements from client-app"
```

### Focused Review
```bash
# Only review changes in shared libraries
/backport --path "src/lib/**"

# Only review deferred items from previous sessions
/backport --deferred
```

## Help Output (for --help/-h):

When user types `/backport --help` or `/backport -h`, display this formatted help:

```
/backport - Cross-project Backport Manager

USAGE:
  /backport [command] [options]

COMMANDS:
  (default)                       Review changes since last session
  apply                           Apply accepted backports to base
  --init                          First-time setup
  --status                        Show status dashboard
  --deferred                      Re-review deferred items
  --reset                         Clear history and start fresh
  -h, --help                      Show this help

OPTIONS:
  -d, --dry-run                   Preview changes only (use with apply)
  --ignore <pattern>              Add ignore pattern
  --path <glob>                   Focus on specific directory/files

STATE FILES:
  .backport.json                  Clone-side config and decisions (gitignored)
  .backport-registry.json         Base-side provenance tracking (committed)

WORKFLOW:
  1. /backport --init              # Configure base project path
  2. /backport                     # Review changes, make decisions
  3. /backport apply               # Push accepted changes to base
  4. cd <base> && npm test         # Test and commit in base

AI ASSESSMENT:
  Each change is presented with an AI recommendation:
  • Generic improvement - likely backportable
  • App-specific customization - suggest skip
  • Mixed - contains both types of changes
  • Ambiguous - needs your judgment

  The AI learns from your decisions over time but never auto-skips.
  You always make the final call on every change.

EXAMPLES:
  /backport --init                       # First-time setup
  /backport                              # Review recent changes
  /backport --path "src/lib/**"          # Review specific directory
  /backport --status                     # Check pending backports
  /backport apply                        # Apply to base project
  /backport apply --dry-run              # Preview what would apply
  /backport --ignore "src/custom/**"     # Add ignore pattern
  /backport --deferred                   # Re-review deferred items
  /backport --reset                      # Start fresh

For more details, see project documentation.
```

## Notes:

- **Safe by default**: Never auto-skips changes, never overwrites uncommitted work in base
- **AI-assisted**: Classifies changes as generic vs app-specific, learns from your decisions
- **Incremental**: Only reviews changes since last session (fast for large projects)
- **Cross-project**: Reads and writes files in both clone and base project directories
- **Provenance tracking**: Base registry records where every backported change came from

---
agent: backport-manager
---

Manage backporting of improvements from a clone project to its base project. Parse the user's arguments and execute the appropriate mode.

## Argument Parsing

Parse the user's input for:

1. **Commands**: `apply`, `--init`, `--status`, `--deferred`, `--reset`, `--help`/`-h`
2. **Options**: `--dry-run`/`-d`, `--ignore <pattern>`, `--path <glob>`
3. **Default**: If no command given, run review mode

## Mode Execution

### INIT MODE (`--init` or no `.backport.json` found)

1. Ask the user for the base project path
2. Validate the path:
   - Must exist as a directory
   - Must contain typical project files (package.json, src/, etc.)
3. Determine the base project name from the directory name or package.json
4. Create `.backport.json` with:
   - `base_project_path`: Absolute path to base
   - `base_project_name`: Project name
   - `initialized_at`: Current ISO timestamp
   - `last_review_commit`: null
   - `last_review_date`: null
   - `ignore_patterns`: Default ignore list
   - `learned_patterns`: Empty `{ "skip_paths": {}, "backport_paths": {} }`
   - `sessions`: Empty array
5. Add `.backport.json` to `.gitignore` if not already present
6. Scan both projects and report file counts (excluding ignored patterns)
7. Output success message with next steps

### REVIEW MODE (default)

1. Load `.backport.json` - if not found, redirect to INIT MODE
2. Validate base project path still exists
3. **Find changed files:**
   - If `last_review_commit` exists: use `git diff --name-status <last_commit>..HEAD` to find changed files in clone
   - If `last_review_commit` is null (first review): compare ALL non-ignored source files between clone and base
   - Filter out files matching `ignore_patterns`
   - If `--path` option given, further filter to matching files only
4. **For each changed file, determine change type:**
   - `modified`: File exists in both clone and base but differs
   - `new`: File exists in clone but not in base
   - `deleted`: File was deleted in clone but exists in base
   - `identical`: File exists in both and is the same (skip silently)
5. **For each non-identical file, perform AI analysis:**
   - Read both versions (clone and base) of the file
   - Identify the specific differences (diff hunks)
   - Classify each hunk using these signals:
     a. **File path**: `src/lib/`, `src/utils/`, `src/middleware/` tend to be generic; `src/app/[route]/`, `src/config/` tend to be app-specific
     b. **Content**: Hardcoded app names, theme colors, environment URLs, branded strings → app-specific
     c. **Imports**: If file imports from clone-specific modules → app-specific
     d. **Learned patterns**: Check `learned_patterns` for path-prefix matches
   - Generate a summary of what changed (1-2 sentences)
   - Assign an assessment label:
     - "Generic improvement - likely backportable"
     - "App-specific customization - suggest skip"
     - "Mixed - contains both generic and app-specific changes"
     - "Ambiguous - needs your judgment"
   - If learned patterns exist for this path, append a note (e.g., "You've backported from src/lib/ 6 times before")
6. **Present changes to user, grouped by assessment:**
   - First: files assessed as "Generic improvement"
   - Then: "Mixed" files
   - Then: "Ambiguous" files
   - Finally: "App-specific" files
   - For each file, show:
     - File path and change type
     - Summary of changes
     - AI assessment with learned pattern note (if any)
     - Ask for decision: Backport / Skip / Defer / View diff
   - If user asks to "View diff", show the actual diff content then re-ask for decision
   - For "Mixed" files, offer "Backport (partial)" option - let user specify which hunks to include
7. **After all decisions collected:**
   - Create a new session entry in `.backport.json`
   - Record current clone HEAD commit as `clone_commit`
   - Record current base HEAD commit as `base_commit` (read from base project)
   - Update `last_review_commit` to current clone HEAD
   - Update `last_review_date` to current timestamp
   - **Update learned patterns:**
     - For each decision, extract the directory prefix of the file path
     - If decision is "skip", increment `skip_paths[prefix].count` and update `last_seen`
     - If decision is "backport", increment `backport_paths[prefix].count` and update `last_seen`
     - Create new entries if path prefix not yet tracked
   - Save updated `.backport.json`
8. **Report summary:**
   - N files marked for backport
   - N files skipped
   - N files deferred
   - Suggest: "Run `/backport apply` to push changes to base"

### APPLY MODE (`apply`)

1. Load `.backport.json`
2. Collect all decisions with `"decision": "backport"` and `"applied": false` across all sessions
3. If no pending backports, report "Nothing to apply" and exit
4. If `--dry-run`, list all files that would be applied and exit
5. **Call the `backport-applier` sub-agent** via the Task tool with:
   - Clone project path (current working directory)
   - Base project path (from config)
   - List of files to apply with their change types
   - The `.backport.json` content for updating applied status
6. After sub-agent completes:
   - Report results (applied, skipped due to conflicts, errors)
   - Display next steps for the user

### STATUS MODE (`--status`)

1. Load `.backport.json`
2. Display formatted dashboard:

```
BACKPORT STATUS for <project-name>
Base: <base-project-path>
Last review: <date> (commit <short-sha>)

PENDING BACKPORTS (not yet applied):
  • src/lib/auth.ts (modified) - "Added refresh token rotation"
  • src/lib/rate-limiter.ts (new) - "New rate limiting middleware"

RECENTLY APPLIED:
  • src/components/ErrorBoundary.tsx (modified) - Applied 2026-02-18

DEFERRED (awaiting re-review):
  • src/utils/logger.ts (modified) - "Enhanced logging format"

LEARNED PATTERNS:
  Tend to backport: src/lib/ (6x), src/middleware/ (2x)
  Tend to skip: src/app/dashboard/ (4x), src/config/branding.ts (3x)
```

### DEFERRED MODE (`--deferred`)

1. Load `.backport.json`
2. Collect all decisions with `"decision": "defer"` across all sessions
3. Present them for re-review using the same flow as REVIEW MODE step 5-7
4. User can change decision to backport, skip, or keep as defer

### RESET MODE (`--reset`)

1. Ask user for confirmation: "This will clear all session history and learned patterns. Continue?"
2. If confirmed:
   - Preserve `base_project_path`, `base_project_name`, `initialized_at`, and `ignore_patterns`
   - Reset `last_review_commit` to null
   - Reset `last_review_date` to null
   - Clear `learned_patterns` to empty
   - Clear `sessions` to empty array
   - Save `.backport.json`
3. Report: "Reset complete. Run `/backport` to start a fresh review."

### IGNORE MODE (`--ignore <pattern>`)

1. Load `.backport.json`
2. Add the pattern to `ignore_patterns` if not already present
3. Save `.backport.json`
4. Report: "Added '<pattern>' to ignore list. N patterns total."

## Important Reminders

**DO:**
- Always present every change to the user - never auto-skip
- Read files from both clone AND base project directories (cross-project operations)
- Validate base project path exists before any operation
- Preserve existing `.backport.json` data when updating
- Use the current git HEAD when recording session commits
- Handle missing files gracefully (file deleted in one project but not the other)
- Group and sort changes by AI assessment for easier review
- Include learned pattern notes when presenting changes

**DON'T:**
- Auto-skip files based on AI assessment or learned patterns
- Modify files in the base project during review mode (only during apply)
- Assume file paths are the same between projects (validate each one)
- Create malformed JSON in state files
- Lose session history when adding new sessions
- Modify the base project's `.backport-registry.json` during review (only during apply)
