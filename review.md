# Code Review Command

Review code changes before committing. This command acts as your AI pair-programmer doing a thorough code review on staged changes, unstaged changes, or a specific scope.

**IMPORTANT**: If the user provides `--help` or `-h` as an argument, display the complete usage information below instead of executing any operations.

## Arguments:

- `--staged` or `-s`: Review only staged changes (default if changes are staged)
- `--all` or `-a`: Review all changes (staged + unstaged)
- `--files <glob>`: Review specific files matching a pattern
- `--commit <sha>`: Review changes introduced by a specific commit
- `--branch`: Review all changes on the current branch vs its base
- `--security`: Focus specifically on security concerns
- `--thorough`: Run a deeper analysis (slower but catches more edge cases)
- `--quick` or `-q`: Quick review — only flag critical issues
- `--help` or `-h`: Show detailed usage information

## Behavior:

### Default mode:

1. Detect what to review:
   - If there are staged changes → review staged
   - If no staged changes but unstaged exist → ask whether to review unstaged
   - If no changes at all → report "Nothing to review"
2. For each changed file, analyze:
   - **Bugs & Logic Errors**: Off-by-one, null/undefined access, race conditions, infinite loops
   - **Security Issues**: Injection vulnerabilities, exposed secrets, unsafe deserialization, missing auth checks, XSS, CSRF
   - **Error Handling**: Missing try/catch, swallowed errors, unclear error messages, missing fallbacks
   - **Edge Cases**: Empty arrays, null inputs, boundary values, concurrent access
   - **Code Quality**: Dead code, duplicated logic, overly complex functions, naming issues
   - **Performance**: N+1 queries, unnecessary re-renders, missing memoization, large bundle imports
   - **Type Safety**: Missing types, unsafe casts, `any` usage (TypeScript)
3. Present findings grouped by severity:
   - 🔴 **Critical** — Must fix before committing (security holes, data loss risks, crashes)
   - 🟡 **Warning** — Should fix, likely to cause problems (bugs, poor error handling)
   - 🔵 **Suggestion** — Nice to fix, improves quality (naming, structure, performance)
4. For each finding, show:
   - File and line number(s)
   - What the issue is (1-2 sentences)
   - Why it matters
   - Suggested fix (code snippet when helpful)
5. End with a summary verdict:
   - ✅ **Clear** — No critical or warning issues found
   - ⚠️ **Review recommended** — Warnings found, consider fixing before commit
   - 🛑 **Issues found** — Critical issues that should be addressed
6. **Post-review next steps** (unless verdict is Clear):
   - Offer to fix the issues found. Present options:
     - **Fix all** — Attempt to fix all Critical and Warning issues automatically
     - **Fix critical only** — Only address Critical items
     - **Pick specific** — Let the user choose which findings to fix
     - **Skip** — Do nothing, the user will handle it manually
   - If the user chooses to fix:
     1. Apply fixes one file at a time
     2. For each fix, briefly explain what was changed
     3. After all fixes are applied, automatically re-run the review on the modified files
     4. Present the new verdict (the re-review should show the fixes resolved the original issues)
     5. If new issues were introduced by the fixes, flag them
   - If verdict is ✅ Clear after fixes (or originally): suggest `/gitco` as the next step

### Security mode (`--security`):

Focus exclusively on security concerns:
- Hardcoded secrets, API keys, tokens
- SQL/NoSQL injection vectors
- XSS and output encoding
- Authentication and authorization gaps
- CSRF protection
- Insecure dependencies or imports
- File path traversal
- Unsafe regular expressions (ReDoS)
- Information leakage in error messages
- Missing rate limiting or input validation

### Branch mode (`--branch`):

1. Determine the base branch (main/master/develop)
2. Get all commits on the current branch since divergence
3. Review the cumulative diff
4. Especially useful before creating a PR

### Thorough mode (`--thorough`):

Everything in default mode plus:
- Cross-file dependency analysis (does this change break callers?)
- Test coverage check (are the changed code paths tested?)
- Documentation freshness (do docs still match the code?)
- Backwards compatibility (API contract changes?)
- Build/lint check (run available linters if configured)

## Output Format:

```
CODE REVIEW — [scope description]
═══════════════════════════════════════

📁 src/lib/auth.ts

  🔴 CRITICAL: JWT token not validated before use (line 42)
     The token from the request header is decoded but never verified
     against the signing secret. Any crafted JWT would be accepted.

     Fix: Add verification step before decoding:
     ```typescript
     const verified = jwt.verify(token, process.env.JWT_SECRET);
     ```

  🟡 WARNING: Error swallowed silently (line 67)
     The catch block is empty — authentication failures will pass
     through without any indication of what went wrong.

     Fix: At minimum, log the error and return a 401 response.

📁 src/components/UserProfile.tsx

  🔵 SUGGESTION: Unnecessary re-render on every parent update (line 15)
     This component receives `user` as a prop but doesn't memoize.
     Consider wrapping with React.memo() since user data changes rarely.

───────────────────────────────────────
SUMMARY: 1 critical, 1 warning, 1 suggestion
Verdict: 🛑 Issues found — address critical items before committing
```

## Integration with /gitco:

When called from `/gitco --review`, output a condensed version:
- Only show 🔴 Critical and 🟡 Warning items
- Skip 🔵 Suggestions
- End with a clear pass/fail that `/gitco` can act on
- If critical issues found, recommend aborting the commit

## Examples:

