# Preflight Check Command

Run a pre-deployment sanity check on your project. This command validates that your codebase is clean, builds successfully, passes tests, and is free of common deployment hazards.

**IMPORTANT**: If the user provides `--help` or `-h` as an argument, display the complete usage information below instead of executing any operations.

## Arguments:

- `(default)`: Run all applicable checks for the detected project type
- `--build`: Only check that the project builds successfully
- `--tests`: Only run the test suite
- `--lint`: Only run linting/formatting checks
- `--deps`: Only check for dependency issues
- `--env`: Only check environment and configuration safety
- `--git`: Only check git state (clean tree, branch, unpushed commits)
- `--skip <check>`: Skip a specific check (e.g., `--skip tests`)
- `--fix`: Attempt to auto-fix issues where possible (formatting, lint)
- `--strict`: Treat warnings as failures
- `--help` or `-h`: Show detailed usage information

## Behavior:

### Auto-detection

On run, detect the project type and available tooling:

1. **Project type**: Same detection as `/review` (Next.js, React, WordPress, Unity, Swift, Node.js)
2. **Available tools**: Check for presence of:
   - Package manager: `package-lock.json` (npm), `yarn.lock` (yarn), `pnpm-lock.yaml` (pnpm)
   - Test runner: jest, vitest, phpunit, xcodebuild test, Unity Test Runner
   - Linter: eslint, prettier, phpcs, swiftlint
   - Build command: `build` script in package.json, `xcodebuild`, Unity build
   - TypeScript: `tsconfig.json` presence

### Check Sequence

Checks run in this order (each check reports PASS, WARN, or FAIL):

#### 1. Git State (`git`)
- Working tree is clean (no uncommitted changes) — WARN if dirty
- Current branch name — INFO
- Unpushed commits — WARN if any
- Detached HEAD state — WARN

#### 2. Environment Safety (`env`)
- No `.env` files staged or committed (check git ls-files)
- No hardcoded API keys, tokens, or passwords in source files (quick grep for common patterns)
- Required environment variables documented (check `.env.example` or `.env.template` exists if `.env` is used)
- No `localhost` or `127.0.0.1` URLs in production code (outside test files)
- No `console.log` / `print` / `debugger` statements in source (outside test files)

#### 3. Dependencies (`deps`)
- Lock file exists and matches package.json (npm ls / yarn check)
- No known vulnerable dependencies (`npm audit` / `yarn audit` — WARN on moderate, FAIL on high/critical)
- No missing peer dependencies
- WordPress: Check plugin/theme `Requires at least` and `Tested up to` headers are current

#### 4. Lint & Formatting (`lint`)
- Run project linter if configured (eslint, prettier --check, phpcs, swiftlint)
- Report errors as FAIL, warnings as WARN
- With `--fix`: attempt auto-fix (`eslint --fix`, `prettier --write`)

#### 5. Type Checking (`types`) — TypeScript projects only
- Run `npx tsc --noEmit` to check for type errors
- Report any type errors as FAIL

#### 6. Tests (`tests`)
- Detect and run the test suite:
  - Node.js: `npm test` or `npx jest` or `npx vitest run`
  - WordPress: `phpunit` if configured
  - Swift: `xcodebuild test` or `swift test`
  - Unity: Check for EditMode/PlayMode test assemblies
- Report: pass count, fail count, skip count
- Any test failure → FAIL

#### 7. Build (`build`)
- Run the project build command:
  - Node.js/Next.js: `npm run build`
  - Swift: `xcodebuild build` or `swift build`
  - WordPress: Check for build script, run if present
  - Unity: Validate project structure (actual builds are platform-specific)
- Any build error → FAIL
- Build warnings → WARN

#### 8. Version Consistency (`version`)
- If `.semverrc` or `.semver-cache.json` exists, check that versions are in sync across all tracked files
- Compare package.json version with README badge if present
- FAIL if versions are mismatched

### Output Format

