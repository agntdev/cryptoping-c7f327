# CryptoPing — Bot specification

**Archetype:** custom

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

A private Telegram bot for tracking crypto price alerts with smart notifications, customizable thresholds, percent change triggers, and optional daily summaries. Users manage watchlists via buttons or typed tickers, configure quiet hours, and avoid spam through cooldown periods. Owner receives anonymized usage statistics.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- individual crypto traders
- crypto hobbyists
- Telegram users seeking in-app price alerts

## Success criteria

- users can add/remove coins via buttons/typed input
- price alerts trigger once per cooldown period
- owner receives daily anonymized trigger statistics

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Initialize user profile and show onboarding menu
- **/price** (command, actor: user, command: /price) — Show current prices for watchlist or specific ticker
- **Add Bitcoin** (button, actor: user, callback: add_coin:BTC) — Quick-add common coin to watchlist
- **Add custom ticker** (button, actor: user, callback: add_custom) — Prompt user to type a custom ticker

## Flows

### onboarding
_Trigger:_ /start

1. greet user
2. request timezone selection
3. show quick-add buttons for BTC/ETH/TON

_Data touched:_ user_profile

### price_alert_setup
_Trigger:_ button: Add price alert

1. ask direction (above/below)
2. request target price
3. confirm and store rule

_Data touched:_ watchlist_entry

### percent_alert_setup
_Trigger:_ button: Add percent alert

1. ask direction (rise/fall/both)
2. request percent threshold
3. confirm window (default 1h)

_Data touched:_ watchlist_entry

### morning_summary
_Trigger:_ scheduled event

1. check quiet hours status
2. compile watchlist prices
3. send summary if not in quiet hours

_Data touched:_ user_profile

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **user_profile** _(retention: persistent)_ — User-specific settings and preferences
  - fields: telegram_id, timezone, quiet_hours_start, quiet_hours_end, summary_time, pause_after_alert
- **watchlist_entry** _(retention: persistent)_ — Cryptocurrency symbol and alert rules
  - fields: ticker, readable_name, price_threshold_alerts, percent_change_alerts, cooldown_timestamps
- **notification_record** _(retention: persistent)_ — Anonymized alert statistics for owner
  - fields: ticker, trigger_type, timestamp, count

## Integrations

- **Telegram** (required) — Bot API messaging and scheduling
- **Market Price API** (required) — Cryptocurrency price data
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- view aggregate user counts
- access anonymized trigger statistics
- configure default cooldown periods

## Notifications

- price threshold alerts
- percent change alerts
- morning summary
- error notifications for price unavailability

## Permissions & privacy

- all user data is private and never shared
- owner only sees anonymized aggregate statistics
- no cross-user data exposure

## Edge cases

- unknown tickers with fuzzy matching
- price API failures with retries
- quiet hours during alert triggers
- cooldown periods preventing alert spam

## Required tests

- alert cooldown prevents repeated notifications
- morning summary respects quiet hours
- percent change calculation over configurable window

## Assumptions

- default timezone uses Telegram-provided value
- price API has exponential backoff retries
- 3-hour default cooldown for alerts
