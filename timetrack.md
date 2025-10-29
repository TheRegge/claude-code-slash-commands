# /timetrack

Analyze your project and create realistic time tracking estimates based on human development effort. This command evaluates code complexity, git history, and project scope to estimate how many hours a senior engineer would have spent building the project.

## Usage

- `/timetrack` - Create or update time tracking for entire project
- `/timetrack init` - Initialize new time tracking file
- `/timetrack update` - Add recent changes to existing tracking
- `/timetrack report` - Generate summary report from existing data
- `/timetrack scale [hours]` - Scale existing report to fit within specified hour limit
- `/timetrack --since [date]` - Track time since specific date
- `/timetrack --category [name]` - Track time for specific category
- `/timetrack --human-pace` - Emphasize realistic human development speed
- `/timetrack --round-full` - Round all hours up to nearest full hour
- `/timetrack --round-half` - Round all hours up to nearest 0.5 increment

## How it works

The command will:
1. Analyze git commits and code changes to understand project scope
2. Evaluate code complexity, architecture, and dependencies
3. Read existing documentation to understand features
4. Calculate realistic human development time (not AI-accelerated time)
5. Categorize time by feature, bug fixes, refactoring, testing, etc.
6. Store data in `.timetrack.json` file at project root
7. Update existing tracking file with new sessions

## Time Estimation Philosophy

The agent estimates time as a **senior developer working efficiently** would spend:
- Accounts for planning, implementation, testing, and debugging
- Considers learning curve for new technologies
- Factors in code review and documentation time
- Adjusts for complexity and risk factors
- **Ignores AI-accelerated development times** - even if git shows rapid commits

## Examples

- `/timetrack` - Analyze entire project and create time estimates
- `/timetrack update` - Add time for changes since last tracking
- `/timetrack report` - Show summary of time spent by category
- `/timetrack scale 40` - Rescale existing 100-hour report to fit within 40 hours
- `/timetrack scale 40 --round-full` - Rescale to 40h, round all hours up to full hours
- `/timetrack scale 40 --round-half` - Rescale to 40h, round all hours up to 0.5 increments
- `/timetrack --round-full` - Round existing tracking to full hours
- `/timetrack --since 2024-01-01` - Track time since beginning of year

The tracking data helps with project estimation, billing, and understanding actual human effort required for similar projects.

---
agent: time-tracker
---

Please analyze this project and create realistic time tracking estimates for development effort.

**Core Task**: Evaluate how many hours a senior software engineer would have spent developing this project at a realistic human pace.

## Process:

### Regular Mode (default):

1. **Project Analysis**
   - Check if `.timetrack.json` exists at project root
   - Analyze git log to understand commit history and development timeline
   - Read key source files to understand architecture and complexity
   - Review documentation to understand features and scope
   - Identify major features, refactorings, and bug fixes

2. **Time Estimation** (Human-Realistic)
   - Estimate time for each major feature/component
   - Consider: planning, coding, testing, debugging, documentation
   - Account for learning curves and research time
   - Factor in code review and iteration cycles
   - **CRITICAL**: Base estimates on human development speed, NOT git commit times
   - If commits show rapid AI-assisted development, still estimate realistic human time

3. **Categorization**
   - Feature Development
   - Bug Fixes
   - Refactoring
   - Testing & QA
   - Documentation
   - Setup & Configuration
   - Code Review & Planning

4. **Update Tracking File**
   - Create or update `.timetrack.json` at project root
   - Preserve existing tracking history
   - Add new entries for recent work
   - Include detailed breakdown by category
   - Add metadata (dates, commit ranges, notes)

### Scale Mode (`scale [hours]`):

**Purpose**: Proportionally adjust existing time tracking to fit within a specified hour limit for business/billing purposes.

1. **Read Existing Data**
   - Read `.timetrack.json` from project root
   - If file doesn't exist, inform user to run `/timetrack` first

2. **Calculate Scaling**
   - Get current total_hours from tracking file
   - Calculate scaling factor: `target_hours / total_hours`
   - Example: 40 target hours / 100 total hours = 0.40 (40% scale)

