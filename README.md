# Trillet for Cursor and Grok Bot

Connect [Trillet](https://trillet.ai) voice AI so Grok Bot or Cursor can claim a phone number, reuse or create an agent, and place a real outbound call from chat. After the call, the bot reports the transcript.

This repository is the connector plugin. Bot templates (a Restaurant Booker persona, and others) come later and will depend on this plugin. They are not part of this repo.

## Install from the marketplace

Once the listing is approved:

1. Open **Customize** in Cursor.
2. Find **Trillet** and choose **Install**.
3. Choose **Configure** and set the two variables below.

Submit this repository at https://cursor.com/marketplace/publish

## Install from the repo (before it is listed)

Copy this repository to `~/.cursor/plugins/local/trillet` so `.cursor-plugin/plugin.json` is inside that folder. Restart Cursor or run **Developer: Reload Window**, then open **Customize** and configure the plugin.

On Teams and Enterprise, local plugin imports must be allowed. A marketplace plugin named `trillet` takes precedence over the local copy.

## Configure

In **Customize → Trillet → Configure**:

| Variable | What to paste |
|---|---|
| `TRILLET_API_KEY` | Secret API key from [Trillet Studio](https://app.trillet.ai): **Settings → Workspace → Developer → API Keys**. Owner or admin. Not available on the Basic plan. |
| `TRILLET_WORKSPACE_ID` | 24-character hex workspace id from the Studio URL or workspace settings. |

Cursor stores these values. Do not commit them.

The plugin starts the official MCP server with:

```text
npx -y @trillet-ai/mcp@latest
```

Package name `@trillet-ai/mcp`, bin `trillet-mcp`, env `TRILLET_API_KEY` and `TRILLET_WORKSPACE_ID`. Optional `TRILLET_API_URL` defaults to `https://api.trillet.ai` inside the server and is not a plugin variable.

## Example

> Call +61… and ask if they can take a table for 2 at 7.

The bot confirms the number and the mission, dials only after you say yes, then returns the transcript. If the workspace has no caller ID, it searches numbers, asks you to pick one, confirms the purchase, and sends a Stripe Checkout link when Studio requires payment.

## MCP package must be published to npm before Install works for strangers

`@trillet-ai/mcp` is **not on npm yet** (`npm view @trillet-ai/mcp` is 404). Until it is published, `npx` cannot start the server for anyone who installs this plugin.

Source of truth: `packages/mcp` in the private repo [TrilletAI/trillet-claude](https://github.com/TrilletAI/trillet-claude) (`@trillet-ai/mcp`, version in `package.json` / `src/version.ts`). A maintainer with the npm org `trillet-ai` runs, from that repo:

```bash
cd packages/mcp
npm install
npm whoami
npm org ls trillet-ai
npm publish --dry-run
npm publish
npm view @trillet-ai/mcp version
```

`publishConfig.access` is `public`. `prepublishOnly` builds `dist/` (the `trillet-mcp` bin). Do not publish a `.env`, API key, or workspace id. If `npm org ls trillet-ai` fails, create the org at https://www.npmjs.com/org/create or get added as a publisher first. Full notes: `packages/mcp/PUBLISH.md` in trillet-claude.

After publish, `npx -y @trillet-ai/mcp@latest` should print `trillet-mcp v… ready` on stderr and wait on stdin. Stop it with Ctrl-C.
