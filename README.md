# Meshtastic Chat Card

A custom [Lovelace](https://www.home-assistant.io/dashboards/) card for [Home Assistant](https://www.home-assistant.io/) that displays Meshtastic channel messages recorded by the [`meshtastic` integration](https://github.com/meshtastic/home-assistant).

## Features

- **History backfill** – on load, pulls up to 7 days of past messages from the HA logbook (`logbook/get_events`).
- **Live updates** – subscribes to the `meshtastic_message_log` event bus so new messages appear instantly without a page refresh.
- **Auto-scroll** – automatically scrolls to the latest message; pauses auto-scroll when you scroll up, and resumes when you scroll back to the bottom.
- **PKI / direct-message badge** – optionally shows a 🔒 badge next to messages delivered over an encrypted direct link.
- **Visual editor** – all options are configurable through the Lovelace UI editor; no YAML required.
- **Channel auto-discovery** – when adding a new card, the editor pre-selects the primary Meshtastic channel entity if one exists.
- **Message deduplication** – messages received from both history and the live event stream are deduplicated so nothing appears twice.

## Requirements

- Home Assistant with the [`meshtastic` custom integration](https://github.com/meshtastic/home-assistant) installed and configured.
- At least one Meshtastic gateway device added to HA with a channel entity (`device_class: channel`).

## Installation

### HACS (recommended)

1. Open **HACS → Frontend** in Home Assistant.
2. Click the three-dot menu → **Custom repositories**.
3. Add `https://github.com/ch0ppy35/meshtastic-integration-chat-log-card` with category **Dashboard**.
4. Search for **Meshtastic Chat** and install it.
5. Reload your browser.

### Manual

1. Download `meshtastic-chat-card.js` from the [latest release](https://github.com/ch0ppy35/meshtastic-integration-chat-log-card/releases).
2. Copy the file to `config/www/meshtastic-chat-card.js` on your Home Assistant instance.
3. Go to **Settings → Dashboards → Resources** and add `/local/meshtastic-chat-card.js` as a **JavaScript module**.
4. Reload your browser.

## Usage

Add the card via the Lovelace UI (**Add card → Meshtastic Chat**) or paste the YAML directly:

```yaml
type: custom:meshtastic-chat-card
channel_entity: meshtastic.my_gateway_channel_primary
```

### Configuration options

| Option            | Type    | Default | Description                                                                 |
|-------------------|---------|---------|-----------------------------------------------------------------------------|
| `channel_entity`  | string  | —       | **Required.** Entity ID of the Meshtastic channel to display (`device_class: channel`). |
| `title`           | string  | —       | Card title. Defaults to the channel entity's `friendly_name`.               |
| `limit`           | number  | `200`   | Maximum number of messages to keep rendered (oldest are dropped first).     |
| `show_timestamps` | boolean | `true`  | Show the `HH:MM` timestamp at the start of each message row.                |
| `show_pki_badge`  | boolean | `true`  | Show a 🔒 badge on messages delivered via PKI / direct encrypted link.      |

### Full YAML example

```yaml
type: custom:meshtastic-chat-card
channel_entity: meshtastic.my_gateway_channel_primary
title: "Base Camp Chat"
limit: 100
show_timestamps: true
show_pki_badge: true
```

## Development

```bash
# Install dependencies
yarn install

# Start the dev rollup (rebuild on save, serves the bundle on :5001)
yarn start

# Production build → dist/meshtastic-chat-card.js
yarn build

# Type-check (uses tsconfig.test.json so test files are included)
yarn typecheck

# Lint
yarn lint
```

The dev build registers the card as `meshtastic-chat-card-dev` so it can coexist with the production card in the same HA instance. The `DEV` flag is injected at build time by `@rollup/plugin-replace` (`true` in `rollup.config.dev.js`, `false` in `rollup.config.js`) — no manual flipping required before cutting a release.

### Where the dev bundle is written

By default `yarn start` writes to `./dist-dev/meshtastic-chat-card.js`. To live-reload directly into a Home Assistant instance, point `DEV_OUTPUT_DIR` at your HA `www` directory:

```bash
DEV_OUTPUT_DIR=/path/to/homeassistant/config/www yarn start
```

Then in HA, register the resource at **Settings → Dashboards → Resources** as `/local/meshtastic-chat-card.js` with type **JavaScript module** ([HA docs](https://developers.home-assistant.io/docs/frontend/custom-ui/registering-resources)).

### Testing the live message path without a radio

The card subscribes to the `meshtastic_message_log` event bus. To exercise the live render path without a real Meshtastic gateway, fire a fake event from HA's **Developer Tools → Events**:

- Event type: `meshtastic_message_log`
- Event data:
  ```yaml
  entity_id: meshtastic.your_channel_entity
  from_name: Tester
  message: hello from devtools
  pki: false
  ```

### Testing

```bash
# Run the unit and component test suites
yarn test

# Watch mode
yarn test:watch

# Coverage report
yarn test:coverage
```

Tests live next to the source under `src/__tests__/` and use Jest with `ts-jest` (ESM mode). Pure helpers (`messages.ts`, `history.ts`, `discovery.ts`, `live.ts`) run in the default `node` environment; the Lit render smoke test opts into `jsdom` via a per-file docblock.
