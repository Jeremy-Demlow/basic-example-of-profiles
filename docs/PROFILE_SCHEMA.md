# Profile Schema (GitHub + Stage)

A **profile** is a single JSON file in `~/.snowflake/cortex/profiles/<Name>.json` that, when
activated in Agent Settings → Profiles, switches on a whole "perspective": which skills, MCP
servers, system prompt, and settings are active.

Sources can be **GitHub** (public, no stage) or **Snowflake stage** (internal, role-governed).

> This page documents both source forms. Neither is in the public docs at time of writing —
> both were verified against the Cortex Code Desktop app itself.

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

## Stage-sourced form

The same profile shape, but with `snowflake_stage` instead of `source`:

```jsonc
{
  "name": "team-perspective",
  "description": "...",
  "ownerTeam": "your-team",
  "version": "1.0",

  "skillRepos": [
    { "snowflake_stage": "@CORTEX_CODE.CONFIG.STG_PROFILE_TEAM_PERSPECTIVE/skills/" }
  ],

  "mcpServers": {
    "snowflake_stage": "@CORTEX_CODE.CONFIG.STG_PROFILE_TEAM_PERSPECTIVE/mcp.json"
  },

  "systemPromptRepo": {
    "snowflake_stage": "@CORTEX_CODE.CONFIG.STG_PROFILE_TEAM_PERSPECTIVE/AGENTS.md"
  },

  // ... same remaining fields as GitHub form
}
```

Stage paths must match `@DATABASE.SCHEMA.STAGE` or `@DATABASE.SCHEMA.STAGE/path`.

## Profile Registry table

When you `cortex profile publish`, the profile is stored in a registry table
(`CORTEX_CODE.CONFIG.PROFILE_REGISTRY` by default) so others can discover it via
`cortex profile list-remote` and fetch it with `cortex profile add`.

### Schema (14 columns)

```sql
CREATE TABLE CORTEX_CODE.CONFIG.PROFILE_REGISTRY (
  CONFIG_NAME        STRING NOT NULL,   -- slug: letters, numbers, hyphens, underscores only
  DESCRIPTION        STRING,
  OWNER_TEAM         STRING,
  SKILL_REPOS        VARIANT,           -- JSON array of repo entries
  MCP_SERVERS        VARIANT,           -- JSON repo entry or inline object
  COMMAND_REPOS      VARIANT,           -- JSON array of repo entries
  ENV_VARS           VARIANT,           -- JSON object
  SETTINGS_OVERRIDES VARIANT,           -- JSON object
  PERMISSIONS        VARIANT,           -- JSON object (nullable)
  HOOKS              VARIANT,           -- JSON object (nullable)
  PLUGINS            VARIANT,           -- JSON array (nullable)
  SUBAGENTS          VARIANT,           -- JSON array (nullable)
  SYSTEM_PROMPT_REPO VARIANT,           -- JSON repo entry (nullable)
  VERSION            STRING DEFAULT '1.0',
  ACTIVE             BOOLEAN DEFAULT TRUE
);
```

### Important constraints

- **`CONFIG_NAME` must be a slug** — starts with a letter, contains only `[A-Za-z0-9_-]`.
  Spaces are not allowed (unlike local profile filenames). Use hyphens: `team-perspective`.
- **ACTIVE** — only rows with `ACTIVE = TRUE` are returned by `list-remote` / `add`.
  Use `cortex profile publish --deactivate` to hide a profile without deleting the row.

> **Versioning note:** the plugin's `version` (in `.cortex-plugin/plugin.json`) and a profile's
> `version` field are **independent** — they describe different things (the plugin package vs. the
> profile definition) and do not need to match.

See [`STAGE_SETUP.md`](STAGE_SETUP.md) for the full setup walkthrough.