```
PREFLIGHT CHECK — [project-name] ([project-type])
═══════════════════════════════════════

  ✅ Git State
     Branch: feature/user-auth | Clean | 0 unpushed

  ✅ Environment Safety
     No secrets detected | .env.example present | No debug statements

  ⚠️ Dependencies
     2 moderate vulnerabilities found (npm audit)
     Run `npm audit fix` to resolve

  ✅ Lint & Formatting
     ESLint: 0 errors, 0 warnings | Prettier: formatted

  ✅ Type Checking
     TypeScript: 0 errors

  ❌ Tests
     42 passed, 1 failed, 0 skipped
     FAIL: src/lib/auth.test.ts > validateToken > should reject expired tokens

  ✅ Build
     Next.js build completed (12.3s) | 0 warnings

  ✅ Version Consistency
     v2.1.0 across all files

───────────────────────────────────────
RESULT: 1 failed, 1 warning, 6 passed

❌ PREFLIGHT FAILED
   Fix the failing test before deploying.
```

### Verdict

- **All checks PASS**: `✅ PREFLIGHT PASSED — ready to deploy`
- **Only WARN issues**: `⚠️ PREFLIGHT PASSED WITH WARNINGS — review before deploying`
- **Any FAIL**: `❌ PREFLIGHT FAILED — fix issues before deploying`
- **With --strict**: `❌ PREFLIGHT FAILED (strict mode) — warnings treated as failures`

## Examples:

```bash
# Run all checks
/preflight

# Only check build and tests
/preflight --build --tests

# Run everything except tests (faster)
/preflight --skip tests

# Check environment safety before going live
/preflight --env --strict

# Auto-fix formatting issues, then run all checks
/preflight --fix

# Just check git and dependency state
/preflight --git --deps
```

## Help Output (for --help/-h):

When user types `/preflight --help` or `/preflight -h`, display this formatted help:

```
/preflight - Pre-deployment Sanity Check

USAGE:
  /preflight [options]

CHECK OPTIONS (run specific checks only):
  --build                         Build check only
  --tests                         Test suite only
  --lint                          Lint & formatting only
  --deps                          Dependency check only
  --env                           Environment safety only
  --git                           Git state only

MODIFIER OPTIONS:
  --skip <check>                  Skip a check (git|env|deps|lint|types|tests|build|version)
  --fix                           Auto-fix lint/formatting issues
  --strict                        Treat warnings as failures
  -h, --help                      Show this help

CHECKS PERFORMED (in order):
  1. Git State        Branch, clean tree, unpushed commits
  2. Environment      Secrets, debug statements, hardcoded URLs
  3. Dependencies     Lock file, vulnerabilities, peer deps
  4. Lint/Format      ESLint, Prettier, phpcs, swiftlint
  5. Type Checking    TypeScript compiler (if applicable)
  6. Tests            Jest, Vitest, PHPUnit, XCTest
  7. Build            npm build, xcodebuild, swift build
  8. Version          Semver consistency across files

SUPPORTED PROJECTS:
  • Next.js / React          • WordPress Plugin/Theme
  • Swift iOS/macOS          • Swift Package (SPM)
  • Unity                    • General Node.js

EXAMPLES:
  /preflight                             # Run all checks
  /preflight --build --tests             # Build and tests only
  /preflight --skip tests                # Everything except tests
  /preflight --fix                       # Auto-fix then check all
  /preflight --env --strict              # Strict environment check
  /preflight --git --deps                # Git and dependency check

WORKFLOW:
  Recommended pre-deployment flow:
    /review → fix issues → /preflight → /gitco → /semver patch --git-tag

For more details, see the full command documentation.
```

## Notes:

- **Non-destructive by default**: Only reads and runs check commands, never modifies files (except with `--fix`)
- **Fail-fast option**: Stops on first FAIL if `--strict` is used
- **Project-aware**: Adapts checks to the detected project type
- **Complements /review**: `/review` checks code quality, `/preflight` checks operational readiness
- **Safe commands only**: All build/test/lint commands run in read-only or check mode
