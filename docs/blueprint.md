# Crypto Price Tracker with Personal Alerts — Bot specification

**Archetype:** custom

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

A Telegram bot that lets users track cryptocurrency prices with customizable alerts, silent hours, and anti-spam logic. Provides personalized price notifications, manual price checks, and admin-level anonymized analytics.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- retail crypto market users
- bot owner/admin

## Success criteria

- Users can set up and manage personalized price alerts
- Notifications are delivered without spam during active hours
- Admin receives anonymized aggregate statistics

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Initialize account setup and show main menu
- **/price** (command, actor: user, command: /price) — Check current price of a specific coin or full tracking list
  - inputs: ticker symbol
  - outputs: price data
- **Manage Coins** (button, actor: user, callback: tracking:manage) — Open tracking list interface with add/delete/alert options
  - inputs: selected coin
  - outputs: tracking list updates
- **Morning Summary** (button, actor: user, callback: summary:toggle) — Enable/disable daily price summary notifications
  - inputs: enabled/disabled
  - outputs: summary preference

## Flows

### Onboarding Setup
_Trigger:_ /start

1. Detect timezone
2. Set silent hours
3. Optional morning summary setup

_Data touched:_ User

### Alert Configuration
_Trigger:_ tracking:manage

1. Select coin
2. Choose alert type (threshold/percentage)
3. Set parameters
4. Confirm alert

_Data touched:_ TrackingPosition

### Price Check
_Trigger:_ /price

1. Parse ticker argument
2. Fetch price data
3. Format response

_Data touched:_ PriceSource

### Notification Delivery
_Trigger:_ price_threshold_crossed

1. Check silent hours
2. Apply anti-spam cooldown
3. Send alert message

_Data touched:_ AlertEvent

### Admin Stats
_Trigger:_ admin:stats

1. Aggregate active users
2. Count alert triggers by ticker
3. Format anonymized report

_Data touched:_ AdminStats

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **User** _(retention: persistent)_ — Telegram user account with personal preferences and tracking list
  - fields: telegram_id, timezone, silent_hours, summary_time, tracking_list
- **TrackingPosition** _(retention: persistent)_ — Coin price monitoring configuration
  - fields: ticker, alert_type, threshold_value, percentage_value, window_minutes, last_triggered
- **AlertEvent** _(retention: session)_ — Record of price alerts sent to users
  - fields: user_id, ticker, old_price, new_price, change_percent, trigger_time
- **AdminStats** _(retention: persistent)_ — Anonymized aggregate usage statistics
  - fields: active_users, top_tickers, alert_type_counts

## Integrations

- **Telegram** (required) — Bot API messaging
- **Price API** (required) — Cryptocurrency price data source
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- View anonymized aggregate statistics
- Configure price source API
- Manage subscription tiers (optional)

## Notifications

- Price threshold alerts
- Percentage change alerts
- Morning summary digest
- Admin statistics report

## Permissions & privacy

- User data stored privately and encrypted
- Admin stats aggregated and anonymized
- No user data shared with third parties

## Edge cases

- Price source API failures
- User input typos in ticker symbols
- Alert cooldown expiration during price spikes
- Silent hour boundary notifications

## Required tests

- Alert suppression during silent hours
- Anti-spam cooldown behavior
- Price source error handling
- Admin stats aggregation accuracy

## Assumptions

- Default price source is CoinGecko API
- Default anti-spam cooldowns: 6h for thresholds, 1h for percentages
- Morning summary defaults to 8:00 AM local time
- Price source errors trigger temporary status updates
