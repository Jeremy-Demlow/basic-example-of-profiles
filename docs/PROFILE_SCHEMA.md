# Profile Schema (GitHub-sourced)

A **profile** is a single JSON file in `~/.snowflake/cortex/profiles/<Name>.json` that, when
activated in Agent Settings → Profiles, switches on a whole "perspective": which skills, MCP
servers, system prompt, and settings are active. Everything can be sourced from **GitHub** — no
Snowflake stage required.

> This page documents the GitHub source form, which the official docs don't yet cover. It was
> verified against the Cortex Code Desktop app itself.

## Full shape

```jsonc
{
  "name": "Team Perspective",
  "description": "...",
  "ownerTeam": "your-team",
  "version": "1.0",

  // Skills: clones the repo subfolder and registers every SKILL.md it finds.
  "skillRepos": [
    { "source": "owner/repo/skills", "ref": "main" }
  ],

  // MCP servers: point at an mcp.json in the repo (or inline the object, or use a stage).
  "mcpServers": { "source": "owner/repo/mcp.json", "ref": "main" },

  // (Slash) command repos — same repo-entry shape as skillRepos.
  "commandRepos": [],

  // System prompt / persona: point at a markdown file in the repo.
  "systemPromptRepo": { "source": "owner/repo/AGENTS.md", "ref": "main" },

  "plugins": [],                 // can also bind GitHub/local plugins
  "subagents": [],
  "hooks": {},
  "envVars": {},

  // Applied to settings on activation (theme, default connection, etc.)
  "settingsOverrides": { "theme": "dark", "sqlConnectionName": "myconn" },

  "localModified": false
}
```

## The repo entry

`skillRepos`, `commandRepos`, `systemPromptRepo`, and `mcpServers` all accept a **repo entry**:

| Field | Required | Meaning |
|---|---|---|
| `source` | yes* | `owner/repo`, optionally with a `/subpath` |
| `ref` | no | Branch, tag, or commit. Defaults to `main`. |
| `local` | no | If `true`, `source` is a local filesystem path instead of a repo. |
| `sparse` | no | If `true` (with a subpath), sparse-checkout only that subpath. |
| `url` | no | Explicit clone URL, used if `source` can't be parsed. |
| `snowflake_stage` | yes* | Alternative to `source`: a `@DB.SCHEMA.STAGE/path`. (Not used in this repo.) |

\* Provide **either** `source` (GitHub/local) **or** `snowflake_stage`.

### Accepted `source` string formats

- `owner/repo`
- `owner/repo/path/to/subfolder`
- `owner/repo@branch` or `owner/repo#branch` (inline ref)
- `https://github.com/owner/repo`
- `git@github.com:owner/repo`
- `github:owner/repo` (the `github:` prefix is stripped)

`mcpServers` and `systemPromptRepo` point at a **file** (`.../mcp.json`, `.../AGENTS.md`); the
others point at a **folder**.

## How activation resolves it

On activate, the app clones each `source` at its `ref` into a cache, appends the subpath, and:
- registers skills found under the skill folders,
- loads the MCP `mcp.json` (prompting for any `${input:...}` secrets on first use),
- applies the system prompt file as the active persona,
- applies `settingsOverrides`.

Switching to a different profile (or deleting the file) reverses all of it.
