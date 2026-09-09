---
name: Configure alerts and notification channels
description: Set threshold, discrete and throughput alerts on building datapoints, and route them to Teams, Telegram, email or the dashboard.
api: openapi/aedifion-openapi.yml
operations: [get_alerts, post_alert_threshold, post_alert_discrete, post_alert_throughput, enable_alert, disable_alert, delete_alert_new]
---

# Configure alerts and notification channels

This is aedifion's push surface. There is no general webhook subscription API — outbound
delivery exists only as alert notification channels.

## The three alert types

- `post_alert_threshold` — a datapoint crosses a numeric threshold. Fields: `threshold_info`,
  `threshold_crit`, `threshold_crit_reset` (hysteresis, so a point oscillating around the
  boundary does not storm), `threshold_dead` (dead band).
- `post_alert_discrete` — a datapoint enters a discrete state.
- `post_alert_throughput` — a datapoint **stops delivering data**. This is the one people
  forget, and in building telemetry a silent sensor is the most common real fault.

Shared fields: `level`, `delay` (how long the condition must hold), `period`, `repeat`,
`negate`, `status`, `description`.

## Routing

`AlertNotification` has four optional channels:

| Channel | Field | Shape |
|---|---|---|
| `ms_teams` | `webhook_urls` | array of Microsoft Teams incoming-webhook URLs |
| `telegram` | `chat_ids` | array of Telegram chat ids |
| `email` | `recipients` | array of addresses |
| `dashboard` | `enabled` | boolean |

`ms_teams.webhook_urls` is the only channel that delivers to an HTTP endpoint you control.

**Know its limits before you depend on it:** aedifion documents no signature or HMAC on the
delivery, no retry policy, no delivery guarantee and no replay endpoint. Do not treat these
notifications as a reliable event stream. If you need one, consume MQTT
(`asyncapi/aedifion-mqtt-asyncapi.yml`) and compute the condition yourself.

## Lifecycle

- `enable_alert` / `disable_alert` — a clean reversible pair. **Prefer disabling to deleting.**
- `delete_alert_new` — permanent, no undo.
- `get_alerts` — list what a project already has, before you add another.

## Discipline

- No idempotency key: check `get_alerts` before creating, or a retry duplicates the alert and
  doubles the notifications.
- Set `delay` and `threshold_crit_reset` deliberately. An alert without hysteresis on a noisy
  building sensor generates notification storms, and the fastest way to get an alerting system
  ignored is to make it noisy.
