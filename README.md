# Discord Twitch Stream Alerts

![Discord Twitch Stream Alerts](assets/banner.svg)

[![TypeScript](https://img.shields.io/badge/TypeScript-5-3178C6?logo=typescript&logoColor=white)](https://www.typescriptlang.org/)
[![Node.js](https://img.shields.io/badge/Node.js-22%2B-5FA04E?logo=nodedotjs&logoColor=white)](https://nodejs.org/)
[![Twitch EventSub](https://img.shields.io/badge/Twitch-EventSub-9146FF?logo=twitch&logoColor=white)](https://dev.twitch.tv/docs/eventsub/)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

A TypeScript Discord bot that posts an alert when configured Twitch channels go live. Each Discord server has its own alert channel and Twitch channel list. Shared Twitch channels are monitored once and announced in every server that subscribes to them.

[Español](README.es.md)

## Features

- Near-real-time alerts from Twitch EventSub `stream.online`, with Helix polling as a safety net.
- One bot for many Discord guilds: each server has its own alert channel and Twitch channel list.
- Shared Twitch channels are monitored once and announced in every guild that subscribes to them.
- The alert embed bar is Twitch purple while the stream is live and turns gray after it ends (detected by the reconciliation poll).
- Per-subscriber delivery tracking retries failed sends without duplicating ones that already succeeded.
- EventSub reconnects with backoff, transient Helix errors are retried, and startup is guarded.
- Minimal Discord permissions and no message-content, member, or presence intents.

## How it works

![Runtime architecture](assets/architecture.svg)

Twitch EventSub provides near-real-time `stream.online` events; Helix polling reconciles missed events after startup or reconnect. The bot resolves each Twitch channel once, tracks per-subscriber delivery in `data/live-state.json`, and posts the embed to every subscribed guild channel.

## Example alert

![Example Twitch live alert](assets/alert-example.svg)

*Illustrative example with fictional data. The real embed shows the stream preview and category. Viewer count is hidden by default; set `SHOW_VIEWER_COUNT=true` in `.env` to opt in.*

## Requirements

- Node.js 22+
- pnpm 11+
- A Discord bot application/token
- A Twitch developer application (Client ID and Client Secret)

## Discord setup

1. Create a Discord application and bot in the [Developer Portal](https://discord.com/developers/applications).
2. Invite it to each guild with `bot` scope and the permissions `View Channels`, `Send Messages`, `Embed Links`, and `Read Message History`.
3. No Administrator permission, message-content intent, or member/presence intent is needed.
4. Copy the bot token to local `.env` as `DISCORD_TOKEN`.

## Twitch setup (no Twitch CLI needed)

1. Register a Twitch application at the [Twitch Developer Console](https://dev.twitch.tv/console/apps).
2. Add this exact OAuth Redirect URL: `http://localhost:3000`.
3. Copy its Client ID and Client Secret to local `.env` as `TWITCH_CLIENT_ID` and `TWITCH_CLIENT_SECRET`.
4. Install dependencies, then run `pnpm auth:twitch` from the project directory.
5. Open the displayed Twitch authorization URL and approve the application. The local callback validates OAuth state and writes the user access and refresh tokens to `.env` without printing them.
6. Twitch EventSub WebSocket requires a user access token. `stream.online` requires no additional scopes. The bot refreshes expired access tokens and persists token rotation.

The redirect server binds to loopback on port 3000 only for the OAuth setup and shuts down after authorization or timeout. Nothing needs a public callback URL.

## Local configuration

```bash
cp .env.example .env
chmod 600 .env
cp config/guilds.example.json config/guilds.json
```

Fill `.env` locally. Edit `config/guilds.json` for each Discord guild:

```json
{
  "guilds": [
    {
      "guildId": "YOUR_DISCORD_SERVER_ID",
      "alertChannelId": "YOUR_TEXT_CHANNEL_ID",
      "twitchChannels": ["streamer_one", "streamer_two"]
    }
  ]
}
```

Enable Developer Mode in Discord, then copy server and text channel IDs. Twitch names are channel logins, not URLs. The same Twitch channel can appear in different guild entries; that stream gets an alert in each listed Discord channel.

If a configured Twitch login cannot be resolved (the account was renamed or deleted), the bot logs a warning and keeps monitoring the remaining channels. Update `config/guilds.json` and restart to fix it.

Bot presence matches `server-status`: `BOT_ACTIVITY_TYPE=Custom`, `BOT_ACTIVITY_NAME=x0void.com` and `BOT_ACTIVITY_STATE=🔗 x0void.com`. This free-form custom state is not a clickable link. `BOT_ACTIVITY_URL` is only used by a `Streaming` activity and then must be an HTTPS Twitch/YouTube URL. Presence is reapplied after Discord shard ready/resume events and each Twitch reconciliation cycle.

Viewer counts are hidden from alert embeds by default. Set `SHOW_VIEWER_COUNT=true` in `.env` to opt in; `true` and `false` are case-insensitive, and any other value is rejected during startup.

## Run

```bash
pnpm install
pnpm auth:twitch
pnpm dev
```

Build/run production JavaScript:

```bash
pnpm build
pnpm start
```

## Project layout

```text
src/
  application/    reconciliation and subscriber indexing
  domain/         entities and ports
  infrastructure/ Discord, Twitch, persistence, and config adapters
  index.ts        composition root and startup
tests/            Vitest suite with fakes
```

## Deploying with systemd

The unit lives in `deploy/announcements.service` and is a template: `make install` replaces `@DEPLOY_USER@`, `@DEPLOY_GROUP@`, `@DEPLOY_HOME@`, `@DEST@` and `@NODE_BIN@` before installing it. By default it uses the user running `make`, `/srv/services/announcements` and the `node` in `PATH`; you can override them with variables:

```bash
make install DEPLOY_USER=announcements DEPLOY_GROUP=announcements DEST=/srv/announcements
```

Installing the service and updating the code are separate steps:

```bash
make install  # creates the destination, installs the unit and the config; does not enable or start it
make deploy   # build, production dependencies and code; enables the unit if needed and restarts it
```

Before the first `make deploy`, prepare `/srv/services/announcements/.env` with `600` permissions and `config/guilds.json` for production. The deploy does not copy `.env` and keeps the existing guild configuration; `make install` creates that configuration from the local copy only if it does not exist yet. The unit starts through a secrets launcher (`~/.secrets/bin/run srv:announcements`) that loads the destination's `.env`; the application also reads it from its working directory. With `LOG_FILE=/var/log/announcements.log` the bot also writes to that file, which `deploy/announcements.tmpfiles.conf` creates and `deploy/announcements.logrotate.conf` rotates (both installed by `make install`).

`TWITCH_RECONCILE_INTERVAL_SECONDS` defaults to 60 (valid 15–3600). EventSub supplies normal near-real-time start notifications; Helix polling reconciles after reconnect/startup and recovers events missed while disconnected. State is stored under `data/live-state.json` by default.

## Development and tests

```bash
pnpm typecheck
pnpm test
pnpm build
```

Tests use fakes and must not send real Discord messages or create Twitch subscriptions.

## Secret handling

- Never commit `.env`, `config/guilds.json`, OAuth tokens, Discord tokens, or Twitch Client Secret.
- `.env.example` and `config/guilds.example.json` are safe templates only.
- Each person hosting a copy must create their own Discord and Twitch applications and provide their own credentials.
- If a token or client secret is accidentally shared or committed, revoke/rotate it immediately.

## License

MIT. See [LICENSE](LICENSE).
