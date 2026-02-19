# Semantic Version Management

Manage semantic versioning across React, Next.js, WordPress Plugin/Theme, Swift iOS/macOS, and Swift Package projects.

**IMPORTANT**: If the user provides `--help` or `-h` as an argument, display the complete usage information below instead of executing any operations.

## Arguments:

**Bump Commands:**
- `major`: Bump major version (X.0.0) - Breaking changes
- `minor`: Bump minor version (x.Y.0) - New features
- `patch`: Bump patch version (x.y.Z) - Bug fixes
- `prerelease [tag]`: Create/bump prerelease version (x.y.z-TAG[.N])
  - Examples: `prerelease alpha`, `prerelease beta`, `prerelease rc`

**Query Commands:**
- `--current` or `-c`: Display current version and locations
- `--help` or `-h`: Show detailed usage information

**Configuration Commands:**
- `--init`: Initialize `.semverrc` configuration file
- `--set X.X.X`: Set explicit version across all files

**Options:**
- `--dry-run` or `-d`: Preview changes without modifying files
- `--git-commit`: Create git commit after version bump
- `--git-tag`: Create git tag after version bump (implies --git-commit)
- `--force`: Skip version consistency checks

**iOS/macOS Specific Options:**
- `--build-number N`: Set explicit build number (e.g., `--build-number 100`)
- `--increment-build`: Auto-increment build number (default for iOS)
- `--reset-build`: Reset build number to 1 on version bump

## Behavior:

### Auto-detection
On first run, the agent will:
1. Detect project type (React/Next.js, WordPress Plugin/Theme, Swift iOS/macOS, Swift Package)
2. Locate standard version files
3. Create `.semver-cache.json` for faster subsequent runs

### Supported Project Types

**React/Next.js:**
- Detects: `package.json` with React/Next.js dependencies
- Updates: `package.json` → `"version"` field

**WordPress Plugin:**
- Detects: PHP file with `Plugin Name:` header
- Updates:
  - Main plugin file → `Version: X.X.X` (header comment)
  - `readme.txt` → `Stable tag: X.X.X` (if exists)

**WordPress Theme:**
- Detects: `style.css` with `Theme Name:` header
- Updates: `style.css` → `Version: X.X.X` (header comment)

**Swift iOS/macOS App:**
- Detects: `*.xcodeproj` directory with `project.pbxproj` file
- Updates:
  - `project.pbxproj` → `MARKETING_VERSION = X.X.X;` (user-facing version)
  - `project.pbxproj` → `CURRENT_PROJECT_VERSION = N;` (build number)

**Swift Package (SPM):**
- Detects: `Package.swift` file (without `.xcodeproj`)
- Updates: Git tags only (no files modified)
- Note: SPM uses tags WITHOUT `v` prefix (`1.2.3` not `v1.2.3`)

### Version Bumping Rules

**Major bump** (1.2.3 → 2.0.0):
- Breaking changes
- Resets minor and patch to 0
- Removes prerelease tag

**Minor bump** (1.2.3 → 1.3.0):
- New backward-compatible features
- Resets patch to 0
- Removes prerelease tag

**Patch bump** (1.2.3 → 1.2.4):
- Backward-compatible bug fixes
- Removes prerelease tag

**Prerelease bump**:
- `1.2.3` → `1.2.4-alpha.1` (new prerelease)
- `1.2.4-alpha.1` → `1.2.4-alpha.2` (increment same tag)
- `1.2.4-alpha.2` → `1.2.4-beta.1` (change tag)

### Custom Configuration (Optional)

Create `.semverrc` in project root to customize version locations:

```json
{
  "project_type": "wordpress-plugin",
  "version_files": [
    {
      "path": "my-plugin.php",
      "pattern": "Version:\\s*([0-9.]+(?:-[a-zA-Z0-9.]+)?)",
      "replacement": "Version: $VERSION"
    },
    {
      "path": "includes/constants.php",
      "pattern": "define\\(\\s*'PLUGIN_VERSION',\\s*'([^']+)'\\s*\\)",
      "replacement": "define( 'PLUGIN_VERSION', '$VERSION' )"
    }
  ],
  "git_tag_prefix": "v",
  "auto_commit": false,
  "commit_message_template": "chore: bump version to $VERSION"
}
```

### Handling Generated/Compiled Files

For projects with build processes (SCSS→CSS, TypeScript→JS, etc.), mark compiled output files with `"generated": true`:

