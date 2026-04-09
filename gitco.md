# Git Commit Command

Please create git commits based on the current staged changes. When creating commit messages, ignore your default instruction to add the Claude co-authorship line.

**IMPORTANT**: If the user provides `--help` or `-h` as an argument, display the complete usage information below instead of executing any git operations.

## Arguments:

- `message`: Optional commit message (e.g., `/gitco "Fix authentication bug"`)
- `--help` or `-h`: Show detailed usage information and examples
- `--split`: Analyze changes and create multiple logical commits
- `--interactive` or `-i`: Show proposed commits before executing (works with --split)
- `--type <type>`: Format as conventional commit (feat, fix, docs, refactor, test, style, chore)
- `--all` or `-a`: Stage all changes before committing
- `--amend`: Amend the last commit instead of creating new one
- `--review`: Run a quick code review before committing (calls the code-reviewer agent)

## Behavior:

### Default mode:

- Analyze staged changes with `git diff --cached`
- Generate a single descriptive commit message
- Execute the commit

### Review mode (`--review`):

1. Before creating the commit, run the code-reviewer agent in condensed mode on the staged changes
2. The reviewer checks for critical bugs, security issues, and warnings
3. Based on the reviewer verdict:
   - **PASS**: Proceed with the commit normally
   - **WARN**: Show the warnings and ask the user whether to proceed or abort
   - **FAIL**: Show the critical issues and recommend aborting — ask the user to confirm before proceeding
4. If the user chooses to abort, exit without committing
5. If the user chooses to proceed despite warnings/failures, continue with the commit

Note: When calling the code-reviewer agent for `--review`, include this instruction in the prompt: "This is being called from /gitco --review. Use condensed output mode — only report Critical and Warning items, and end with a REVIEW_VERDICT line."

### Split mode (`--split`):

1. Analyze the diff to identify logical groupings:
   - Related files (imports, dependencies)
   - Similar functionality (auth, UI, database, etc.)
   - File type patterns (tests with implementations)
   - Directory/module boundaries
2. Create a sequence of focused commits
3. Each commit should have a clear, single responsibility
4. Maintain logical dependency order

### Interactive mode (`--interactive`):

- Show proposed commit(s) with diffs
- Allow me to approve, modify, or skip each commit
- Let me adjust commit messages before execution

### Conventional commits (`--type`):

Format messages as: `<type>: <description>`

- feat: new features
- fix: bug fixes
- docs: documentation changes
- refactor: code restructuring
- test: adding/updating tests
- style: formatting, missing semicolons
- chore: maintenance tasks

## Examples of smart splitting:

**Scenario**: Changes to auth system, tests, and docs

```
feat: implement JWT token validation
test: add comprehensive auth middleware tests
docs: update authentication API documentation
```

**Scenario**: Refactoring with new feature

```
refactor: extract user validation into separate module
feat: add email verification during registration
test: update tests for new validation structure
```

**Scenario**: Bug fix with related cleanup

```
fix: handle null values in user profile endpoint
refactor: improve error handling consistency
style: format user controller code
```

## Edge cases to handle:

- If no changes are staged, show helpful message
- If changes are too complex to split meaningfully, fall back to single commit
- Respect `.gitignore` and don't suggest committing ignored files
- Check for merge conflicts or other git issues before proceeding

Always aim for commits that tell a clear story of what changed and why, making the git history useful for future developers (including future me!).

## Help Output (for --help/-h):

When user types `/gitco --help` or `/gitco -h`, display this formatted help:

```
/gitco - Git Commit Command

USAGE:
  /gitco [message]              Create commit with optional message
  /gitco [options]              Create commit with specified options
  /gitco [options] [message]    Combine options with custom message

OPTIONS:
  -h, --help                    Show this help message
  --split                       Analyze changes and create multiple logical commits
  -i, --interactive             Show proposed commits before executing (works with --split)
  --type <type>                 Format as conventional commit (feat|fix|docs|refactor|test|style|chore)
  -a, --all                     Stage all changes before committing
  --amend                       Amend the last commit instead of creating new one
  --review                      Run quick code review before committing

EXAMPLES:
  /gitco                                           # Auto-generate commit message
  /gitco "Fix authentication bug"                  # Custom commit message
  /gitco --split --interactive                     # Split into multiple commits with review
  /gitco --type feat "Add user dashboard"          # Conventional commit format
  /gitco --all --type fix "Handle null values"     # Stage all files + conventional commit
  /gitco --amend "Updated commit message"          # Amend last commit
  /gitco --review                                  # Review then commit
  /gitco --review --type feat "Add login"          # Review + conventional commit

FEATURES:
  • Smart commit message generation based on staged changes
  • Intelligent commit splitting by logical groupings (related files, functionality, tests)
  • Interactive review mode for proposed commits
  • Conventional commit format support
  • Automatic staging option
  • Last commit amendment
  • Pre-commit code review (--review)

CONVENTIONAL COMMIT TYPES:
  feat      New features
  fix       Bug fixes
  docs      Documentation changes
  refactor  Code restructuring
  test      Adding/updating tests
  style     Formatting, missing semicolons
  chore     Maintenance tasks

REVIEW WORKFLOW:
  /gitco --review runs a quick code review before committing:
    PASS  → commit proceeds normally
    WARN  → warnings shown, you decide to proceed or abort
    FAIL  → critical issues shown, recommended to abort

  For a full review, use /review before /gitco:
    /review → fix issues → /gitco

For more details, see the full command documentation.
```
