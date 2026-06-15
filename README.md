# Cortex Code Desktop — Profile Starter Kit

**Set up your entire team's Cortex Code Desktop experience in one step.**

Cortex Code Desktop is Snowflake's agentic IDE. A **profile** is how you configure what the
assistant knows, how it talks, and what tools it has — for a whole team at once. This repo is
a working, forkable template that gives you a complete profile ready to distribute.

## What a profile does

When someone activates a profile, Cortex Code Desktop automatically:

- Loads **skills** (reusable workflows the assistant can run — like slash commands)
- Connects **MCP servers** (external tools the assistant can use — knowledge bases, APIs, etc.)
- Applies a **system prompt** (the assistant's personality, tone, and behavior rules)
- Sets **defaults** (theme, Snowflake connection, permission mode)

One profile replaces dozens of manual configuration steps. Your team members activate it once
and they're ready to work.

## What's in this repo

```
profile/
  team-perspective.profile.json           ← GitHub-sourced profile
  team-perspective-stage.profile.json     ← Snowflake stage-sourced profile (same content)

AGENTS.md                                 ← System prompt (persona / behavior rules)
mcp.json                                  ← MCP server templates (secrets never committed)

skills/
  setup-my-perspective/SKILL.md           ← Guided setup: asks one question, installs everything
  getting-started/SKILL.md                ← Friendly tour for first-time users

agents/data-helper.md                     ← Example subagent (read-only data helper)
.cortex-plugin/plugin.json                ← Plugin manifest (makes this installable as one bundle)
examples/hooks.example.json               ← Optional automation hooks (not active by default)

docs/
  PROFILE_SCHEMA.md                       ← Full profile JSON schema reference
  STAGE_SETUP.md                          ← How to publish to Snowflake (internal distribution)
  CUSTOMIZE.md                            ← Fork-and-fill guide for your team
```

## Quick start

### Option A — GitHub (public teams, no Snowflake stage needed)

Your team members do this in Cortex Code Desktop:

1. **Agent Settings → Skills → + (GitHub Skills)** → paste:
   ```
   YOUR-ORG/YOUR-FORK-OF-THIS-REPO
   ```
2. In a chat, type: **`/setup-my-perspective`**
3. Answer one question (their name), then follow the activation prompt.

Done. Skills, MCP, persona, and settings are all live.

### Option B — Snowflake stage (internal teams, role-restricted)

You publish the profile to your Snowflake account once (see [`docs/STAGE_SETUP.md`](docs/STAGE_SETUP.md)),
then team members run:

```bash
cortex profile add team-perspective -c <their-connection>
```

Or in the desktop UI: **Agent Settings → Profiles** → find it in the remote list → **Add → Activate**.

## Two distribution channels

| | GitHub | Snowflake Stage |
|---|---|---|
| Best for | Public / external / open-source teams | Internal / enterprise / restricted access |
| Access control | Git permissions (public = anyone) | Snowflake roles and grants |
| How users find it | You give them the repo URL | `cortex profile list-remote` shows it |
| Profile file | `profile/team-perspective.profile.json` | `profile/team-perspective-stage.profile.json` |
| Update flow | Push to GitHub; users sync in Agent Settings | Re-PUT files to stage; users run `cortex profile sync` |

Both channels deliver the same result: skills + MCP + persona + settings, activated in one click.

## Restricting access by role

Profiles published to a Snowflake stage are governed by standard Snowflake grants. To make a
profile visible **only to certain roles**:

```sql
-- Grant access to a specific role
GRANT USAGE ON DATABASE CORTEX_CODE TO ROLE DATA_ANALYSTS;
GRANT USAGE ON SCHEMA CORTEX_CODE.CONFIG TO ROLE DATA_ANALYSTS;
GRANT SELECT ON TABLE CORTEX_CODE.CONFIG.PROFILE_REGISTRY TO ROLE DATA_ANALYSTS;
GRANT READ ON STAGE CORTEX_CODE.CONFIG.STG_PROFILE_TEAM_PERSPECTIVE TO ROLE DATA_ANALYSTS;

-- Revoke from PUBLIC to keep it hidden from everyone else
REVOKE ALL ON TABLE CORTEX_CODE.CONFIG.PROFILE_REGISTRY FROM ROLE PUBLIC;
REVOKE ALL ON STAGE CORTEX_CODE.CONFIG.STG_PROFILE_TEAM_PERSPECTIVE FROM ROLE PUBLIC;
```

Users without these grants won't see the profile in `cortex profile list-remote` and can't
fetch it. You can create **multiple profiles for different roles** — each with its own stage,
own skills, and own persona — and gate them independently.

## How to make this yours

1. **Fork this repo** into your org.
2. **Edit these files:**

   | File | What to change |
   |---|---|
   | `AGENTS.md` | Rewrite the persona for your team's voice and use case |
   | `mcp.json` | Replace placeholder servers with your real MCP endpoints |
   | `profile/team-perspective.profile.json` | Change `source` fields from `Jeremy-Demlow/basic-example-of-profiles` to your fork's `owner/repo` |
   | `skills/` | Add your own slash-command skills |

3. **Tell your team** to add the repo (Option A) or publish to a stage (Option B).

Full fork guide: [`docs/CUSTOMIZE.md`](docs/CUSTOMIZE.md)

## Profile schema reference

The profile JSON format is documented in [`docs/PROFILE_SCHEMA.md`](docs/PROFILE_SCHEMA.md),
including:
- The GitHub source form (`"source": "owner/repo/path"`)
- The Snowflake stage form (`"snowflake_stage": "@DB.SCHEMA.STAGE/path"`)
- The registry table schema (for publishing)
- All accepted source string formats

## Key concepts

- **Profile** = the activatable configuration (binds skills + MCP + persona + settings).
- **Plugin** = a distributable package of capabilities (this repo is structured as one).
- **Skill** = a markdown file (`SKILL.md`) that teaches the assistant a specific workflow.
- **MCP server** = an external tool connection (knowledge search, APIs, databases).
- **System prompt** = `AGENTS.md` — controls *how* the assistant communicates (tone, safety rules).

A profile *references* plugins, skills, and MCP servers. It doesn't contain them inline — it
points at where they live (GitHub or a stage) and the app pulls them on activation.

## Safety

- The assistant defaults to **ask-before-acting** mode. Nothing runs without explicit approval.
- MCP secrets use `${input:...}` prompts — entered once, stored encrypted by the OS, never in
  this repo.
- Profiles are not cryptographically signed. Review what you're installing before activating,
  the same way you'd review any code from an author you choose to trust.
- Everything is reversible: switch profiles or delete the profile file to undo all changes.
