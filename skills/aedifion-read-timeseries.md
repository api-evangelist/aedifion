---
name: Read and import building timeseries
description: Pull observation history for a datapoint over a time window, and bulk-import timeseries into a project via CSV or InfluxDB line protocol.
api: openapi/aedifion-openapi.yml
operations: [get_datapoint_timeseries, post_project_import_timeseries, get_project_datapoints_bytag]
---

# Read and import building timeseries

Assumes you have a token and a resolved `dataPointID` — see
`aedifion-authenticate-and-discover.md`.

## Reading observations

`get_datapoint_timeseries` takes `project_id`, `dataPointID`, and a time window:

- `start` and `end` are datetimes. If `start` is given without `end`, you get the first `max`
  observations after `start`.
- `closed_interval` controls boundary inclusion.
- `units_system` converts engineering units server-side.

Timeseries reads are **windowed, not paged** — there is no cursor. Ask for a bounded interval
and iterate the window yourself for long histories. Keep windows modest; there is no published
response-size limit and no rate-limit header to guide you.

A `422` here usually means a unit problem, not a syntax problem: the spec says it is returned
when "at least one datapoint could not be converted, e.g., due to the missing `units` tag or
the requested unit conversion is not supported." Fix the tag or drop `units_system`.

## Importing observations

`post_project_import_timeseries` uploads a file as `multipart/form-data`:

- `format` — `csv` or `influx_line_protocol` (both are enum values in the spec).
- `time_precision` — `s`, `ms`, `u` or `n`, default `s`. **Only applies to
  `influx_line_protocol`.**
- `on_error` — `continue` or `abort`. Choose deliberately: `continue` will partially import.

There is no idempotency key. A retried import is a second import. If a request times out,
verify with `get_datapoint_timeseries` over the same window before re-sending.

## Streaming instead

For live data, do not poll this endpoint. aedifion runs an MQTT broker at `mqtt.aedifion.io`
(TLS only, 8883 native / 9001 WebSockets) carrying the same observations in InfluxDB line
protocol on topic `<load-balancing-group>/<project-handle>`. See
`asyncapi/aedifion-mqtt-asyncapi.yml`. MQTT credentials are minted through the HTTP API's MQTT
user-management endpoints.
