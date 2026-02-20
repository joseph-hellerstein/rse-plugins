# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repository Is

A collection of Claude Code plugins (agents, skills, and commands) for Research Software Engineering (RSE) and scientific computing. Plugins are published to the Claude Code plugin marketplace and used by researchers and scientific software developers.

## Validation

The CI pipeline validates all plugins. To run validation locally:

```bash
# Validate all JSON files
find . -name "*.json" -not -path "./.git/*" | xargs -I {} sh -c 'jq . "{}" > /dev/null && echo "OK: {}" || echo "FAILED: {}"'

# Validate all YAML frontmatter (agents and skills have YAML frontmatter)
find . -name "*.yml" -not -path "./.git/*" | xargs -I {} sh -c 'python3 -c "import yaml,sys; yaml.safe_load(sys.stdin)" < "{}" && echo "OK: {}" || echo "FAILED: {}"'
```

CI also checks that every plugin directory under `plugins/` and `community-plugins/` contains both `.claude-plugin/plugin.json` and `README.md`.

## Plugin Architecture

```
plugins/
  <plugin-name>/
    .claude-plugin/plugin.json   # Plugin metadata (name, description, version, keywords, license)
    agents/                      # AI personas (.md files with YAML frontmatter + prompt)
    skills/                      # Reusable knowledge modules
      <skill-name>/
        SKILL.md                 # Skill entrypoint with YAML frontmatter
        assets/                  # Templates and examples referenced by the skill
        references/              # Detailed reference docs (PATTERNS.md, EXAMPLES.md, etc.)
    commands/                    # Slash commands (used in workflow-style plugins)
    README.md
community-plugins/               # Community-contributed plugins (same structure)
.claude-plugin/marketplace.json  # Root marketplace registry listing all plugins
```

## File Formats

**Agent files** (`agents/<name>.md`): YAML frontmatter followed by the agent prompt in Markdown.
```yaml
---
name: agent-name           # kebab-case
description: |
  When to invoke this agent (written for the Claude Code agent selector)
examples:
  - context: "..."
    user: "..."
    assistant: "..."
model: inherit
color: green               # One of: red, orange, yellow, green, blue, purple, pink, cyan
skills:
  - skill-name
---
[Full agent system prompt in Markdown]
```

**Skill files** (`skills/<name>/SKILL.md`): YAML frontmatter followed by skill documentation.
```yaml
---
name: skill-name
description: When this skill should be invoked (written for the agent selector)
---
[Comprehensive documentation: decision trees, patterns, examples]
```

**Command files** (`commands/<name>.md`): YAML frontmatter with description, followed by the command instructions.
```yaml
---
description: What this command does
---
[Step-by-step instructions for the command]
```

**Plugin metadata** (`.claude-plugin/plugin.json`): JSON with `name`, `version`, `description`, `author`, `license`, `keywords`, `agents`, `skills` arrays listing the files in the plugin.

## Conventions

- Agent and skill filenames: **kebab-case** (e.g., `scientific-python-expert.md`)
- Agent `description` fields are written from the perspective of Claude Code's agent routing — they describe *when* to use the agent, not just what it does
- Skills are linked to agents via the `skills:` list in the agent frontmatter; skill names must match the directory name under `skills/`
- The marketplace registry (`.claude-plugin/marketplace.json`) must be updated when adding a new plugin
- All plugins use BSD-3-Clause license

## Adding a New Plugin

1. Create `plugins/<plugin-name>/` with the standard structure above
2. Add `.claude-plugin/plugin.json` and `README.md` (required by CI)
3. Register the plugin in `.claude-plugin/marketplace.json`
4. Follow the agent template structure in `CONTRIBUTING.md`: Expertise section, When to Use section, then the agent prompt
