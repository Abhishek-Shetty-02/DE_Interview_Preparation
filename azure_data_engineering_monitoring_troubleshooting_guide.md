# Monitoring & Troubleshooting for Azure Data Engineering

## Azure Monitor + Log Analytics + Logic Apps — End-to-End Interview and Production Guide

**Purpose:** A practical reference for designing, implementing, operating, and troubleshooting production Azure data platforms. Examples cover Azure Data Factory (ADF), Azure Databricks, ADLS Gen2, Azure SQL, Synapse Analytics, Event Hubs, and related orchestration services.

**Core stack:** Azure Monitor, Log Analytics workspace, Kusto Query Language (KQL), diagnostic settings, alert rules, action groups, Logic Apps, Azure Activity Log, Azure Service Health, Azure Resource Health, dashboards/workbooks, and operational runbooks.

> **Implementation note:** Table availability, diagnostic categories, metric names, connector behavior, and permissions differ by resource type, region, SKU, and product version. Verify the current settings for each resource. Queries and code snippets are templates; test them against your workspace's actual tables and schemas before production use.

## Table of Contents

1. Executive summary and interview answer
2. Reference architecture
3. Monitoring principles and operating model
4. Azure Monitor in depth
5. Log Analytics in depth
6. KQL essentials
7. Azure Data Factory monitoring
8. Azure Databricks monitoring
9. ADLS Gen2, Azure SQL, Synapse, and Event Hubs monitoring
10. Logic Apps for alert enrichment and automation
11. Alert engineering and severity
12. End-to-end implementation runbook
13. Production KQL query cookbook
14. Troubleshooting methodology
15. Production incident scenarios
16. Reliability, security, cost, and governance
17. Testing, recovery, and disaster readiness
18. Interview questions and model answers
19. Advanced scenario-based questions
20. Implementation checklist and rollout plan
21. Glossary and official references
22. Troubleshooting playbooks and operational appendices

---

# 1. Executive summary and interview answer

## 1.1 Short interview answer

In a production Azure data engineering platform, I use **Azure Monitor as the central observability and alerting layer**, **Log Analytics as the searchable log store for cross-service investigation using KQL**, and **Logic Apps as the workflow automation layer for alert enrichment, routing, notifications, and approved remediation steps**.

I enable diagnostic settings on services such as ADF, Databricks, ADLS Gen2, Azure SQL, Synapse, and Event Hubs where supported, and route selected resource logs to a Log Analytics workspace. I monitor platform metrics and operational outcomes: pipeline failure rate, activity duration, data freshness, record-count variance, streaming lag, cluster/job failures, storage throttling, SQL timeouts, and SLA breaches. Azure Monitor alert rules evaluate metrics, logs, activity events, and service health signals. An action group invokes a Logic App or other notification/action endpoint. The Logic App normalizes the alert payload, enriches it with pipeline and ownership context, checks for duplicates, sends a concise notification to the appropriate team, and optionally opens an incident or runs a tightly controlled recovery workflow.

I do not rely on a single “pipeline succeeded” signal. I correlate orchestration runs, individual activity runs, compute logs, storage/SQL errors, and data-quality checks. Alerts are designed around actionable thresholds, severity, ownership, suppression, and runbooks. I validate the complete chain by generating controlled failures, verifying that logs arrive, the query returns expected records, the alert fires, the Logic App completes, and the notification contains enough context for an engineer to respond.

## 1.2 Component responsibilities

| Component | Responsibility | Data engineering example |
|---|---|---|
| Azure Monitor | Central telemetry, metrics, alerts, dashboards, and response | Alert on a critical pipeline failure or resource metric threshold |
| Log Analytics workspace | Stores supported logs for query and investigation | Correlate ADF errors with Databricks and storage failures |
| KQL | Query language for Log Analytics | Find failed pipeline runs and calculate failure rate |
| Diagnostic settings | Route supported resource logs/metrics to destinations | Send ADF activity and pipeline logs to Log Analytics |
| Alert rule | Evaluates a condition and creates an alert | Detect repeated failures within a defined time window |
| Action group | Reusable notifications/actions | Notify on-call and invoke a Logic App |
| Logic Apps | Workflow automation and integration | Enrich alert, notify Teams/email, create ticket, record outcome |
| Azure Activity Log | Subscription-level control-plane events | Investigate who changed a resource or role assignment |
| Service/Resource Health | Azure platform incident and resource state signals | Separate a platform incident from an application defect |
| Workbooks | Operational visualizations | Show failures, SLA misses, freshness, and volume trends |

## 1.3 Telemetry versus business correctness

A pipeline can finish successfully while producing incomplete or incorrect data. Monitor several layers:

1. **Infrastructure:** availability, capacity, network, throttling, service health.
2. **Execution:** pipeline/job/activity status, duration, retries, queue time, errors.
3. **Data movement:** source-to-target counts, rejected rows, file arrival, watermark movement.
4. **Data quality:** nulls, duplicates, schema drift, referential integrity, business-rule failures.
5. **Business SLA:** data available by deadline, critical tables refreshed, downstream reports ready.
6. **Response workflow:** correct owner notified, incident recorded, duplicate alerts controlled, recovery verified.

## 1.4 Operating loop

**Observe → Detect → Correlate → Diagnose → Respond → Recover → Validate → Learn.**

- Observe with metrics, resource logs, application logs, and data-quality measurements.
- Detect with alert rules tied to thresholds or known failure patterns.
- Correlate by resource, pipeline, run ID, time window, batch, and business domain.
- Diagnose with KQL, ADF run details, Databricks job/Spark UI, storage logs, and SQL diagnostics.
- Respond using an actionable runbook and a clearly assigned owner.
- Recover through retry, replay, checkpoint recovery, credential repair, capacity changes, or controlled backfill.
- Validate target data and downstream freshness before closing the incident.
- Learn by reducing repeat failures, tuning alerts, and improving tests.

---

# 2. Reference architecture

## 2.1 Logical architecture

```text
Sources: SQL / APIs / files / events / SaaS / on-premises
                         |
                         v
               Azure Data Factory
              (pipeline/activity logs)
                         |
             ADLS Gen2 <--> Databricks
                         |      |
                         |      +-- Delta Lake / Spark jobs
                         |
          Azure SQL / Synapse / Event Hubs
                         |
              Diagnostic settings
                         v
                  Azure Monitor
            metrics | logs | activity
                         |
              Log Analytics workspace
                   KQL / Workbooks
                         |
               Azure Monitor alert rules
                         |
                    Action group
                         |
                     Logic App
              /          |          \
          Teams/email  ITSM ticket  Approved automation
                         |
                On-call / operations
                         |
              Runbook and validation
```

## 2.2 Example production flow

Assume ADF ingests order data from a source database into ADLS, invokes Databricks transformations, and loads a curated Delta table for reporting.

1. ADF emits pipeline-run, activity-run, and trigger-run diagnostics where configured.
2. Databricks job/cluster telemetry is collected through supported diagnostics and job-level logging. Spark UI and driver/executor logs support deeper execution analysis.
3. ADLS diagnostics help identify authorization failures, throttling, request errors, and unexpected access patterns when relevant categories are enabled.
4. Custom pipeline metrics record expected source count, target count, rejected rows, watermark, and completion timestamp.
5. Azure Monitor evaluates service metrics and log-query alerts.
6. A critical alert invokes an action group that calls a Logic App.
7. The Logic App parses the alert, maps resource/pipeline to a support team, enriches the message, suppresses duplicates, and routes the incident.
8. An engineer uses the runbook to isolate the failing layer and recover safely.
9. Post-recovery validation checks counts, freshness, duplicate keys, and downstream availability.

## 2.3 Control plane, data plane, and business plane

| Plane | What to observe | Examples |
|---|---|---|
| Control plane | Resource configuration and management operations | Resource updates/deletes, role changes, diagnostic settings, deployments |
| Data plane | Actual reads, writes, executions, service requests | ADF activities, storage requests, SQL queries, streaming processing |
| Business/data plane | Completeness, correctness, readiness | Count reconciliation, freshness timestamp, failed business rules, report SLA |

A control-plane event may explain why a workload suddenly failed. A data-plane error identifies the failed operation. A business-plane check tells you whether a technically successful workload produced the expected result.

## 2.4 Design decisions to make early

- **Workspace strategy:** central workspace, separate workspaces by environment, or a hybrid model based on security, access, retention, and cost.
- **Environment separation:** prevent dev/test/prod data and alert routes from mixing accidentally.
- **Table mode:** use resource-specific tables where supported; some configurations still use `AzureDiagnostics`.
- **Retention:** align retention to incident response, audit, and compliance requirements; archive long-term data where appropriate.
- **Alert ownership:** every critical alert needs an owner, severity, runbook, and response target.
- **Correlation IDs:** pass pipeline name, pipeline run ID, activity name, batch ID, source system, and business date through logs.
- **Privacy:** do not place patient, payment, credential, or other sensitive values in alert payloads or free-text logs.
- **Resilience:** make notifications idempotent and provide a fallback for critical incidents.

---

# 3. Monitoring principles and operating model

## 3.1 SLI, SLO, and SLA

- **SLI (Service Level Indicator):** a measured property, such as the percentage of critical pipelines completing successfully or the age of the latest curated partition.
- **SLO (Service Level Objective):** a target for the SLI, such as 99% of daily pipelines completing before 06:00 local time.
- **SLA (Service Level Agreement):** a committed service level, if one exists contractually or organizationally.

Derive thresholds from business deadlines, performance baselines, variability, and recovery time—not just from what is easy to configure.

| Indicator | Example target | Alert behavior |
|---|---|---|
| Critical daily pipeline | Complete by 05:30 | Warn before deadline; critical after deadline |
| Curated freshness | Latest business date by 06:00 | Alert when freshness age exceeds objective |
| Ingestion success rate | At least 99% over rolling 7 days | Trend alert plus immediate critical alert for tier-1 feed failure |
| Source-to-target reconciliation | Within defined tolerance | Alert on unexplained variance, not expected filtering |
| Streaming lag | Below workload-specific threshold | Warning when sustained; critical near SLA/retention risk |
| Alert delivery | On-call receives critical incident within agreed minutes | Monitor workflow failures and maintain a secondary path |

These are examples. Define real thresholds with data owners and test against actual workload baselines.

## 3.2 Golden signals for data engineering

1. **Availability:** can the pipeline, cluster, storage endpoint, database, and downstream target be reached?
2. **Latency:** how long do ingestion, transformation, and end-to-end refresh take?
3. **Traffic/throughput:** rows, files, events, bytes, requests, and batches processed per time interval.
4. **Errors:** failed runs, rejected rows, exceptions, throttling, timeouts, and retries.
5. **Saturation:** compute capacity, concurrency limits, queueing, storage request limits, SQL pressure, or Event Hubs throughput limits.
6. **Freshness/completeness:** latest business timestamp, expected partitions, record-count reconciliation, and watermark movement.

## 3.3 Operational event contract

A consistent custom event can include:

```json
{
  "eventTimeUtc": "2026-10-17T04:20:00Z",
  "environment": "prod",
  "domain": "orders",
  "pipelineName": "pl_orders_ingestion",
  "pipelineRunId": "<run-id>",
  "activityName": "CopySourceToBronze",
  "batchId": "<batch-id>",
  "sourceSystem": "orders-db",
  "targetObject": "bronze.orders",
  "status": "Failed",
  "recordsRead": 100000,
  "recordsWritten": 0,
  "rejectedRecords": 0,
  "durationSeconds": 245,
  "errorCode": "<sanitized-code>",
  "correlationId": "<correlation-id>"
}
```

Use a stable schema and sanitize values. Do not log secrets, connection strings, tokens, raw sensitive payloads, or personal identifiers without an approved protected requirement. Prefer opaque IDs and secure links to authorized diagnostic views.

## 3.4 Monitoring maturity

- **Level 1 — Reactive:** engineers manually open each service portal after failure.
- **Level 2 — Centralized:** selected logs go to Log Analytics; basic alerts exist.
- **Level 3 — Correlated:** KQL queries combine run status, duration, and data-quality signals; owners and runbooks are defined.
- **Level 4 — Automated response:** Logic Apps enrich, route, and ticket incidents; safe retries are automated.
- **Level 5 — Proactive reliability:** baselines, trend analysis, deployment verification, game days, and recurring-incident prevention are routine.

Increase automation only when the failure mode is understood and the recovery action is safe, bounded, auditable, and idempotent.

---

# 4. Azure Monitor in depth

## 4.1 What Azure Monitor is

Azure Monitor is the monitoring and observability platform for Azure resources and applications. It brings together metrics, logs, traces, and events and provides tools for analysis, visualization, alerts, and response. The precise signals available depend on the resource and its instrumentation.

For data engineering, use Azure Monitor to answer:

- Did a service metric cross a threshold?
- Did a pipeline, trigger, or resource operation fail?
- Did an Azure platform incident coincide with the workload failure?
- Is the workload slower than its baseline?
- Which team should be notified, and what action should happen next?

## 4.2 Metrics versus logs

| Dimension | Metrics | Logs |
|---|---|---|
| Shape | Numeric time series with dimensions | Records/events with fields and detail |
| Best for | Fast thresholding, trends, capacity, availability | Root-cause investigation, joins, detailed errors, custom logic |
| Example | Event Hubs incoming messages; storage availability metric | Failed ADF activity with error code and run ID |
| Analysis | Metrics Explorer and supported metric alerts | Log Analytics using KQL |
| Alert example | Request errors rise above a threshold | More than five critical pipeline failures in ten minutes |
| Limitation | May lack contextual error text | Logs require collection configuration and may arrive with delay |

Use both. A metric can show that something degraded; logs often explain what happened. Not every metric is available for every SKU or resource.

## 4.3 Metrics, resource logs, Activity Log, traces, and events

