# How We Figured This Out

This document captures the discovery process used to verify the profile schema and build this
repo. None of this was in the public Snowflake documentation at time of writing (June 2026) —
it was reverse-engineered from the Cortex Code Desktop app's own source code and then tested
end-to-end.

## The problem

The official docs cover **skills**, **plugins**, and **MCP servers** well, but **profiles** —
the primitive that *binds them all together and makes them activatable* — have no published
schema reference. The docs show stage-backed profiles exist (Agent Settings → Profiles) but
don't document:

- What JSON keys a profile supports
- How to reference GitHub (instead of a Snowflake stage) as a source
- The exact string formats the parser accepts
- The registry table schema for `cortex profile publish`

## Discovery method

### 1. Read existing profiles on disk

We started by reading real profile files:

```
~/.snowflake/cortex/profiles/se_manager.json        # stage-backed, team-deployed
~/.snowflake/cortex/profiles/_CORTEX_CODE_DEFAULT.json  # auto-apply default
~/.snowflake/cortex/profiles/My Local Development Profile.json  # empty shell
```

This revealed the JSON shape (skillRepos, mcpServers, systemPromptRepo, settingsOverrides, etc.)
and the `snowflake_stage` key for stage-backed sources.

### 2. Read the app's own TypeScript source (compiled JS)

The CoCo Desktop app bundles its core logic in:
```
/Applications/SnowWork.app/Contents/Resources/app/node_modules/@coco-sdk/core/dist/
```

Key files we read:

| File | What it told us |
|---|---|
| `profiles/types.js` | `isRepository()` — a repo entry needs either `source` (string) or `snowflake_stage` (string), plus optional `ref` |
| `remote/parser.js` | `parseRemoteSource()` — the exact string formats accepted: `owner/repo`, `owner/repo/path`, `owner/repo@branch`, URLs, `github:` prefix |
| `profiles/profileApplier.js` | How the app clones repos: strips `github:` prefix, calls parseRemoteSource, resolves subpath, clones via RemoteResourceManager |
| `profiles/profileManager.js` | The MERGE statement for `publishProfileToRegistry()` — revealed all 14 columns of the registry table |

### 3. Tested with the app's own functions (not a mock)

We imported the app's actual ESM modules in Node.js and ran our profile's `source` strings
through them:

```javascript
const { parseRemoteSource } = await import(".../remote/parser.js");
const { isRepository } = await import(".../profiles/types.js");

// Result: all 3 sources resolve to {type: "github", owner, repo, ref, path}
```

Then we drove the app's real clone engine:

```javascript
const { getRemoteResourceManager } = await import(".../remote/remoteResourceManager.js");
const mgr = getRemoteResourceManager();
const result = mgr.cloneOrUpdate(parsedSource, {});
// Result: cloned to ~/.snowflake/cortex/remote-cache/github_Jeremy-Demlow_..., files exist
```

### 4. Verified with the `cortex` CLI

```bash
cortex profile show "Team Perspective TEST"
# Shows: Skills (1 repos), MCP Servers (2), System Prompt — all from GitHub sources
```

### 5. Confirmed in the desktop UI

Activated the profile in Agent Settings → Profiles. Skill repo, MCP servers, and system prompt
all appeared and loaded correctly.

### 6. Published to Snowflake stage + registry

Created `CORTEX_CODE.CONFIG.PROFILE_REGISTRY` with the 14-column schema from the MERGE statement,
uploaded files to `@CORTEX_CODE.CONFIG.STG_PROFILE_TEAM_PERSPECTIVE`, and ran:

```bash
cortex profile publish "team-perspective" --from-file profile/team-perspective-stage.profile.json
```

Verified with `cortex profile list-remote` — profile appears and is fetchable.

## Key findings (not in public docs)

1. **GitHub-sourced profiles work with zero stage.** A profile repo entry is
   `{ "source": "owner/repo/subpath", "ref": "main" }`. The parser accepts `owner/repo`,
   `owner/repo/path`, `owner/repo@branch`, full URLs, and `git@` SSH URLs.

2. **`mcpServers` accepts a repo entry** (not just an inline object or stage). Point it at
   an `mcp.json` file: `{ "source": "owner/repo/mcp.json", "ref": "main" }`.

3. **The registry table has 14 columns** including `SUBAGENTS` (VARIANT) which isn't in the
   initial DDL examples anywhere — we discovered it from a publish failure and added it.

4. **Profile names in the registry must be slugs** (letters, numbers, hyphens, underscores —
   no spaces). Local profile filenames can have spaces, but publishing requires a slug.

5. **The UI normalizes profiles on activation** — it re-saves the JSON, drops unknown keys,
   and reorders fields. This is expected and harmless.

6. **`permissions.json` is a runtime grant cache**, not a policy file. Don't write "baseline
   permissions" into it — the safe posture is just Default Approvals mode (the default).

7. **Hook events are:** SessionStart, PreToolUse, PostToolUse, UserPromptSubmit, Stop,
   SubagentStop, Notification.

## Reproducibility

Anyone can repeat this discovery by reading the app bundle at the path above. The functions are
stable ESM exports and the MERGE statement in `profileManager.js` is the single source of truth
for the registry schema.
