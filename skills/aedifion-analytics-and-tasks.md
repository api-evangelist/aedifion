---
name: Run analytics and act on optimization findings
description: Read analytics results and KPIs for a building, aggregate them across a portfolio, and turn findings into tracked optimization tasks.
api: openapi/aedifion-openapi.yml
operations: [get_analysis_instances, get_instance_results, get_component_results, get_kpi_aggregation, get_projects_kpis_aggregation, get_project_tasks, post_task]
---

# Run analytics and act on optimization findings

aedifion.analytics detects inefficiencies in building plant and produces prioritized
recommendations. This skill reads those findings and turns them into work.

## Read the findings

- `get_analysis_instances` — the analytics instances configured for a project. An instance is
  one analysis function bound to one component instance.
- `get_instance_results` — results for an instance.
- `get_component_results` — results for one component in a project, which is usually the more
  natural question ("how is air handling unit 3 doing?").

An `AnalysisResult` carries more than a number:

- `kpi` and `summary_kpi` — the computed indicators.
- `interpretation` and `recommendation` — prose explaining what the numbers mean and what to
  do. Surface this to a human; do not paraphrase it away.
- `signal_color` — the triage signal.
- `plots` — chart payloads.
- `analytics_version` — **record this.** Analysis functions change between versions and the
  changelog shows algorithm changes landing regularly (2.1.17 changed how the Flexible Tariff
  analysis sources electricity pricing). A result is only comparable to another result from the
  same version.

## Aggregate across the portfolio

- `get_kpi_aggregation` — KPIs aggregated for a project.
- `get_projects_kpis_aggregation` — across projects, for portfolio-level energy, cost and CO2
  reporting.

Pass `units_system` and `currency_system` so the numbers come back in the units you report in.

## Turn findings into work

- `get_project_tasks` — existing optimization tasks.
- `post_task` — create one.

A `Task` links back to the evidence: `analytics_result`, `componentinproject`, `plot_view`, plus
`assignee`, `reporter`, `priority`, `status` and `savings_potential`. Populate
`analytics_result` so the task carries its provenance — a task without its originating result
is an assertion nobody can check later.

## Discipline

- There is no idempotency key. Check `get_project_tasks` before creating a task, or you will
  create duplicates on retry.
- `delete_task` is permanent. There is no restore operation anywhere in this API.
- Savings figures are model output, not measurement. Report them as estimates and keep the
  `analytics_version` alongside.