- **Platform metrics:** numerical telemetry emitted by Azure resources, often available without routing resource logs to Log Analytics.
- **Resource logs:** detailed events emitted by a resource. They generally require a diagnostic setting to route them to a destination.
- **Azure Activity Log:** subscription-level control-plane events such as resource writes/deletes and service health events. Export it to Log Analytics when KQL correlation and longer retention are needed.
- **Application traces/logs:** application-specific diagnostic output, such as custom ETL events or application instrumentation.
- **Distributed traces:** correlated operations across components when tracing is instrumented and supported.
- **Service Health:** Azure platform incidents, planned maintenance, and advisories relevant to the subscription.
- **Resource Health:** the health state of a particular resource, where supported.

## 4.4 Diagnostic settings

Diagnostic settings tell a supported resource which log categories and metrics to send to destinations such as Log Analytics, Storage, or Event Hubs. They do not create logs the service does not emit. Select categories based on operational value, not simply “everything forever.”

1. Open the resource in the Azure portal and navigate to **Monitoring → Diagnostic settings** (navigation can vary).
2. Create or update a setting with a meaningful name, such as `diag-prod-adf-to-law`.
3. Select supported resource-log categories relevant to the workload.
4. Select metrics such as `AllMetrics` where applicable and useful.
5. Choose the intended Log Analytics workspace and/or archival destination.
6. Save, then generate or wait for an event and verify the correct table receives records.
7. Document category coverage, destination, owner, retention, and cost implications.

For ADF, Microsoft documents resource-specific tables including `ADFActivityRun`, `ADFPipelineRun`, and `ADFTriggerRun` when resource-specific mode is used. Older configurations may use `AzureDiagnostics`. Prefer resource-specific mode where supported and appropriate, but query the tables actually enabled in the environment.

**Important:** diagnostic data can take time to appear. A missing record immediately after enabling diagnostics is not conclusive evidence of failure. Check ingestion delay, selected categories, destination workspace, permissions, and table name.

## 4.5 Alert rule types

| Alert type | Suitable for | Example |
|---|---|---|
| Metric alert | Numerical resource signal | Storage availability drops or request errors rise |
| Log search alert | KQL query over log records | Critical ADF pipeline failed in the last ten minutes |
| Activity log alert | Management/control-plane events | Critical resource deleted or role assignment changed |
| Service Health alert | Azure platform incidents and maintenance | Relevant Azure service incident in the subscription/region |
| Resource Health alert | Resource health changes where supported | Resource enters an unavailable state |
| Dynamic threshold | Signals with meaningful historical patterns | Detect unusual metric behavior when static thresholds are inadequate |

Choose the alert type that matches the signal. A log alert is not a replacement for a metric alert when a metric provides a more direct or faster signal.

## 4.6 Action groups

An action group is a reusable set of notifications and actions attached to alert rules. It can notify people or invoke supported endpoints and automation such as a Logic App. Multiple alert rules can reuse an action group; do not assume actions execute in a particular order.

Suggested design:

- `ag-prod-data-critical`: on-call notification, incident ticket, Logic App workflow.
- `ag-prod-data-warning`: team channel/email with lower urgency.
- `ag-platform-service-health`: platform operations notification.
- `ag-nonprod-data`: development/test notification that cannot page production on-call.

Avoid putting every team and alert into one unstructured group. Use ownership-aware routing and avoid duplicate notifications from overlapping rules.

## 4.7 Workbooks and dashboards

A useful operations workbook should answer quickly:

- What failed recently?
- Which pipelines are late?
- Which data domains are stale?
- What is the failure rate by environment and source?
- Which activities are slow relative to baseline?
- Are errors clustered by dependency, code, region, or deployment?
- What changed before the first failure?

Suggested pages:

1. **Executive health:** critical pipelines, current SLA status, latest freshness, open critical incidents.
2. **Pipeline operations:** run status, duration percentiles, retries, failures, activity details.
3. **Data quality:** expected/actual counts, rejected records, duplicate rate, schema failures, watermark status.
4. **Dependencies:** service metrics, throttling, storage errors, SQL failures, service health.
5. **Alert quality:** alert counts by rule/severity, repeated incidents, notification workflow failures.

## 4.8 Common Azure Monitor mistakes

- Assuming resource logs are collected automatically without diagnostic settings.
- Sending every category to a high-cost destination without a use case or retention plan.
- Building alert rules on a table that is not enabled in the workspace.
- Using a broad query that fires for every row rather than one meaningful incident.
- Alerting on transient blips without a sustained-duration condition.
- Forgetting to monitor alert delivery itself.
- Treating pipeline success as proof of data completeness.
- Ignoring UTC versus business-local time.
- Relying on a Logic App without a fallback for critical incidents.

---

# 5. Log Analytics in depth

## 5.1 What a Log Analytics workspace provides

A Log Analytics workspace is a data store for supported log and trace data. Engineers use KQL to filter, aggregate, join, and correlate records from enabled sources. Workspaces also support alert rules, workbooks, and integrations with other Azure services.

Workspace decisions include which subscriptions/environments send logs, who can query which data, interactive retention and archival requirements, collection volume and cost, environment isolation, data residency, and table-level access.

## 5.2 Common tables for Azure data platforms

| Table | What it can contain | Typical use |
|---|---|---|
| `ADFActivityRun` | ADF activity execution records in resource-specific mode | Find failing copy, lookup, notebook, or stored-procedure activities |
| `ADFPipelineRun` | ADF pipeline-run records | Determine pipeline status and duration |
| `ADFTriggerRun` | ADF trigger execution records | Diagnose missed or failing trigger invocations |
| `AzureDiagnostics` | Legacy/consolidated resource diagnostic records, depending on configuration | Query older settings; schema varies by resource/category |
| `AzureActivity` | Exported Azure Activity Log events | Correlate resource configuration changes and control-plane operations |
| `Heartbeat` | Azure Monitor Agent heartbeat data, if collected | Determine whether an agent-managed machine is reporting |
| `AzureMetrics` | Metrics stored in logs when configured | Analyze metric records available in this table |
| `AppTraces`, `AppExceptions`, `AppRequests` | Application Insights telemetry where configured | Troubleshoot instrumented applications and API dependencies |

Table names and schemas vary by source, diagnostic mode, and configuration. Start with the **Tables** pane in the actual workspace, inspect sample records, and use `getschema` or `take` before adapting a query.

## 5.3 Workspace strategy

### Centralized workspace

**Advantages:** cross-service correlation, shared workbooks, simpler operations, centralized alert rules.

**Trade-offs:** access boundaries become more important; high-volume sources can dominate cost; environment separation and retention require deliberate design.

### Separate workspaces

**Advantages:** stronger environment separation, independent retention/access patterns, reduced risk of cross-environment exposure.

**Trade-offs:** cross-workspace investigations and dashboards require additional query design; duplicated configurations can drift.

### Practical recommendation

Choose boundaries based on security, environment, regulatory, and operating-model requirements. Do not select one workspace solely for convenience if it violates least privilege or retention needs. Use naming conventions, tags, RBAC, and policy to make the intended architecture visible.

## 5.4 Missing logs: investigation order

1. Was the relevant diagnostic category enabled?
2. Is the setting attached to the correct resource?
3. Is the destination workspace correct?
4. Is the event expected to be emitted by the resource?
5. Has enough time passed for delivery and ingestion?
6. Are you querying the correct table and time range?
7. Does the user have permission to read the workspace/table?
8. Is a filter excluding the event due to resource ID, status casing, or timestamp assumptions?
9. Is the resource using resource-specific mode or legacy `AzureDiagnostics`?
10. Is the destination or query affected by a service, permission, or configuration issue?

## 5.5 Retention and cost

Log Analytics costs depend on pricing model, volume, and retention. Manage cost by enabling only categories with a clear diagnostic/audit purpose; estimating volume before enabling verbose sources; logging compact structured events; using appropriate retention and archival; reviewing ingestion by table/resource; and avoiding duplicate routing unless required.

Do not shorten retention or disable a diagnostic category merely to reduce cost without checking operational, security, audit, and regulatory requirements.

## 5.6 Access control

Use least privilege. Separate the right to **read logs** from the right to **change alert rules**, **modify diagnostic settings**, and **execute remediation**. Exact permissions depend on the operation and resource scope. Use managed identities for Logic Apps where supported and avoid embedding credentials in workflow definitions or notifications.

---

# 6. KQL essentials

Kusto Query Language (KQL) is used to analyze Log Analytics data. The examples below are templates; column names may differ by table/schema. Inspect sample rows and adjust accordingly.

## 6.1 Core operators

| Operator/function | Purpose | Example |
|---|---|---|
| `where` | Filter records | `| where Status == "Failed"` |
| `project` | Select/rename columns | `| project TimeGenerated, PipelineName, Status` |
| `extend` | Add calculated columns | `| extend DurationMinutes = DurationMs / 60000.0` |
| `summarize` | Aggregate records | `| summarize Failures=count() by PipelineName` |
| `bin()` | Bucket timestamps | `| summarize count() by bin(TimeGenerated, 5m)` |
| `join` | Correlate tables | Join pipeline runs to activity runs by run identifier |
| `parse_json()` | Parse JSON stored in a string | `| extend p=parse_json(Properties)` |
| `make_set()` | Collect distinct values | `| summarize ErrorCodes=make_set(ErrorCode)` |
| `top` / `sort` | Rank or order results | `| top 20 by DurationMinutes desc` |
| `ago()` | Relative time filter | `| where TimeGenerated > ago(24h)` |
| `dcount()` | Approximate distinct count | `| summarize UniqueRuns=dcount(RunId)` |
| `arg_max()` | Return row with maximum value | `| summarize arg_max(TimeGenerated, *) by PipelineName` |

## 6.2 Inspect the table first

```kusto
ADFActivityRun
| take 10
```

```kusto
ADFActivityRun
| getschema
```

```kusto
ADFActivityRun
| where TimeGenerated > ago(2h)
| project TimeGenerated, PipelineName, ActivityName, Status, ErrorCode, ErrorMessage
| order by TimeGenerated desc
```

If a query fails, do not assume the service is not logging. Verify that the table exists and inspect its schema. Legacy `AzureDiagnostics` columns may have type suffixes such as `_s` or `_d`.

## 6.3 Time handling and query quality

Azure telemetry commonly uses UTC timestamps; business SLAs may use local time. Use explicit conversion for business deadlines. In alert rules, account for query period, evaluation frequency, ingestion delay, and late-arriving records.

- Filter by time early (`where TimeGenerated > ago(...)`).
- Filter by resource, environment, or pipeline before broad joins.
- Project only required columns.
- Avoid expensive joins over unbounded history.
- Use stable identifiers rather than display names where possible.
- Check output row count and sample results before creating an alert.
- Handle nulls and status casing using actual records.
- Keep lookback and evaluation frequency aligned with workload behavior.

---

# 7. Azure Data Factory monitoring

## 7.1 What to monitor

**Pipeline level:** run status, start/end time, duration, trigger type, parameters, pipeline run ID, parent/child runs, SLA deadline, business date, retries, recurring failures.

**Activity level:** activity name/type/status, duration, queue time where exposed, integration runtime, rows/files read and written, throughput, source/sink errors, connector codes, notebook/job outcome, retry and timeout behavior.

**Trigger level:** trigger state and last invocation, schedule/timezone, tumbling-window dependencies, event subscription, and file-arrival assumptions.

**Integration Runtime:** self-hosted IR node availability, connectivity, CPU/memory pressure, version/maintenance, network path, firewall, DNS, credential reachability, queueing, and concurrency.

## 7.2 Enable ADF diagnostics

1. Open the Data Factory resource in the Azure portal.
2. Open **Diagnostic settings** and add a setting.
3. Select Log Analytics as the destination.
4. Choose supported logs for pipeline, activity, and trigger investigations. Add SSIS categories only if SSIS workloads are used.
5. Select metrics where useful and supported.
6. Save and verify that records appear in `ADFPipelineRun`, `ADFActivityRun`, and `ADFTriggerRun` when resource-specific mode is enabled.
7. Test with a controlled failed run and compare portal details with the log record.

ADF diagnostics can use resource-specific tables or legacy `AzureDiagnostics`. A diagnostic setting is not retroactive; do not expect it to populate events from before it was configured.

## 7.3 ADF operational dashboard

Recommended widgets: pipeline runs by status; failures by pipeline/source; top pipelines by duration; activity failure counts; critical SLA status; failed or unexpectedly absent triggers; retries/error codes; source-to-target count variance; latest successful watermark and freshness.

## 7.4 ADF KQL templates

**Recent failed pipeline runs** (adjust columns to your schema):

```kusto
ADFPipelineRun
| where TimeGenerated > ago(24h)
| where Status =~ "Failed"
| project TimeGenerated, PipelineName, RunId, Status, FailureType, ErrorMessage
| order by TimeGenerated desc
```

**Failed activity runs:**

```kusto
ADFActivityRun
| where TimeGenerated > ago(24h)
| where Status =~ "Failed"
| project TimeGenerated, PipelineName, ActivityName, ActivityType,
          Status, ErrorCode, ErrorMessage, RunId
| order by TimeGenerated desc
```

**Failure count by pipeline:**

```kusto
ADFPipelineRun
| where TimeGenerated > ago(7d)
| summarize TotalRuns=count(),
            FailedRuns=countif(Status =~ "Failed"),
            CancelledRuns=countif(Status =~ "Cancelled")
  by PipelineName
| extend FailurePct = round(100.0 * FailedRuns / TotalRuns, 2)
| order by FailedRuns desc
```

**Pipeline duration percentiles:**

```kusto
ADFPipelineRun
| where TimeGenerated > ago(7d)
| where Status =~ "Succeeded"
| extend DurationMinutes = datetime_diff('second', End, Start) / 60.0
| summarize AvgMinutes=avg(DurationMinutes),
            P95Minutes=percentile(DurationMinutes, 95),
            MaxMinutes=max(DurationMinutes), Runs=count()
  by PipelineName
| order by P95Minutes desc
```

Some tables expose a precomputed duration column or use different timestamp names. Inspect the schema and replace `Start`/`End` as needed.

## 7.5 Custom pipeline logging

A production pipeline should log business-level milestones in addition to built-in ADF telemetry:

