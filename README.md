# google-workspace

A starter repo for bootstrapping Google Workspace work — Drive, Gmail, Calendar,
Sheets — on [DAM](https://github.com/dam-agents/dam). Clone it, point an agent at
it, grant a Google connection, and you have an agent that can drive Google
Workspace through the [Google Workspace CLI (`gws`)](https://github.com/googleworkspace/cli).

It's config only — a `CLAUDE.md` operating manual plus a set of skills. There's
no image to build: `gws` is baked into the DAM base image, so **every** agent
already has the CLI on its `PATH`.

## Works with any harness

Because `gws` ships in the base image, this works regardless of harness —
Claude Code, Codex, Pi, etc. The config in this repo happens to be written for
**Claude Code** (`CLAUDE.md` + `.claude/skills/`); to use another harness, adapt
those files to its conventions. The `gws` commands themselves are identical
everywhere.

## How auth works (out of the box)

`gws` reads its OAuth access token from the `GOOGLE_WORKSPACE_CLI_TOKEN`
environment variable. DAM wires this up automatically:

1. When you grant a Google connection to the agent, the platform injects
   `GOOGLE_WORKSPACE_CLI_TOKEN=dummy-placeholder` into the agent pod's env.
2. `gws` sends `Authorization: Bearer dummy-placeholder` to `*.googleapis.com`.
3. The in-pod Envoy sidecar swaps the sentinel for the real Bearer token,
   sourced from a Secret mounted into the sidecar only ([ADR-033](https://github.com/dam-agents/dam/blob/main/docs/adrs/033-envoy-credential-gateway.md)).
4. Google receives a valid access token.

The agent container never sees your real Google credentials, and the api-server
refreshes the access token before it expires — no manual steps after the initial
OAuth consent.

## Getting started

### 1. Create a Google Cloud OAuth app

1. Open the [Google Cloud Console](https://console.cloud.google.com/).
2. Create (or pick) a project.
3. **Enable APIs** — *APIs & Services > Library* — enable the APIs for the
   services you want, e.g. Google Drive API, Gmail API, Google Sheets API,
   Google Calendar API.
4. **Configure the OAuth consent screen** — *APIs & Services > OAuth consent
   screen*:
   - User type: **External** (or **Internal** on a Workspace org).
   - Add the scopes for the services you enabled.
   - Add your Google account as a **test user**.
5. **Create an OAuth Client ID** — *APIs & Services > Credentials > Create
   Credentials > OAuth client ID*:
   - Application type: **Web application**.
   - Add the **Authorized redirect URIs** the platform shows you on the
     connection's setup screen (e.g.
     `http://localhost:4444/api/apps/google-drive/callback`).
   - Save the **Client ID** and **Client Secret**.

### 2. Add the connection in DAM

1. In the DAM UI, go to **Connections**.
2. Add a **Google** connection (e.g. Google Drive) with your Client ID and
   Client Secret, and complete the OAuth consent flow.
3. Grant any Google connection to your agent — one is enough to authenticate
   `gws`; grant more to widen the scopes.

### 3. Clone this repo

Clone this repo and use it as your agent's config (`CLAUDE.md` +
`.claude/skills/`). Then ask the agent something like *"list my Google Drive
files"* or *"triage my Gmail inbox"*.

## Files

- [`CLAUDE.md`](CLAUDE.md) — operating manual loaded by the agent.
- [`.claude/skills/`](.claude/skills/) — the Google Workspace skills.
