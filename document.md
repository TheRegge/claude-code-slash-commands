# /document

Create intelligent documentation for your codebase. This command analyzes recent changes to identify what needs documentation, then documents the current state of your code - optimized for both developers and LLMs.

## Usage

- `/document` - Document the current state of recently modified code
- `/document feature [name]` - Document a feature as it currently exists
- `/document fix [issue]` - Document the corrected behavior
- `/document api [endpoint]` - Document API in its current form
- `/document architecture [component]` - Document architectural decisions
- `/document update [topic]` - Update existing documentation
- `/document verify` - Check documentation freshness
- `/document [freeform instructions]` - Document based on natural language instructions
- `/document --changes` - Include historical context about what changed
- `/document --full` - Ignore tracking, document entire codebase
- `/document --reset` - Clear tracking state and start fresh
- `/document --since [commit]` - Document code modified since specific commit

## How it works

The command will:
1. Use git history to identify which parts of the code need documentation
2. Analyze the actual source code files to understand current implementation
3. Search for related existing documentation to update or extend
4. Generate documentation describing how the code works now
5. Place documentation in the project's docs/ directory with LLM-optimized structure

By default, documentation focuses on the current state. Use `--changes` to include historical context.

## Examples

- `/document feature "user authentication"` - Documents auth system as it currently works
- `/document fix "memory leak"` - Documents the corrected implementation
- `/document api "/api/v2/users"` - Documents current API endpoints and parameters
- `/document --changes` - Documents current state plus what changed from before
- `/document update "database schema"` - Updates existing schema documentation
- `/document verify` - Checks if docs match current code
- `/document Please ensure the authentication system and API endpoints are documented` - Natural language request

The documentation will include metadata, code examples, and structured information that helps both human developers and AI assistants understand your codebase better.

---
agent: dual-purpose-docs-writer
---

Please create comprehensive documentation for this codebase, using git history to identify what needs to be documented.

Process:
1. Use git commits/changes to identify which files and functions have been modified
2. Read and analyze the actual source code files to understand current implementation
3. Document how the code works in its current state
4. Search for existing documentation that should be updated
5. Create or update the `.aidoc` tracking file at the project root

Focus on documenting the current state of the code - describe what exists now and how it works. Do NOT include historical information about what changed or how things used to work, unless the user specifically included the --changes flag.

With --changes flag: Include historical context explaining what was modified, added, or fixed.
Without --changes flag (default): Document only the current implementation.

Create or update documentation in the docs/ directory. Include code examples from the actual codebase, usage instructions based on current APIs, and any architectural decisions reflected in the code.

**CRITICAL: .aidoc Tracking File Management**
After completing all documentation work, you MUST call the aidoc-tracker agent to create/update the `.aidoc` tracking file.

Call the aidoc-tracker agent with:
- `project_root`: Current working directory path
- `docs_created`: Array of all documentation files you created or updated during this run
- `from_commit`: The commit SHA you started documenting from (if available from existing .aidoc)

The aidoc-tracker agent will handle:
- Getting the current git commit SHA
- Reading existing .aidoc file (if present)
- Creating proper JSON structure
- Preserving documentation history
- Writing the .aidoc file in correct format

**Do not attempt to create the .aidoc file yourself - always use the aidoc-tracker agent for this task.**

## Argument Parsing

The command supports both structured and freeform arguments. Parse the user's input as follows:

**1. Check for structured patterns first:**
- `feature [name]` - Document a specific feature
- `fix [issue]` - Document a bug fix or corrected behavior
- `api [endpoint]` - Document API endpoints
- `architecture [component]` - Document architectural decisions
- `update [topic]` - Update existing documentation
- `verify` - Check documentation freshness

**2. Check for flags:**
- `--changes` - Include historical context about what changed
- `--full` - Ignore tracking, document entire codebase
- `--reset` - Clear tracking state and start fresh
- `--since [commit]` - Document code modified since specific commit

**3. Handle freeform instructions:**
If the user's input doesn't match any structured pattern above, treat it as freeform natural language instructions. Examples:
- "Please ensure the authentication system and API endpoints are documented"
- "Look at database migrations and make sure they're covered"
- "Document the payment processing flow and error handling"

For freeform instructions:
- Parse the natural language to identify what needs to be documented
- Extract key topics, components, or areas mentioned (e.g., "authentication system", "API endpoints", "database migrations")
- Apply the appropriate documentation approach based on the context
- If the instruction mentions multiple areas, document all of them
- Use the same git history and code analysis approach as structured commands

**4. Execution priority:**
- Structured patterns take precedence over freeform (e.g., "feature auth" is treated as structured, not freeform)
- Flags can be combined with any mode (structured or freeform)
- If unclear, treat as freeform and parse the intent

If the user provided specific arguments (structured patterns like "feature auth system", freeform instructions like "ensure auth is documented", or flags), focus on those areas while documenting the current state.