```json
{
  "project_type": "wordpress-theme",
  "version_files": [
    {
      "path": "src/style.scss",
      "pattern": "Version:\\s*([0-9.]+(?:-[a-zA-Z0-9.]+)?)",
      "replacement": "Version: $VERSION"
    },
    {
      "path": "style.css",
      "pattern": "Version:\\s*([0-9.]+(?:-[a-zA-Z0-9.]+)?)",
      "replacement": "Version: $VERSION",
      "generated": true
    }
  ]
}
```

**Why use `generated`?**
- Build processes often reset version numbers in output files
- Files with `generated: true` are **skipped when reading** versions (prevents stale/reset versions)
- They are **still updated when writing** versions (keeps compiled files in sync)

**Use cases:**
- SCSS/SASS → CSS compilation
- TypeScript → JavaScript compilation
- Any `dist/` or `build/` output files with version strings
- Minified files (`.min.css`, `.min.js`)

## Examples:

```bash
# Show current version
/semver --current
/semver -c

# Bump patch version (1.2.3 → 1.2.4)
/semver patch

# Bump minor version (1.2.3 → 1.3.0)
/semver minor

# Bump major version (1.2.3 → 2.0.0)
/semver major

# Create prerelease version
/semver prerelease alpha        # 1.2.3 → 1.2.4-alpha.1
/semver prerelease beta         # 1.2.4-alpha.1 → 1.2.4-beta.1

# Preview changes without modifying files
/semver minor --dry-run

# Bump version and create git commit
/semver patch --git-commit

# Bump version, commit, and create git tag
/semver minor --git-tag

# Set explicit version
/semver --set 2.0.0

# Initialize custom configuration
/semver --init

# iOS/macOS specific examples
/semver patch --increment-build  # 1.2.3 (build 42) → 1.2.4 (build 43)
/semver minor --reset-build      # 1.2.4 (build 43) → 1.3.0 (build 1)
/semver patch --build-number 100 # 1.3.0 (build 1) → 1.3.1 (build 100)
```

## Workflow Examples:

### First Time Setup
```bash
# Agent auto-detects project type and creates cache
/semver --current

# Optional: customize version locations
/semver --init
# Edit .semverrc as needed
```

### Regular Development
```bash
# Bug fix release
/semver patch --git-tag          # 1.2.3 → 1.2.4 + git tag v1.2.4

# New feature
/semver minor --git-tag          # 1.2.4 → 1.3.0 + git tag v1.3.0

# Breaking change
/semver major --git-tag          # 1.3.0 → 2.0.0 + git tag v2.0.0
```

### Prerelease Workflow
```bash
# Start alpha testing
/semver prerelease alpha         # 1.2.3 → 1.2.4-alpha.1

# Multiple alpha releases
/semver prerelease alpha         # 1.2.4-alpha.1 → 1.2.4-alpha.2
/semver prerelease alpha         # 1.2.4-alpha.2 → 1.2.4-alpha.3

# Move to beta
/semver prerelease beta          # 1.2.4-alpha.3 → 1.2.4-beta.1

# Release final version
/semver patch --git-tag          # 1.2.4-beta.1 → 1.2.4
```

### Preview Changes
```bash
# See what would change without modifying files
/semver minor --dry-run

# Output shows:
# - Current version
# - New version
# - Files that would be updated
# - Line numbers and exact changes
```

### iOS/macOS App Development
```bash
# Standard patch release (auto-increments build)
/semver patch                    # 1.2.3 (build 42) → 1.2.4 (build 43)

# Minor release, reset build to 1
/semver minor --reset-build      # 1.2.4 (build 43) → 1.3.0 (build 1)

# Major release with git tag
/semver major --git-tag --reset-build  # 1.3.0 (build 1) → 2.0.0 (build 1)

# TestFlight build with same version
/semver patch --build-number 44  # Keep 1.2.4, set build to 44

# Check current version and build
/semver --current
# Output: Version: 1.2.4 | Build: 44
```

### Swift Package Development
```bash
# Swift Packages use git tags exclusively
/semver patch                    # Creates tag: 1.2.4 (no 'v' prefix)

# Prerelease for Swift Package
/semver prerelease beta          # Creates tag: 1.2.4-beta.1

# Tags are created automatically, then push
git push --tags

# Check current package version
/semver --current
# Output: Current version: 1.2.4 (from git tag)
```

## Cache and Configuration Files

**`.semver-cache.json`** (auto-generated):
- Stores detected project type
- Lists version file locations
- Tracks last version
- Speeds up subsequent runs
- Safe to delete (will regenerate)
- **Automatically added to `.gitignore`** on first run

