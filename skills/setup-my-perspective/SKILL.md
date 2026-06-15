---
name: setup-my-perspective
description: >-
  One-step onboarding for a new Cortex Code Desktop user. Installs and personalizes
  a GitHub-sourced "perspective" profile (skills + MCP + system prompt + settings),
  then guides activation. No terminal, no stage, everything from GitHub.
  Triggers: setup my perspective, set me up, onboard me, get me started,
  configure my profile, build my perspective, first time setup, new user setup.
---

# Setup My Perspective

You are onboarding a **non-technical partner** on Cortex Code Desktop. They may know
Claude / Cowork but never used VSCode. Be warm, plain-spoken, jargon-free. Do the work
for them; they only answer a few questions and click Approve.

The deliverable is a **profile** — a single JSON file under
`~/.snowflake/cortex/profiles/` that, when activated, pulls skills, MCP servers, and the
system prompt directly from a GitHub repo. You install + personalize that profile; the
app does the rest on activation.

## Golden rules

1. **Back up before writing.** If a profile file already exists, copy it to
   `<file>.bak.<timestamp>` first and say where.
2. **Confirm before each write**, in plain language.
3. **Never commit or print secrets.** MCP secrets use `${input:...}` and are prompted later.
4. **Reversible.** Switching profiles or deleting the file fully undoes everything.

## Where things live

- Profiles: `~/.snowflake/cortex/profiles/<Name>.json`
- This repo's profile template: `profile/team-perspective.profile.json`
- Once the repo is installed, it's cloned under `~/.snowflake/cortex/plugins/`
  (plugin install) or `~/.snowflake/cortex/remote_cache/` (skill-from-GitHub install).

## Step 1 — Find the source repo

Ask the user (question tool, one short batch):

- **Which repo is their perspective in?** Default to this example,
  `Jeremy-Demlow/basic-example-of-profiles`. If their team forked it, get their
  `owner/repo` (and branch if not `main`). This is what every `source` field will point at.
- **Their name** — for the profile name + greeting.
- **Snowflake connection name** — their default connection. If unknown, run
  `snow connection list` and let them pick; if none, that's fine, skip connection defaults.

## Step 2 — Load the profile template

Locate `profile/team-perspective.profile.json` from the installed repo (search
`plugins/` and `remote_cache/`). If you can't find it locally, fetch it from the repo's
raw GitHub URL for the chosen `owner/repo@ref`.

## Step 3 — Personalize it

Produce a personalized copy:

- `name` → `"<Name>'s Perspective"`
- `ownerTeam` → their team (or leave blank)
- Replace every `source` value's `owner/repo` segment with the repo from Step 1, keeping
  the subpath (`/skills`, `/mcp.json`, `AGENTS.md`) and `ref`. If they're using the default
  repo unchanged, leave the sources as-is.
- `settingsOverrides.sqlConnectionName` / `cortexAgentConnectionName` → their connection
  (remove these keys if they have no connection).
- Strip the `//*` comment keys.

Show the final JSON and confirm.

## Step 4 — Install the profile

Write it to `~/.snowflake/cortex/profiles/<Name>'s Perspective.json`. Back up first if a
file with that name exists.

## Step 5 — Activate + verify (do this WITH them, in plain steps)

1. Open **Agent Settings** (left activity bar) → **Profiles**.
2. Select **"<Name>'s Perspective"** → click **Activate**. On activation the app clones the
   skills, MCP config, and system prompt from GitHub automatically.
3. Start a **new chat** so the persona and MCP load.

Then tell them:
- The MCP servers in the repo's `mcp.json` are **placeholders** — they'll only connect once
  someone fills in real values (their team's fork). That's expected; nothing breaks.
- Try **"What can I do here?"** to get the guided tour from `getting-started`.

## Step 6 — Wrap up

Summarize in 3 bullets: what profile you created, where it is, and that everything is
reversible (switch profiles or delete the file). Keep it warm and short.
