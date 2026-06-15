# Basic Example of Profiles

A starter **"perspective"** for Cortex Code Desktop, built for **non-technical partners** who
have just installed the app and want to be set up in a few clicks — no terminal, no coding.

It bundles everything a newcomer needs into one place:

- A friendly **system prompt / persona** (`AGENTS.md`) that makes the assistant talk in plain
  language and always ask before acting.
- **Starter skills** — `/setup-my-perspective` (one-step setup) and `/getting-started` (a tour).
- **MCP server** templates (e.g. knowledge search) that wire up with secret prompts, never
  hardcoded keys.
- Safe **settings** and a recommended **permissions posture**.
- A switchable **profile** so the whole setup is reversible.

---

## For non-technical partners — 3 steps

1. **Add this from GitHub.** In Cortex Code Desktop, open **Agent Settings** (left activity
   bar) → **Skills** → click **+** next to *GitHub Skills* → paste:

   ```
   Jeremy-Demlow/basic-example-of-profiles
   ```

   *(Or use **Plugins → Add from GitHub** with the same path to get everything as one bundle.)*

2. **Run the setup.** Open a chat and type:

   ```
   /setup-my-perspective
   ```

   Answer a couple of simple questions (your name, your Snowflake connection, which tools to
   turn on). The assistant does the rest and asks before every change.

3. **Activate your profile.** When it finishes, go to **Agent Settings → Profiles**, select
   **"&lt;Your Name&gt;'s Perspective"**, click **Activate**, and start a new chat.

Then try: **"What can I do here?"** for a guided tour.

> Everything is reversible. To undo, switch back to the default profile or delete your profile
> file under `~/.snowflake/cortex/profiles/`.

---

## What gets installed where

| Piece | Source in this repo | Lands at |
|---|---|---|
| Persona / system prompt | `AGENTS.md` | referenced by your profile |
| Starter skills | `skills/` | your Skills list |
| MCP servers (selected) | `mcp.json` | `~/.snowflake/cortex/mcp.json` |
| Settings (theme, connection) | `settings.snippet.json` | `~/.snowflake/cortex/settings.json` |
| Permissions posture | `permissions.snippet.json` | *(verified, not written)* |
| Switchable profile | `profile/perspective.profile.json` | `~/.snowflake/cortex/profiles/` |

## Customizing for your team

Fork this repo and edit:
- `AGENTS.md` — change the persona/voice for your audience.
- `mcp.json` — add your team's MCP servers (replace `YOUR-COMPANY.glean.com`, add others).
- `settings.snippet.json` — set defaults.
- `skills/` — add your own starter skills.

Point your partners at your fork instead of this repo. Nothing here contains secrets, so it is
safe to keep public.

## Notes

- Plugins and skills are **not** cryptographically signed. Review the manifest and skills before
  installing, as you would any code from an author you're choosing to trust.
- MCP secrets use `${input:...}` prompts and are stored encrypted by the OS, never in this repo.
