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

## Behavior:

### Default mode:

- Analyze staged changes with `git diff --cached`
- Generate a single descriptive commit message
- Execute the commit

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

EXAMPLES:
  /gitco                                           # Auto-generate commit message
  /gitco "Fix authentication bug"                  # Custom commit message
  /gitco --split --interactive                     # Split into multiple commits with review
  /gitco --type feat "Add user dashboard"          # Conventional commit format
  /gitco --all --type fix "Handle null values"     # Stage all files + conventional commit
  /gitco --amend "Updated commit message"          # Amend last commit

FEATURES:
  • Smart commit message generation based on staged changes
  • Intelligent commit splitting by logical groupings (related files, functionality, tests)
  • Interactive review mode for proposed commits
  • Conventional commit format support
  • Automatic staging option
  • Last commit amendment

CONVENTIONAL COMMIT TYPES:
  feat      New features
  fix       Bug fixes  
  docs      Documentation changes
  refactor  Code restructuring
  test      Adding/updating tests
  style     Formatting, missing semicolons
  chore     Maintenance tasks

For more details, see the full command documentation.
```
