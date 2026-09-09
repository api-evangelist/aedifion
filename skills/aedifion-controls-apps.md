---
name: Deploy and supervise autonomous controls apps
description: Start, stop and monitor the cloud control applications that autonomously operate HVAC plant in a building.
api: openapi/aedifion-openapi.yml
operations: [get_controls_algorithms_by_component, get_controls_apps, get_controls_app, post_controls_app, post_controls_app_run, post_controls_apps_run, get_controls_app_logs, delete_controls_app]
---

# Deploy and supervise autonomous controls apps

A controls app is a cloud algorithm that **operates real building plant without a human in the
loop**. This is the highest-authority surface in the aedifion API. Everything else reads data or
writes one value; this delegates ongoing control.

## Survey before deploying

- `get_controls_algorithms_by_component` — the available control algorithms and their variations,
  grouped by the component type they apply to.
- `get_controls_apps` — what is already deployed in the project.
- `get_controls_apps_summary` — deployed apps grouped by algorithm.

Check for an existing app on the same component first. Two algorithms fighting over the same
plant is a real failure mode and there is no idempotency key to protect you.

## Deploy

- `post_controls_app` — create a controls app in the project.
- `post_controls_app_custom` — create a custom one.

## Run and stop

`post_controls_app_run` — "Request to run or stop a controls app in the project." The same
operation does both, so **your reversal path is the operation you already called.**
`post_controls_apps_run` does it in bulk, and `post_controls_apps_update` updates several at once.

Bulk operations across a live building deserve more caution than single ones, not less. There is
no dry-run on this surface — `dryrun` exists only on `post_datapoint_setpoint`.

## Supervise

- `get_controls_app` — current state.
- `get_controls_app_logs` — what the algorithm actually did.
- `ControlsAppStatus` carries `code`, `message`, `name` (a stable language-independent key such
  as `running` or `died`) and `type` (`normal`, `error`, `pending`, `ready`).

**Match on `name` and `type`, not on `message`** — `name` is documented as the stable,
language-independent key, and `message` is localized prose.

An app in state `died` is not controlling anything. Nothing pushes that to you: there is no
webhook for controls app state. Poll `get_controls_apps`, or configure a throughput alert on a
datapoint the app should be writing.

## Discipline

- Deploying a controls app hands over ongoing authority. Require explicit human approval.
- `delete_controls_app` is permanent.
- Stopping an app leaves the plant wherever it was last commanded. If you stop an app, check
  whether the setpoints it wrote need resetting — see
  `aedifion-write-setpoint-safely.md` and reset with `value='null'`.
- As of changelog 2.1.13 (2026-08-11) **schedules deploy via a Schedule controls app rather
  than by writing datapoints**. Older integrations that wrote schedule values directly are
  broken; this is the one breaking change in the recent changelog window.
