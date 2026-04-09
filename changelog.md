# Changelog Command

Generate or update a changelog from git history. This command analyzes commits between versions (or since the last changelog entry) and produces a structured, human-readable changelog following the Keep a Changelog format.

**IMPORTANT**: If the user provides `--help` or `-h` as an argument, display the complete usage information below instead of executing any operations.

## Arguments:

**Commands:**
- `(default)`: Generate changelog entries since last tagged version or last changelog entry
- `--init`: Create a new CHANGELOG.md file with proper structure
- `--full`: Regenerate the entire changelog from all tags/history
- `--preview` or `-p`: Preview what would be added without writing to file

**Options:**
- `--from <tag|sha>`: Start from a specific tag or commit
- `--to <tag|sha>`: End at a specific tag or commit (default: HEAD)
- `--version <version>`: Label the new section with this version (default: "Unreleased")
- `--group`: Group entries by conventional commit type (default behavior)
- `--flat`: List entries chronologically without grouping
- `--help` or `-h`: Show detailed usage information

## Behavior:

### Init mode (`--init`):

Create a new `CHANGELOG.md` at the project root:

```markdown
# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]
```

If `CHANGELOG.md` already exists, report it and do nothing (unless `--full` is also passed).

### Default mode:

1. **Find the starting point:**
   - If `--from` specified, use that
   - If CHANGELOG.md exists, find the most recent version heading and locate its corresponding git tag
   - If no changelog exists, find the most recent git tag
   - If no tags exist, use the first commit

2. **Collect commits** from starting point to `--to` (or HEAD):
   ```bash
   git log <from>..<to> --pretty=format:"%H|%s|%b|%an|%aI" --no-merges
   ```

3. **Parse each commit message:**
   - If conventional commit format (`type: description` or `type(scope): description`):
     - Extract type, scope, and description
     - Map type to changelog category:
       - `feat` → **Added**
       - `fix` → **Fixed**
       - `docs` → **Documentation**
       - `refactor` → **Changed**
       - `perf` → **Changed** (with performance note)
       - `test` → (skip unless `--include-tests`)
       - `style` → (skip unless `--include-style`)
       - `chore` → (skip unless `--include-chore`)
       - `breaking` or `!` suffix → **Breaking Changes** (always shown first)
   - If not conventional commit format:
     - Analyze the commit message and diff to classify
     - Use best judgment to categorize as Added/Fixed/Changed/Removed

4. **Group entries** by category (unless `--flat`):
   - **Breaking Changes** (if any — always first)
   - **Added** (new features)
   - **Fixed** (bug fixes)
   - **Changed** (refactors, improvements, performance)
   - **Removed** (removed features)
   - **Security** (security-related changes)
   - **Documentation** (doc changes — only if substantive)

5. **Format the section:**

   ```markdown
   ## [2.1.0] - 2026-04-09

   ### Breaking Changes
   - **auth**: Token format changed from JWT to Paseto — update all clients

   ### Added
   - **auth**: User session refresh with sliding window expiry
   - **dashboard**: Export button for analytics data (CSV and PDF)

   ### Fixed
   - **api**: Race condition in concurrent checkout requests
   - **ui**: Profile image upload failing on Safari

   ### Changed
   - **core**: Refactored database connection pooling for better performance
   ```

6. **Write to CHANGELOG.md:**
   - If file exists: insert the new section after the header and before the previous version
   - If `[Unreleased]` section exists with content: replace it with the versioned section
   - Preserve all existing content below
   - If `--preview`: display the section but don't write

### Full mode (`--full`):

1. Get all tags sorted by version: `git tag --sort=-version:refname`
2. For each pair of consecutive tags, generate a changelog section
3. Include an `[Unreleased]` section for commits after the latest tag
4. Write the complete CHANGELOG.md (backs up existing file first)

## Integration with /semver:

When `/semver` bumps a version, suggest running:
```
/changelog --version X.Y.Z
```

This pairs naturally: `/semver minor --git-tag` creates the version, `/changelog --version 1.3.0` documents what's in it.

## Examples:

```bash
# Generate changelog entries since last version
/changelog

# Preview without writing
/changelog --preview

# Create initial changelog file
/changelog --init

# Generate for a specific version
/changelog --version 2.1.0

# Generate from a specific tag to HEAD
/changelog --from v2.0.0

# Generate between two specific points
/changelog --from v1.0.0 --to v2.0.0 --version 2.0.0

# Regenerate entire changelog from all tags
/changelog --full

# Flat list without grouping
/changelog --flat
```

## Help Output (for --help/-h):

When user types `/changelog --help` or `/changelog -h`, display this formatted help:

```
/changelog - Changelog Generator

USAGE:
  /changelog [options]

COMMANDS:
  (default)                       Generate entries since last version
  --init                          Create new CHANGELOG.md
  --full                          Regenerate entire changelog from history
  -p, --preview                   Preview without writing to file
  -h, --help                      Show this help

OPTIONS:
  --from <tag|sha>                Start from specific tag or commit
  --to <tag|sha>                  End at specific point (default: HEAD)
  --version <version>             Label section with version number
  --group                         Group by commit type (default)
  --flat                          Chronological list, no grouping

COMMIT TYPE MAPPING:
  feat     → Added               fix      → Fixed
  refactor → Changed             perf     → Changed
  docs     → Documentation       chore    → (skipped)
  test     → (skipped)           style    → (skipped)
  BREAKING → Breaking Changes (always first)

FORMAT:
  Follows Keep a Changelog (https://keepachangelog.com)
  Entries are grouped by category and tagged with scope

EXAMPLES:
  /changelog                             # Since last version
  /changelog --preview                   # Preview changes
  /changelog --init                      # Create CHANGELOG.md
  /changelog --version 2.1.0            # Label as version 2.1.0
  /changelog --from v1.0.0              # From specific tag
  /changelog --full                      # Regenerate everything

WORKFLOW:
  Pairs with /semver:
    /semver minor --git-tag → /changelog --version 1.3.0

For more details, see the full command documentation.
```

## Notes:

- **Preserves existing content**: Never overwrites previous changelog entries (except with `--full`, which backs up first)
- **Conventional commit aware**: Best results with conventional commit messages, but works with any commit style
- **Scope detection**: If commits include scopes, they're shown as bold prefixes in entries
- **Deduplication**: Won't add entries that already exist in the changelog
- **Keep a Changelog format**: Industry-standard format, human-readable and parseable
