# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- Code-reviewer agent and `/review` command with project-type-aware checks and severity-based findings
- Unity project support for `/semver` — ProjectSettings.asset version management, per-platform build numbers, subfolder support
- README version badge auto-detection and sync for all project types
- `/changelog` command for generating Keep a Changelog entries from git history
- `/preflight` command for pre-deployment sanity checks (build, tests, lint, deps, env)
- `--review` flag for `/gitco` — integrates code-reviewer for pre-commit review

### Fixed
- Version badge format in README and .semverrc

### Changed
- Updated .gitignore for local settings and temp files

## [1.1.1] - 2026-02-19

### Documentation
- Renamed README.txt to README.md and updated semver references

## [1.1.0] - 2026-02-19

### Added
- `/backport` command and backport-manager/backport-applier agent specifications for cross-project backporting
- Agent definition files extracted into dedicated `agents/` directory (aidoc-tracker, backport-applier, backport-manager, dual-purpose-docs-writer, semver-manager, time-tracker)

### Documentation
- Documented all slash commands and custom agents in README (/semver, /document, /timetrack, /aidoc)

## [1.0.0] - 2025-09-09

### Added
- `/gitco` command for smart git commits with auto-generated messages, split mode, and conventional commit support
- `/document` command for project documentation with aidoc-tracker integration and freeform natural language support
- `/semver` command for semantic versioning across React, Next.js, WordPress, and Swift projects
- `/aidoc` command for standalone .aidoc tracking file management
- `/timetrack` command for realistic human-effort time tracking estimates
- Generated file handling (`"generated": true`) for `/semver` command
- MIT License
- README documenting all custom slash commands