1. Generate a batch ID at pipeline start.
2. Record source object, extraction watermark, expected partition/date, and start time.
3. Record source rows/files read.
4. Record rejected records and reason counts.
5. Record target rows written and target partition/table.
6. Commit the watermark only after target write and validations succeed.
7. Record end time, status, and sanitized error code.
8. Alert on failed milestones or incomplete batches.

A custom control table in Azure SQL or Delta Lake supports durable batch state and restartability; it does not replace Azure Monitor. The control table supports reconciliation, while Monitor/Log Analytics supports detection and investigation.

## 7.6 ADF failure categories

| Category | Common symptoms | First checks |
|---|---|---|
| Authentication/authorization | 401/403, login failure | Managed identity/service principal, role assignments, secret expiry, target permissions |
| Network/connectivity | Timeout, DNS failure | Firewall, private endpoint, DNS, IR network path, NSG/routes |
| Schema/data | Conversion error, missing column, malformed file | Source schema, mapping, drift, encoding, nullability, corrupt records |
| Throttling/capacity | 429, resource busy, slow copy | Service limits, concurrency, source/sink capacity, retry/backoff |
| Integration Runtime | Queued activities, offline self-hosted IR | Node status, service health, network, runtime version, CPU/memory |
| Orchestration logic | Wrong branch, missing parameter | Expression, actual activity output shape, parameters, dependencies |
| Sink/write conflict | Duplicate keys, lock conflict, rejected rows | Idempotency, merge keys, transaction behavior, concurrent writers |
| Trigger | Expected pipeline never started | Trigger state, timezone, event subscription, schedule/window dependencies |

## 7.7 ADF troubleshooting sequence

1. Locate the pipeline run ID and confirm status in ADF Monitor.
2. Open the failed activity and read the full error code/message and retry history.
3. Determine whether the failure is orchestration, integration runtime, source, transformation, or sink.
4. Check the same time window in Log Analytics for correlated failures and configuration changes.
5. Validate credentials, networking, schema, input files, source query, and destination state.
6. Decide whether retry is safe. Confirm idempotency and whether partial writes occurred.
7. Retry only the failed activity or rerun the pipeline according to the recovery design.
8. Reconcile source/target counts, watermark, duplicates, and downstream freshness.
9. Add a permanent fix and record the incident cause.

---

# 8. Azure Databricks monitoring

## 8.1 Monitoring layers

Azure Databricks needs Azure resource-level telemetry and Databricks-native execution details.

- **Azure resource diagnostics:** workspace logs supported by diagnostic settings, sent to Log Analytics, Event Hubs, or Storage. Availability and required plan/features depend on log type; audit-log delivery has documented plan requirements.
- **Job/workflow runs:** status, duration, task failures, retries, parameters, job IDs, and run IDs.
- **Cluster/compute:** start failures, termination reasons, autoscaling, node allocation, resource pressure, runtime compatibility.
- **Spark execution:** stages, tasks, shuffle, skew, spills, executor loss, GC, query plans, and failed tasks via Spark UI and driver/executor logs.
- **Delta Lake:** transaction conflicts, schema enforcement/evolution, merge performance, small files, and streaming checkpoints.
- **Data correctness:** record counts, duplicate keys, invalid records, late events, and target freshness.

Azure Monitor does not replace Spark UI, query plans, or Databricks job run details. Use Monitor for central detection/correlation and native Databricks tools for execution-level diagnosis.

## 8.2 Enable supported diagnostics

1. Confirm the required workspace plan/features for desired diagnostic logs.
2. Open the Azure Databricks workspace resource in the Azure portal.
3. Configure diagnostic settings and select supported categories.
4. Send selected logs to the intended workspace or approved destination.
5. Confirm events are present and identify the table/schema used.
6. Keep detailed job parameters and business identifiers in structured job logs or a controlled run ledger where appropriate.
7. Validate access restrictions and redact sensitive values.

Diagnostic logs do not necessarily include every event available through Databricks system tables or the native UI. Verify current documentation for the log source and workspace plan.

## 8.3 Key Databricks indicators

| Indicator | What it can reveal | Follow-up |
|---|---|---|
| Job failure rate | Code, dependency, permission, or compute failures | Inspect task error and run output |
| Job duration/P95 | Data growth, skew, regressions, slow dependencies | Compare stages, source volume, query plan, cluster size |
| Cluster start failure | Quota, capacity, configuration, network, policy | Inspect termination reason and permissions |
| Executor loss / failed tasks | Memory pressure, node issue, unstable workload | Review executor logs, shuffle, partition sizing |
| Spill/shuffle growth | Join/aggregation pressure or skew | Inspect query plan, partitioning, broadcast feasibility |
| Streaming input/output rate | Throughput imbalance | Compare source rate, processing rate, trigger interval |
| Streaming batch duration | Backlog and processing delay | Tune workload, micro-batch size, partitioning, writes |
| Checkpoint/restart errors | Invalid path, access, concurrent query, incompatible state | Verify checkpoint isolation and restart strategy |
| Delta transaction conflicts | Concurrent writes or overlapping merges | Review concurrency and write predicates |
| Small-file growth | Inefficient reads and write overhead | Review partitioning and compaction strategy |

## 8.4 Databricks diagnostic troubleshooting

If events do not appear in Log Analytics, verify that logging is supported for the workspace/feature/plan; check the setting is attached to the correct workspace and destination; confirm categories and ingestion delay; inspect actual tables; distinguish diagnostic logs from system tables/native job logs; check permissions; and verify that the delivery path covers the event types required.

## 8.5 Troubleshooting a slow Spark job

1. Compare current runtime with historical median/P95 and input volume.
2. Identify the slowest task/stage in Spark UI.
3. Check skew: a small number of tasks run much longer or process disproportionately large partitions.
4. Inspect shuffle read/write, spill, garbage collection, and executor failures.
5. Review joins: broadcast only when the smaller side is genuinely small enough for executor memory; otherwise consider sort-merge join and appropriate partitioning.
6. Check partition count and file sizes; too few partitions underutilize compute, while too many tiny tasks create scheduling overhead.
7. Review Delta writes, merge predicates, partition pruning, and concurrent writers.
8. Confirm whether upstream latency or downstream sink throttling is the real bottleneck.
9. Apply one change at a time and compare throughput, duration, and cost.
10. Add a regression baseline and alert for sustained degradation.

## 8.6 Streaming monitoring

For Structured Streaming or Event Hubs/Kafka ingestion, monitor input rows/bytes per trigger, processing rate, trigger duration, source lag/backlog, watermark progression, late-event/drop counts, checkpoint errors, batch failures, restarts, output commits, target freshness, and replay/deduplication behavior.

A stream can remain “running” while falling behind. Alert on lag and freshness, not only process state. Thresholds must account for normal bursts, retention limits, and business latency objectives.

---

# 9. ADLS Gen2, Azure SQL, Synapse, and Event Hubs monitoring

## 9.1 ADLS Gen2 / Azure Storage

Monitor supported metrics and logs for request failures, latency, availability, throttling, authorization errors, and unexpected access patterns. Configure diagnostics for the relevant storage service/resource scope; categories and table names vary.

| Symptom | Likely causes | Checks |
|---|---|---|
| 403/AuthorizationPermissionMismatch | RBAC, ACL, identity, path permissions | Effective identity, data-plane role, directory ACL traversal, network rules |
| 404/path not found | Incorrect path, case mismatch, missing partition/file | Exact path, upstream arrival, date format, file naming |
| 409/conflict | Lease/rename/account-operation conflict | Full error code, operation type, account features, concurrent writes |
| 429/503/throttling | Request burst, hot path, service capacity | Request rate, concurrency, retry/backoff, file layout |
| High latency | Small files, network, service pressure, large listing | File count, listing scope, request metrics, path layout |
| Missing logs | Diagnostics disabled or wrong scope | Setting, categories, destination, expected delay |

Operational patterns:

- Use stable folder conventions such as `/raw/<source>/<entity>/ingest_date=YYYY-MM-DD/`.
- Record file arrival and ingestion status in a control ledger.
- Validate file size, schema, count, and checksum where required.
- Make writes restartable and avoid exposing partial output as complete data.
- Alert on missing expected files after the agreed arrival deadline, not because a folder is empty at an arbitrary time.

## 9.2 Azure SQL Database / SQL Server

Monitor supported metrics and diagnostics for connection failures, query duration, deadlocks, timeouts, CPU/DTU/vCore pressure, storage pressure, failed logins, and workload contention. Exact signals depend on the SQL offering and configuration.

Troubleshooting order:

1. Capture sanitized SQL error number, operation, target database, and timestamp.
2. Check service health and database utilization around the incident.
3. Check network/private endpoint/firewall and DNS.
4. Check managed identity/login permissions and token/credential validity.
5. Inspect blocking, deadlocks, query plans, indexes, and parameters when performance is involved.
6. Check connection pooling, concurrency, transaction duration, and retry strategy.
7. Validate whether a timed-out write committed before the client lost its response.
8. Reconcile target data before retrying non-idempotent inserts.

Avoid logging full SQL parameter values when sensitive. Prefer operation names, query IDs, error numbers, duration, and correlation identifiers.

## 9.3 Azure Synapse Analytics

Monitor pipeline orchestration, SQL pool health/utilization, query duration, queued requests, failed queries, concurrency, data movement, and supported storage/workload signals.

Common failure classes include login/permission failures, SQL timeouts/resource pressure, type conversion/schema mismatch, distribution skew/data movement overhead, concurrent workload saturation, external table/storage authorization failure, and upstream ADF copy failure.

Use Synapse monitoring views and SQL diagnostics for query-level detail; use Azure Monitor and Log Analytics to correlate symptoms with ADF, storage, and deployment events.

## 9.4 Azure Event Hubs / Kafka-compatible ingestion

Monitor incoming requests/messages/bytes, outgoing traffic, throttled requests, server errors, consumer lag where exposed, partition utilization, and downstream processing latency.

Separate layers:

- **Producer:** authentication, endpoint, TLS, send errors, retries.
- **Event Hubs:** namespace/network access, throughput/capacity, partitioning, retention, throttling.
- **Consumer:** group/offset progress, partition assignment, lag, checkpoint/offset commit, restart loops.
- **Sink:** processing rate, write failures, idempotency, data freshness.

A healthy namespace does not guarantee a healthy consumer. If arrival rate exceeds processing capacity, backlog grows. Compare incoming rate, processing rate, lag trend, and sink latency before scaling consumers or changing partition count.

---

# 10. Logic Apps for alert enrichment and incident automation

## 10.1 Why Logic Apps

Logic Apps provides a workflow layer for integrating monitoring alerts with messaging, ticketing, HTTP endpoints, approvals, and other services. It is useful when the default alert notification is not sufficiently actionable or an alert needs routing and enrichment.

Typical uses:

- Format alert details into a consistent message.
- Route incidents by environment, resource tag, domain, or severity.
- Add ownership, runbook, dashboard, and affected-pipeline context.
- Query an approved source to enrich the alert with recent failure details.
- Create/update an ITSM incident or work item.
- Notify Teams/email/on-call endpoints.
- Deduplicate repeated notifications using alert ID, resource ID, and time window.
- Start an approved recovery workflow after validating failure type and safety conditions.
- Write an audit record for the notification/remediation outcome.

## 10.2 Recommended workflow architecture

```text
Azure Monitor alert
       |
       v
Action group invokes Logic App
       |
       v
Parse and validate payload
       |
       v
Normalize alert fields: resource, rule, severity, time, alert ID
       |
       v
Deduplicate / correlate incident
       |
       v
Enrich from approved metadata: owner, domain, runbook, dashboard
       |
       v
Severity and environment routing
       +---- Critical: on-call + incident ticket
       +---- Warning: team channel/email
       +---- Informational: dashboard / daily summary
       |
       v
Optional approved remediation
       |
       v
Record outcome, failures, and correlation ID
```

## 10.3 Triggering a Logic App from Azure Monitor

1. Create a Logic App workflow using a supported trigger pattern for Azure Monitor/action-group integration.
2. Design it to accept the alert payload and validate required fields.
3. Create an Azure Monitor action group that invokes the Logic App using the supported action type.
4. Attach the action group to a test alert rule.
5. Generate a controlled alert and inspect alert history and Logic App run history.
6. Verify routing, formatting, and behavior when an optional field is missing.
7. Configure access, secure connections, retry policies, and failure notification.

The precise trigger and payload shape depend on the integration selected. Inspect a real test payload before hard-coding JSON paths. Azure Monitor's documented Logic Apps integration supports custom alert notifications and enrichment; follow current product documentation for the chosen alert type.

## 10.4 Example normalized alert contract

```json
{
  "alertId": "<stable-alert-id>",
  "alertRule": "prod-adf-critical-failure",
  "severity": "Sev1",
  "environment": "prod",
  "resourceId": "<azure-resource-id>",
  "resourceName": "adf-prod-orders",
  "pipelineName": "pl_orders_daily",
  "pipelineRunId": "<run-id-if-available>",
  "firedAtUtc": "2026-10-17T04:20:00Z",
  "ownerTeam": "Data Platform On-Call",
  "runbookUrl": "<approved-runbook-url>",
  "dashboardUrl": "<approved-dashboard-url>",
  "summary": "Critical data pipeline failed"
}
```

This is an internal design example, not a guaranteed native Azure Monitor payload. Map the actual incoming schema to this contract and handle missing optional values safely.

## 10.5 Example notification format

```text
[SEV1][PROD] Critical ADF pipeline failure
Resource: adf-prod-orders
Pipeline: pl_orders_daily
Run ID: <run-id>
Detected at (UTC): 2026-10-17 04:20
Impact: Orders curated dataset may miss the reporting SLA.
First checks: Open ADF run details; inspect failed activity and linked Log Analytics query.
Runbook: <runbook link>
Dashboard: <workbook link>
Owner: Data Platform On-Call
Correlation ID: <correlation-id>
```