3. **Apply Scaling**
   - Multiply ALL hour values by scaling factor:
     - Each category total
     - Each session's category hours
     - Each session total
     - Overall total
   - **Rounding options:**
     - Default: Round to 1 decimal place for readability
     - With `--round-full`: Round UP to nearest full hour (e.g., 3.2 → 4.0, 5.0 → 5.0)
     - With `--round-half`: Round UP to nearest 0.5 increment (e.g., 3.2 → 3.5, 3.6 → 4.0, 3.5 → 3.5)
   - **Preserve all descriptions, dates, commits, and structure**
   - Keep percentages/proportions identical (before rounding)

4. **Output Options**
   - **Option A** (default): Display side-by-side comparison without saving
   - **Option B** (with `--save` flag): Create `.timetrack.scaled.json` with scaled values
   - **Option C** (with `--replace` flag): Replace original with scaled values (keep backup)

5. **Display Format**
   ```
   Time Tracking - Scaling Report
   ================================

   Original Total: 100.0 hours
   Target Total:   40.0 hours
   Scaling Factor: 40% (0.40)

   Category Breakdown:

   Category              Original    Scaled    Proportion
   ───────────────────────────────────────────────────────
   Feature Development    48.0h      19.2h     48.0%
   Testing                16.0h       6.4h     16.0%
   Bug Fixes               8.0h       3.2h      8.0%
   Documentation           6.0h       2.4h      6.0%
   Refactoring            12.0h       4.8h     12.0%
   Setup                   8.0h       3.2h      8.0%
   Planning               10.0h       4.0h     10.0%
   ───────────────────────────────────────────────────────
   TOTAL                 108.0h      43.2h    100.0%

   Note: Proportions maintained. Scaled report suitable for
   business estimates while preserving work distribution.
   ```

6. **Important Notes**
   - Scaling does NOT modify original tracking by default
   - All work descriptions remain unchanged
   - Git commit references preserved
   - Session history maintained
   - Use `--save` to export scaled version
   - Use `--replace` to update original (creates `.timetrack.backup.json`)
   - Rounding flags work with both scaling and regular tracking
   - Rounding always rounds UP (ceiling) for conservative estimates

## JSON Schema for .timetrack.json:

```json
{
  "version": "1.0",
  "project_name": "string",
  "total_hours": 0,
  "last_updated": "ISO_timestamp",
  "last_commit_tracked": "git_sha",
  "sessions": [
    {
      "session_id": "unique_id",
      "date": "ISO_timestamp",
      "commit_range": {
        "from": "git_sha_or_null",
        "to": "git_sha"
      },
      "categories": {
        "feature_development": {"hours": 0, "description": "string"},
        "bug_fixes": {"hours": 0, "description": "string"},
        "refactoring": {"hours": 0, "description": "string"},
        "testing": {"hours": 0, "description": "string"},
        "documentation": {"hours": 0, "description": "string"},
        "setup": {"hours": 0, "description": "string"},
        "planning": {"hours": 0, "description": "string"}
      },
      "session_total": 0,
      "notes": "string"
    }
  ],
  "category_totals": {
    "feature_development": 0,
    "bug_fixes": 0,
    "refactoring": 0,
    "testing": 0,
    "documentation": 0,
    "setup": 0,
    "planning": 0
  }
}
```

## Estimation Guidelines:

**Feature Development:**
- Simple CRUD feature: 4-8 hours
- Complex feature with integrations: 16-40 hours
- Major architectural change: 40-80+ hours

**Bug Fixes:**
- Simple fix: 0.5-2 hours
- Complex debugging: 4-16 hours
- Critical production issue: 8-24 hours

**Testing:**
- Unit tests for module: 2-4 hours
- Integration test suite: 8-16 hours
- E2E test coverage: 16-32 hours

**Documentation:**
- API documentation: 4-8 hours
- Architecture docs: 8-16 hours
- User guides: 8-24 hours

**Setup/Config:**
- Initial project setup: 2-8 hours
- CI/CD pipeline: 8-16 hours
- Infrastructure as code: 16-40 hours

## Important Reminders:

- Always read existing `.timetrack.json` first to preserve history
- Use `git rev-parse HEAD` to get current commit SHA
- Create ISO timestamps for all entries
- Be realistic about human development pace
- Include detailed notes about major work items
- Aggregate category totals across all sessions
- Consider the full software development lifecycle, not just coding time

After completing the analysis, provide a summary to the user showing:
- Total estimated hours
- Breakdown by category
- Recent session details
- Location of `.timetrack.json` file