```bash
# Review staged changes (default)
/review

# Review everything including unstaged
/review --all

# Quick review — only critical issues
/review --quick

# Security-focused review
/review --security

# Review specific files
/review --files "src/lib/**"

# Review current branch vs main
/review --branch

# Deep analysis before a release
/review --thorough

# Review a specific commit
/review --commit abc1234
```

## Help Output (for --help/-h):

When user types `/review --help` or `/review -h`, display this formatted help:

```
/review - AI Code Review

USAGE:
  /review [options]

SCOPE OPTIONS:
  -s, --staged                    Review staged changes (default)
  -a, --all                       Review all changes (staged + unstaged)
  --files <glob>                  Review specific files
  --commit <sha>                  Review a specific commit
  --branch                        Review current branch vs base

MODE OPTIONS:
  --security                      Security-focused review
  --thorough                      Deep analysis (slower, more thorough)
  -q, --quick                     Quick review — critical issues only
  -h, --help                      Show this help

SEVERITY LEVELS:
  🔴 Critical    Must fix — security holes, crashes, data loss
  🟡 Warning     Should fix — bugs, poor error handling
  🔵 Suggestion  Nice to fix — naming, structure, performance

CHECKS PERFORMED:
  • Bugs & logic errors          • Security vulnerabilities
  • Error handling gaps          • Edge case coverage
  • Code quality issues          • Performance concerns
  • Type safety (TypeScript)     • Cross-file impacts (--thorough)

EXAMPLES:
  /review                                # Review staged changes
  /review --all                          # Review all changes
  /review --security                     # Security-focused review
  /review --branch --thorough            # Deep review of branch
  /review --files "src/api/**"           # Review specific directory
  /review --quick                        # Quick critical-only scan

INTEGRATION:
  Use before /gitco for a review-then-commit workflow:
    /review → fix issues → /gitco

  Or use the built-in flag:
    /gitco --review    (runs quick review before committing)

For more details, see the full command documentation.
```

## Notes:

- **Non-destructive by default**: This command only reads and reports — unless the user opts into fixes
- **Context-aware**: Understands project type (React, Next.js, WordPress, Unity, Swift) from project structure
- **Incremental**: Reviews only what changed, not the entire codebase
- **Pairs with /gitco**: Use `/review` → fix → `/gitco` as your standard workflow

---
agent: code-reviewer
---

Review code changes and report findings. Parse the user's arguments to determine scope and mode.

## Argument Parsing

Parse the user's input for:

1. **Scope**: `--staged`/`-s`, `--all`/`-a`, `--files <glob>`, `--commit <sha>`, `--branch`
2. **Mode**: `--security`, `--thorough`, `--quick`/`-q`
3. **Help**: `--help`/`-h`
4. **Default**: If no scope given, use staged changes (or prompt if none staged)

## Execution

1. **Determine the diff to review:**
   - Staged: `git diff --cached`
   - All: `git diff HEAD`
   - Files: `git diff -- <glob>` (or read files directly if not in git)
   - Commit: `git diff <sha>^..<sha>`
   - Branch: `git diff $(git merge-base HEAD main)..HEAD` (try main, then master, then develop)

2. **Detect project type** from files present:
   - `package.json` with next → Next.js
   - `package.json` with react → React
   - `style.css` with Theme Name or `.php` with Plugin Name → WordPress
   - `ProjectSettings/ProjectSettings.asset` → Unity
   - `*.xcodeproj` → Swift/iOS
   - `Package.swift` → Swift Package

3. **For each changed file**, read the full file (not just the diff) for context, then analyze the diff hunks against the review checklist. Tailor checks to the project type:

   **Next.js/React:**
   - Missing `use client` / `use server` directives
   - Unsafe usage of `dangerouslySetInnerHTML`
   - Missing key props in lists
   - useEffect dependency array issues
   - Server component data leaking to client
   - Unprotected API routes

   **WordPress:**
   - Missing nonce verification
   - Unescaped output (missing `esc_html`, `esc_attr`, `wp_kses`)
   - Direct database queries without `$wpdb->prepare()`
   - Missing capability checks
   - Unsanitized `$_GET`/`$_POST` usage

   **Unity (C#):**
   - Missing null checks on GetComponent results
   - Allocations in Update/FixedUpdate loops
   - Missing OnDestroy cleanup
   - Coroutine error handling
   - Serialization issues

   **Swift/iOS:**
   - Force unwrapping optionals
   - Missing main thread dispatch for UI updates
   - Retain cycles in closures
   - Missing error handling in async/await

   **General (all projects):**
   - Hardcoded secrets or credentials
   - Console.log/print statements left in
   - TODO/FIXME/HACK comments in new code
   - Empty catch blocks
   - Unused imports or variables

4. **Classify findings** by severity (Critical, Warning, Suggestion)

5. **Format output** according to the output format specification

6. **If called from /gitco --review** (indicated by a note in the prompt):
   - Only report Critical and Warning items
   - Return a structured verdict: `PASS` (no critical/warnings) or `FAIL` (issues found)
   - Keep output concise

## Important Reminders

**DO:**
- Read full files for context, not just diff hunks
- Include line numbers in all findings
- Provide actionable fix suggestions with code snippets
- Tailor checks to the detected project type
- Group findings by file, then by severity
- Be specific — "line 42 has an issue" not "there might be issues"

**DON'T:**
- Modify any files
- Run any commands that change state (no git commit, no npm install, etc.)
- Flag issues that exist in unchanged code (only review the diff)
- Be overly pedantic — focus on issues that actually matter
- Flag style issues unless they're genuinely confusing
- Report the same issue multiple times across files
