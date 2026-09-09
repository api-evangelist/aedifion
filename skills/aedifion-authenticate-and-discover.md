---
name: Authenticate and discover a building
description: Obtain a token for the aedifion HTTP API, list the buildings you can reach, and locate datapoints by semantic meaning rather than by protocol identifier.
api: openapi/aedifion-openapi.yml
operations: [get_user, get_user_projects, get_project, get_project_components, get_project_datapoints_bytag, get_project_datapoints_by_page, get_datapoint]
---

# Authenticate and discover a building

Base URL: `https://api.aedifion.io` — every path is prefixed `/v2`. Dedicated customers are on
`https://api.<REALM>.aedifion.io`.

Note the published spec declares `servers: [{"url": ""}]`, so a generated client will have no
host configured. Set the base URL explicitly.

## 1. Get a token

Prefer OpenID Connect. The issuer is the aedifion Keycloak realm:

```
POST https://auth.aedifion.io/realms/aedifion/protocol/openid-connect/token
grant_type=password&client_id=<client>&username=<email>&password=<password>
```

Send it as `Authorization: Bearer <access_token>` on every call.

HTTP Basic (`Authorization: Basic base64(email:password)`) also works but aedifion documents it
as legacy and says it "may be deprecated in future". Do not build on it.

Confirm who you are with `get_user`.

## 2. List the buildings you can reach

`get_user_projects` returns the projects (buildings/sites) your principal can access.
`get_project` gives one project's detail — address, latitude/longitude, `handle`, and
`load_balancing_group`. Keep the `handle` and `load_balancing_group`: they are the MQTT and
Kafka topic components if you later stream from this building.

For a portfolio rollup across buildings use `get_projects_kpis_aggregation`.

## 3. Find datapoints by meaning, not by name

This is the step people get wrong. Datapoint identifiers are protocol-shaped strings — the
spec's own example is `bacnet100-4120-CO2`, which encodes the bus and device address. **Never
guess or construct a `dataPointID`.**

Resolve points semantically instead:

- `get_project_components` — the semantic components in the building (boilers, pumps, air
  handling units). Components have pins; a datapoint mapped to a pin has a known role.
- `get_project_datapoints_bytag` — find datapoints by tag. Tags carry a `confirmed` flag and a
  `probability`, because some are AI-proposed and awaiting human confirmation. Prefer
  `confirmed` tags when acting on the result.
- `get_project_datapoints_by_page` — paged enumeration when you need everything. Use `page` and
  `per_page`; the response carries `PaginationMeta` with `total_pages`.
- `get_datapoint` — detail for one point.

## Conventions that apply throughout

- Errors are `application/json` with `{error, success, operation, details}` — **not** RFC 9457
  problem+json. The cause is free text in `error`; there is no error code to match on.
- `401` is declared on 175 of 208 operations. On a 401, re-authenticate before retrying.
- A `404` can mean "not visible to your role" as well as "does not exist" — role-based access
  control scopes projects and individual datapoints.
- Many operations accept `units_system` and `currency_system`, and 27 accept a language
  parameter. Ask the API for the units you want rather than converting building-engineering
  units yourself.
- There are no `ETag`, `If-Match` or request-id headers.
