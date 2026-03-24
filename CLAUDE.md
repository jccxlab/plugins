# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a Claude Code skills/plugins marketplace repository owned by April. It contains custom Claude Code plugins that can be loaded into Claude Code sessions.

## Structure

```
.claude-plugin/marketplace.json   # Marketplace index listing available plugins
plugins/                          # Plugin directory
  wk-review/                      # iOS/Swift/ObjC code review plugin
    .claude-plugin/plugin.json    # Plugin metadata (name, version, author)
    README.md                     # User-facing documentation
    commands/                     # Slash command definitions (.md files with frontmatter)
    skills/                       # Skill implementation files
      wk-review/
        SKILL.md                  # Core skill logic and workflow
        references/               # Supporting reference files used by the skill
```

## Plugin Architecture

Each plugin follows this structure:

- **`plugin.json`** — metadata: `name`, `version`, `description`, `author`
- **`commands/<name>.md`** — defines the slash command with YAML frontmatter (`description`, `mode: skill`, `skill_file`) and user-facing docs; ends with `$ARGUMENTS` to pass user input
- **`skills/<name>/SKILL.md`** — the actual skill implementation with frontmatter (`name`, `description`) and detailed workflow instructions for Claude
- **`skills/<name>/references/`** — reference files (output formats, checklists, etc.) cited by SKILL.md

The command file is the entry point; it delegates to the skill file via `skill_file:` frontmatter. Reference files are linked from SKILL.md using relative paths like `references/output-format.md`.

## Marketplace Index

`/.claude-plugin/marketplace.json` is the root index. When adding a new plugin, register it here:

```json
{
  "plugins": [
    { "name": "plugin-name", "source": "./plugins/plugin-name", "description": "..." }
  ]
}
```

## wk-review Skill

An iOS code review skill that uses a parallel Agent architecture:

- **Input**: `scope` (staged/unstaged/all/branch/commit:\<hash\>), `focus` (logic/crash/memory/performance/thread/resource/bestpractice), `severity` (CRITICAL/HIGH/MEDIUM/LOW)
- **Workflow**: Fetches git diff → groups files → dispatches parallel review Agents (one per file group, max 5) → aggregates JSON results → renders structured Markdown report
- **Agent dispatch rule**: ≤3 files → one Agent per file; >3 files → group by module; single file diff >200 lines → dedicated Agent
- **Output**: Structured Markdown report with severity-graded issue table (CRITICAL/HIGH/MEDIUM/LOW), impact analysis, and summary
- **Constraint**: Read-only — never modifies source files or runs git write operations
