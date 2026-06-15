---
name: setup-my-perspective
description: >-
  One-step onboarding for a brand-new Cortex Code Desktop user. Sets up a complete
  personalized "perspective": creates a switchable profile, wires up MCP servers,
  installs a system prompt (persona), applies safe settings + permissions, and
  registers starter skills — all conversationally, with no terminal required.
  Triggers: setup my perspective, set me up, onboard me, get me started,
  configure my profile, build my perspective, first time setup, new user setup.
---

# Setup My Perspective

You are onboarding a **non-technical partner** who has just installed Cortex Code Desktop
for the first time. They may have used Claude / Cowork but never VSCode/Cursor. Be warm,
plain-spoken, and avoid jargon. Explain *why* before each action. Never assume technical
knowledge. Do everything for them — they should only have to answer a few questions and
click "Approve".

## Golden rules

1. **Always back up before you write.** Copy any file you modify to `<file>.bak.<timestamp>`
   first, and tell the user where the backup is.
2. **Merge, never clobber.** When editing `mcp.json`, `settings.json`, or `permissions.json`,
   read the existing content and merge — do not overwrite keys the user already has.
3. **Confirm before each write.** Show the user what you're about to add in plain language
   and wait for a yes.
4. **Never commit or print secrets.** MCP secrets use `${input:...}` prompts; do not hardcode.
5. **Everything is reversible.** Everything you create is captured in one profile JSON the
   user can switch away from or delete.

## Where things live (CoCo Desktop config)

- Profiles: `~/.snowflake/cortex/profiles/<Name>.json`
- MCP servers: `~/.snowflake/cortex/mcp.json`  (schema: `{ "mcpServers": { ... }, "inputs": [...] }`)
- Settings: `~/.snowflake/cortex/settings.json`
- Permissions: `~/.snowflake/cortex/permissions.json`
- This repo (once installed) is cloned under `~/.snowflake/cortex/plugins/` (plugin install)
  or `~/.snowflake/cortex/remote_cache/` (skill-from-GitHub install). Locate `AGENTS.md`
  by searching both; if not found, fetch it from the repo's raw GitHub URL.

---

## Step 1 — Welcome + collect a few basics

Greet the user and ask (use the question tool, keep it to one short batch):

- **Your name** — used to personalize the profile and greetings.
- **Snowflake connection name** — the connection they want as default. If they don't know,
  run `snow connection list` for them and let them pick; if there are none, tell them that's
  fine and skip connection defaults.
- **Which MCP servers to enable** — offer the friendly options from this repo's `mcp.json`
  (e.g. *Snowflake*, *Glean knowledge search*). Multi-select. "None for now" is valid.

## Step 2 — Apply settings

Read `~/.snowflake/cortex/settings.json` (create `{}` if missing). Back it up. Merge in the
values from this repo's `settings.snippet.json` (theme, default approvals, and the connection
name they chose). Show the merged result and confirm before writing.

## Step 3 — Confirm a safe permissions posture (no file write)

Do **not** write to `~/.snowflake/cortex/permissions.json` — it is a runtime grant cache, not a
policy file. Instead, read `permissions.snippet.json` for the recommended posture and **verify +
explain** it to the user in plain terms:
"You're in *ask-before-acting* mode (Default Approvals), so nothing runs without your OK. For
anything unfamiliar, use **Plan Mode** — it just shows the plan without doing anything."
Tell them how to confirm/switch the mode from the chat input, and advise against Bypass Approvals
for now. No file is changed in this step.

## Step 4 — Wire up MCP servers (only the ones they picked)

Read `~/.snowflake/cortex/mcp.json` (create the skeleton if missing). Back it up. For each
server the user selected in Step 1, copy its entry from this repo's `mcp.json` into the user's
`mcpServers` object, and copy any matching `inputs` entries. **Do not duplicate** a server that
already exists. Explain that secrets will be prompted on first use, never stored in plain text.
Confirm, then write.

## Step 5 — Install the system prompt (persona)

Locate this repo's `AGENTS.md` (see "Where things live"). This is the persona that makes the
assistant talk like a friendly guide for non-technical partners. You will reference it from the
profile in Step 6 via `systemPromptRepo` (local file path is fine). Tell the user what the
persona does in one sentence.

## Step 6 — Create the switchable profile

Write `~/.snowflake/cortex/profiles/<Name>'s Perspective.json` using this shape (fill in real
values from the steps above; omit empty blocks):

```json
{
  "name": "<Name>'s Perspective",
  "description": "Starter perspective created by setup-my-perspective for <Name>",
  "ownerTeam": "",
  "version": "1.0",
  "skillRepos": [],
  "mcpServers": {},
  "commandRepos": [],
  "systemPromptRepo": null,
  "plugins": [],
  "subagents": [],
  "hooks": {},
  "envVars": {},
  "settingsOverrides": {},
  "localModified": true
}
```

- Put the MCP servers they enabled under `mcpServers` (inline), the chosen settings under
  `settingsOverrides`, and point `systemPromptRepo` at the located `AGENTS.md` path.
- Because they installed this repo as a plugin/skill, the starter skills are already available;
  you do not need to re-register them. If they installed only this one skill from GitHub, tell
  them the other starter skills come along automatically with the plugin install.

## Step 7 — Activate + verify

Tell the user, in numbered plain steps, how to switch on the new profile:

1. Open **Agent Settings** (activity bar) → **Profiles**.
2. Select **"<Name>'s Perspective"** and click **Activate**.
3. Restart the chat (new conversation) so the persona + MCP load.

Then give them a friendly first thing to try, e.g.:
> Type: *"What can I do here?"* — and the `getting-started` skill will give you a tour.

End by summarizing, in 3 bullets, exactly what you set up and where the backups are, so they
feel in control. Remind them everything is reversible by switching profiles or deleting the
profile JSON.