The message should tell the receiver what failed, where, when, likely impact, and what to do next. Avoid credentials, raw payloads, and unnecessary sensitive identifiers.

## 10.6 Branching and routing logic

- If environment is `prod` and severity is critical, page on-call and create/update an incident.
- If severity is warning, notify the owning engineering team and include a dashboard link.
- If the alert duplicates an open incident, append context rather than creating another ticket.
- If ownership is unknown, route to a central triage queue and flag missing metadata.
- If the payload is malformed, record a workflow error and send a fallback notification with a safe correlation ID.
- If recovery is eligible, validate preconditions; otherwise route to a human.

## 10.7 Idempotency and duplicate suppression

Azure Monitor can evaluate alert rules repeatedly, and one underlying incident can create multiple related alerts. Logic Apps should not blindly create a ticket or restart a pipeline on every invocation.

Use a deduplication key such as:

`hash(environment + resourceId + alertRule + pipelineRunId-or-time-bucket)`

Store the key in a durable store with an expiry/closure strategy. When a duplicate arrives, update the existing incident or add a note. Ensure concurrency handling prevents simultaneous workflow runs from creating duplicate tickets.

## 10.8 Safe automated remediation

Good candidates for controlled automation:

- Retry a transient, idempotent operation after confirming no conflicting run exists.
- Re-run a specific pipeline after verifying source data and target write behavior.
- Notify an owner and request approval for a high-impact action.
- Run a validation query and attach the result to an incident.

Do not automate deletion of production data, checkpoint resets without understanding replay implications, broad permission changes, table drop/recreate operations, or expensive scaling without strong safeguards.

A remediation workflow should check eligibility, use a managed identity with minimum permissions, enforce timeouts/retry limits, log the action, verify the result, and stop when the outcome is uncertain.

## 10.9 Logic Apps failure handling

- Configure bounded retries for transient connector failures.
- Use structured error handling to separate success and failure paths.
- Record failed workflow run IDs and the original alert ID.
- Prevent a notification failure from silently losing the incident.
- Create a fallback route for critical failures.
- Monitor Logic App failures and run duration with Azure Monitor.
- Test connector authentication expiry, rate limits, unavailable endpoints, malformed payloads, and duplicate delivery.

---

# 11. Alert engineering and severity model

## 11.1 Design alerts that lead to action

Every alert rule should define:

- **Signal:** metric, log, or event being evaluated.
- **Scope:** resource, subscription, workspace, pipeline, or business domain.
- **Condition:** threshold, count, query result, or state transition.
- **Evaluation window/frequency:** detection speed and potential noise.
- **Severity:** operational impact and urgency.
- **Owner:** accountable response team.
- **Action:** notification, ticket, workflow, or automation.
- **Runbook:** first checks, safe recovery, validation, escalation.
- **Suppression/deduplication:** treatment of repeat evaluations and related signals.
- **Test:** evidence that it fires when expected and stays quiet for known healthy behavior.

## 11.2 Example severity model

| Severity | Definition | Example | Response pattern |
|---|---|---|---|
| Sev 1 / Critical | Major impact or imminent critical SLA breach | Tier-1 dataset unavailable before reporting deadline | Page on-call, create incident, immediate triage |
| Sev 2 / High | Important workload impaired with limited workaround | Several production pipelines failing for one domain | Notify owner, ticket, prompt response |
| Sev 3 / Medium | Degradation with workaround or non-critical feed impact | Repeated retries, moderate latency increase | Team notification and planned investigation |
| Sev 4 / Low | Informational or trend issue | Gradual duration increase, non-prod failure | Dashboard/digest and backlog item |

Use the organization's incident policy; this is an example, not a universal standard.

## 11.3 Threshold selection

Combine:

- **Business deadline:** alert when remaining recovery time becomes smaller than expected recovery time.
- **Absolute threshold:** no successful run for a defined period.
- **Relative threshold:** duration/failure rate is materially worse than baseline.
- **Persistence:** signal remains above threshold across evaluations.
- **Volume-aware thresholds:** normalize failures by total runs/events when workload volume varies.
- **Suppression:** avoid paging for approved maintenance or an already-open incident while preserving evidence.

Avoid overfitting to a few historical days. Revisit thresholds after source volume, architecture, or schedules change.

## 11.4 Alert on absence, not only failure

Important data pipeline incidents may not produce an explicit failure:

- A scheduled pipeline never started.
- A source file did not arrive.
- A trigger was disabled.
- A streaming job is alive but processing no new data.
- A pipeline succeeded but wrote zero records unexpectedly.
- The target's latest business date is stale.

Use freshness checks, expected-run calendars, source-arrival checks, and heartbeat/last-success timestamps. Account for weekends, holidays, and low-volume periods.

## 11.5 Alert fatigue controls

- Group related alerts under one incident where possible.
- Separate root-cause alerts from downstream symptoms.
- Use meaningful time windows and sustained conditions.
- Set ownership and runbooks before enabling paging.
- Review noisy rules weekly during rollout, then periodically.
- Track duplicate rate, false-positive rate, alert-to-incident conversion, and time to acknowledge.
- Do not suppress all alerts for a resource merely because one rule is noisy; fix the rule or route.

---

# 12. End-to-end implementation runbook

## 12.1 Phase 1 — Inventory and requirements

1. Inventory resources by environment, domain, owner, and criticality.
2. Map each pipeline from source through transformation to target and reporting.
3. Identify supported metrics, diagnostic categories, native monitoring surfaces, and gaps.
4. Define SLIs, SLOs, SLA deadlines, and business calendars.
5. Agree on severity, escalation path, support hours, and ticketing requirements.
6. Decide retention, access control, workspace boundaries, and cost constraints.
7. Define correlation fields and naming conventions.

Deliverables: resource inventory, monitoring matrix, ownership map, alert catalogue, retention/access decision, and runbook template.

## 12.2 Phase 2 — Configure Log Analytics

1. Select workspace architecture based on security and operating model.
2. Configure RBAC and, where appropriate, table-level access.
3. Set retention and archival policies.
4. Establish tags and naming conventions.
5. Create verification queries for each supported log source.
6. Record expected tables and columns in a monitoring catalogue.

## 12.3 Phase 3 — Enable diagnostics

For each supported resource:

1. Open diagnostic settings.
2. Select relevant resource-log categories and metrics.
3. Route to the correct workspace/destination.
4. Save and wait for telemetry.
5. Trigger a controlled operation or failure.
6. Verify record arrival, timestamps, identifiers, and schema.
7. Document unsupported signals and alternative native monitoring surfaces.

Repeat for ADF, Databricks, ADLS Gen2, SQL/Synapse, Event Hubs, and other dependencies as required.

## 12.4 Phase 4 — Build queries and workbooks

Create tested KQL queries for recent failures, failure rates, long-running activities, missing expected runs, stale data, reconciliation failures, invalid-record spikes, dependency failures, control-plane changes, and alert delivery/workflow failures.

Pin useful queries to workbooks and ensure the on-call role can read them. Avoid embedding secrets or exposing data outside authorized scopes.

## 12.5 Phase 5 — Create alert rules and action groups

1. Start with a small set of high-value alerts.
2. Validate queries against historical and healthy periods.
3. Set evaluation frequency, lookback, threshold, and severity.
4. Add the action group and correct environment route.
5. Include runbook and workbook references.
6. Test end to end.
7. Tune false positives before broad rollout.

## 12.6 Phase 6 — Build Logic App workflows

1. Choose the appropriate trigger/action-group integration.
2. Inspect a real alert payload.
3. Normalize required fields.
4. Add routing, enrichment, deduplication, and failure paths.
5. Configure secure connections and identity.
6. Add ticket creation and safe remediation only after notification is reliable.
7. Record workflow run IDs and the original alert ID.
8. Configure alerts on Logic App failures.
9. Test malformed payloads, duplicate alerts, connector outages, and permission failures.

## 12.7 Phase 7 — Operational acceptance tests

- Force a known ADF activity failure in a safe environment.
- Verify the diagnostic record arrives in Log Analytics.
- Run the KQL query and confirm expected output.
- Confirm the alert fires and action group invokes the Logic App.
- Confirm the correct team receives the notification.
- Confirm the incident is not duplicated on repeated delivery.
- Verify a successful retry updates/closes the incident appropriately.
- Simulate missing files, stale data, and data-quality failure.
- Verify sensitive values do not appear in logs/messages.
- Document recovery steps and evidence.

## 12.8 Naming convention

| Object | Convention | Example |
|---|---|---|
| Diagnostic setting | `diag-<env>-<resource>-to-law` | `diag-prod-adf-orders-to-law` |
| Alert rule | `alert-<env>-<domain>-<condition>` | `alert-prod-orders-pipeline-failure` |
| Action group | `ag-<env>-<team>-<severity>` | `ag-prod-data-sev1` |
| Logic App | `la-<env>-<purpose>` | `la-prod-monitoring-router` |
| Workbook | `wb-<env>-<domain>-operations` | `wb-prod-data-platform-operations` |
| Runbook | `rb-<service>-<failure-class>` | `rb-adf-authentication-failure` |

---

# 13. Production KQL query cookbook

> **Schema warning:** These are query patterns. Confirm column names in your workspace. Some examples assume resource-specific ADF tables; legacy diagnostic tables use different names. Do not deploy a query as an alert until it has been tested against real data and its output semantics are understood.

## 13.1 Recent ADF failures

```kusto
ADFPipelineRun
| where TimeGenerated > ago(24h)
| where Status =~ "Failed"
| project TimeGenerated, PipelineName, RunId, Status, ErrorMessage
| order by TimeGenerated desc
```

## 13.2 Pipeline runs by status

```kusto
ADFPipelineRun
| where TimeGenerated > ago(24h)
| summarize Runs=count() by PipelineName, Status
| order by PipelineName asc, Runs desc
```

## 13.3 Failure percentage by pipeline

```kusto
ADFPipelineRun
| where TimeGenerated > ago(7d)
| summarize Total=count(), Failed=countif(Status =~ "Failed") by PipelineName
| extend FailurePercent = iff(Total == 0, 0.0, 100.0 * Failed / Total)
| order by FailurePercent desc
```

## 13.4 Long-running pipeline candidates

```kusto
ADFPipelineRun
| where TimeGenerated > ago(7d)
| where Status =~ "Succeeded"
| extend DurationMinutes = datetime_diff('second', End, Start) / 60.0
| summarize P50=percentile(DurationMinutes, 50),
            P95=percentile(DurationMinutes, 95),
            Max=max(DurationMinutes), Runs=count()
  by PipelineName
| order by P95 desc
```

## 13.5 Failed activities grouped by type

```kusto
ADFActivityRun
| where TimeGenerated > ago(24h)
| where Status =~ "Failed"
| summarize Failures=count(), Pipelines=dcount(PipelineName)
  by ActivityType, ErrorCode
| order by Failures desc
```

## 13.6 Top recurring error messages

```kusto
ADFActivityRun
| where TimeGenerated > ago(7d)
| where Status =~ "Failed"
| summarize Occurrences=count(), AffectedPipelines=dcount(PipelineName)
  by ErrorCode, ErrorMessage
| top 20 by Occurrences desc
```

For messages containing changing IDs, normalize/group on error code and a sanitized message pattern rather than the entire raw message.

## 13.7 Pipeline duration by day

```kusto
ADFPipelineRun
| where TimeGenerated > ago(30d)
| where Status =~ "Succeeded"
| extend DurationMinutes = datetime_diff('second', End, Start) / 60.0
| summarize AvgDuration=avg(DurationMinutes), P95Duration=percentile(DurationMinutes, 95)
  by PipelineName, Day=bin(TimeGenerated, 1d)
| order by Day asc
```

## 13.8 Find unusually long runs compared with baseline

```kusto
let Baseline = ADFPipelineRun
| where TimeGenerated between (ago(14d) .. ago(1d))
| where Status =~ "Succeeded"
| extend DurationMinutes = datetime_diff('second', End, Start) / 60.0
| summarize BaselineP95=percentile(DurationMinutes, 95) by PipelineName;
ADFPipelineRun
| where TimeGenerated > ago(24h)
| where Status =~ "Succeeded"
| extend DurationMinutes = datetime_diff('second', End, Start) / 60.0
| join kind=inner Baseline on PipelineName
| where DurationMinutes > BaselineP95
| project TimeGenerated, PipelineName, DurationMinutes, BaselineP95, RunId
| order by DurationMinutes desc
```

This is a simple candidate detector, not a full statistical anomaly model. Avoid using the same small window for baseline and current values.

## 13.9 Expected pipelines with no recent successful run

Maintain a governed expected-pipeline inventory with one row per critical pipeline and schedule. A simplified pattern:

```kusto
let ExpectedPipelines = datatable(PipelineName:string, MaxAgeHours:real)
[
  "pl_orders_daily", 8.0,
  "pl_customer_daily", 12.0,
  "pl_inventory_hourly", 2.0
];
let LatestSuccess = ADFPipelineRun
| where TimeGenerated > ago(7d)
| where Status =~ "Succeeded"
| summarize LastSuccess=max(End) by PipelineName;
ExpectedPipelines
| join kind=leftouter LatestSuccess on PipelineName
| extend AgeHours = datetime_diff('minute', now(), LastSuccess) / 60.0
| where isnull(LastSuccess) or AgeHours > MaxAgeHours
| project PipelineName, MaxAgeHours, LastSuccess, AgeHours
```

In production, use a governed inventory rather than hard-coding the list. Account for weekends, holidays, backfills, and intentionally paused pipelines.

## 13.10 Azure Activity Log: recent failed operations

```kusto
AzureActivity
| where TimeGenerated > ago(24h)
| where ActivityStatusValue =~ "Failure"
| project TimeGenerated, Caller, OperationNameValue, ResourceGroup,
          ResourceId, ActivityStatusValue, Properties
| order by TimeGenerated desc
```

Column names can differ across table versions. Use `AzureActivity | take 10` to inspect the actual schema.

## 13.11 Resource change correlation

