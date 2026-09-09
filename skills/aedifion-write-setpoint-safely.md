---
name: Write a setpoint to building equipment safely
description: Actuate physical HVAC plant through the aedifion API — rehearse with a dry run, confirm the write actually landed, and reverse it.
api: openapi/aedifion-openapi.yml
operations: [get_project_setpoints, post_datapoint_setpoint, get_datapoint_setpoint, get_datapoint]
---

# Write a setpoint to building equipment safely

**This skill actuates physical equipment in an occupied building.** Heating, cooling and
ventilation setpoints affect comfort, energy cost and, at the extremes, safety. Treat every
call here as irreversible until you have confirmed otherwise.

## Before you write

1. `get_project_setpoints` — see what setpoints the project already defines.
2. `get_datapoint` — confirm the point is writable and is the point you think it is. Resolve it
   through tags or component pins, never by constructing a `dataPointID`.

## Rehearse first

`post_datapoint_setpoint` accepts a `dryrun` query parameter, documented as "Do a dry run
without actually writing anything." **Always dry-run first.** This is the one high-consequence
operation on the API and aedifion ships a rehearsal facility for it; use it.

Parameters:

- `dataPointID` (required) — the point to write.
- `project_id` (required) — the project it belongs to.
- `value` (required) — a float, **or the string `null` to reset**.
- `priority` — BACnet-style write priority.
- `acked` — request an acknowledgement. Set this.
- `dryrun` — rehearse without writing.
- `keep_out_of_service` — pin the value so the local building automation system will not
  overwrite it. This makes the write *sticky*; only use it when you mean it, and plan the
  reversal.

## A 200 does not mean it worked

aedifion is explicit about this. The endpoint is "a no-frills, non-acked, stateless,
best-effort write service", and:

> This endpoint returns '200 - success' if the setpoint operation is authorized and correctly
> specified. It does not provide any feedback whether the setpoint has actually been written
> successfully by the remote building network.

So:

1. Call with `acked=true`. The response carries a `reference` token.
2. Redeem it with `get_datapoint_setpoint` using that `reference`.
3. Read the `SetpointAck`: `status`, `log`, and **`state_before`** — the value the point held
   before you touched it. **Record `state_before`.** It is the only way to restore rather than
   merely reset.

## Reversing

- **Reset:** re-issue `post_datapoint_setpoint` with `value='null'`. The parameter description
  defines `null` as a reset — it hands the point back to the local building automation system.
- **Restore:** re-issue with the `state_before` value you captured from the acknowledgement.

No reversal *window* is published, and reversal is best-effort for exactly the same reason the
original write is. Confirm the reversal with a second acknowledgement.

## Retry discipline

There is **no `Idempotency-Key` header on this API**. A retried `POST` writes a second time.
If a call times out, do not blindly retry — read back with `get_datapoint_setpoint` or
`get_datapoint_timeseries` and decide.

## Errors

`401` re-authenticate. `403` your role lacks write access on this datapoint — do not retry with
the same principal. `423` on password change means "locked due to too many requests"; on this
API throttling is signalled as 423 or 429 with no `Retry-After`, so back off on your own clock.
