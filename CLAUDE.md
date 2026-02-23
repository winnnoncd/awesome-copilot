# CLAUDE.md — Awesome GitHub Copilot

This file provides guidance for AI assistants (Claude Code and similar) working in this repository. The authoritative human-facing documentation lives in [AGENTS.md](AGENTS.md) and [CONTRIBUTING.md](CONTRIBUTING.md).

---

## Project Overview

**Awesome GitHub Copilot** is a community-driven, content-first repository. There is no traditional application source code to compile or run. Instead, this repo is a curated collection of GitHub Copilot customization resources:

| Resource type | Directory | File pattern |
|---|---|---|
| Agents | `agents/` | `*.agent.md` |
| Prompts | `prompts/` | `*.prompt.md` |
| Instructions | `instructions/` | `*.instructions.md` |
| Skills | `skills/<name>/` | `SKILL.md` + optional assets |
| Hooks | `hooks/<name>/` | `README.md` + `hooks.json` |
| Plugins | `plugins/<name>/` | `.github/plugin/plugin.json` + `README.md` |

The build system (Node.js) reads these files and generates the top-level `README.md` and `.github/plugin/marketplace.json`. CI enforces that both are up to date.

---

## Repository Structure

```
awesome-copilot/
├── agents/                  # *.agent.md — GitHub Copilot custom agent definitions
├── prompts/                 # *.prompt.md — reusable task-specific prompts
├── instructions/            # *.instructions.md — coding standards applied to file patterns
├── skills/                  # folders: each skill is <name>/SKILL.md + optional assets
├── hooks/                   # folders: each hook is <name>/README.md + hooks.json
├── plugins/                 # folders: each plugin is <name>/.github/plugin/plugin.json + README.md
├── docs/                    # README.<type>.md — generated docs per resource type
├── eng/                     # Node.js build/automation scripts (.mjs)
├── scripts/                 # Utility shell scripts (e.g., fix-line-endings.sh)
├── cookbook/                # Practical copy-paste examples
├── website/                 # Vite-based documentation website
├── AGENTS.md                # Full project + workflow reference for AI agents
├── CONTRIBUTING.md          # Contribution guidelines
└── package.json             # npm scripts (the primary build interface)
```

---

## Essential Commands

Always run from the repository root.

```bash
# Install dependencies (first-time or after pulling)
npm ci

# Build: regenerates README.md and marketplace.json — run after ANY content change
npm run build

# Validate plugin manifests
npm run plugin:validate

# Validate skill manifests
npm run skill:validate

# Scaffold a new plugin
npm run plugin:create -- --name <plugin-name>

# Scaffold a new skill
npm run skill:create -- --name <skill-name> --description "<description>"

# Normalize line endings (CRLF → LF) — MUST run before every commit
bash scripts/fix-line-endings.sh
```

**What `npm run build` does:**
- Runs `eng/update-readme.mjs` → regenerates top-level `README.md` from all resource files
- Runs `eng/generate-marketplace.mjs` → regenerates `.github/plugin/marketplace.json`

---

## Critical Rules

1. **Run `npm run build` after every content change.** CI (`validate-readme.yml`) checks that the README is current and will fail if you forget.
2. **Run `bash scripts/fix-line-endings.sh` before every commit.** CI (`check-line-endings.yml`) rejects CRLF line endings.
3. **All PRs must target the `staged` branch, not `main`.** CI (`check-pr-target.yml`) enforces this.
4. **Description field values must be wrapped in single quotes** in all frontmatter.
5. **File and folder names must be lowercase with hyphens** (e.g., `my-new-agent.agent.md`, not `MyNewAgent.agent.md`).
6. **Skill `name` frontmatter must exactly match the folder name** (lowercase, hyphens, max 64 chars).

---

## File Naming Conventions

| Resource | Directory | Name format | Example |
|---|---|---|---|
| Agent | `agents/` | `<name>.agent.md` | `code-reviewer.agent.md` |
| Prompt | `prompts/` | `<name>.prompt.md` | `create-readme.prompt.md` |
| Instruction | `instructions/` | `<name>.instructions.md` | `typescript-style.instructions.md` |
| Skill | `skills/<name>/` | folder + `SKILL.md` | `skills/my-skill/SKILL.md` |
| Hook | `hooks/<name>/` | folder + `README.md` + `hooks.json` | `hooks/session-logger/` |
| Plugin | `plugins/<name>/` | folder + `.github/plugin/plugin.json` | `plugins/my-plugin/` |

---

## Frontmatter Reference

### Agent files (`*.agent.md`)

```yaml
---
description: 'Brief description of what the agent does'  # required
name: 'Human Readable Name'                              # strongly recommended
model: gpt-4.1                                           # strongly recommended
tools: ['codebase', 'terminalCommand', 'edit/editFiles'] # recommended
---
```

### Prompt files (`*.prompt.md`)