```kusto
AzureActivity
| where TimeGenerated > ago(24h)
| where OperationNameValue has_any ("write", "delete", "network", "roleAssignments")
| project TimeGenerated, Caller, OperationNameValue, ResourceId, ActivityStatusValue
| order by TimeGenerated desc
```

This is a broad search pattern. Refine it to exact operation names and resource IDs before using it for incident investigation or alerting.

## 13.12 Agent heartbeat (where applicable)

```kusto
Heartbeat
| where TimeGenerated > ago(30m)
| summarize LastHeartbeat=max(TimeGenerated) by Computer
| extend AgeMinutes = datetime_diff('minute', now(), LastHeartbeat)
| where AgeMinutes > 10
| order by AgeMinutes desc
```

This applies where Azure Monitor Agent heartbeat data is collected. It is not a generic heartbeat for every Azure PaaS resource.

## 13.13 Application exceptions (when configured)

```kusto
AppExceptions
| where TimeGenerated > ago(24h)
| summarize ExceptionCount=count() by ProblemId, Type, AppRoleName
| order by ExceptionCount desc
```

## 13.14 Application request failures (when available)

```kusto
AppRequests
| where TimeGenerated > ago(24h)
| summarize Requests=count(), Failed=countif(Success == false),
            P95Duration=percentile(DurationMs, 95)
  by AppRoleName, Name
| extend FailurePercent = 100.0 * Failed / Requests
| order by FailurePercent desc
```

## 13.15 Legacy `AzureDiagnostics` inspection

```kusto
AzureDiagnostics
| where TimeGenerated > ago(1h)
| take 20
```

Do not guess suffixes such as `_s` or `_d`; inspect the table. Legacy tables can contain multiple resource categories and a wide schema. Filter by resource ID/category and project only needed columns.

## 13.16 Logic App workflow failures

The exact tables depend on how Logic Apps diagnostics are configured. Inspect available tables and diagnostic categories for the workflow resource, then query supported records for failed runs, connector failures, or action duration. If no diagnostic logs are enabled, use Logic App run history and configure supported diagnostics rather than assuming a standard table exists.

## 13.17 Count reconciliation pattern

If custom audit events are written to `IngestionAudit_CL` (illustrative table name only):

```kusto
IngestionAudit_CL
| where TimeGenerated > ago(7d)
| summarize arg_max(TimeGenerated, *) by PipelineName_s, BatchId_s
| extend CountVariance = tolong(TargetCount_d) - tolong(SourceCount_d)
| where CountVariance != 0 or Status_s != "Succeeded"
| project TimeGenerated, PipelineName_s, BatchId_s,
          SourceCount_d, TargetCount_d, CountVariance, RejectedCount_d, Status_s
| order by TimeGenerated desc
```

Custom log ingestion can use different table and column names. Validate numeric types and whether expected filtering or late-arriving records should be excluded from the variance check.

## 13.18 Freshness monitoring pattern

```kusto
// Illustrative custom event schema: DataFreshness_CL
DataFreshness_CL
| summarize LatestBusinessTimestamp=max(BusinessTimestamp_t) by Dataset_s
| extend AgeMinutes = datetime_diff('minute', now(), LatestBusinessTimestamp)
| where AgeMinutes > 180
| project Dataset_s, LatestBusinessTimestamp, AgeMinutes
| order by AgeMinutes desc
```

For date-based datasets, calculate freshness using the business calendar and expected schedule. A fixed 180-minute threshold is not appropriate for every dataset.

## 13.19 Alert query design checklist

Before saving a query as an alert, verify:

- It returns results only for the condition that should alert.
- The time window is explicit.
- Duplicate records will not create repeated incidents.
- Environment/resource scope is correct.
- Nulls, missing runs, and delayed ingestion are handled.
- Output is small enough to be useful in the alert payload.
- Query cost is reasonable at its evaluation frequency.
- The query has been tested against healthy and unhealthy periods.

---

# 14. Troubleshooting methodology

## 14.1 Start with impact, not the tool

When an incident starts, establish:

1. Which business dataset or report is impacted?
2. Is the issue freshness, completeness, correctness, availability, or performance?
3. What is the last known successful batch/business date?
4. Which source, pipeline, job, and target are involved?
5. Is the failure isolated or shared across workloads?
6. Is there an Azure service health incident or recent configuration change?
7. What is the latest safe recovery time before the business SLA is missed?

This prevents engineers from spending time on a noisy symptom while overlooking the actual business impact.

## 14.2 Use a dependency graph

For each dataset, document dependencies such as:

`Source → ADF extraction → ADLS raw → Databricks transformation → Delta curated → SQL/Synapse serving → semantic model/report`

For streaming workloads, include producer, Event Hubs/Kafka, consumer/checkpoint, Delta sink, and downstream serving layers. When a downstream table is stale, work backward from the last successful node rather than assuming the downstream service is the root cause.

## 14.3 Correlation strategy

Correlate using UTC time range, Azure resource ID, pipeline name/run ID, activity name/run ID, Databricks job/run and cluster IDs, batch ID, source/target object, event partition/offset or checkpoint reference, deployment/change record, and a propagated correlation ID.

Keep correlation IDs in logs and notifications. They reduce manual searching and make incidents reproducible.

## 14.4 Triage decision tree

```text
Is the dataset/report stale or incorrect?
  |
  +-- No --> Investigate alert/query correctness; correct a false positive safely.
  |
  +-- Yes --> Did the scheduled pipeline/job start?
                 |
                 +-- No --> Trigger/schedule/identity/dependency/resource state
                 |
                 +-- Yes --> Did the activity/job fail?
                                |
                                +-- Yes --> Read exact error; classify auth/network/data/capacity/code
                                |
                                +-- No --> Check counts, watermark, rejected rows, target commit,
                                           downstream refresh, and freshness calculation
```

## 14.5 Retry versus replay versus backfill

- **Retry:** rerun a failed operation expected to be safe, often with the same input and idempotent output behavior.
- **Replay:** reprocess previously captured events/files; verify deduplication and offset/checkpoint strategy.
- **Backfill:** intentionally process a historical period or missing partition with a controlled date range.

Before retrying, determine whether a partial write occurred. For SQL sinks, a timeout may occur after server commit but before the client receives confirmation. For Delta tables, consider transaction semantics and concurrent writers. For streams, do not delete/reset a checkpoint casually; understand replay effects on duplicates and stateful operations.

## 14.6 Incident closure criteria

Do not close an incident merely because a retry succeeded. Confirm the batch completed or was safely abandoned; counts and quality checks pass; watermark/checkpoint progress is correct; no duplicate/missing partitions were introduced; downstream tables/reports refreshed; monitoring returned to normal; root cause or uncertainty is recorded; and follow-up work has an owner.

---

# 15. Production incident scenarios

Each scenario follows **symptom → investigation → likely root cause → recovery → prevention**. Validate evidence before assigning a root cause.

## Scenario 1 — ADF pipeline failed, but the alert did not fire

**Symptom:** The pipeline failed in ADF Monitor, but no alert was generated.

**Investigation:** Confirm diagnostic setting, selected category, destination workspace, expected table, ingestion delay, KQL status filter, query time range, alert evaluation frequency, and action-group association. Run the query manually with a wider time window.

**Likely causes:** Diagnostics not enabled; query uses wrong table/column; status casing mismatch; query returns no rows; alert disabled; scope/action group misconfigured.

**Recovery:** Fix diagnostic or query configuration and generate a controlled test failure. Do not assume past events will appear retroactively.

**Prevention:** Include alert-path testing in deployment and maintain a diagnostic coverage inventory.

## Scenario 2 — Logic App ran, but no notification arrived

**Symptom:** Azure Monitor shows a fired alert and Logic App run, but the team received no notification.

**Investigation:** Inspect workflow run history, trigger input, parsing/condition branches, connector authentication, recipient/channel configuration, throttling, and failure scopes.

**Likely causes:** Payload path changed, optional field assumed mandatory, connector expired, wrong route selected, or workflow action failed after trigger success.

**Recovery:** Correct mapping/connection, reprocess the incident safely, and use a fallback path for critical events.

**Prevention:** Validate real alert payloads, test failure branches, and monitor Logic App failures independently.

## Scenario 3 — ADF succeeded but target count is lower

**Symptom:** ADF reports success, but the target has fewer rows than the source.

**Investigation:** Compare source extraction query, source count, copy metrics, filtering, rejected rows, transformations, deduplication, target write mode, and business keys.

**Likely causes:** Incremental watermark skipped a range; filters were misinterpreted; malformed rows were skipped; mapping dropped a column; merge/partition overwrite replaced data.

**Recovery:** Identify missing range, repair extraction/watermark logic, backfill safely, and reconcile counts and keys.

**Prevention:** Persist source/target counts, rejected counts, watermark boundaries, and reconciliation status per batch; alert on unexplained variance.

## Scenario 4 — Pipeline duration suddenly doubles

**Symptom:** A daily ingestion normally takes 20 minutes but now takes 45 minutes.

**Investigation:** Compare input volume, source query time, IR queueing, copy throughput, network, target throttling, Databricks stage duration, and recent deployments.

**Likely causes:** Data growth, source query plan change, skew, too many small files, capacity pressure, concurrency changes, or downstream throttling.

**Recovery:** Isolate the slowest stage and apply a targeted fix; avoid blindly scaling every component.

**Prevention:** Track P50/P95 duration, throughput per GB/row, and input volume; alert on sustained regression rather than one slow run.

## Scenario 5 — Self-hosted Integration Runtime is offline

**Symptom:** Multiple ADF activities queue or fail with connectivity errors.

**Investigation:** Check node status, service process, VM health, CPU/memory/disk, outbound connectivity, DNS, firewall/proxy changes, and maintenance events.

**Likely causes:** Host restart, service stopped, network change, expired credential, capacity saturation, or node version issue.

**Recovery:** Restore service/connectivity, verify IR health, and rerun only safe failed activities.

**Prevention:** Monitor node availability/resource pressure; use documented high-availability patterns and clear ownership.

## Scenario 6 — Managed identity suddenly receives 403 from ADLS

**Symptom:** A pipeline that previously wrote to ADLS now receives authorization failures.

**Investigation:** Confirm runtime identity, account network rules, data-plane RBAC, directory ACLs, execute permissions on parent directories, and recent role/ACL changes.

**Likely causes:** Role assignment removed, wrong identity selected, ACL/path change, private endpoint/DNS issue misclassified as authorization, or deployment changed the resource.

**Recovery:** Restore minimum required permissions through approved change control; validate read/write/list behavior on the exact path.

**Prevention:** Monitor role changes, test identity access in deployment, and maintain least-privilege documentation.

## Scenario 7 — Databricks job is running but target table is stale

**Symptom:** Job is marked running or succeeded, but consumers see old data.

**Investigation:** Check task status, streaming progress, input offsets, watermark, checkpoint state, output commit, target path/table, partition date, and downstream refresh.

**Likely causes:** Job processing backlog; no new source data; wrong target; merge condition excludes updates; checkpoint/offset not advancing; downstream cache/model refresh delayed.

**Recovery:** Find last committed batch and missing interval; repair target mapping or resume processing without resetting state blindly.

**Prevention:** Alert on target freshness and checkpoint/offset progress, not only job status.

## Scenario 8 — Spark job fails with executor loss or out-of-memory

**Symptom:** Tasks fail repeatedly, executors are lost, or job terminates with memory pressure.

**Investigation:** Inspect Spark UI, executor logs, shuffle size, spill, GC, partition sizes, skew, cached datasets, join plan, and cluster sizing.

**Likely causes:** Oversized partitions, skewed key, unsafe broadcast, excessive caching, large shuffle, or under-sized compute.

**Recovery:** Fix the data/plan issue first; adjust partitions, join strategy, caching, or compute based on evidence.

**Prevention:** Track stage/task skew, spill, runtime, and input size; test representative peak data.

## Scenario 9 — Azure SQL sink times out and retry causes duplicates

**Symptom:** The first write times out, and retry creates duplicate or conflicting records.

**Investigation:** Check whether the first transaction committed, target keys, transaction boundaries, query duration, locks, and retry semantics.

**Likely causes:** Client timeout after server commit, non-idempotent inserts, concurrent pipeline runs, or long transaction.

**Recovery:** Reconcile target rows by business key/batch ID; use idempotent merge/upsert or controlled cleanup before replay.

**Prevention:** Design idempotency keys, use transaction-aware retry logic, and record batch commit state.

## Scenario 10 — Event Hubs throughput is healthy but streaming lag increases

**Symptom:** Namespace is available, but the consumer is increasingly behind.

**Investigation:** Compare incoming events/sec with processed events/sec; inspect consumer lag, partition distribution, micro-batch duration, downstream Delta write latency, checkpoint errors, and skew.

**Likely causes:** Consumer compute undersized, slow transformation, uneven partitions, downstream throttling, or repeated failures/replays.

**Recovery:** Remove bottleneck and scale consumers/compute only when partitioning and sink support the change.

**Prevention:** Alert on sustained lag growth and time-to-retention-risk; monitor source and processing rates.

## Scenario 11 — Logs disappear after resource deployment

**Symptom:** A service is redeployed and expected records no longer arrive.

**Investigation:** Check whether diagnostic settings remain attached, whether resource ID changed, and whether IaC recreated the resource or overwrote settings.

**Likely causes:** Diagnostics omitted from IaC, wrong destination, changed categories, or replacement resource.

**Recovery:** Reapply diagnostic configuration and verify delivery.

**Prevention:** Manage settings through IaC/Azure Policy where supported and test observability after deployment.

## Scenario 12 — Alert storm from one upstream outage

**Symptom:** Dozens of downstream alerts fire simultaneously.

**Investigation:** Group by time, dependency, source, error code, and common upstream resource. Identify whether one dependency explains the failures.

**Likely causes:** Independent rules have no correlation/suppression; downstream symptoms are treated as separate incidents.

**Recovery:** Triage the root dependency first, preserve evidence, and communicate impact once. Avoid disabling all rules.

**Prevention:** Correlate incidents, use ownership-aware routing, deduplicate, and distinguish root-cause alerts from downstream symptoms.

