# Publishing to a Snowflake Stage

This guide walks through setting up a **stage-backed profile** so anyone on a Snowflake account
can pull the perspective without touching GitHub. This is the same pattern used by official
Snowflake profiles like `se_manager`.

## When to use this vs. GitHub

| Channel | Best for | Access governed by |
|---|---|---|
| **GitHub** (`source: "owner/repo/..."`) | Public/external partners, open-source teams, anyone with internet | Git credentials (public = none) |
| **Snowflake stage** (`snowflake_stage: "@DB.SCHEMA.STAGE/..."`) | Internal teams, enterprise accounts, role-restricted access | Snowflake roles + grants |

Both produce the same result: an activatable profile that pulls skills, MCP, and a system prompt.

## Step-by-step setup

### 1. Create the database + schema + stage

```sql
CREATE DATABASE IF NOT EXISTS CORTEX_CODE;
CREATE SCHEMA IF NOT EXISTS CORTEX_CODE.CONFIG;

CREATE STAGE IF NOT EXISTS CORTEX_CODE.CONFIG.STG_PROFILE_TEAM_PERSPECTIVE
  ENCRYPTION = (TYPE = 'SNOWFLAKE_SSE')
  COMMENT = 'Stage for team perspective profile: skills, system prompt, MCP config';
```

### 2. Upload the profile's files to the stage

From SnowSQL or any SQL client that supports PUT:

```sql
-- System prompt / persona
PUT 'file:///path/to/basic-example-of-profiles/AGENTS.md'
  @CORTEX_CODE.CONFIG.STG_PROFILE_TEAM_PERSPECTIVE/
  AUTO_COMPRESS=FALSE OVERWRITE=TRUE;

-- MCP server config
PUT 'file:///path/to/basic-example-of-profiles/mcp.json'
  @CORTEX_CODE.CONFIG.STG_PROFILE_TEAM_PERSPECTIVE/
  AUTO_COMPRESS=FALSE OVERWRITE=TRUE;

-- Skills (one PUT per skill's SKILL.md)
PUT 'file:///path/to/basic-example-of-profiles/skills/setup-my-perspective/SKILL.md'
  @CORTEX_CODE.CONFIG.STG_PROFILE_TEAM_PERSPECTIVE/skills/setup-my-perspective/
  AUTO_COMPRESS=FALSE OVERWRITE=TRUE;

PUT 'file:///path/to/basic-example-of-profiles/skills/getting-started/SKILL.md'
  @CORTEX_CODE.CONFIG.STG_PROFILE_TEAM_PERSPECTIVE/skills/getting-started/
  AUTO_COMPRESS=FALSE OVERWRITE=TRUE;
```

Verify:
```sql
LIST @CORTEX_CODE.CONFIG.STG_PROFILE_TEAM_PERSPECTIVE;
-- Should show: AGENTS.md, mcp.json, skills/getting-started/SKILL.md, skills/setup-my-perspective/SKILL.md
```

### 3. Create the profile registry table

The registry table is where `cortex profile publish` stores profile metadata so users can
discover and fetch profiles with `cortex profile add`.

```sql
CREATE TABLE IF NOT EXISTS CORTEX_CODE.CONFIG.PROFILE_REGISTRY (
  CONFIG_NAME       STRING NOT NULL,
  DESCRIPTION       STRING,
  OWNER_TEAM        STRING,
  SKILL_REPOS       VARIANT,
  MCP_SERVERS       VARIANT,
  COMMAND_REPOS     VARIANT,
  ENV_VARS          VARIANT,
  SETTINGS_OVERRIDES VARIANT,
  PERMISSIONS       VARIANT,
  HOOKS             VARIANT,
  PLUGINS           VARIANT,
  SUBAGENTS         VARIANT,
  SYSTEM_PROMPT_REPO VARIANT,
  VERSION           STRING DEFAULT '1.0',
  ACTIVE            BOOLEAN DEFAULT TRUE
);
```

### 4. Publish the profile

Using the `cortex` CLI:

```bash
cortex profile publish "team-perspective" \
  --from-file profile/team-perspective-stage.profile.json \
  --connection myconnection \
  --version "1.0" \
  --description "Starter team perspective from stage." \
  --owner-team "your-team"
```

On success you'll see:
```
✓ Profile 'team-perspective' published successfully!
Users can now run:
  cortex profile add team-perspective -c <connection>
  cortex --profile team-perspective
```

### 5. Verify it's visible

```bash
cortex profile list-remote --connection myconnection
```

Should show `team-perspective` in the remote profiles table.

### 6. Users add and activate it

**CLI:**
```bash
cortex profile add team-perspective -c myconnection
```

**Desktop UI:**
Agent Settings → Profiles → the profile appears in the remote list → Add → Activate.

## Grant access to other roles

By default only the role that created the objects can see them. To let others use the profile:

```sql
GRANT USAGE ON DATABASE CORTEX_CODE TO ROLE <target_role>;
GRANT USAGE ON SCHEMA CORTEX_CODE.CONFIG TO ROLE <target_role>;
GRANT SELECT ON TABLE CORTEX_CODE.CONFIG.PROFILE_REGISTRY TO ROLE <target_role>;
GRANT READ ON STAGE CORTEX_CODE.CONFIG.STG_PROFILE_TEAM_PERSPECTIVE TO ROLE <target_role>;
```

## Updating the profile

To push new versions of skills or the system prompt:
1. Re-PUT the updated files to the stage (with `OVERWRITE=TRUE`).
2. Optionally bump the version: re-run `cortex profile publish` with `--version "1.1"`.
3. Users run `cortex profile sync team-perspective -c myconnection` to pull the update.
