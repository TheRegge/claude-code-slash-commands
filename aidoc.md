# /aidoc

Create or update the `.aidoc` tracking file in the current project directory. This command only manages the tracking file without creating any documentation.

## Usage

- `/aidoc` - Create or update .aidoc file with current git state
- `/aidoc --init` - Initialize new .aidoc file (ignores existing file)
- `/aidoc --docs file1.md file2.md` - Update .aidoc with specific documentation files
- `/aidoc --from-commit abc123` - Specify starting commit for this documentation range

## How it works

This command calls the specialized `aidoc-tracker` agent to:
1. Get the current git commit SHA
2. Read existing .aidoc file (if present)
3. Create/update the JSON tracking file with proper structure
4. Preserve documentation history

## Examples

- `/aidoc` - Update tracking file with current commit, no new docs
- `/aidoc --docs docs/api/new-endpoint.md docs/features/auth.md` - Record specific docs as created
- `/aidoc --init` - Create fresh .aidoc file, clearing any existing history
- `/aidoc --from-commit a1b2c3d4` - Specify that this documentation run covers changes from commit a1b2c3d4 to HEAD

The .aidoc file enables incremental documentation by tracking what has already been documented and when.

---
agent: aidoc-tracker
---

Please create or update the `.aidoc` tracking file for this project.

Process:
1. Determine the current working directory as the project root
2. Get the current git commit SHA using `git rev-parse HEAD`
3. Check for existing `.aidoc` file and read it if present
4. Create or update the `.aidoc` file with proper JSON structure

Parameters to use:
- `project_root`: Use the current working directory
- `docs_created`: Use files specified with --docs flag, or empty array if none specified
- `from_commit`: Use commit specified with --from-commit flag, or determine from existing .aidoc, or null for initial run

Handle special flags:
- If `--init` flag is used: Create new .aidoc file ignoring any existing file
- If `--docs` flag is used: Include the specified documentation files in the tracking
- If `--from-commit` flag is used: Use the specified commit as the starting point for this range

Create the .aidoc file with current timestamp and git commit information.