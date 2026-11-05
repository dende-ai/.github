---
name: dende-plan
description: Write the plan comment of a Dende issue before code, and the proposal of a big change. Use when an issue has no plan comment yet, when the plan changes, or when you see the need for a big change.
---

# Plan comment

Post one comment on the Linear issue before code. Every plan passes the same points, in this order.
Every line stays. Write `none` where nothing applies, except in Done where, Files and PRs.

```markdown
**Plan** · protocol <revision from AGENT_PROTOCOL.md> · kind <api · web · lambda · job · package · infra · database · workflows>

| # | Input | Expected output |
| --- | --- | --- |
| 1 | <state and action> | <observable result> |

**Interface**: <what a caller or a user sees>
**Tracking**: <event, properties, transport, the insight that reads it>
**Alarms**: <the alarm that fires when this fails, its threshold, where it lives>
**Performance**: <the budget, the measurement, the production row count of each table that the change touches>
**Data**: <schema object, migration order, backfill, lock>
**Linked**: <repository: issue, for each change outside this repository>
**Scope**: <what you do not build, and why>
**Done where**: <environment · the examples that run there, in words · the evidence>
**Files**: <paths to change>
**PRs**: <one, or the sequence that the owner asked for>
**Question**: <one owner decision, with your recommendation, its reason and what happens on yes>
```

- At most 12 rows, one row per test. At least one row shows a refusal and one a failure, or the plan says why
  the change has none.
- If the issue has no examples, draft the rows and ask for the OK on them.
- Take each default from the section of the repository's kind below, and each name (route, screen, event) from
  the issue. Do not ask the owner for the threshold of a new alarm or budget, an environment or an event name.
- The fixture of the performance test holds the production row count of each table that the change reads or
  writes. A smaller fixture is the Question.
- A failure that an alarm watches logs at the level that the alarm matches.
- One Question per plan. A second owner decision is its own issue.
- For a new pattern, post the design note of `AGENT_PROTOCOL.md` instead of this form, then stop and ask.
- Progress goes in later comments, not in the description.

## Production change

When Done where names PRO, add three lines to the plan:

```markdown
**Fails halfway**: <what the system shows and does>
**Undo**: <the command or step>
**Signal**: <the metric, log line or check that shows it worked>
```

## Big change

A big change is a new schema object, route, capability, public interface, service, queue, worker, scheduler,
repository or dependency. Propose it in a second comment, not as the Question, as soon as you see the need.
At plan time, post it together with the plan:

```markdown
**Big change**: <name>
**Without it**: <the way without it and why it is not enough>
**Production facts**: <row counts, query plans, traffic; name each fact that you cannot read>
```

The plan does not wait for the answer. Code on the big change starts only when the owner asked for it by name
or said yes to this comment. Write the answer and its date in the issue.

# Kinds

Use the section of the repository's kind. A line that the section does not name holds what the issue needs,
or `none`. A repository that fits no kind uses the plan lines without defaults. A template has the kind of the repositories that it generates; its Done where is a generated
repository that passes its check command.

## api: api-web V3

- **Interface**: the diff of `contracts/v3/openapi.json`, additive or breaking; a public route also in
  `security/v3/public-routes.json`.
- **Tracking**: `none`. A server-side event needs the V3 analytics port, which is a big change.
- **Alarms**: the `api_class` of the route in `observability/betterstack/v3/alerts.json`. A user-critical route
  adds a p95 above 2 s line and an any-5xx line, routed to incidents-p2.
- **Performance**: the SLO of the `api_class`; a `QueryContract` with its `QueryPlan` budget for each statement.
- **Data**: the database issue, and the migration revision that DEV needs before the code merges.
- **Linked**: database; the consumers of the route.
- **Done where**: the DEV QA table; the PRO release; the Betterstack signal for 7 days.

## web: web-app

- **Interface**: the Figma link of each screen and breakpoint; components from `src/components/ui/`; the states
  loading, empty, error and denied as rows.
- **Tracking**: the event in `analytics/events.ts` and `tracker.ts` of the feature, named
  `<feature>_<object>_<past participle>` with one prefix per feature; `trackProduct` for a product event, `track`
  for a conversion; a tracker test; the insight that reads it. No event outside the typed catalog.
- **Alarms**: Sentry `pro`.
- **Performance**: the bundle delta of the route from `vite build`, at most 30 kB gzipped.
- **Linked**: the api-web endpoint; the GTM repository for a conversion.
- **Done where**: one screenshot per state, attached to the issue; the DEV QA table; the PRO release.

## lambda

- **Interface**: the event source and its recorded fixture; the batch size; a partial failure.
- **Alarms**: Errors ≥ 1 and Throttles ≥ 1 per 300 s in `lambda_monitoring.tfvars`; queue depth above 100 and
  DLQ above 10; a Betterstack sustained-errors alert, routed to incidents-p1.
- **Performance**: timeout, memory and concurrency; the cost at the expected volume.
- **Linked**: infrastructure, for the trigger, the IAM statements and the alarms.
- **Done where**: the first real event after the CI deploy, in DEV and in PRO.

## job: maintenance lanes

- **Interface**: the eligibility rule; the excluded classes; the three phases.
- **Alarms**: a Betterstack heartbeat with the period of the schedule and one period of grace, routed to
  incidents-p1; a RunningTaskCount alarm.
- **Performance**: the fixture profile against the production rows; batch size, lock timeout and SQL budget.
- **Data**: the archive is verified before a delete; the receipt row.
- **Linked**: infrastructure, for the schedule, the task definition and the heartbeat; the library pin.
- **Done where**: the first run after the owner activates the lane, and its receipts.

## package: dende-sql, dende-log, dende-events, dende-s3, dende-db-maintenance

- **Interface**: the public symbols added, changed or removed; the semver bump.
- **Linked**: each consumer and its pin bump issue, in order.
- **Done where**: the CI of each consumer, green on the tag.

## infra: infrastructure

- **Interface**: `variables_<domain>.tf` and `inputs/{dev,pro}`; the add, change and destroy counts of the plan.
  A destroy is the Question.
- **Alarms**: the alarm of each new resource, in the same PR.
- **Performance**: the monthly cost delta.
- **Linked**: the application issue that needs the resource.
- **Done where**: the DEV apply run; the PRO apply run; the next plan shows no changes.

## database

- **Interface**: the schema object; the up and the down migration.
- **Performance**: the `EXPLAIN` of each query that an index serves, on production row counts; `CONCURRENTLY`
  on a live table.
- **Data**: the migration before its readers; the backfill; retention and personal data.
- **Linked**: the repository that reads the object.
- **Done where**: the applied migration version in DEV and in PRO, read with the repository's migration tool.

## workflows: dende-cicd

- **Interface**: the inputs and outputs of each changed workflow or action.
- **Linked**: each consumer and its release tag bump.
- **Done where**: a consumer run at the new tag.
