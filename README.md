# agent-skills

A collection of reusable agent skill files for AI coding assistants. Each skill lives in its own folder and contains a `SKILL.md` with domain-specific instructions the agent follows when invoked.

## Skills

| Skill | Folder | Description |
|---|---|---|
| Changelog Maintenance | `plugins/common/skills/changelog-maintenance/` | Keep a root `CHANGELOG.md` grouped by date headings, bootstrapped from git history when missing |
| FreeCAD Project Setup | `free-cad-project-setup/` | Project structure, params.py conventions, part files, assembly, export, and print orientation for FreeCAD Python projects |
| FreeCAD Visual Validation | `freecad-visual-validation/` | MCP-based visual validation procedure — required views, what to check, and defect handling |
| FreeCAD Threading | `freecad-threading/` | FDM-printable thread construction using `makeHelix` + `makePipeShell`, clearances, male/female patterns |
| FreeCAD Fits & Tolerances | `freecad-fits-tolerances/` | Sliding, rotating, press/glue fits, snap tabs, and hinge knuckle construction for FDM printing |

---

## Installation

### Plugin Marketplace (GitHub Copilot CLI & Claude Code)

This repo is a plugin marketplace. Register it once, then install any plugin by name.

**Register the marketplace:**

```sh
# GitHub Copilot CLI
copilot plugin marketplace add intelligentmachine/agent-plugins

# Claude Code
claude plugin marketplace add intelligentmachine/agent-plugins
```

**Install the FreeCAD plugin:**

```sh
# GitHub Copilot CLI
copilot plugin install freecad@agent-plugins

# Claude Code
claude plugin install freecad@agent-plugins
```

**Install the common workflow plugin:**

```sh
# GitHub Copilot CLI
copilot plugin install common@agent-plugins

# Claude Code
claude plugin install common@agent-plugins
```

**Browse available plugins:**

```sh
copilot plugin marketplace browse agent-plugins
```

---

### Manual installation (GitHub Copilot CLI / Claude Code)

Install directly from the repository without registering the marketplace:

```sh
# Install the freecad plugin directly from the repo subdirectory
copilot plugin install intelligentmachine/agent-plugins:plugins/freecad

# Or from a local clone
copilot plugin install ./plugins/freecad
```

---

### GitHub Copilot (VS Code)

Skills are loaded via `.github/copilot-instructions.md` or VS Code instruction files (`.instructions.md`).

1. Copy the skill folder(s) you want into your project, or reference them from a shared location.
2. In your project root create (or append to) `.github/copilot-instructions.md`:

   ```markdown
   When setting up a FreeCAD project, follow the instructions in:
   free-cad-project-setup/SKILL.md
   ```

   Alternatively, use a VS Code workspace instruction file at `.vscode/<name>.instructions.md` with a `applyTo` glob:

   ```markdown
   ---
   applyTo: "**/*.py"
   ---
   Follow the FreeCAD project conventions defined in free-cad-project-setup/SKILL.md.
   ```

3. In chat, reference the skill explicitly: **"Follow the free-cad-project-setup skill"** or configure the skill path in your Copilot agent settings so it is loaded automatically.

### Claude Code

Claude Code reads instructions from `CLAUDE.md` at the project root (and optionally nested `CLAUDE.md` files in sub-folders).

1. Copy the desired skill folder(s) into your project.
2. Add a reference in your project's `CLAUDE.md`:

   ```markdown
   ## FreeCAD Projects
   When working on FreeCAD Python code, read and follow:
   - `free-cad-project-setup/SKILL.md` — project structure and part conventions
   - `freecad-threading/SKILL.md` — thread construction
   - `freecad-visual-validation/SKILL.md` — visual validation after every change
   - `freecad-fits-tolerances/SKILL.md` — fits and tolerances
   ```

3. Claude Code will automatically load `CLAUDE.md` on startup and apply the referenced skill instructions during the session.

---

## Repository layout

```
agent-plugins/
├── .github/plugin/
│   └── marketplace.json          # GitHub Copilot CLI marketplace manifest
├── .claude-plugin/
│   └── marketplace.json          # Symlink → .github/plugin/marketplace.json (Claude Code)
└── plugins/
    ├── common/
    │   ├── plugin.json
    │   ├── .claude-plugin/
    │   │   └── plugin.json
    │   └── skills/
    │       └── changelog-maintenance/
    │           └── SKILL.md
    └── freecad/
        ├── plugin.json           # Plugin manifest
        ├── .claude-plugin/
        │   └── plugin.json       # Claude Code plugin manifest (mirrors plugin.json)
        ├── .mcp.json             # MCP server config (FreeCAD HTTP server)
        ├── agents/
        │   ├── freecad-project.agent.md
        │   └── freecad-visual-validation.agent.md
        └── skills/
            ├── freecad-fits-tolerances/
            │   └── SKILL.md
            └── freecad-threading/
                └── SKILL.md
```

## Example usage

For manual pre-merge changelog updates in any project, invoke the shared skill explicitly:

```text
Use the changelog-maintenance skill from the common plugin before merging.
```

The skill keeps `CHANGELOG.md` at the project root, groups entries by `YYYY-MM-DD`, and bootstraps the file from git commits if it does not exist yet.
