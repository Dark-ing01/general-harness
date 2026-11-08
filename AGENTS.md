# AGENTS.md

You are the coding assistant of Darking, a Software Engineer.
Call the human Dark everytime you communicate with him please.

This is a common repo to store general harness configurations like skills, mcps configurations, Tools, multiagents workflows, etc.
Review all agent configurations and make sure that all of them are installed in the global agent config.


## Project Architecture

This repository (`general-harness`) serves as the **source-of-truth library** for all reusable Pi Coding Agent resources. It is designed to be the single place where skills, MCP servers, plugins, workflows, and other agent configurations are authored, versioned, and maintained.

The runtime agent configuration lives in the **global agent directory** at `~/.pi/agent/`. This repository's resources must be **installed/linked** into that global directory for the agent to use them.

```
┌─────────────────────────────────────────────────────────────────────┐
│                    general-harness (this repo)                      │
│  /home/dark/Documents/ai-workflows/general-harness/                 │
│  ├── AGENTS.md          ← This file (agent instructions)            │
│  ├── README.md          ← Project overview                          │
│  ├── mcps/              ← MCP server configurations (JSON/YAML)     │
│  ├── plugins/           ← Pi extensions (TypeScript/JS)             │
│  │   └── pi/            ← Pi-specific plugin entry points           │
│  ├── workflows/         ← Multi-agent workflow definitions          │
│  └── skills/            ← Portable skill directories (SKILL.md)     │
└─────────────────────────────────────────────────────────────────────┘
                              │
                              ▼  (install / symlink / copy)
┌─────────────────────────────────────────────────────────────────────┐
│              Global Agent Config (~/.pi/agent/)                     │
│  ~/.pi/agent/                                                       │
│  ├── settings.json        ← User settings (models, tools, etc.)     │
│  ├── mcp.json             ← MCP servers available in every project  │
│  ├── auth.json            ← API keys / OAuth credentials            │
│  ├── AGENTS.override.md   ← Global agent instructions (optional)    │
│  ├── extensions/          ← Installed extensions (symlinked here)   │
│  ├── skills/              ← Installed skills (symlinked here)       │
│  ├── prompts/             ← Installed prompt templates              │
│  ├── themes/              ← Custom themes                           │
│  ├── packages/            ← Pi packages (npm/git/local)             │
│  ├── projects-memory/     ← Per-project memory & skills             │
│  │   └── general-harness/ ← This project's project-scoped memory   │
│  ├── pi-hermes-memory/    ← Agent session & skill memory            │
│  ├── install/             ← Pi binary releases & managed installs   │
│  ├── npm/                 ← npm package cache for Pi packages       │
│  └── bin/                 ← CLI binaries (pi, rg, etc.)             │
└─────────────────────────────────────────────────────────────────────┘
```

### Resource Types in This Repository

| Directory | Purpose | Installation Target |
|-----------|---------|---------------------|
| `mcps/` | MCP server definitions (JSON) | `~/.pi/agent/mcp.json` (merged) |
| `plugins/` | Pi extensions (TypeScript/JS) | `~/.pi/agent/extensions/` |
| `plugins/pi/` | Pi-specific plugin entry points | `~/.pi/agent/extensions/` |
| `workflows/` | Multi-agent workflow definitions | `~/.pi/agent/extensions/` or project `.pi/extensions/` |
| `skills/` | Portable skill directories with `SKILL.md` | `~/.pi/agent/skills/` |

> **Note**: Currently the `mcps/`, `plugins/`, `workflows/`, and `skills/` directories are empty placeholders. Resources should be added here as needed, then installed to the global config.


## Mandatory Verification Protocol

**When the user explicitly requests it (e.g., "verify resources", "check installation", "sync harness"), the coding agent MUST verify that all resources defined in this `general-harness` repository are correctly installed and available in the global agent configuration (`~/.pi/agent/`).**

### Verification Steps

1. **Inventory this repo's resources**
   - List all files/directories under `mcps/`, `plugins/`, `workflows/`, `skills/`
   - For skills: each subdirectory containing a `SKILL.md` is one skill
   - For plugins: each `.ts`/`.js` file or directory with `package.json` is one plugin
   - For MCPs: each `.json` file defines MCP servers to merge into `mcp.json`
   - For workflows: each workflow definition file

2. **Check global agent config**
   - Read `~/.pi/agent/settings.json` → check `extensions`, `skills`, `prompts`, `themes`, `packages` arrays
   - Read `~/.pi/agent/mcp.json` → check MCP server definitions
   - List `~/.pi/agent/extensions/` → verify plugin files/dirs exist
   - List `~/.pi/agent/skills/` → verify skill directories exist
   - List `~/.pi/agent/projects-memory/general-harness/skills/` → verify project-scoped skills

3. **Report discrepancies**
   - Missing resources (in repo but not in global config)
   - Orphaned resources (in global config but not in repo)
   - Version mismatches (if applicable)

4. **Remediation guidance**
   - For missing skills: symlink or copy skill directory to `~/.pi/agent/skills/`
   - For missing plugins: symlink or copy to `~/.pi/agent/extensions/`
   - For MCPs: merge into `~/.pi/agent/mcp.json`
   - For workflows: install as extensions or project-scoped resources
   - Run `/reload` in the agent after changes

### Example Verification Command Sequence

```bash
# 1. Inventory repo resources
find /home/dark/Documents/ai-workflows/general-harness -type f \( -name "*.json" -o -name "*.ts" -o -name "*.js" -o -name "SKILL.md" \) -not -path "*/node_modules/*" -not -path "*/.git/*"

# 2. Check global config
cat ~/.pi/agent/settings.json | jq '.extensions, .skills, .prompts, .themes, .packages'
cat ~/.pi/agent/mcp.json
ls -la ~/.pi/agent/extensions/
ls -la ~/.pi/agent/skills/
ls -la ~/.pi/agent/projects-memory/general-harness/skills/
```

---

## Agent Behavior Rules

1. **Always address the user as "Dark"**
2. **This repo is the source of truth** — global config must reflect what's here
3. **On explicit verification request**: perform the full verification protocol above
4. **When adding new resources** to this repo, remind the user to install them to global config
5. **Use relative paths** from this repo root when referencing files
6. **Prefer symlinks** over copies for development (so changes are immediate)
7. **Run `/reload`** after modifying global config to pick up changes


## Quick Reference: Global Config Paths

| Resource | Global Path | This Repo Source |
|----------|-------------|------------------|
| Settings | `~/.pi/agent/settings.json` | N/A (managed separately) |
| MCP Servers | `~/.pi/agent/mcp.json` | `mcps/*.json` |
| Extensions | `~/.pi/agent/extensions/` | `plugins/`, `plugins/pi/` |
| Skills | `~/.pi/agent/skills/` | `skills/*/SKILL.md` |
| Prompts | `~/.pi/agent/prompts/` | (not yet in this repo) |
| Themes | `~/.pi/agent/themes/` | (not yet in this repo) |
| Packages | `~/.pi/agent/npm/` + `settings.json` | (not yet in this repo) |
| Project Memory | `~/.pi/agent/projects-memory/general-harness/` | Auto-managed |


## Related Documentation

- [Pi Skills Spec](https://agentskills.io/specification) — Skill directory structure
- [Pi Configuration](configuration.md) — Config file locations and precedence
- [Pi Extensions](extensions.md) — Writing TypeScript extensions
- [MCP Servers](mcp.md) — Model Context Protocol integration
- [Pi Packages](packages.md) — Distributing skills/extensions via npm/git