## Scenario 13 — Logic App creates multiple tickets for one alert

**Symptom:** One production failure creates repeated incidents.

**Investigation:** Compare alert IDs, rule evaluations, workflow retries, connector behavior, and concurrency. Determine whether ticket creation is idempotent.

**Likely causes:** No deduplication key, retry after ticket creation but before response, or simultaneous runs.

**Recovery:** Merge duplicates and add a durable idempotency check before ticket creation.

**Prevention:** Use alert/resource/run identifiers, concurrency control, and an update-existing-incident branch.

## Scenario 14 — Log Analytics ingestion cost spikes

**Symptom:** Ingestion costs rise sharply.

**Investigation:** Break down ingestion by table, resource, category, and time. Look for verbose diagnostics, repeated payload logging, new resources, loops, or changed collection.

**Likely causes:** Debug category enabled in production, duplicate routing, high-volume request logs, or custom logs containing full records.

**Recovery:** Remove unnecessary verbosity only after confirming audit/diagnostic needs; optimize event schema and retention.

**Prevention:** Cost dashboards, budgets/alerts, category review, sampling where supported, and ownership for high-volume tables.

## Scenario 15 — Alert is delayed by log ingestion

**Symptom:** A log alert fires late or misses a narrow failure window.

**Investigation:** Compare event time with ingestion time, alert lookback/evaluation frequency, query aggregation, and resource log delivery behavior.

**Likely causes:** Ingestion latency, query lookback too short, or late-arriving events.

**Recovery:** Adjust lookback/evaluation model to observed delay without making the window so broad that stale failures keep firing.

**Prevention:** Test alert latency end to end; use metric or Activity Log alerts where those are better signals.

## Scenario 16 — Trigger did not start expected pipeline

**Symptom:** No pipeline run exists for the scheduled time.

**Investigation:** Check trigger state, schedule/timezone, deployment, event subscription, tumbling-window dependency, and expected-run calendar.

**Likely causes:** Trigger disabled, timezone misunderstanding, event not delivered, dependency blocked, or trigger definition changed.

**Recovery:** Run a controlled catch-up for the missed interval and verify no duplicate processing.

**Prevention:** Alert on missing expected runs and test trigger state after deployment.

## Scenario 17 — Copy succeeded but output files are missing

**Symptom:** Pipeline succeeded, but expected output files/partitions are absent.

**Investigation:** Check sink path expression, partition date, output naming, empty source behavior, write mode, permissions, and whether data landed in a different container/environment.

**Likely causes:** Incorrect dynamic path, zero-row input, overwrite behavior, path casing, or validation checking the wrong partition.

**Recovery:** Correct path/partition and backfill the intended interval after checking target contents.

**Prevention:** Validate expected files and non-empty output when required; log resolved target paths and counts.

## Scenario 18 — Count variance is expected for one feed but alert fires

**Symptom:** Reconciliation alerts occur for a source that legitimately filters deleted or invalid rows.

**Investigation:** Compare extraction semantics, expected filters, rejected-record rules, CDC behavior, and whether counts represent the same snapshot.

**Likely causes:** The check compares incompatible populations or lacks an expected-rejection model.

**Recovery:** Compare equivalent populations and report accepted, rejected, filtered, and written counts separately.

**Prevention:** Define reconciliation contracts with data owners; do not apply one equality rule to every feed.

## Scenario 19 — Deployment changes a parameter and production breaks

**Symptom:** Failures begin immediately after release.

**Investigation:** Correlate Activity Log/deployment records with first failure; compare environment parameters, linked services, Key Vault references, network settings, and code version.

**Likely causes:** Incorrect environment configuration, renamed resource, changed identity, incompatible schema, or untested parameter.

**Recovery:** Roll back or hotfix through approved release controls; rerun impacted batches only after validating target state.

**Prevention:** CI/CD validation, parameterized environment configuration, smoke tests, and deployment-linked monitoring checks.

## Scenario 20 — No alert when the pipeline never runs

**Symptom:** There is no failed run, but the dataset is stale.

**Investigation:** Compare expected schedule with latest run and successful data timestamp. Check trigger health and source arrival.

**Likely causes:** Alert only looks for failed runs; no absence/freshness monitoring exists.

**Recovery:** Execute the missing interval safely and validate target data.

**Prevention:** Create expected-run and freshness checks; treat missing data as a first-class failure mode.

---

# 16. Reliability, security, cost, and governance

## 16.1 Reliability patterns

- Keep monitoring configuration in version control or infrastructure-as-code where feasible.
- Use reusable alert rules/action groups with clear naming.
- Create an observability coverage checklist for each new resource.
- Monitor the monitor: alert delivery, Logic App failures, diagnostic gaps, and workspace ingestion health.
- Preserve enough context to investigate without logging entire data payloads.
- Keep runbooks short, actionable, and tested.
- Define what “recovered” means at the data level.
- Review incidents for recurring causes and implement permanent fixes.

## 16.2 Security patterns

- Use managed identities and least-privilege roles wherever supported.
- Separate log-reader permissions from remediation permissions.
- Protect Log Analytics workspaces and sensitive tables.
- Do not place secrets, tokens, connection strings, or sensitive source rows in logs.
- Use Key Vault or secure connection mechanisms for workflow credentials.
- Validate and sanitize alert fields before rendering messages or constructing URLs.
- Audit automated remediation and require approval for high-impact actions.
- Keep production and non-production notification routes separate.

## 16.3 Cost controls

- Inventory diagnostic categories and identify high-volume ones.
- Estimate ingestion before enabling verbose telemetry broadly.
- Use retention appropriate to incident response and compliance requirements.
- Avoid logging entire request/response bodies unless approved.
- Build a dashboard for ingestion by table/resource and review changes after deployments.
- Use sampling/filtering only where supported and where required evidence is preserved.
- Review duplicated routing and unnecessary cross-workspace copies.
- Tune alert query frequency and complexity to business need.

## 16.4 Compliance and privacy

Monitoring data can contain identifiers, error payloads, query text, file paths, and user identities. Define classification and masking standards for telemetry. For healthcare, financial, or otherwise sensitive workloads, keep alert messages concise and link to authorized systems rather than embedding sensitive records. Align retention and access with organizational policy and applicable obligations.

## 16.5 Ownership model

| Role | Responsibility |
|---|---|
| Data engineering | Pipeline/job logic, data quality, retry/replay, target validation |
| Platform engineering | Workspace, networking, compute, identity, resource availability |
| Cloud operations/SRE | Alert policies, incident process, on-call, reliability reviews |
| Security | Access control, sensitive logging, audit requirements |
| Data/business owner | Data correctness, expected volume, business calendar, SLA impact |
| Application/source team | Source availability, schema changes, API/database behavior |

Define one accountable owner for each alert even when several teams contribute to resolution.

---

# 17. Testing, recovery, and disaster readiness

## 17.1 Test the complete monitoring chain

A test is complete only when the entire chain has been verified:

1. The failure or threshold condition occurs.
2. The source service emits the expected telemetry.
3. Diagnostic settings deliver the record to the intended destination.
4. The KQL query returns the expected record.
5. The alert rule evaluates and fires within expected time.
6. The action group invokes the expected action.
7. The Logic App parses and routes the alert.
8. The recipient receives an actionable notification or ticket.
9. Failure handling works when a downstream connector is unavailable.
10. Recovery is verified against the original data issue.

## 17.2 Game-day tests

Run controlled, approved tests for:

- ADF activity failure.
- Missing expected source file.
- Invalid schema or malformed record.
- Delayed source arrival.
- Databricks job failure or intentional slow workload in a safe environment.
- Storage authorization denial.
- SQL timeout/transient connectivity issue in non-production.
- Logic App connector failure.
- Duplicate alert delivery.
- Diagnostic setting removed/changed in a test subscription.
- Service Health notification routing.

Do not create disruptive production failures without approval and a rollback plan.

## 17.3 Recovery evidence

Capture original alert/incident ID; affected resource, pipeline, run ID, batch ID, and business date; error classification and evidence; recovery action and execution time; source/target counts and quality results; freshness and downstream availability; and follow-up owner.

## 17.4 Disaster recovery considerations

Monitoring should remain useful when a regional or service-level incident affects the workload. Consider where logs/workflows reside, how responders access runbooks, whether notification paths are independent of the failing component, and whether alerts cover service/resource health. Do not assume a monitoring workspace or Logic App in the same failure domain will always be available during a regional incident. Validate the architecture against recovery objectives.

---

# 18. Interview questions and model answers

Answers are written in a practical interview style. Adapt them to the architecture actually implemented; do not claim production experience with a component you have only studied.

## Q1. Explain Azure Monitor in an Azure data engineering project.

**Answer:** Azure Monitor is the central observability service I use to collect and analyze metrics, logs, and events from Azure resources. In a data platform, I monitor ADF pipeline/activity execution, Databricks and compute signals where supported, storage/database health, and business-level freshness or reconciliation signals. I configure alert rules and action groups so the correct team is notified when an actionable condition occurs. For root-cause analysis, I use Log Analytics and KQL to correlate events across services.

## Q2. What is the difference between Azure Monitor and Log Analytics?

**Answer:** Azure Monitor is the broader monitoring and response platform. Log Analytics is the workspace-based log analysis experience and data store used to query supported log and trace data with KQL. Azure Monitor includes metrics, alert rules, action groups, service health, and visualization; Log Analytics is one of the main tools used to investigate detailed log records.

## Q3. What is a diagnostic setting?

**Answer:** A diagnostic setting configures which supported resource logs and metrics are routed to destinations such as Log Analytics, Storage, or Event Hubs. Resource logs are not necessarily collected into a workspace by default. I enable categories needed for operational, audit, and compliance requirements, then verify that expected records arrive.

## Q4. Metrics versus logs — which would you use?

**Answer:** Metrics are best for numeric time-series monitoring and straightforward thresholds, such as request errors or resource utilization. Logs provide detailed event records and contextual fields, such as an ADF activity error code and run ID. I use metrics for quick detection where possible and logs for detailed correlation/diagnosis. The choice depends on the signal and what the resource exposes.

## Q5. How do you monitor ADF pipelines centrally?

**Answer:** I enable ADF diagnostic settings to send pipeline, activity, and trigger records to Log Analytics using supported resource-specific tables or the configured legacy table. I create KQL queries for failures, duration, retries, and recurring error codes, then add alerts for critical failures and missing expected runs. I also capture custom batch metrics such as source count, target count, rejected rows, watermark, and freshness because built-in pipeline status does not prove data completeness.

## Q6. How would you detect a pipeline that never ran?

**Answer:** A failure-only query cannot detect a missing run because no run record exists. I maintain an expected-run schedule or pipeline inventory, compare it with the latest run/success timestamp, and alert when the expected execution is missing or data freshness exceeds its deadline. The check accounts for business calendars, holidays, paused workloads, and intentional backfills to avoid false positives.

## Q7. How do you troubleshoot missing ADF logs in Log Analytics?

**Answer:** I check whether the correct diagnostic setting is enabled on the correct factory, whether required categories and workspace destination are selected, and whether enough time has passed for ingestion. I inspect actual tables/schema, confirm permissions, and verify that KQL uses the right table and time range. I then generate a controlled event and compare portal details with the log record.

## Q8. What is KQL, and why is it useful?

**Answer:** KQL is the query language used to analyze Azure Monitor Logs. I use `where` to filter, `project` to select fields, `summarize` to aggregate failures and durations, `bin()` to trend events over time, and `join` to correlate records. I filter by time early and inspect the actual schema before building a query-based alert.

## Q9. How do you design an actionable alert?

**Answer:** I define the signal, scope, condition, evaluation window, severity, owner, action group, and runbook. I test against healthy and failed periods and check that it fires only when an engineer should act. I account for late-arriving logs, repeated evaluations, duplicate incidents, and missing-run conditions. An alert without an owner or response procedure is incomplete.

## Q10. What is an action group?

**Answer:** An action group is a reusable collection of notifications and automated actions attached to Azure Monitor alerts. It can notify people or invoke supported endpoints/workflows, including Logic Apps. I use separate groups for critical production alerts, warnings, and non-production notifications so routing matches severity and environment.

## Q11. Why integrate Logic Apps with Azure Monitor?

**Answer:** I use Logic Apps when the default alert notification is not enough. The workflow normalizes the payload, enriches it with ownership/runbook information, routes by severity, creates or updates an incident, notifies Teams/email, deduplicates alerts, and records the outcome. It can initiate approved recovery actions, but only with safeguards and idempotency checks.

## Q12. How do you avoid duplicate incidents from Logic Apps?

**Answer:** I create a stable deduplication key from fields such as environment, resource ID, alert rule, pipeline run ID, and a controlled time bucket. I check a durable incident store before creating a ticket, and update the existing incident if the same issue is already open. I also account for workflow retries and concurrent runs so ticket creation is idempotent.

## Q13. How do you monitor Azure Databricks?

**Answer:** I combine Azure-level diagnostics with Databricks-native job and Spark execution details. I monitor job status/duration, cluster start/termination failures, streaming lag, checkpoint errors, and data-quality outcomes. For slow jobs I inspect Spark UI, stages, skew, shuffle, spill, executor logs, query plans, and Delta write behavior. Azure Monitor centralizes alerts and correlates dependencies, but does not replace detailed Spark diagnostics.

## Q14. A Databricks job succeeded but data is stale. How do you investigate?

**Answer:** I check the latest successful batch, input offsets, watermark progression, checkpoint state, output commit, target path/table, partition date, and downstream refresh. I verify row counts and freshness rather than trusting job status alone. If the job is running but lagging, I compare input/processing rates and investigate the slowest stage or sink.

## Q15. How do you monitor ADLS Gen2 authorization failures?

**Answer:** I configure supported storage diagnostics and metrics, then correlate failed requests with the pipeline run and identity. I verify the effective managed identity, data-plane RBAC role, directory ACLs and execute permissions on parent paths, account network rules, private endpoint, and DNS. I apply the minimum permission needed and validate access to the exact path.

