# Basic Example of Profiles

A complete, **fork-and-go** example of a Cortex Code Desktop **perspective** — built so a team
can clone it, change a few placeholders, and onboard non-technical partners in minutes.

This repo showcases **both distribution channels** for profiles:
- **GitHub-sourced** — public, no Snowflake needed, works for anyone with internet.
- **Snowflake stage-sourced** — internal, role-governed, discoverable via `cortex profile list-remote`.

Both are verified working end-to-end and documented here.

## Plugin vs Profile — what's what

This repo is **both**, because they do different jobs:

- **Plugin** = the *package* of capabilities (the `skills/`, `agents/`, `mcp.json` here). It's
  how you distribute reusable content. Defined by `.cortex-plugin/plugin.json`.
- **Profile** = the *active identity* a user switches on — which skills, MCP, system prompt, and
  settings are live.

The profile is what a partner "loads." It *references* this repo's content. See
[`docs/PROFILE_SCHEMA.md`](docs/PROFILE_SCHEMA.md) for the exact schema (GitHub + stage forms).

## What's in here

```
basic-example-of-profiles/
  profile/
    team-perspective.profile.json         # GitHub-sourced profile (the default)
    team-perspective-stage.profile.json   # Snowflake stage-sourced profile (same content, different transport)
  AGENTS.md                               # system prompt / persona
  mcp.json                                # MCP servers (placeholders, ${input:} secrets)
  skills/
    setup-my-perspective/SKILL.md         # one-question install skill (vanilla fast path)
    getting-started/SKILL.md              # friendly, no-jargon tour
  agents/data-helper.md                   # example subagent
  .cortex-plugin/plugin.json              # installable as one plugin bundle
  examples/hooks.example.json             # optional event-hook example (not auto-loaded)
  docs/
    PROFILE_SCHEMA.md                     # GitHub + stage profile schema (verified from app source)
    CUSTOMIZE.md                          # fork-and-fill guide
    STAGE_SETUP.md                        # full stage + registry setup walkthrough
    DISCOVERY.md                          # how we reverse-engineered and verified all of this
```

## Two channels, same result

| | GitHub-sourced | Stage-sourced |
|---|---|---|
| **Profile file** | `profile/team-perspective.profile.json` | `profile/team-perspective-stage.profile.json` |
| **Source key** | `"source": "owner/repo/path"` | `"snowflake_stage": "@DB.SCHEMA.STAGE/path"` |
| **Best for** | Public/external, open-source, no Snowflake needed | Internal teams, enterprise, role-restricted |
| **Access governed by** | Git credentials (public = none) | Snowflake roles + grants |
| **Discovery** | Partners paste the repo URL in Agent Settings | `cortex profile list-remote` shows it |
| **Setup guide** | Below (Load paths A & B) | [`docs/STAGE_SETUP.md`](docs/STAGE_SETUP.md) |

## Load paths (GitHub channel)

**A. One-click profile (cleanest).** Copy `profile/team-perspective.profile.json` into
`~/.snowflake/cortex/profiles/`, then **Agent Settings → Profiles → Activate**. Activation pulls
skills + MCP + persona straight from GitHub.

**B. Guided setup skill (friendliest for beginners).** In Cortex Code Desktop:

1. **Agent Settings → Skills → +** (GitHub Skills) → paste `Jeremy-Demlow/basic-example-of-profiles`
   *(or use Plugins → Add from GitHub to get the whole bundle).*
2. Open a chat and type **`/setup-my-perspective`**.
3. Confirm your name (one question) → the skill installs a personalized profile.
4. **Agent Settings → Profiles → Activate** → start a new chat.

Then try **"What can I do here?"** for a guided tour.

## Load paths (Stage channel)

**C. Stage-backed (internal teams).** After setup ([`docs/STAGE_SETUP.md`](docs/STAGE_SETUP.md)):

```bash
# CLI
cortex profile add team-perspective -c myconnection

# Or in the Desktop UI:
# Agent Settings → Profiles → remote list → Add → Activate
```

## Make it yours

Fork and replace placeholders — full instructions in [`docs/CUSTOMIZE.md`](docs/CUSTOMIZE.md).

## How we figured this out

The profile schema, registry table DDL, GitHub source format, and hook events are **not in the
public Snowflake documentation**. We reverse-engineered them from the CoCo Desktop app's own
compiled source code (`@coco-sdk/core`), tested with the app's real functions, and verified
end-to-end in both the CLI and desktop UI.

The full story: [`docs/DISCOVERY.md`](docs/DISCOVERY.md).

## Safe-by-default posture

This example assumes a new, non-technical user, so it relies on the default **ask-before-acting**
posture (Default Approvals): the assistant prompts before each tool call, edit, or SQL run. For
unfamiliar tasks, use **Plan Mode** (read-only — shows the plan before anything happens). Don't
enable Bypass Approvals for newcomers.

## Security notes

- Plugins and profiles are **not** cryptographically signed. Review the manifest, skills, MCP
  servers, and hooks before installing — same as running any code from an author you trust.
- The `mcp.json` servers are **placeholders** and won't connect until you fill in real values.
  Secrets use `${input:...}` prompts and are stored encrypted by the OS, never committed here.