```yaml
---
agent: 'agent'                                           # required (agent | ask | Plan)
description: 'Brief description of the prompt'          # required, single quotes
name: 'Prompt Display Name'                              # recommended
model: gpt-4.1                                           # strongly recommended
tools: ['codebase', 'edit/editFiles']                    # recommended
---
```

### Instruction files (`*.instructions.md`)

```yaml
---
description: 'Brief description of the coding standard'  # required, single quotes
applyTo: '**.ts, **.tsx'                                 # required; glob patterns for target files
---
```

### Skill SKILL.md (`skills/<name>/SKILL.md`)

```yaml
---
name: skill-folder-name          # required; must match folder name; max 64 chars; lowercase hyphens
description: |                   # required; 10–1024 chars; can be multiline
  What this skill does.
---
```

### Hook README.md (`hooks/<name>/README.md`)

```yaml
---
name: 'Human Readable Hook Name'  # required
description: 'What it does'      # required, single quotes
tags: ['logging', 'audit']       # optional; array of lowercase-hyphenated strings
---
```

### Plugin manifest (`plugins/<name>/.github/plugin/plugin.json`)

```json
{
  "name": "plugin-folder-name",
  "description": "What this plugin does",
  "version": "1.0.0",
  "keywords": ["tag-one", "tag-two"],
  "agents": ["../../agents/my-agent.agent.md"],
  "commands": ["../../prompts/my-prompt.prompt.md"],
  "skills": ["../../skills/my-skill/"]
}
```

---

## Workflow: Adding New Resources

### Agent, Prompt, or Instruction

1. Create the file in the appropriate directory with correct frontmatter.
2. `npm run build` — updates README.
3. `bash scripts/fix-line-endings.sh` — normalize line endings.
4. Verify your resource appears in the relevant `docs/README.<type>.md`.

### Skill

1. `npm run skill:create -- --name <name> --description "<desc>"` — scaffolds `skills/<name>/SKILL.md`.
2. Edit `SKILL.md` with your instructions.
3. Add bundled assets (scripts, templates, data files) to the skill folder; reference them in SKILL.md.
   - Keep each asset under 5 MB.
4. `npm run skill:validate` — check structure.
5. `npm run build` — update README.
6. `bash scripts/fix-line-endings.sh`.

### Hook

1. Create folder: `hooks/<name>/`.
2. Create `README.md` with frontmatter (`name`, `description`, optional `tags`).
3. Create `hooks.json` following the [GitHub Copilot hooks spec](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/coding-agent/use-hooks).
4. Add any scripts and make them executable: `chmod +x script.sh`.
5. `npm run build` — update README.
6. `bash scripts/fix-line-endings.sh`.

### Plugin

1. `npm run plugin:create -- --name <plugin-name>` — scaffolds the plugin directory.
2. Edit `plugins/<name>/.github/plugin/plugin.json` with metadata and resource references.
   - Resources are referenced as relative paths to existing top-level files.
   - CI materializes them into the plugin — do not duplicate source files inside the plugin folder.
3. `npm run plugin:validate` — check structure.
4. `npm run build` — update README and marketplace.json.
5. `bash scripts/fix-line-endings.sh`.

---

## CI/CD Checks

All of these run automatically on pull requests. Fix failures before merging.

| Workflow | What it checks |
|---|---|
| `check-line-endings.yml` | Rejects files with CRLF line endings |
| `check-plugin-structure.yml` | Validates plugin manifest structure and referenced files |
| `check-pr-target.yml` | Enforces PRs target `staged`, not `main` |
| `validate-readme.yml` | Ensures `npm run build` was run and README is up to date |
| `codespell.yml` | Spell checking across all markdown files |
| `contributors.yml` | Updates contributor list |

---

## Pre-Commit Checklist

Before committing any change:

- [ ] `npm run build` completed without errors
- [ ] `bash scripts/fix-line-endings.sh` was run
- [ ] All new files have correct frontmatter with required fields
- [ ] File/folder names are lowercase with hyphens
- [ ] Description values are wrapped in single quotes
- [ ] Skill `name` field matches its folder name exactly
- [ ] Plugin does not reference non-existent files
- [ ] PR will target `staged` branch (not `main`)

---

## Tech Stack

- **Build tooling**: Node.js 18+, npm; scripts are ES modules in `eng/*.mjs`
- **Key dependencies**: `js-yaml`, `vfile`, `vfile-matter` (frontmatter parsing)
- **Dev dependencies**: `all-contributors-cli`
- **Website**: Vite-based app in `website/`
- **MCP server**: Docker image `ghcr.io/microsoft/mcp-dotnet-samples/awesome-copilot:latest`

---

## Further Reading

- [AGENTS.md](AGENTS.md) — complete project overview, full field-by-field checklists for each resource type
- [CONTRIBUTING.md](CONTRIBUTING.md) — contribution guidelines and code of conduct link
- [.github/copilot-instructions.md](.github/copilot-instructions.md) — code review instructions (for agents performing reviews)
- [docs/](docs/) — generated documentation per resource type
- [cookbook/README.md](cookbook/README.md) — practical examples