## Q16. How would you detect data loss when the pipeline reports success?

**Answer:** I record expected/actual source and target counts, rejected rows, batch ID, watermark boundaries, and target partition. I reconcile equivalent populations and alert on unexplained differences. I validate keys, duplicates, and freshness. Reconciliation must account for intentional filtering, CDC semantics, and rejected records rather than assuming source count always equals target count.

## Q17. How do you monitor streaming pipelines?

**Answer:** I monitor input rate, processing rate, source lag, micro-batch duration, watermark progression, late-event behavior, checkpoint failures, restart frequency, and target freshness. A job can be running while it falls behind, so I alert on sustained lag and time-to-SLA/retention risk, not only a stopped process. Recovery must preserve checkpoint and deduplication semantics.

## Q18. What is Azure Activity Log used for?

**Answer:** It records subscription-level control-plane operations such as resource updates, deletes, and other management events. I use it to correlate a sudden failure with a configuration or permission change. For KQL correlation or longer retention, I configure export to Log Analytics and query the resulting `AzureActivity` records.

## Q19. How do you distinguish Azure service issues from application issues?

**Answer:** I check Service Health and Resource Health where applicable, then compare the incident time and affected region/resource with pipeline errors and dependency metrics. If several unrelated workloads fail against the same dependency at the same time, that suggests a shared platform/network issue. I still validate application-specific errors before concluding root cause.

## Q20. How do you manage Log Analytics costs?

**Answer:** I review ingestion by table, resource, and diagnostic category; remove unnecessary verbosity only after checking audit/compliance requirements; avoid logging full payloads; and use appropriate retention or archival. I inspect sudden volume increases after deployments and avoid duplicate routing. Cost optimization must not silently remove required evidence.

## Q21. What should an operational workbook contain?

**Answer:** It should show critical pipeline status, SLA/freshness status, recent failures, duration trends, failure rates, data-quality/reconciliation exceptions, dependency health, and alert-delivery health. Each panel should answer an operational question and link to a detailed view or runbook.

## Q22. How do you troubleshoot a failed pipeline efficiently?

**Answer:** I begin with business impact and the last successful data timestamp. I identify the run and failed activity, classify the error as authentication, network, schema/data, capacity, orchestration, or sink conflict, and correlate it with Log Analytics, Activity Log, and dependency metrics. Before retrying, I check for partial writes and idempotency. After recovery, I validate counts, keys, watermark, freshness, and downstream readiness.

## Q23. How do you handle alert fatigue?

**Answer:** I separate root-cause alerts from downstream symptoms, tune thresholds against baselines, use sustained conditions where appropriate, deduplicate related alerts, and ensure every alert has an owner/runbook. I review false positives and duplicate rates regularly. I do not solve alert fatigue by disabling all alerts for a noisy resource.

## Q24. How do you monitor the monitoring system itself?

**Answer:** I alert on Logic App failures, notification connector failures, unexpected gaps in diagnostic data, and critical pipeline events that lack the expected incident workflow. I test the full alert path periodically and maintain a fallback notification method for critical incidents. The test includes data-level recovery, not just successful email delivery.

## Q25. How do you protect sensitive data in logs?

**Answer:** I use structured, minimal logs with correlation IDs and sanitized error details. I exclude credentials, tokens, connection strings, and raw sensitive payloads. I apply least-privilege access to workspaces/tables, separate monitoring permissions from remediation permissions, and keep alert messages brief with links to authorized diagnostic views.

## Q26. How do you build production-grade monitoring during CI/CD?

**Answer:** I version diagnostic settings, alert rules, action groups, Logic App definitions, and workbooks where supported. I parameterize environment-specific resource IDs and notification routes. After deployment, I run smoke tests for diagnostics, query behavior, alert routing, and freshness. I also detect configuration drift so deployment cannot silently remove observability.

## Q27. What is the difference between retry and replay?

**Answer:** Retry repeats a failed operation and is appropriate when safe and idempotent. Replay reprocesses previously captured events/files and requires correct checkpoint/offset and deduplication behavior. Before either action, I determine whether a partial write occurred and whether the target tolerates duplicate delivery.

## Q28. How do you monitor a self-hosted Integration Runtime?

**Answer:** I monitor node availability, service state, host resource utilization, network connectivity, and activity queueing. I check node status, DNS/firewall/proxy, credentials, version, and host capacity. If multiple pipelines fail together, I look for a shared runtime/network dependency before debugging each pipeline separately.

## Q29. What should a critical alert message contain?

**Answer:** Severity, environment, affected resource/pipeline, run/batch ID when available, UTC detection time, business impact, link to run details, workbook/query, runbook, owner, and correlation ID. It should not contain secrets or sensitive raw data.

## Q30. How do you prove monitoring is production-ready?

**Answer:** I run end-to-end acceptance tests for pipeline failure, missing data, data-quality variance, delayed telemetry, duplicate alerts, Logic App connector failure, and recovery. I verify the record arrives, the query returns it, the alert fires, the correct team is notified, and the dataset is validated after recovery. I document owners, runbooks, severity, and known signal gaps.

---

# 19. Advanced scenario-based interview questions

## Scenario A — Source count is 10 million but target contains fewer records

**Answer approach:** First confirm whether counts represent the same extraction window and filtering rules. Compare source predicates and watermark boundaries, copy read/write metrics, rejected rows, transformation filters, deduplication, target merge/overwrite behavior, and batch IDs. Use the control ledger and Log Analytics to locate the first stage where counts diverge. Repair the cause, backfill only the missing interval, and validate keys, duplicates, and final counts.

## Scenario B — Three pipelines fail simultaneously with storage 403 errors

**Answer approach:** Multiple pipelines failing against the same storage account suggests a shared dependency. Compare timestamps and identities, inspect storage/network diagnostics, and review Azure Activity Log for role, ACL, firewall, or private endpoint changes. Verify the effective identity used by each pipeline and exact path permissions. After correction, rerun only safe batches and reconcile target data.

## Scenario C — ADF pipeline duration rises every day

**Answer approach:** Plot P50/P95 duration and input volume by day, then split the pipeline into activity durations. Compare source query time, IR queueing, copy throughput, transformation duration, sink latency, and code/schema changes. Growing data volume may be expected, but rising duration-per-GB can indicate a regression. Optimize the bottleneck, establish a baseline, and create a sustained regression alert.

## Scenario D — Alert query returns a row, but no alert fires

**Answer approach:** Check alert rule scope, enabled state, evaluation frequency, lookback, query semantics, threshold, dimension splitting, suppression, and action group. Verify the rule uses the same workspace/table and that the result meets the alert engine's expected condition. Test with a known record and inspect alert history rather than relying only on a manual query.

## Scenario E — Logic App runs but routes to the wrong team

**Answer approach:** Inspect normalized payload and metadata lookup, including resource tags, environment, domain, and owner mapping. Validate missing/unknown owners and route them to a central triage queue rather than silently defaulting. Test production and non-production routes separately and log the routing decision with alert ID.

## Scenario F — Freshness alert fires during a planned backfill

**Answer approach:** Determine whether the freshness SLO is intentionally paused or the backfill is expected to maintain the serving table. Use a governed maintenance/backfill state with expiry and owner, rather than permanently suppressing the alert. Preserve visibility of delay and notify consumers if the SLA is affected. After backfill, validate freshness and remove the temporary exception.

## Scenario G — Streaming job restarts repeatedly and duplicates events

**Answer approach:** Inspect driver/executor errors, checkpoint path, concurrent query execution, offset commits, sink idempotency, and stateful-operation compatibility. Identify the last committed output and compare it with source offsets before resetting anything. Restore a valid checkpoint/restart path and deduplicate by stable event/business keys. Test controlled replay and confirm expected logical records are present.

## Scenario H — Production logs contain sensitive data

**Answer approach:** Treat it as a security/privacy incident. Restrict access to affected logs, notify the responsible security/privacy team under policy, and stop unnecessary sensitive logging. Determine what data was collected and who could access it, apply approved retention/remediation steps, and update redaction tests. Do not copy sensitive values into tickets or chat messages.

## Scenario I — Log Analytics ingestion volume spikes after a release

**Answer approach:** Break down ingestion by table/resource and correlate the start time with deployment records. Check new diagnostic categories, repeated loops, verbose debug messages, payload logging, and duplicate destinations. Reduce unnecessary volume while preserving required telemetry and audit evidence, then add ingestion-cost regression checks to release validation.

## Scenario J — Azure regional incident affects multiple workloads

**Answer approach:** Check Service Health and Resource Health, identify affected dependencies/regions, and map impacted pipelines from the dependency graph. Communicate business impact and expected freshness, avoid repeated retries that worsen the incident, and follow the approved regional recovery strategy. After service restoration, replay/backfill missing intervals safely and validate data before reopening downstream consumption.

## Scenario K — Design monitoring for a new data product

**Answer approach:** Start with business SLIs and dependencies, then define execution, freshness, completeness, quality, throughput, and cost indicators. Enable supported diagnostics, add structured batch events and correlation IDs, create tested KQL queries/workbooks, define alert owners/runbooks, and implement Logic App routing where useful. Include monitoring tests in CI/CD and run a game day before production acceptance.

## Scenario L — A successful pipeline writes zero rows

**Answer approach:** Treat zero rows as valid behavior or an anomaly based on the source contract. Compare source arrival, query predicates, watermark, partition date, API pagination, schema changes, and target path. Alert only when the expected source/volume contract says zero is unexpected. After correction, backfill the missing interval and validate the target.

## Scenario M — Alert repeatedly fires after the issue is resolved

**Answer approach:** Check whether the query includes stale events within a long lookback, whether the alert is stateful/stateless, whether the signal has returned to normal, and whether the alert lifecycle is being interpreted correctly. Refine the query to evaluate the intended current condition and tune evaluation/suppression. Preserve incident evidence and test the clear/recovery path.

## Scenario N — Team wants automatic restart for every pipeline failure

**Answer approach:** Do not restart every failure indiscriminately. Classify failure types and allowlist transient, idempotent failures where bounded retry is safe. Authentication, schema, invalid-data, quota, and partial-write errors need different handling. The Logic App validates run state and batch ID, limits attempts, avoids concurrent duplicate runs, records actions, and escalates uncertain outcomes.

## Scenario O — Dataset meets pipeline SLA but misses reporting SLA

**Answer approach:** Trace the full path from source arrival to reporting readiness. Pipeline completion may exclude downstream SQL/Synapse loading, semantic model refresh, or report cache refresh. Define an end-to-end freshness SLI, add milestone timestamps for each dependency, and alert on the last consumer-visible readiness signal rather than ADF success alone.

---

# 20. Implementation checklist and rollout plan

## 20.1 Resource onboarding checklist

- [ ] Resource is in the inventory with environment, owner, domain, and criticality.
- [ ] Supported metrics and diagnostic categories have been reviewed.
- [ ] Diagnostic setting sends required logs to the correct destination.
- [ ] Expected table and schema are documented and verified with real events.
- [ ] Time range and ingestion delay are understood.
- [ ] Access controls and sensitive-data handling are reviewed.
- [ ] KQL query exists for key failure modes.
- [ ] Alert rule has severity, owner, action group, and runbook.
- [ ] Logic App routing is tested where used.
- [ ] Alert delivery and workflow failures are monitored.
- [ ] Workbook includes the resource's operational signals.
- [ ] Recovery procedure is documented and tested.

## 20.2 Production alert checklist

- [ ] Signal is tied to a real operational or business risk.
- [ ] Query/metric is validated against healthy and unhealthy periods.
- [ ] Evaluation frequency and lookback are appropriate.
- [ ] Ingestion delay and late-arriving events are considered.
- [ ] Duplicate rows/repeated evaluations will not create alert storms.
- [ ] Message contains resource, time, severity, owner, runbook, and correlation context.
- [ ] Alert route is environment-specific.
- [ ] A clear/recovery path exists.
- [ ] Rule is tested after deployments and configuration changes.

## 20.3 Logic App checklist

- [ ] Real trigger payload has been inspected.
- [ ] Required fields are validated; optional fields have safe defaults.
- [ ] Alert normalization is separate from routing logic.
- [ ] Routing uses governed owner/environment metadata.
- [ ] Duplicate detection is durable and concurrency-safe.
- [ ] Connector failures and retry behavior are handled.
- [ ] Sensitive data is excluded from notifications.
- [ ] Remediation is allowlisted, idempotent, bounded, and audited.
- [ ] Workflow failures have a fallback route.
- [ ] End-to-end tests are recorded.

## 20.4 Suggested four-week rollout

### Week 1 — Inventory and baseline

- Inventory critical resources and dependencies.
- Define owners, business calendars, freshness SLIs, and SLA deadlines.
- Decide workspace/access/retention architecture.
- Identify diagnostic coverage gaps and high-risk pipelines.

### Week 2 — Diagnostics and query pack

- Enable diagnostics for critical ADF, Databricks, storage, database, and event resources where supported.
- Validate tables and schemas.
- Create KQL queries for failure, duration, missing-run, freshness, and reconciliation.
- Build the first operations workbook.

### Week 3 — Alerts and Logic Apps

- Create critical and warning alert rules.
- Configure action groups and Logic App routing.
- Add incident deduplication and ownership enrichment.
- Monitor workflow failures and delivery outcomes.

### Week 4 — Testing and handover

- Run controlled failure and recovery tests.
- Test delayed telemetry, missing data, duplicate alerts, and connector failures.
- Finalize runbooks and escalation paths.
- Review cost, security, alert noise, and observability gaps.
- Obtain sign-off from data owners and operations.

## 20.5 Definition of done

A workload is monitoring-ready only when the team can detect an important failure or missing-data condition, find enough context to diagnose it, route it to an accountable owner, recover safely, and verify data correctness afterward. A dashboard alone is insufficient; neither is an email alert without a runbook.

---

# 21. Glossary and official references

## 21.1 Glossary

