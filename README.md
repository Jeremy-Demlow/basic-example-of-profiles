# Basic Example of Profiles

A complete, **fork-and-go** example of a Cortex Code Desktop **perspective** — built so a team
can clone it, change a few placeholders, and onboard non-technical partners in minutes.

**Everything lives in GitHub. No Snowflake stage required.** A single profile file references
the skills, MCP servers, and system prompt in this repo, so activating it wires up the whole
experience.

## Plugin vs Profile — what's what

This repo is **both**, because they do different jobs:

- **Plugin** = the *package* of capabilities (the `skills/`, `agents/`, `mcp.json` here). It's
  how you distribute reusable content. Defined by `.cortex-plugin/plugin.json`.
- **Profile** = the *active identity* a user switches on — which skills, MCP, system prompt, and
  settings are live. Defined by `profile/team-perspective.profile.json`.

The profile is what a partner "loads." It *references* this repo's content from GitHub. See
[`docs/PROFILE_SCHEMA.md`](docs/PROFILE_SCHEMA.md) for the exact schema.

## What's in here

```
basic-example-of-profiles/
  profile/team-perspective.profile.json   # THE profile — binds everything, all GitHub-sourced
  AGENTS.md                               # system prompt / persona (referenced by the profile)
  mcp.json                                # MCP servers (placeholders, ${input:} secrets)
  skills/
    setup-my-perspective/SKILL.md         # installs + personalizes the profile for a newcomer
    getting-started/SKILL.md              # friendly, no-jargon tour
  agents/data-helper.md                   # example subagent
  .cortex-plugin/plugin.json              # makes the repo installable as one plugin bundle
  examples/hooks.example.json             # optional event-hook example (not auto-loaded)
  docs/PROFILE_SCHEMA.md                  # the GitHub profile schema (verified, not in public docs)
  docs/CUSTOMIZE.md                       # fork-and-fill guide for your team
```

## Two ways to load it

**A. One-click profile (cleanest).** Copy `profile/team-perspective.profile.json` into
`~/.snowflake/cortex/profiles/`, then **Agent Settings → Profiles → Activate**. Activation pulls
skills + MCP + persona straight from GitHub.

**B. Guided setup skill (friendliest for beginners).** In Cortex Code Desktop:

1. **Agent Settings → Skills → +** (GitHub Skills) → paste `Jeremy-Demlow/basic-example-of-profiles`
   *(or use Plugins → Add from GitHub to get the whole bundle).*
2. Open a chat and type **`/setup-my-perspective`**.
3. Answer a couple of simple questions; it installs and personalizes the profile, then walks you
   through activating it.

Then try **"What can I do here?"** for a guided tour.

## Make it yours

This is a template. Fork it and replace the placeholders — full instructions in
[`docs/CUSTOMIZE.md`](docs/CUSTOMIZE.md). The short version: point every `source` in the profile
at your fork, fill in `mcp.json` with your real servers, and rewrite `AGENTS.md` in your team's
voice.

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