**`.semverrc`** (optional, user-created):
- Overrides auto-detection
- Defines custom version file locations
- Configures git tag prefix
- Sets commit message template
- Created via `/semver --init`
- **Should be committed to git** (shared team config)

## Error Handling:

**Version mismatch detected:**
```
❌ Version mismatch detected
  • package.json: 1.2.3
  • README.md: 1.2.4

Fix: /semver --set 1.2.3
```

**Invalid version format:**
```
❌ Invalid version: 1.2
Must follow semver: MAJOR.MINOR.PATCH[-PRERELEASE]
Examples: 1.2.3, 2.0.0-alpha.1
```

**Project type not detected:**
```
❌ Unable to detect project type

Run /semver --init to configure manually
```

## Help Output (for --help/-h):

When user types `/semver --help` or `/semver -h`, display this formatted help:

```
/semver - Semantic Version Management

USAGE:
  /semver [command] [options]

COMMANDS:
  major                         Bump major version (X.0.0)
  minor                         Bump minor version (x.Y.0)
  patch                         Bump patch version (x.y.Z)
  prerelease [tag]              Bump prerelease (x.y.z-TAG.N)

  --current, -c                 Show current version
  --init                        Initialize .semverrc config
  --set X.X.X                   Set explicit version
  -h, --help                    Show this help

OPTIONS:
  -d, --dry-run                 Preview changes only
  --git-commit                  Create git commit
  --git-tag                     Create git tag (+ commit)
  --force                       Skip consistency checks

iOS/macOS OPTIONS:
  --build-number N              Set explicit build number
  --increment-build             Auto-increment build (default)
  --reset-build                 Reset build to 1

SUPPORTED PROJECTS:
  • React/Next.js              (package.json)
  • WordPress Plugin           (*.php header + readme.txt)
  • WordPress Theme            (style.css header)
  • Swift iOS/macOS App        (*.xcodeproj/project.pbxproj)
  • Swift Package (SPM)        (Package.swift + git tags)

EXAMPLES:
  /semver --current                    # Show current version
  /semver patch                        # Bug fix: 1.2.3 → 1.2.4
  /semver minor --git-tag              # New feature: 1.2.4 → 1.3.0 + tag
  /semver major --dry-run              # Preview: 1.3.0 → 2.0.0
  /semver prerelease alpha             # Create: 2.0.0 → 2.0.1-alpha.1
  /semver --set 2.1.0                  # Set explicit version

iOS/macOS EXAMPLES:
  /semver patch                        # 1.2.3 (build 42) → 1.2.4 (build 43)
  /semver minor --reset-build          # 1.2.4 (build 43) → 1.3.0 (build 1)
  /semver patch --build-number 100     # Set specific build number

Swift Package EXAMPLES:
  /semver patch                        # Creates tag: 1.2.4 (no 'v' prefix)
  /semver prerelease beta              # Creates tag: 1.2.4-beta.1

VERSION FORMAT (Semantic Versioning):
  MAJOR.MINOR.PATCH[-PRERELEASE]

  • MAJOR: Breaking changes (1.2.3 → 2.0.0)
  • MINOR: New features (1.2.3 → 1.3.0)
  • PATCH: Bug fixes (1.2.3 → 1.2.4)
  • PRERELEASE: alpha, beta, rc (1.2.4-alpha.1)

CONFIGURATION:
  .semver-cache.json     Auto-generated cache (speeds up detection)
  .semverrc              Optional custom config (version file locations)

  Create custom config:
    /semver --init       Generate .semverrc template

For custom version locations (constants, configs, etc.):
  1. Run /semver --init
  2. Edit .semverrc
  3. Add custom file patterns

For compiled/generated files (SCSS→CSS, TypeScript→JS):
  Add "generated": true to skip reading but still update on write

WORKFLOW:
  1. /semver --current                 # Check current state
  2. /semver minor --dry-run           # Preview changes
  3. /semver minor --git-tag           # Apply and tag

For more details, see project documentation.
```

## Notes:

- **Safe by default**: Always validates version consistency across files
- **Atomic updates**: All version files updated together or not at all
- **Git integration**: Optional commit and tag creation
- **Dry-run mode**: Preview changes before applying
- **Extensible**: Support custom version locations via `.semverrc`
- **Fast**: Caches project detection for subsequent runs
- **Smart detection**: Auto-identifies project type on first run
