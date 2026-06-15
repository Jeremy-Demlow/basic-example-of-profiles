# Customize this for your team

This repo is a working example. To make it *yours*, fork it and change a handful of
placeholders. Nothing here contains secrets, so your fork can stay public or be private —
your choice.

## 1. Fork the repo

Fork `Jeremy-Demlow/basic-example-of-profiles` into your own org/account, e.g.
`your-org/your-team-perspective`.

## 2. Replace placeholders

Search the repo for these markers and replace them:

| Placeholder | Where | Replace with |
|---|---|---|
| `Jeremy-Demlow/basic-example-of-profiles` | `profile/team-perspective.profile.json` (every `source`) | your fork's `owner/repo` |
| `REPLACE_WITH_YOUR_TEAM` | `profile/...json` `ownerTeam` | your team name |
| `REPLACE_WITH_YOUR_CONNECTION` | `profile/...json` `settingsOverrides` | your default Snowflake connection name |
| `@YOUR_DB.YOUR_SCHEMA.YOUR_STAGE/...` | `profile/team-perspective-stage.profile.json` (every `snowflake_stage`) | your Snowflake stage path (only if you use the stage channel) |
| `REPLACE-WITH-YOUR-COMPANY.glean.com` | `mcp.json` | your knowledge-search MCP URL (or delete the server) |
| `REPLACE-WITH-YOUR-MCP-PACKAGE` | `mcp.json` | your MCP server package (or delete the server) |

If you renamed the default branch, update each `ref` (defaults to `main`).

## 3. Make it sound like your team

- **`AGENTS.md`** — the persona/system prompt. Rewrite the voice for your audience.
- **`skills/`** — add your own starter skills (each is a folder with a `SKILL.md`).
- **`agents/`** — add subagents (each is a `.md` with `name` + `description` frontmatter).
  The included `agents/data-helper.md` (a read-only, plain-language data helper) ships with
  the plugin and is auto-discovered when the repo is installed as a plugin. To bind a subagent
  explicitly to a profile, add its name to the profile's `subagents` array.
- **`mcp.json`** — add the MCP servers your team actually uses; secrets via `${input:...}`.
  The two examples are: `team_knowledge_search` (a remote knowledge/wiki search over
  `npx mcp-remote`, e.g. Glean) and `example_api_server` (a generic template showing the
  `${input:...}` secret-prompt pattern). Replace or delete them.
- **`examples/hooks.example.json`** — copy to a root `.hooks.json` if you want event hooks.

## 4. Tell your team how to load it

Two paths — pick what fits your audience:

**A. One-click profile (cleanest).** A teammate copies
`profile/<your>.profile.json` into `~/.snowflake/cortex/profiles/`, then activates it in
Agent Settings → Profiles. Because every source points at your fork on GitHub, activation
pulls skills + MCP + persona automatically.

**B. Guided setup skill (friendliest for beginners).** A teammate adds your fork from
Agent Settings → Skills → Add from GitHub (`your-org/your-team-perspective`), then runs
`/setup-profile`, which installs and personalizes the profile for them.

## 5. Verify

Activate the profile, start a new chat, and run `/getting-started`. Confirm the persona,
skills, and (filled-in) MCP servers all show up.