| Term | Meaning |
|---|---|
| Azure Monitor | Azure's monitoring and observability platform |
| Log Analytics workspace | Log and trace data store queried using KQL |
| KQL | Kusto Query Language |
| Diagnostic setting | Configuration routing supported resource logs/metrics to destinations |
| Resource log | Detailed telemetry emitted by an Azure resource |
| Metric | Numeric time-series signal |
| Azure Activity Log | Record of Azure control-plane/subscription events |
| Action group | Reusable collection of alert notifications/actions |
| Logic App | Workflow automation and integration service |
| SLI | Service Level Indicator |
| SLO | Service Level Objective |
| SLA | Service Level Agreement |
| Data freshness | Age of latest expected data available to consumers |
| Idempotency | Repeating an operation does not create unintended additional effects |
| Replay | Reprocessing previously captured inputs/events |
| Backfill | Processing historical or missing data intervals intentionally |
| Alert fatigue | Reduced response quality caused by excessive/noisy alerts |
| Correlation ID | Identifier linking events across components |
| Runbook | Documented steps to investigate, recover, and validate an issue |

## 21.2 Official Microsoft documentation

Use current documentation to verify exact resource categories, table schemas, feature availability, permissions, and setup steps in your environment.

1. [Azure Monitor overview](https://learn.microsoft.com/en-us/azure/azure-monitor/overview) — monitoring platform, telemetry, analysis, and response.
2. [Configure diagnostic settings and a workspace for Azure Data Factory](https://learn.microsoft.com/en-us/azure/data-factory/monitor-configure-diagnostics) — ADF diagnostic configuration and resource-specific tables.
3. [Configure diagnostic log delivery for Azure Databricks](https://learn.microsoft.com/en-us/azure/databricks/admin/account-settings/audit-log-delivery) — supported diagnostic destinations and setup considerations.
4. [Create and manage action groups](https://learn.microsoft.com/en-us/azure/azure-monitor/alerts/action-groups) — alert notifications and actions.
5. [Customize alert notifications by using Logic Apps](https://learn.microsoft.com/en-us/azure/azure-monitor/alerts/alerts-logic-apps) — integrate Azure Monitor alerts with Logic Apps.
6. [Collect resource logs from an Azure resource](https://learn.microsoft.com/en-us/azure/azure-monitor/data-collection/tutorial-resource-logs) — diagnostic settings and log collection.
7. [Log queries in Azure Monitor](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/log-query-overview) — KQL and query use cases.
8. [Log Analytics workspace overview](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/log-analytics-workspace-overview) — workspace design, access, retention, and cost.
9. [Azure Activity Log](https://learn.microsoft.com/en-in/azure/azure-monitor/platform/activity-log) — control-plane events and export options.
10. [Azure Monitor documentation](https://learn.microsoft.com/en-us/azure/azure-monitor/) — current monitoring, alerts, data collection, and visualization docs.

## 21.3 Final interview summary

A strong production monitoring design connects **technical telemetry to business outcomes**. Azure Monitor provides the central detection and response framework; Log Analytics and KQL provide detailed cross-service investigation; Logic Apps turns alerts into consistent, enriched, and auditable workflows. The most important practices are to enable and verify diagnostics, monitor missing data as well as explicit failures, preserve correlation IDs and batch-level reconciliation, design actionable low-noise alerts, automate only safe recovery steps, and validate actual data after recovery.

---

# Appendix A — Troubleshooting playbooks by symptom

Use these as operational cards. Verify the exact error and environment before taking action.

## A1. Authentication and authorization

**Indicators:** 401/403, token acquisition errors, login failed, permission denied.

**Checks:** effective identity; secret/certificate expiry; managed identity assignment; data-plane RBAC versus management-plane RBAC; ACL traversal; Key Vault access; SQL database role; private endpoint/DNS; recent identity or role changes.

**Actions:** validate with the same identity used by the runtime; request only the missing minimum permission; avoid switching to a broadly privileged identity as a shortcut; test the exact source/target operation after the change.

**Monitoring improvement:** alert on repeated authentication failures and role/credential changes; never log token or secret values.

## A2. Network and endpoint errors

**Indicators:** timeout, host not found, connection refused, TLS failure, intermittent connection reset.

**Checks:** DNS resolution from runtime network; private endpoint DNS zone; firewall allowlist; routing/NSG; proxy; service endpoint; TLS/certificate validity; service health; whether only one region/runtime is affected.

**Actions:** determine whether failure is name resolution, routing, connection establishment, or application response. Do not treat all timeouts as credential issues. Retry only after confirming safety.

**Monitoring improvement:** correlate failures by endpoint/runtime; alert when multiple pipelines share the same failing dependency.

## A3. Schema drift and malformed data

**Indicators:** missing/extra columns, type conversion errors, invalid JSON/CSV, corrupt files, null constraint failures.

**Checks:** source schema version; sample payload; delimiter/encoding; nested fields; timestamp formats; nullable fields; schema enforcement/evolution settings; producer deployment.

**Actions:** quarantine invalid records with reason codes; preserve original input; avoid silently coercing critical business fields; coordinate schema changes with data owners.

**Monitoring improvement:** track schema version, accepted/rejected counts, and reject rate. Alert on spikes or new schema signatures.

## A4. Throttling and capacity

**Indicators:** 429, 503, busy/overloaded, retries, queue growth, duration increase.

**Checks:** service metrics; concurrent activities/jobs; account/database limits; source-side workload; request rate; partition distribution; retry policy; time-of-day contention.

**Actions:** use exponential backoff where appropriate; reduce excessive concurrency; optimize request/query pattern; scale only after confirming bottleneck and capacity model.

**Monitoring improvement:** alert on sustained throttling and queue/lag growth rather than one isolated retry.

## A5. Pipeline orchestration logic

**Indicators:** branch not executed, unexpected skip, missing parameter, expression error, child pipeline not invoked.

**Checks:** pipeline parameters/defaults; activity dependency conditions; actual output JSON; array/object/string type; expression path; trigger-supplied values; environment parameter files.

**Actions:** capture a sanitized sample of actual activity output and validate the expression against it. Do not assume a property exists because it existed in a prior run.

**Monitoring improvement:** validate required parameters and log resolved business date/path without sensitive values.

## A6. Incorrect incremental watermark

**Indicators:** missing rows, duplicate extraction, no new data despite source changes, repeated old partitions.

**Checks:** lower/upper watermark semantics; inclusive/exclusive comparison; timezone and precision; null timestamps; late updates; CDC deletes; commit timing; parallel-run overlap.

**Actions:** derive the missing interval, repair the watermark, and backfill idempotently. Advance the watermark only after target write and validation succeed.

**Monitoring improvement:** record previous watermark, extraction lower/upper bounds, and committed watermark per batch. Alert when the watermark fails to advance unexpectedly.

## A7. Delta Lake write or merge issue

**Indicators:** transaction conflict, duplicate business keys, slow MERGE, schema mismatch, too many small files.

**Checks:** overlapping writers; merge predicate; source duplicate keys; partition pruning; schema enforcement; file sizes; concurrent maintenance.

**Actions:** deterministically deduplicate source rows before merge; use stable merge keys; control overlapping writes; optimize based on table size and query patterns.

**Monitoring improvement:** track merge duration, input/output rows, duplicate-key counts, transaction failures, and file-count growth.

## A8. Missing source file

**Indicators:** no new batch, empty folder, skipped pipeline, stale target.

**Checks:** expected delivery calendar; source SLA; filename convention; event trigger; storage permissions; upstream export status; timezone/date partition; alternate directory.

**Actions:** confirm whether file is late or intentionally absent. If late, communicate impact and catch up after arrival. Do not create an empty target partition merely to make a pipeline green.

**Monitoring improvement:** maintain per-source arrival windows and alert only after the grace period.

## A9. Slow SQL or warehouse load

**Indicators:** query timeout, long-running load, blocking, resource saturation, queue delay.

**Checks:** query plan; statistics/indexes; lock waits; concurrent workloads; data skew/distribution; transaction size; resource tier; storage latency.

**Actions:** isolate the slow query, reduce contention, optimize query/load strategy, and use controlled batch size. Confirm transaction outcome before retrying.

**Monitoring improvement:** record operation name, duration, row count, error number, and correlation ID; alert on sustained P95 regression and saturation.

## A10. Monitoring data not arriving

**Indicators:** empty table, stale workbook, missing alert input.

**Checks:** diagnostic setting; selected categories; destination; resource ID; table mode; permissions; ingestion delay; time filter; actual schema; recent resource recreation.

**Actions:** test with a new event, query a wider time range, inspect workspace tables, and compare native service history with collected records.

**Monitoring improvement:** maintain diagnostic coverage report and periodic synthetic tests for critical pipelines.

# Appendix B — Operational metrics catalogue

A metric should have a definition, unit, aggregation, owner, target, and response procedure.

| Metric | Definition | Why it matters | Typical owner |
|---|---|---|---|
| Pipeline success rate | Successful runs / eligible runs | Overall orchestration reliability | Data engineering |
| Critical pipeline SLA attainment | Runs completed by deadline / eligible runs | Business readiness | Data engineering + data owner |
| Pipeline duration P95 | 95th percentile runtime | Slow-tail regressions | Data engineering |
| Activity retry count | Retries by activity/type | Transient instability | Data engineering/platform |
| Source-to-target variance | Difference between comparable counts | Missing/extra records | Data engineering + source owner |
| Rejected record rate | Rejected / processed records | Data quality and schema health | Data engineering/data owner |
| Duplicate business-key count | Rows beyond expected uniqueness | Merge/dedup correctness | Data engineering |
| Latest successful business timestamp | Latest expected data timestamp available | Freshness | Data owner |
| Watermark progress | Change from previous committed watermark | Incremental load health | Data engineering |
| File arrival delay | Actual minus expected arrival | Source dependency health | Source owner |
| Databricks job duration P95 | 95th percentile runtime | Compute/workload regression | Data engineering |
| Spark task skew ratio | Slowest/median task duration or data volume | Skew detection | Data engineering |
| Streaming processing lag | Source progress versus consumer progress | Real-time SLA/retention risk | Streaming/platform team |
| Trigger delay | Expected schedule/event to run start | Orchestration responsiveness | Data engineering |
| Storage request failure rate | Failed / total requests | Storage health | Platform engineering |
| SQL timeout count | Timed-out operations by workload | Database pressure/query defects | Database/platform team |
| Logic App failure rate | Failed / total workflow runs | Notification reliability | Cloud operations |
| Alert delivery latency | Alert fired to notification delivered | Incident readiness | Cloud operations |
| Duplicate incident rate | Duplicate tickets / total tickets | Alert quality | SRE/operations |
| Log ingestion volume | Data ingested by table/resource/time | Cost and verbosity | Platform engineering |

### B1. Metric ownership rules

- Every metric has one accountable owner.
- A metric with no defined action should generally be dashboard-only rather than a paging alert.
- Define the denominator and exclusions for every percentage.
- Separate operational freshness from business-date completeness.
- Track definition changes so historical comparisons remain meaningful.
- Use UTC for correlation and explicit conversion for business reporting.

# Appendix C — Runbook template

## Runbook: [Alert name]

**Owner:** [Team/person/rotation]  
**Severity:** [Sev1/Sev2/Sev3/Sev4]  
**Last reviewed:** [Date]  
**Alert rule:** [Name/link]  
**Workbook/query:** [Link]  
**Affected services:** [Resources]  
**Business impact:** [Dataset/report/process]  
**Expected response time:** [Policy target]

### 1. Confirm impact

- Confirm alert is current and identify environment/resource.
- Check last successful run and latest business timestamp.
- Determine whether downstream consumers are affected.

### 2. Gather identifiers

- Pipeline/job run ID:
- Activity/task:
- Batch/business date:
- Source and target:
- Correlation ID:
- First failure time (UTC):

### 3. Initial checks

- Open native service run details.
- Run linked KQL query.
- Check service/resource health and recent control-plane changes.
- Identify the first failing dependency in the pipeline graph.

### 4. Classify the failure

- [ ] Authentication/authorization
- [ ] Network/connectivity
- [ ] Schema/data quality
- [ ] Capacity/throttling
- [ ] Code/logic
- [ ] Trigger/orchestration
- [ ] Sink/transaction conflict
- [ ] Missing data/freshness
- [ ] Platform/service health
- [ ] Monitoring/notification defect

### 5. Recovery decision

- Is retry safe?
- Could a partial write have committed?
- Is replay/backfill required?
- Is approval required?
- What is the maximum number of attempts?
- What is the rollback/stop condition?

### 6. Execute and validate

- Record the action and timestamp.
- Verify target counts, keys, quality checks, watermark/checkpoint, and freshness.
- Confirm downstream refresh.
- Confirm alert returned to a healthy state.

### 7. Close and learn

- Root cause or remaining uncertainty:
- Evidence links:
- Follow-up action and owner:
- Alert tuning needed:
- Permanent engineering fix:
- Runbook update needed:

# Appendix D — Design review questions

1. What business outcome is affected if the workload fails or is late?
2. What is the expected schedule, calendar, and arrival window?
3. What signal proves the data is correct and complete?
4. What is the last known-good timestamp or batch ID?
5. Which resources and dependencies are involved?
6. Which diagnostics are supported, and have they been tested in the actual environment?
7. Which logs belong in Log Analytics and which should be archived elsewhere?
8. Who can read sensitive telemetry?
9. What is expected log volume and cost?
10. Which conditions require paging, and which should be dashboard-only?
11. What is the minimum useful alert payload?
12. How are duplicates and correlated failures handled?
13. What happens if Logic Apps or the notification connector is unavailable?
14. Which actions are safe to automate, and which require human approval?
15. How will retry, replay, and backfill avoid duplicate writes?
16. How is recovery verified at the data level?
17. What synthetic tests prove monitoring works after deployment?
18. How are diagnostic settings and alert rules protected from drift?
19. What is the escalation path outside business hours?
20. When was the runbook last exercised?

A “yes” to the relevant questions does not guarantee zero incidents, but it makes incidents more detectable, explainable, recoverable, and auditable.
