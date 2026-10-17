# Azure Databricks — Complete Interview Question Guide

**80 questions with detailed answers:** 10 core, 15 advanced, 40 scenario-based, 10 practical exercises and 5 architecture/project questions.

**Focus:** Distributed lakehouse transformations, orchestration of Spark workloads and governed analytical serving.

**Audience:** Azure data engineering interviews, from fundamentals through production troubleshooting and design. Each question has exactly two detailed prose paragraphs: the first explains the concept or resolution, and the second develops its practical application and checks. Scenario answers cover the cause, investigation, correction and verification. Check source semantics, engine/runtime, workload identity and acceptance evidence for your actual configuration. Practice scenarios are hypothetical and are not claims about the named client projects.

## Contents

- A. Core interview questions — C01–C10.
- B. Advanced implementation questions — A01–A15.
- C. Scenario-based interview questions — S01–S40.
- D. Practical implementation exercises — P01–P10.
- E. Architecture and project discussion — D01–D05.
- F. Additional advanced implementation questions — A06–A15.
- G. Additional production scenarios — S26–S40.
- H. Additional practical exercises — P06–P10.
- Revision notes, scoring and official references.

## Role in the full project flow

```text
ADF or event source -> ADLS/Delta bronze -> Databricks job -> silver transformations -> gold Delta -> Databricks SQL or reporting
```

For this guide, the primary unit is **one Databricks job or notebook task**. The engineering artifacts are job definition, notebook/package version and compute configuration. Troubleshooting starts with Spark UI stages, task skew, job logs and Delta operation history; durable progress/evidence is represented by Delta table version, job run ID and streaming checkpoint where relevant. Distinguish platform availability, technical completion and accepted business correctness.

## A. Core interview questions

### C01. What is Azure Databricks?

An Azure-integrated data/AI platform used for distributed processing and lakehouse workloads. Distinguish the platform from Apache Spark, which is one processing engine it uses. The platform coordinates compute, governance and serving, while Spark distributes computation. Explain which workload uses which engine and where its data is persisted.

A Databricks Spark job creates silver Delta tables and a SQL warehouse serves the accepted gold product. Keep workload identity, data location and execution engine explicit so platform features are not confused with Spark's distributed processing behavior.

### C02. What is the driver versus an executor?

The driver coordinates application execution and planning; executors process distributed tasks. Collecting large data to the driver defeats distributed memory scaling. Driver memory and executor memory solve different problems. Distributed transformations scale across executors; collecting an unbounded result centralizes data on the driver.

Write a large usage aggregation directly to a table and collect only a bounded diagnostic sample. Driver-side diagnostics should remain bounded; executor memory, shuffle pressure and partition sizes need separate investigation when distributed tasks fail.

### C03. How do jobs differ from exploratory notebooks?

Jobs provide scheduled/triggered production tasks, parameters and execution controls. Notebooks support development; production requires versioning, ownership and reproducible deployment. Production execution should not depend on cells previously run by a human. Package dependencies, parameters and task order make the job reproducible.

A scheduled job receives an immutable input manifest and rule version instead of reading an interactive variable. Define dependencies, retries and publication gates in the job graph, and capture the input/rule versions needed to reproduce its accepted result.

### C04. What is a Delta table?

A table with transaction-log metadata over data files, providing supported transactional and schema behavior. It is not merely a folder of Parquet files. The transaction log defines active table state, including updates and removed files. Compatible readers use that state rather than reading every physical Parquet file.

After a corrected transaction MERGE, the active table contains the accepted line while obsolete files may remain for retention. Validate the active table snapshot and schema through supported readers; directly scanning storage files can include records removed from current table state.

### C05. What are bronze, silver and gold layers?

Bronze preserves raw/source context, silver conforms business records, and gold serves defined analytical products. They are design conventions, not required platform names. Layer names matter less than clear contracts. Preserve raw provenance, conform business semantics, then publish grain-specific products with acceptance evidence.

Bronze CDRs retain payloads; silver resolves durable events; gold contains subscriber/day/service totals. Each layer should declare grain, permitted changes, ownership and acceptance controls rather than treating directory names alone as a complete architecture.

### C06. How do SQL warehouses differ from Spark job compute?

SQL warehouses serve SQL workloads; Spark compute executes distributed jobs/notebooks. Match compute to workload, isolation and measured concurrency. Serving concurrency and transformation throughput have different resource patterns. Compute separation can reduce interference, but it introduces cost and operational configuration.

A heavy backfill should not automatically share the same execution resource as latency-sensitive BI queries. Measure workload contention and query latency before deciding compute isolation, and include startup/utilization costs when comparing serving and processing configurations.

### C07. What does lazy execution mean?

Spark transformations build a plan and actions trigger computation. Several actions can recompute the same lineage unless reuse is deliberately materialized or persisted. An action can execute the same logical lineage again. Persisting is useful only when actual reuse justifies its resource footprint and lifecycle.

A validated intermediate reused by several downstream calculations may be persisted, then released after those actions. Repeated actions may repeat source reads and wide transformations; cache only genuinely reused data and release it when downstream work completes.

### C08. What should a production notebook accept?

Validated input/output identifiers, run ID, bounded processing scope and rule/configuration version. Return meaningful status without embedding credentials. Input/output parameters must identify an approved scope and remain valid under retry. Arbitrary target names and paths should not become unrestricted execution instructions.

The notebook accepts entity and run IDs and resolves approved target tables from reviewed configuration. Use a run manifest to identify input files/boundaries and target scope, and fail closed when a supplied identifier is outside approved configuration.

### C09. How do you govern access?

Use Unity Catalog object grants and approved storage credentials/external locations where appropriate, plus platform identity/workspace policies. Platform access, catalog grants and cloud storage permissions form different layers. Test the effective workload identity and eliminate unintended direct-storage bypasses.

A production job reads its domain tables through the approved catalog storage abstraction with narrowly scoped grants. Review inherited grants and direct ADLS access as well as table permissions, since broad cloud access can bypass the intended governed table path.

### C10. What does the Spark UI reveal?

Stages, task duration, shuffle volume, skew, spills and execution plans help identify bottlenecks. Correlate them with job input and output evidence. Task-level evidence identifies skew and spill better than overall runtime alone. Compare successful and slow runs with the same data boundary and workload.

A join with one unusually long task suggests a hot key rather than insufficient capacity everywhere. Look at task distributions rather than only averages; one hot-key task can explain why a larger cluster produces little improvement.

## B. Advanced implementation questions

### A01. When should you broadcast a dimension?

When the actual serialized relation is small enough for executor memory and the join plan benefits. Verify size, skew and runtime behavior instead of forcing broadcasts universally. Broadcasting replicates the build-side data where supported, trading shuffle for memory. Estimates, actual relation size and executor pressure determine whether it helps.

A genuinely small approved reference can broadcast; a unexpectedly expanded dimension may require another join strategy. Check actual relation size, serialization and executor capacity; a dimension that was small last month may no longer be safe to broadcast.

### A02. How do you handle late dimension changes?

Maintain effective intervals, reconstruct affected key timelines and re-resolve impacted facts. Closing only the current row may mishandle out-of-order history. Historical integrity requires reconstructing intervals for late changes, not only updating current attributes. Then determine which existing facts used the affected interval.

A backdated account segment correction can require re-resolving last month's transaction facts. Keep source effective history so a correction can reconstruct the timeline, and identify all gold facts that joined to the changed interval.

### A03. How do you make foreachBatch safe?

Design the handler to be idempotent using batch/business keys and transactional target operations where supported. The function can execute more than once. Microbatch handlers can repeat after recovery. Durable target keys, version ordering and accepted batch evidence must make repetition safe.

A replayed microbatch MERGE does not append another daily usage total. Apply only authoritative new source versions and verify that repeated batches leave keys and aggregates unchanged even after a crash between write and checkpoint.

### A04. How do you tune cost and performance together?

Measure cost per accepted workload, startup/runtime, utilization, shuffle and query latency. Select compute and scaling from these measures, not only job duration. Cost improvements must preserve freshness and correctness. Measure startup, runtime, utilization and accepted output rather than reporting only a smaller cluster bill.

An idle-heavy job may benefit from different scheduling/compute configuration while retaining the agreed completion target. Compare cost per accepted run and freshness under representative concurrency; an inexpensive run that misses its business deadline is not an equivalent improvement.

### A05. How do you upgrade runtimes safely?

Pin dependencies, validate API/connector compatibility and run correctness/performance/replay fixtures in test. Promote with a rollback and data compatibility plan. Runtime upgrades can change APIs, connectors and execution behavior. Fixtures should validate values and recovery as well as whether code still imports.

Test decimal rounding, timestamp parsing, MERGE duplicates and stream restart before promoting a new runtime. Retain the previous runtime/package configuration and confirm that any schema/protocol changes still allow a compatible recovery or rollback.

## C. Scenario-based interview questions — 25 scenarios

Answer production scenarios in this order: clarify the business impact and processing boundary; inspect the specific evidence; isolate the cause; apply a bounded correction; replay/recover safely; prove the accepted outcome; prevent recurrence. Say which assumptions require checking rather than guessing an API/limit.

### Technology-specific design and troubleshooting

### S01. A Spark job spends most time in one task. What do you investigate?

Check skewed keys and partition sizes in task metrics. Filter early, redesign keys or use supported skew handling, then compare the new plan. Task-level evidence identifies skew and spill better than overall runtime alone. Compare successful and slow runs with the same data boundary and workload.

A join with one unusually long task suggests a hot key rather than insufficient capacity everywhere. Validate the accepted result and the intended input schema, source keys and silver/gold table contract. Retain the original failure and recovered result with their operation/run references. Where the operation mutates data or an external outcome, test a repeated attempt; for an access incident, also confirm the unrelated forbidden operation remains denied.

### S02. Executors fail after a forced broadcast. What do you do?

Confirm relation size and memory pressure, remove the forced hint if unsuitable, and evaluate a shuffle join with appropriate partitioning. Broadcasting replicates the build-side data where supported, trading shuffle for memory. Estimates, actual relation size and executor pressure determine whether it helps.

A genuinely small approved reference can broadcast; a unexpectedly expanded dimension may require another join strategy. Validate the accepted result and the intended input schema, source keys and silver/gold table contract. Retain the original failure and recovered result with their operation/run references. Where the operation mutates data or an external outcome, test a repeated attempt; for an access incident, also confirm the unrelated forbidden operation remains denied.

### S03. The driver crashes during collect(). What is the fix?

Remove large driver materialization. Write distributed output or use bounded samples; increasing driver size alone preserves the faulty scaling design. Driver memory and executor memory solve different problems. Distributed transformations scale across executors; collecting an unbounded result centralizes data on the driver.

Write a large usage aggregation directly to a table and collect only a bounded diagnostic sample. Validate the accepted result and the intended input schema, source keys and silver/gold table contract. Retain the original failure and recovered result with their operation/run references. Where the operation mutates data or an external outcome, test a repeated attempt; for an access incident, also confirm the unrelated forbidden operation remains denied.

### S04. A job creates thousands of tiny Delta files. How do you improve it?

Measure batch sizes and writer partitions, use suitable compaction/layout maintenance and verify retention. Avoid arbitrary repartitioning without workload evidence. Cost improvements must preserve freshness and correctness. Measure startup, runtime, utilization and accepted output rather than reporting only a smaller cluster bill.

An idle-heavy job may benefit from different scheduling/compute configuration while retaining the agreed completion target. Validate the accepted result and the intended input schema, source keys and silver/gold table contract. Retain the original failure and recovered result with their operation/run references. Where the operation mutates data or an external outcome, test a repeated attempt; for an access incident, also confirm the unrelated forbidden operation remains denied.

### S05. Two tasks MERGE the same target partition. How do you respond to conflicts?

Partition ownership or serialize conflicting writes, handle supported optimistic conflicts with bounded retry and verify idempotent results. Input/output parameters must identify an approved scope and remain valid under retry. Arbitrary target names and paths should not become unrestricted execution instructions.

The notebook accepts entity and run IDs and resolves approved target tables from reviewed configuration. Validate the accepted result and the intended input schema, source keys and silver/gold table contract. Retain the original failure and recovered result with their operation/run references. Where the operation mutates data or an external outcome, test a repeated attempt; for an access incident, also confirm the unrelated forbidden operation remains denied.

### S06. A reference change arrives after gold facts were published. What must change?

Rebuild the affected dimension timeline and identify dependent fact dates. Publish a reconciled revision rather than only changing the current dimension. Historical integrity requires reconstructing intervals for late changes, not only updating current attributes. Then determine which existing facts used the affected interval.

A backdated account segment correction can require re-resolving last month's transaction facts. Validate the accepted result and the intended input schema, source keys and silver/gold table contract. Retain the original failure and recovered result with their operation/run references. Where the operation mutates data or an external outcome, test a repeated attempt; for an access incident, also confirm the unrelated forbidden operation remains denied.

### S07. A notebook works interactively but not in a job. What do you compare?

Compare execution identity, runtime, dependencies, parameters, catalog/context and network permissions. Reproduce under the job identity. Input/output parameters must identify an approved scope and remain valid under retry. Arbitrary target names and paths should not become unrestricted execution instructions.

The notebook accepts entity and run IDs and resolves approved target tables from reviewed configuration. Validate the accepted result and the intended input schema, source keys and silver/gold table contract. Retain the original failure and recovered result with their operation/run references. Where the operation mutates data or an external outcome, test a repeated attempt; for an access incident, also confirm the unrelated forbidden operation remains denied.

### S08. A stream checkpoint is deleted. Is restart from latest safe?

Not automatically. Identify recoverable offsets/raw input and target idempotency; plan controlled replay instead of silently skipping data. Microbatch handlers can repeat after recovery. Durable target keys, version ordering and accepted batch evidence must make repetition safe.

A replayed microbatch MERGE does not append another daily usage total. Validate the accepted result and the intended input schema, source keys and silver/gold table contract. Retain the original failure and recovered result with their operation/run references. Where the operation mutates data or an external outcome, test a repeated attempt; for an access incident, also confirm the unrelated forbidden operation remains denied.

### S09. SQL warehouse queries slow during a batch workload. What would you isolate?

Measure concurrency and resource contention; separate serving and transformation compute where justified and tune the expensive query plan. Serving concurrency and transformation throughput have different resource patterns. Compute separation can reduce interference, but it introduces cost and operational configuration.

A heavy backfill should not automatically share the same execution resource as latency-sensitive BI queries. Validate the accepted result and the intended input schema, source keys and silver/gold table contract. Retain the original failure and recovered result with their operation/run references. Where the operation mutates data or an external outcome, test a repeated attempt; for an access incident, also confirm the unrelated forbidden operation remains denied.

### S10. A runtime upgrade changes output decimal rounding. How do you detect and prevent it?

Run explicit decimal/null/timezone fixtures and financial reconciliation before promotion. Record the runtime/rule version and roll back incompatible outputs. Runtime upgrades can change APIs, connectors and execution behavior. Fixtures should validate values and recovery as well as whether code still imports.

Test decimal rounding, timestamp parsing, MERGE duplicates and stream restart before promoting a new runtime. Validate the accepted result and the intended input schema, source keys and silver/gold table contract. Retain the original failure and recovered result with their operation/run references. Where the operation mutates data or an external outcome, test a repeated attempt; for an access incident, also confirm the unrelated forbidden operation remains denied.

### Production integration, recovery and operational scenarios

### S11. After moving one Databricks job or notebook task to production, an authorization error appears. How would you distinguish identity, permission and connectivity failures?

Capture the effective caller and exact error, then validate job identity, Unity Catalog grants, storage credential and network connectivity. Compare production with the working configuration using Spark UI stages, task skew, job logs and Delta operation history; do not resolve the incident by giving every identity broad access. Platform access, catalog grants and cloud storage permissions form different layers. Test the effective workload identity and eliminate unintended direct-storage bypasses. Investigate under the real caller rather than the interactive administrator. The relevant path is job identity, Unity Catalog grants, storage credential and network connectivity. Capture the exact failure and compare one allowed operation with one deliberately denied operation, so fixing connectivity does not accidentally broaden authorization.

A production job reads its domain tables through the approved catalog storage abstraction with narrowly scoped grants. Validate the accepted result and the intended input schema, source keys and silver/gold table contract. Retain the original failure and recovered result with their operation/run references. Where the operation mutates data or an external outcome, test a repeated attempt; for an access incident, also confirm the unrelated forbidden operation remains denied.

### S12. The health page says Azure Databricks is available, but the business result has not advanced. What would you check?

Trace the expected operation through ADLS source, Unity Catalog metadata and upstream orchestration and compare progress in Delta table version, job run ID and streaming checkpoint where relevant. Inspect Spark UI stages, task skew, job logs and Delta operation history to distinguish a service being reachable from an accepted business operation actually completing. The platform coordinates compute, governance and serving, while Spark distributes computation. Explain which workload uses which engine and where its data is persisted. Walk the dependency chain through ADLS source, Unity Catalog metadata and upstream orchestration and locate the last accepted evidence in Delta table version, job run ID and streaming checkpoint where relevant. Distinguish a source that delivered nothing, an execution that never started, a held result and a consumer that still reads an old version; those require different fixes.

A Databricks Spark job creates silver Delta tables and a SQL warehouse serves the accepted gold product. Validate the accepted result and the intended input schema, source keys and silver/gold table contract. Retain the original failure and recovered result with their operation/run references. Where the operation mutates data or an external outcome, test a repeated attempt; for an access incident, also confirm the unrelated forbidden operation remains denied.

### S13. A new release of job definition, notebook/package version and compute configuration works in test but breaks a dependent production consumer. How do you diagnose it?

Compare versions, environment settings and the actual input schema, source keys and silver/gold table contract. Hold the affected change/consumer path, identify an accepted compatible version, and recover both configuration and any changed durable state. The platform coordinates compute, governance and serving, while Spark distributes computation. Explain which workload uses which engine and where its data is persisted. Treat the release as a change to job definition, notebook/package version and compute configuration and its dependencies, not just to one code file. Compare input schema, source keys and silver/gold table contract and the execution identity between test and production. Restore a compatible state before allowing unsafe consumers to continue.

A Databricks Spark job creates silver Delta tables and a SQL warehouse serves the accepted gold product. Validate the accepted result and the intended input schema, source keys and silver/gold table contract. Retain the original failure and recovered result with their operation/run references. Where the operation mutates data or an external outcome, test a repeated attempt; for an access incident, also confirm the unrelated forbidden operation remains denied.

### S14. Latency doubles for one Databricks job or notebook task although the request or data volume is unchanged. What is your investigation sequence?

Compare a known-good baseline with Spark UI stages, task skew, job logs and Delta operation history and inspect shuffle spill, skewed tasks, scan volume and compute utilization. Check dependency/configuration changes before scaling; make one measured correction and validate business results are unchanged. The platform coordinates compute, governance and serving, while Spark distributes computation. Explain which workload uses which engine and where its data is persisted. Use Spark UI stages, task skew, job logs and Delta operation history to compare the same business boundary and concurrency against a known-good baseline. Examine shuffle spill, skewed tasks, scan volume and compute utilization and changed dependencies/configuration. A faster run is not acceptable if it changes values, permitted access or recovery behavior.

A Databricks Spark job creates silver Delta tables and a SQL warehouse serves the accepted gold product. Validate the accepted result and the intended input schema, source keys and silver/gold table contract. Retain the original failure and recovered result with their operation/run references. Where the operation mutates data or an external outcome, test a repeated attempt; for an access incident, also confirm the unrelated forbidden operation remains denied.

### S15. An operation involving Azure Databricks may have completed, but the caller timed out before receiving confirmation. How do you retry safely?

Inspect the durable outcome in Delta table version, job run ID and streaming checkpoint where relevant before repeating the operation. For mutations or external side effects, use a stable operation key and supported idempotent/transactional behavior; read-only retries have a different risk. Driver memory and executor memory solve different problems. Distributed transformations scale across executors; collecting an unbounded result centralizes data on the driver. Separate an uncertain response from an uncertain operation. The caller's timeout does not undo a completed write, grant, deployment or external request. Read the outcome represented by Delta table version, job run ID and streaming checkpoint where relevant and repeat only through a supported safe operation/status mechanism.

Write a large usage aggregation directly to a table and collect only a bounded diagnostic sample. Validate the accepted result and the intended input schema, source keys and silver/gold table contract. Retain the original failure and recovered result with their operation/run references. Where the operation mutates data or an external outcome, test a repeated attempt; for an access incident, also confirm the unrelated forbidden operation remains denied.

### S16. Two automated callers perform conflicting changes to one Databricks job or notebook task at the same time. How do you prevent lost or inconsistent state?

Identify the shared resource and ownership boundary in Delta table version, job run ID and streaming checkpoint where relevant. Use the platform's supported transaction/lease/lock or a serialized owner, and reconcile the final intended outcome rather than accepting whichever request finishes last. The transaction log defines active table state, including updates and removed files. Compatible readers use that state rather than reading every physical Parquet file. Define who owns the shared mutation and which state version each caller expected. Preserve Delta table version, job run ID and streaming checkpoint where relevant so a rejected or losing caller can reconcile rather than overwrite blindly. The exact transaction, lease or lock must be supported by the target service.

After a corrected transaction MERGE, the active table contains the accepted line while obsolete files may remain for retention. Validate the accepted result and the intended input schema, source keys and silver/gold table contract. Retain the original failure and recovered result with their operation/run references. Where the operation mutates data or an external outcome, test a repeated attempt; for an access incident, also confirm the unrelated forbidden operation remains denied.

### S17. An upstream contract change reaches Azure Databricks without prior notice. How do you contain the impact?

Validate against input schema, source keys and silver/gold table contract and distinguish compatible from breaking changes. Retain the original input/change evidence, stop unsafe dependent work and obtain a reviewed migration/version before resuming. Layer names matter less than clear contracts. Preserve raw provenance, conform business semantics, then publish grain-specific products with acceptance evidence. Classify the change against input schema, source keys and silver/gold table contract: additional optional information may be compatible, while key/type/meaning changes can be breaking. Preserve evidence and identify all affected consumers before approving a migration or versioned release.

Bronze CDRs retain payloads; silver resolves durable events; gold contains subscriber/day/service totals. Validate the accepted result and the intended input schema, source keys and silver/gold table contract. Retain the original failure and recovered result with their operation/run references. Where the operation mutates data or an external outcome, test a repeated attempt; for an access incident, also confirm the unrelated forbidden operation remains denied.

### S18. You need to investigate last month's incident, but some Delta table version, job run ID and streaming checkpoint where relevant evidence has expired. What can you recover and what should change?

Inventory actual retained records/backups and state honestly which boundaries can still be reconstructed. Align retention of Delta table version, job run ID and streaming checkpoint where relevant and operational evidence with recovery/audit requirements; do not promise recovery from expired artifacts. The transaction log defines active table state, including updates and removed files. Compatible readers use that state rather than reading every physical Parquet file. Recovery can reconstruct only what still exists and is consistent. Inventory Delta table version, job run ID and streaming checkpoint where relevant, relevant backups and dependency records, then state any gap explicitly. Retention must be set from agreed audit, replay and recovery use cases rather than an arbitrary cleanup interval.

After a corrected transaction MERGE, the active table contains the accepted line while obsolete files may remain for retention. Validate the accepted result and the intended input schema, source keys and silver/gold table contract. Retain the original failure and recovered result with their operation/run references. Where the operation mutates data or an external outcome, test a repeated attempt; for an access incident, also confirm the unrelated forbidden operation remains denied.

### S19. Sensitive identifiers appear in Spark UI stages, task skew, job logs and Delta operation history. How would you fix the logging and access design?

Minimize/redact payload values and use restricted references/correlation IDs. Review job identity, Unity Catalog grants, storage credential and network connectivity for logs and source objects, then follow the approved exposure response and validate logging with synthetic sensitive fields. Platform access, catalog grants and cloud storage permissions form different layers. Test the effective workload identity and eliminate unintended direct-storage bypasses. Keep identity, run/object references and reason codes needed for troubleshooting while excluding restricted payload values. Review who can read Spark UI stages, task skew, job logs and Delta operation history and derived exports; minimizing the source log alone does not remove already copied sensitive output.

A production job reads its domain tables through the approved catalog storage abstraction with narrowly scoped grants. Validate the accepted result and the intended input schema, source keys and silver/gold table contract. Retain the original failure and recovered result with their operation/run references. Where the operation mutates data or an external outcome, test a repeated attempt; for an access incident, also confirm the unrelated forbidden operation remains denied.

### S20. An automation for Azure Databricks accidentally points to production while rehearsing in development. What controls would you add?

Validate environment/resource identifiers in job definition, notebook/package version and compute configuration, use environment-scoped identities and fail closed on a configuration mismatch. Review any completed mutations against Delta table version, job run ID and streaming checkpoint where relevant before restoring normal processing. The platform coordinates compute, governance and serving, while Spark distributes computation. Explain which workload uses which engine and where its data is persisted. Environment validation belongs before mutation. Bind job definition, notebook/package version and compute configuration to approved resource names and scoped identities so the development execution cannot reach unintended production targets. If the mistake already happened, reconcile the exact affected state before rerunning.

A Databricks Spark job creates silver Delta tables and a SQL warehouse serves the accepted gold product. Validate the accepted result and the intended input schema, source keys and silver/gold table contract. Retain the original failure and recovered result with their operation/run references. Where the operation mutates data or an external outcome, test a repeated attempt; for an access incident, also confirm the unrelated forbidden operation remains denied.

### S21. Adls source, unity catalog metadata and upstream orchestration is temporarily unavailable. What retry and recovery policy fits Azure Databricks?

Distinguish transient connectivity/rate errors from invalid credentials/contracts. Use bounded platform-supported retry and preserved Delta table version, job run ID and streaming checkpoint where relevant; coordinate the owning system and avoid endless retries that duplicate side effects or hide stale results. The platform coordinates compute, governance and serving, while Spark distributes computation. Explain which workload uses which engine and where its data is persisted. Use the dependency's documented error and retry contract. Invalid permissions/schema are not transient outages; retries do not repair them. During an outage, keep Delta table version, job run ID and streaming checkpoint where relevant durable and make accepted-data/service staleness visible instead of reporting a healthy end-to-end operation.

A Databricks Spark job creates silver Delta tables and a SQL warehouse serves the accepted gold product. Validate the accepted result and the intended input schema, source keys and silver/gold table contract. Retain the original failure and recovered result with their operation/run references. Where the operation mutates data or an external outcome, test a repeated attempt; for an access incident, also confirm the unrelated forbidden operation remains denied.

### S22. Costs associated with Azure Databricks rise while accepted business volume stays flat. How do you investigate without losing recovery capability?

Break cost down by relevant compute, storage, requests, retention and dependency use, then inspect shuffle spill, skewed tasks, scan volume and compute utilization. Remove demonstrated waste while preserving required Delta table version, job run ID and streaming checkpoint where relevant and correctness/access controls. Cost improvements must preserve freshness and correctness. Measure startup, runtime, utilization and accepted output rather than reporting only a smaller cluster bill. Measure cost at an accepted business workload boundary, including repeated attempts and retained artifacts. Link changes to shuffle spill, skewed tasks, scan volume and compute utilization and test the same service targets afterward. Deleting recovery evidence to lower cost changes capabilities and requires an explicit approved retention decision.

An idle-heavy job may benefit from different scheduling/compute configuration while retaining the agreed completion target. Validate the accepted result and the intended input schema, source keys and silver/gold table contract. Retain the original failure and recovered result with their operation/run references. Where the operation mutates data or an external outcome, test a repeated attempt; for an access incident, also confirm the unrelated forbidden operation remains denied.

### S23. During a recovery exercise, job definition, notebook/package version and compute configuration is restored but the business path still fails. What else is required?

Restore/validate job identity, Unity Catalog grants, storage credential and network connectivity, ADLS source, Unity Catalog metadata and upstream orchestration and the appropriate Delta table version, job run ID and streaming checkpoint where relevant. Reconcile durable data/external outcomes and consumer routing before declaring the application path recovered. Platform access, catalog grants and cloud storage permissions form different layers. Test the effective workload identity and eliminate unintended direct-storage bypasses. Configuration restoration alone is incomplete when identities, network routing, dependencies or durable business state remain unavailable. Revalidate job identity, Unity Catalog grants, storage credential and network connectivity, reconcile Delta table version, job run ID and streaming checkpoint where relevant and execute a normal authorized consumer path before calling the recovery successful.

A production job reads its domain tables through the approved catalog storage abstraction with narrowly scoped grants. Validate the accepted result and the intended input schema, source keys and silver/gold table contract. Retain the original failure and recovered result with their operation/run references. Where the operation mutates data or an external outcome, test a repeated attempt; for an access incident, also confirm the unrelated forbidden operation remains denied.

### S24. Only a small subset of records, requests or objects fails in Azure Databricks. How do you isolate it without silently accepting incomplete results?

Correlate failing business keys/objects with Spark UI stages, task skew, job logs and Delta operation history and validate their input schema, source keys and silver/gold table contract. Preserve restricted exception evidence and apply the approved hold/quarantine/partial-success policy; do not blanket-ignore errors. The platform coordinates compute, governance and serving, while Spark distributes computation. Explain which workload uses which engine and where its data is persisted. Find the smallest complete failed business scope using keys/objects and Spark UI stages, task skew, job logs and Delta operation history. Distinguish invalid input from missing reference, permission or ordering conflicts. Any quarantine or partial success must follow input schema, source keys and silver/gold table contract and preserve a clear accounting of accepted versus failed scope.

A Databricks Spark job creates silver Delta tables and a SQL warehouse serves the accepted gold product. Validate the accepted result and the intended input schema, source keys and silver/gold table contract. Retain the original failure and recovered result with their operation/run references. Where the operation mutates data or an external outcome, test a repeated attempt; for an access incident, also confirm the unrelated forbidden operation remains denied.

### S25. You must change job definition, notebook/package version and compute configuration while dependent workloads remain active. What release sequence would you use?

Inventory consumers and input schema, source keys and silver/gold table contract compatibility, deploy additive/compatible prerequisites first and validate the actual execution path. Control conflicting activation, retain Delta table version, job run ID and streaming checkpoint where relevant and define a tested rollback/correction route for incompatible behavior. Input/output parameters must identify an approved scope and remain valid under retry. Arbitrary target names and paths should not become unrestricted execution instructions. Order changes by dependency and compatibility. Introduce safe prerequisites, verify both existing and transitioning consumers, then activate the intended route. Keep Delta table version, job run ID and streaming checkpoint where relevant and a tested correction/rollback boundary so a partially applied release remains recoverable.

The notebook accepts entity and run IDs and resolves approved target tables from reviewed configuration. Validate the accepted result and the intended input schema, source keys and silver/gold table contract. Retain the original failure and recovered result with their operation/run references. Where the operation mutates data or an external outcome, test a repeated attempt; for an access incident, also confirm the unrelated forbidden operation remains denied.

## D. Practical implementation exercises

These exercises have two-paragraph answers describing the implementation and expected evidence. Selected code or flow references are collected later in this same file. They do not provision Azure resources. Use development resources and synthetic/approved data. For code questions, name the engine/runtime and supply reproducible input/expected output; for governance/deployment questions, supply configuration and access/recovery evidence.

### P01. Create a parameterized bronze-to-silver notebook.

Use explicit schemas, bad-record handling, ordered deduplication and target controls. Prove replay produces the same business output. Layer names matter less than clear contracts. Preserve raw provenance, conform business semantics, then publish grain-specific products with acceptance evidence.

Bronze CDRs retain payloads; silver resolves durable events; gold contains subscriber/day/service totals. The corresponding code or flow reference is included in the implementation-reference section of this same file.

### P02. Diagnose a slow Spark join from a plan.

Identify join type, shuffle boundaries, estimates and task skew; propose a measured change and explain its tradeoff. Task-level evidence identifies skew and spill better than overall runtime alone. Compare successful and slow runs with the same data boundary and workload.

A join with one unusually long task suggests a hot key rather than insufficient capacity everywhere. Provide the implementation described above, its explicit inputs and expected results, and evidence for a normal case plus the failure/recovery case relevant to this question. Use the actual engine/runtime and approved identity; the example is a design/code starting point, not an executed cloud solution.

### P03. Build a multi-task job dependency graph.

Separate independent tasks, reference dependencies and critical publication gates. Include durable run IDs and failure recovery. Production execution should not depend on cells previously run by a human. Package dependencies, parameters and task order make the job reproducible.

A scheduled job receives an immutable input manifest and rule version instead of reading an interactive variable. Provide the implementation described above, its explicit inputs and expected results, and evidence for a normal case plus the failure/recovery case relevant to this question. Use the actual engine/runtime and approved identity; the example is a design/code starting point, not an executed cloud solution.

### P04. Create a failing-and-recovered test for one Databricks job or notebook task.

Capture the initial Delta table version, job run ID and streaming checkpoint where relevant, introduce one realistic dependency or validation failure and use Spark UI stages, task skew, job logs and Delta operation history to isolate it. Recover through the supported path and compare accepted outcomes before/after; demonstrate no extra mutation, duplicate side effect or unauthorized access. The platform coordinates compute, governance and serving, while Spark distributes computation. Explain which workload uses which engine and where its data is persisted.

A Databricks Spark job creates silver Delta tables and a SQL warehouse serves the accepted gold product. Provide the implementation described above, its explicit inputs and expected results, and evidence for a normal case plus the failure/recovery case relevant to this question. Use the actual engine/runtime and approved identity; the example is a design/code starting point, not an executed cloud solution.

### P05. Prepare the operating handoff for Azure Databricks.

Document job definition, notebook/package version and compute configuration, the input schema, source keys and silver/gold table contract, required job identity, Unity Catalog grants, storage credential and network connectivity, owner, normal/failure signals and recovery instructions using Delta table version, job run ID and streaming checkpoint where relevant. Include one normal and one failure example plus actual service targets and retention assumptions. The platform coordinates compute, governance and serving, while Spark distributes computation. Explain which workload uses which engine and where its data is persisted.

A Databricks Spark job creates silver Delta tables and a SQL warehouse serves the accepted gold product. Provide the implementation described above, its explicit inputs and expected results, and evidence for a normal case plus the failure/recovery case relevant to this question. Use the actual engine/runtime and approved identity; the example is a design/code starting point, not an executed cloud solution.

## E. Architecture and project discussion

### D01. Where does Azure Databricks fit in an end-to-end Azure data platform?

Use this specific flow: ADF or event source -> ADLS/Delta bronze -> Databricks job -> silver transformations -> gold Delta -> Databricks SQL or reporting. Explain what this component owns, how ADLS source, Unity Catalog metadata and upstream orchestration connects, and what the accepted consumer/result boundary is. The platform coordinates compute, governance and serving, while Spark distributes computation. Explain which workload uses which engine and where its data is persisted.

A Databricks Spark job creates silver Delta tables and a SQL warehouse serves the accepted gold product. Relate this example to the actual source boundary, required access and accepted consumer result. Explain which component you personally implemented, why the design suited the workload, and what measurable evidence establishes its correctness or improvement. Distinguish confirmed experience from a proposed reference architecture when discussing a named client.

### D02. How would you justify using Azure Databricks for lake modernization, banking transformations and telecom usage processing?

Start with actual volume, freshness, correctness, access and support requirements. Explain how one Databricks job or notebook task meets them, compare a credible alternative and validate using Spark UI stages, task skew, job logs and Delta operation history rather than choosing a tool merely because it is on the stack. Task-level evidence identifies skew and spill better than overall runtime alone. Compare successful and slow runs with the same data boundary and workload.

A join with one unusually long task suggests a hot key rather than insufficient capacity everywhere. Relate this example to the actual source boundary, required access and accepted consumer result. Explain which component you personally implemented, why the design suited the workload, and what measurable evidence establishes its correctness or improvement. Distinguish confirmed experience from a proposed reference architecture when discussing a named client.

### D03. What controls must surround Azure Databricks in a financial or telecom project?

Tie input schema, source keys and silver/gold table contract to accountable business ownership, verify job identity, Unity Catalog grants, storage credential and network connectivity, preserve Delta table version, job run ID and streaming checkpoint where relevant and validate outputs at matching business grain. Separate approved reference designs from evidence of work actually performed. Layer names matter less than clear contracts. Preserve raw provenance, conform business semantics, then publish grain-specific products with acceptance evidence.

Bronze CDRs retain payloads; silver resolves durable events; gold contains subscriber/day/service totals. Relate this example to the actual source boundary, required access and accepted consumer result. Explain which component you personally implemented, why the design suited the workload, and what measurable evidence establishes its correctness or improvement. Distinguish confirmed experience from a proposed reference architecture when discussing a named client.

### D04. Which measures would prove your Azure Databricks optimization helped?

Record the old/new configuration and comparable workload, measure shuffle spill, skewed tasks, scan volume and compute utilization with Spark UI stages, task skew, job logs and Delta operation history, and report latency/throughput/cost as relevant. Confirm unchanged correctness, access and recovery behavior before claiming improvement. Cost improvements must preserve freshness and correctness. Measure startup, runtime, utilization and accepted output rather than reporting only a smaller cluster bill.

An idle-heavy job may benefit from different scheduling/compute configuration while retaining the agreed completion target. Relate this example to the actual source boundary, required access and accepted consumer result. Explain which component you personally implemented, why the design suited the workload, and what measurable evidence establishes its correctness or improvement. Distinguish confirmed experience from a proposed reference architecture when discussing a named client.

### D05. How would you explain your own contribution to a Azure Databricks interview project?

Identify which job definition, notebook/package version and compute configuration you personally built/reviewed/operated, one specific failure you resolved, the evidence in Delta table version, job run ID and streaming checkpoint where relevant and a verified outcome. State team-owned components separately and do not invent scale or savings. The platform coordinates compute, governance and serving, while Spark distributes computation. Explain which workload uses which engine and where its data is persisted.

A Databricks Spark job creates silver Delta tables and a SQL warehouse serves the accepted gold product. Relate this example to the actual source boundary, required access and accepted consumer result. Explain which component you personally implemented, why the design suited the workload, and what measurable evidence establishes its correctness or improvement. Distinguish confirmed experience from a proposed reference architecture when discussing a named client.

## Revision notes and common mistakes

- Explain Azure Databricks's specific responsibility in the flow before naming surrounding tools.
- Use the real input schema, source keys and silver/gold table contract; an accepted request/job does not establish a correct business result.
- Validate job identity, Unity Catalog grants, storage credential and network connectivity under the actual execution identity, rather than your administrator account.
- Preserve Delta table version, job run ID and streaming checkpoint where relevant and inspect it before retrying uncertain side effects.
- Tune shuffle spill, skewed tasks, scan volume and compute utilization from measured evidence and confirm business correctness remains unchanged.
- Keep secret/personal data out of answers, fixtures and general logs; use synthetic examples.
- Runtime, connector, tier, licensing, limits and deployment support can change. Check the official documentation for the target configuration before quoting exact values.

## Self-assessment

Score each answer 0–3: **0** cannot explain; **1** can define the concept; **2** can propose a correct implementation; **3** can explain tradeoffs, failure handling and verification evidence. Revisit weak answers and complete at least one practical exercise before describing it as hands-on experience.

For a mock round, choose 3 core, 2 advanced, 5 scenario and 1 practical question. Answer aloud, draw the relevant flow and distinguish what you know from what you would inspect. Follow up with: “Which evidence proves this?”, “What happens after a retry?”, and “How would the consumer know the result is accepted?”

## Implementation references for the practical questions

These code and flow examples support the two-paragraph answers above. They are illustrative, require the stated engine/configuration and have not provisioned or executed cloud resources.

**Reference for P01 — Create a parameterized bronze-to-silver notebook.**

```python
# Illustrative notebook setup; paths and tables are reviewed job parameters.
from pyspark.sql import functions as F
raw = (spark.read.schema(input_schema).json(input_path)
       .withColumn("run_id", F.lit(run_id)))
raw.createOrReplaceTempView("staged_input")
# Next: validate the approved schema/business rules, detect contradictory
# source ties, select latest records by authoritative sequence, and MERGE
# into an existing compatible Delta target. Publish only after controls.
```

## Official references

The questions and detailed answers are original interview preparation guidance. The linked primary documentation describes the platform and selected semantics; use its version-specific pages for exact syntax, limits and capabilities. Overview pages do not independently substantiate every proposed production design.

- [Azure Databricks official documentation](https://learn.microsoft.com/en-us/azure/databricks/introduction/).
- [Structured Streaming with Unity Catalog](https://learn.microsoft.com/en-us/azure/databricks/structured-streaming/unity-catalog).

[Back to all 20 interview guides](README.md).


## F. Additional advanced implementation questions — A06–A15

### A06. How do you choose between repartition(), coalesce() and adaptive query execution?

Use `repartition()` when you need a shuffle to redistribute data or establish a useful partitioning pattern; use `coalesce()` mainly to reduce partitions with less movement when the current distribution is already acceptable. Neither operation should be added by habit: partition count, row width, skew and downstream operations determine whether it helps. Adaptive Query Execution (AQE), when enabled and supported by the runtime and query, can use runtime statistics to adjust shuffle partitioning, optimize certain joins and mitigate skewed partitions.

Inspect the physical plan and Spark UI before and after a change. Compare task duration distribution, shuffle read/write, spill, executor utilization and output file sizes on representative data. AQE does not fix every skew pattern, and an aggressive coalesce can leave too few tasks to use the cluster; retain the change only when repeatable measurements show an improvement without correctness or stability regressions.

### A07. How do you decide whether to cache or persist a DataFrame?

Cache or persist when an expensive DataFrame is reused by multiple downstream actions and recomputing its lineage costs more than retaining it. Choose the storage level based on data size, memory availability and the consequences of recomputation; avoid caching one-time intermediates or very large data that displaces more valuable cached data. Remember that a DataFrame is lazy, so the cache is populated only when an action evaluates it.

Measure execution time, storage usage, eviction and recomputation in the Spark UI. Unpersist when the reuse window ends, and test the job under realistic concurrency because a cache that helps in isolation can hurt other jobs. Do not treat caching as a default fix for slow queries; projection, predicate pushdown, join strategy, partition pruning and avoiding repeated actions may provide a better improvement.

### A08. How do you use Delta MERGE safely for CDC processing?

Define the source event contract first: business key, source sequence or change timestamp, operation type, event identity and delete semantics. Before `MERGE`, deduplicate the incoming batch to one deterministic winning record per target key, using a trustworthy source ordering field and a stable tie-breaker. Apply inserts, updates and deletes according to the source contract, and keep the target key and update condition explicit.

Test duplicate events, out-of-order events, equal timestamps, deletes and retry after an uncertain commit. A later-arriving event must not overwrite a newer target state simply because it was processed later. Capture batch/run IDs and source boundaries, compare inserted/updated/deleted counts, and use Delta transaction history as evidence; exact guarantees depend on source ordering, target design and the retry strategy rather than on `MERGE` alone.

### A09. What is the difference between schema enforcement and schema evolution?

Schema enforcement rejects incompatible writes according to the target table's schema and supported casting rules, protecting consumers from accidental shape changes. Schema evolution deliberately changes the accepted schema when a new field or compatible change is approved. Enabling automatic evolution broadly can make upstream drift silently alter curated contracts, so it should be limited to cases where evolving the schema is part of the intended data contract.

For production, validate required columns, types, nullability expectations and allowed extra fields before writing. Version the contract, route invalid records to quarantine with a reason, and test downstream consumers before publishing a changed schema. When evolution is approved, record the schema version and use a controlled deployment or explicit write option supported by the selected runtime; do not assume every type change is safe or automatically compatible.

### A10. How do you implement idempotent batch processing in Databricks?

An idempotent batch produces the same accepted target state when the same logical input is retried. Define a stable run or batch identifier, source boundary or manifest, deterministic transformation rules and target write semantics. Depending on the workload, use an overwrite limited to a validated partition, a key-based Delta `MERGE`, or a staging-and-publish pattern instead of blindly appending on every retry.

Simulate failures before write, during write and after commit but before orchestration receives success. On retry, verify whether the batch already committed and avoid duplicating its effects. Record source counts, accepted/rejected counts, target version and run status in an operational ledger; the ledger and target update must be designed together so a crash cannot mark an uncommitted output as complete.

### A11. How do you design a reliable Auto Loader ingestion pipeline?

Auto Loader incrementally discovers new files and tracks ingestion progress using its configured source and checkpoint state. Define the expected schema, schema location, checkpoint location, file notification or listing mode, rescued-data behavior and target write semantics. Separate source discovery state from the Delta target and avoid reusing one checkpoint for unrelated streams or incompatible query definitions.

Test newly arriving files, schema additions, malformed records, duplicate delivery and restart after cluster termination. Monitor file discovery lag, input rate, processed rate, rescued records and checkpoint health. A checkpoint is operational state, not a substitute for a source manifest or business reconciliation; retain enough source metadata and counts to investigate whether every expected file was processed and accepted.

### A12. How do you choose Structured Streaming trigger intervals and output modes?

The trigger interval controls how often the query attempts to process available data; shorter intervals can reduce processing delay but increase scheduling overhead and may not improve throughput if each micro-batch takes longer than the interval. Choose the interval using the business latency objective, observed batch duration, source arrival pattern and cost. Output mode—append, update or complete—must match the query's semantics and sink support.

Measure end-to-end event age, input rows, processed rows, batch duration and backlog rather than judging latency from the trigger setting alone. Validate how aggregations, watermarks and sink semantics affect which rows can be emitted. Use a representative load test and confirm restart behavior, because a fast trigger is not useful if the workload continually falls behind or emits incomplete results.

### A13. How do watermarks affect late data and state size?

A watermark expresses how late the engine is prepared to wait for event-time processing in supported stateful operations; it is not a guarantee that every event older than that threshold will be retained. Watermarks can bound state for aggregations and deduplication, but records arriving beyond the allowed lateness may not affect results depending on the query and output mode. Choose the threshold from observed event delay and business correction requirements.

Track event-time lag and late-record counts, and test data arriving just before and after the threshold. If late events have financial or operational significance, preserve raw events and define a correction or replay path rather than assuming the streaming result is final forever. Watermark settings should be documented with the downstream SLA and validated against actual source clocks and timestamp quality.

### A14. How do you design Unity Catalog access for production jobs?

Separate workspace access, catalog/schema/table privileges, storage access and workload identity. Grant a job identity only the privileges needed for its source and target objects, and use approved storage credentials or external locations for governed cloud storage access. Avoid embedding secrets in notebooks or granting broad storage permissions simply to bypass a table-level permission issue.

Troubleshoot an authorization failure by identifying the identity actually used by the job, the object being accessed, inherited and direct grants, external-location permissions, and relevant cloud IAM/network settings. Validate access through the same execution path as production, not only as an administrator in an interactive notebook. Include grant review and negative access tests in deployment so the job can read and write its intended data without expanding unrelated access.

### A15. How do you deploy Databricks jobs and notebooks across dev, test and production?

Treat notebooks or packages, job definitions, parameters, cluster or warehouse configuration, permissions and dependent data contracts as versioned release artifacts. Use source control and a repeatable deployment process, with environment-specific values supplied through reviewed configuration rather than hard-coded paths or credentials. Validate the deployed artifact and job graph, not just the source notebook, because production can differ in identity, dependencies, network access and runtime.

Promote the same tested artifact through environments, run smoke and contract tests, and use a canary or limited-scope run where practical before full publication. Define rollback criteria, compatible schema changes and ownership for downstream consumers. A rollback must account for data already written: reverting code alone does not reverse a bad Delta commit or safely undo external side effects.

## G. Additional production scenarios — S26–S40

### S26. A Delta MERGE fails because multiple source rows match the same target key. How do you fix it?

The immediate issue is that the incoming source is not unique at the target's merge grain, so the engine cannot deterministically decide which source row should update a target row. Inspect duplicate keys and the source event ordering fields, then define the business rule for selecting the winner—such as highest source sequence, latest valid change timestamp and a stable event ID tie-breaker. Do not simply drop duplicates without a deterministic rule.

Deduplicate the source before `MERGE`, write duplicate examples and counts to diagnostics, and test ties and out-of-order updates. Re-run the affected batch idempotently and compare target keys and row counts with the expected source state. Add a pre-merge uniqueness assertion so the pipeline fails early with an actionable message if the contract is violated again.

### S27. A streaming query restarts successfully but processes duplicate business events. What do you investigate?

Separate transport-level duplicate delivery from business-level duplicate events and from target-side duplicate writes. Check whether the checkpoint path is stable and unique to the query, whether the query definition or source changed, and whether the sink operation is idempotent. A checkpoint supports streaming progress, but it does not automatically enforce uniqueness for every business key across every downstream side effect.

Use a stable event ID or business key plus an appropriate event-time or source-sequence rule for deduplication. If using `foreachBatch`, make the batch write idempotent and test a retry after a partial or uncertain commit. Reconcile source offsets or input counts with target accepted counts, and avoid deleting the checkpoint as a routine way to “fix” duplicates because doing so can reset progress and replay data.

### S28. A Databricks job succeeds, but the target table contains fewer records than the source. How do you troubleshoot it?

A successful task status only proves that the code completed according to its own logic; it does not prove business completeness. Compare source count and source scope first, including date filters, watermark boundaries, pagination, partition pruning and deleted or updated source rows. Then examine filtering, joins that remove unmatched records, deduplication rules, invalid-record quarantine and write predicates.

Create a reconciliation by batch and business key, reporting source rows, transformed rows, rejected rows, duplicates, inserted/updated/deleted rows and final target rows. Verify that the comparison uses the same grain and snapshot boundary, and inspect Delta history to confirm the intended commit. Add explicit acceptance thresholds and fail publication when unexplained variance exceeds the approved tolerance.

### S29. Spark UI shows high shuffle spill and long-running tasks. What is your action plan?

Spill indicates that intermediate data could not be held entirely in memory and was written to disk; it can result from large shuffles, oversized partitions, wide rows, skew or insufficient resources. Inspect the physical plan and Spark UI for shuffle read/write, spill, task-size distribution, GC and executor utilization. Determine whether one or a few tasks dominate or whether all tasks are under pressure.

Reduce unnecessary columns before shuffles, filter earlier, correct join strategy, adjust partitioning and use AQE where supported. If the workload remains resource-bound after plan improvements, evaluate compute sizing based on measured executor and memory behavior. Benchmark with the same input scale and verify output equality; increasing cluster size alone may mask a poor plan or worsen cost without fixing skew.

### S30. A job starts failing after an upstream source adds a column and changes a data type. How do you respond?

Treat the change as a source-contract event rather than assuming schema evolution should accept it. Compare the actual schema with the expected contract, identify the changed field and determine whether the type change is compatible with the transformation and downstream table. Preserve the raw payload or source files and quarantine incompatible records when that is part of the ingestion design.

Coordinate an explicit contract update, add tests for old and new shapes, and deploy the transformation and target schema change in a controlled sequence. Verify the impact on existing Delta data, SQL consumers and semantic definitions. Record the accepted schema version and ensure the next run can replay the affected source range without duplicating already accepted records.

### S31. A scheduled job runs twice for the same business date. How do you prevent double counting?

Identify why the duplicate execution occurred—overlapping schedules, a manual run, orchestration retry or a caller retry after timeout. Determine whether the job's output is append-only or keyed by business date and entity. A schedule-level setting can reduce overlapping execution, but target-side idempotency is still needed because retries and ambiguous outcomes can occur after a write has committed.

Use a stable logical batch key, a concurrency policy appropriate to the job and deterministic target writes such as partition replacement or `MERGE`. Track run attempts separately from the logical batch, and check the target commit before replaying. Test two simultaneous invocations and a retry after commit to demonstrate that the accepted business total is unchanged.

### S32. A Databricks job is waiting for compute for a long time. How do you investigate?

Distinguish cluster startup delay, quota or capacity constraints, policy restrictions, unavailable instance types, dependency installation, workspace/network problems and job concurrency limits. Compare run lifecycle timestamps with prior successful runs and inspect the job's compute configuration, pool or serverless settings where applicable, workspace events and any quota or service health information available to the team.

If the wait is caused by repeated environment setup, consider supported pools or managed/serverless compute options after checking policy and workload compatibility. Reduce unnecessary concurrent launches and avoid increasing retries for a capacity issue that will not resolve quickly. Alert on queue/startup time separately from execution duration so the team can distinguish scheduling latency from slow Spark processing.

### S33. A notebook succeeds interactively but fails under the production job identity. What should you check?

Interactive users may have broader permissions, a different identity, environment variables, installed libraries, current working context or previously initialized state. Compare the actual job run identity, catalog grants, external-location and cloud permissions, secret access, runtime version, package versions, parameters and network path. Review the first failing operation rather than granting administrator access as a shortcut.

Reproduce the failure through a job run using the intended identity and a clean environment. Make dependencies explicit, replace notebook-session assumptions with parameters, and apply least-privilege grants to the exact required objects. Add a deployment smoke test that reads a small approved input and writes to a controlled target before promoting the job.

### S34. A Delta table's storage folder contains files that do not appear in query results. Is the table corrupt?

Not necessarily. Delta tables use a transaction log to define the active snapshot; physical files may remain after updates or deletes until supported cleanup removes eligible files. A direct file listing or raw Parquet scan can therefore show data that is not part of the current table state. First establish whether the table is Delta and whether the reader is respecting its transaction log.

Query the table through a supported Delta reader, inspect table history and metadata, and check whether retention, concurrent writes or unsupported direct file manipulation occurred. Do not manually delete files from a live table directory. Use supported maintenance operations and follow retention and time-travel requirements; if files were changed outside the protocol, escalate and recover from a known-good version or backup strategy.

### S35. A production MERGE takes much longer as the target table grows. How do you optimize it?

Inspect whether the merge scans a large portion of the target, whether the source is deduplicated, whether the join keys align with partitioning or clustering strategy, and whether file sizes or data skipping are effective. Review the query plan, Delta operation metrics, shuffle, file counts and the number of files added or removed. Avoid partitioning by high-cardinality or low-selectivity columns without evidence that it narrows reads.

Reduce the source to the intended change range, filter safely using target-compatible predicates, and consider supported clustering, compaction or partition design based on actual access patterns. Benchmark on representative table sizes and verify correctness for updates and deletes that cross any filtering boundary. Monitor both runtime and maintenance cost so an optimization does not merely shift the cost into frequent rewrites.

### S36. A downstream report reads a gold table while the pipeline is updating it. How do you avoid inconsistent publication?

Define what consumers are allowed to see: a transactional table snapshot, a complete business-date partition, or a set of mutually consistent tables. Delta provides table-level transactional behavior, but a multi-table pipeline should not assume that separate table commits become visible atomically as one business release. A consumer could otherwise observe gold tables from different processing versions.

Use a publication pattern such as versioned outputs plus a controlled release pointer or a completion manifest that consumers honor. Validate all required tables and quality checks before publishing the release, and include a batch/version identifier in downstream products where useful. Test consumer behavior during a failed or delayed publish and document how to revert to the last accepted release.

### S37. A streaming aggregation's state store grows continuously and batches slow down. What would you do?

Inspect whether the query uses stateful operations such as aggregations, joins or deduplication, and whether event-time progress and watermarks are advancing. Invalid timestamps, stalled source partitions or a watermark configured too conservatively can retain state longer than expected; high-cardinality grouping keys can also make the state inherently large. Check batch metrics, state rows/bytes and input event-time distribution.

Correct timestamp parsing and source clock issues, select a watermark based on observed lateness and business needs, and reduce unnecessary state keys or aggregation scope. Preserve raw events and design a late-data correction path before tightening the watermark. Validate that the query remains correct around the lateness boundary and that state size stabilizes under representative input.

### S38. An executor is repeatedly lost during a large shuffle. How do you distinguish data skew from infrastructure failure?

Compare failed task identities, partition sizes, executor logs, container termination reasons, GC, spill and host or network events. Skew usually appears as a small number of unusually large or slow partitions; infrastructure failures may affect executors independently of key distribution or coincide with resource termination and node events. Both can happen together, so task-duration shape alone is not conclusive.

Reproduce with task-level metrics and inspect the largest keys or partitions without exposing sensitive values. If skew is confirmed, consider AQE skew handling, salting for suitable aggregations/joins, pre-aggregation or a revised join strategy. If the executor is being killed for memory or infrastructure reasons, adjust the plan or resource profile and involve the platform team with timestamps and run identifiers; verify stability and output correctness after the fix.

### S39. A job retry succeeds, but the incident keeps recurring every day. What should change?

A successful retry is a recovery outcome, not root-cause resolution. Group failures by error signature, upstream dependency, input volume, run time and environment change; determine whether the pattern is transient throttling, schema drift, data-dependent skew, resource pressure or a deterministic code defect. Check whether retries are hiding repeated delay and causing overlapping runs or late publication.

Set alerts on recurrence rate, retry count, time-to-start and business SLA impact, then implement a corrective action with a named owner and verification test. Use bounded retries with backoff for transient errors and fail fast for deterministic contract or permission failures. Close the incident only after the fix is demonstrated over representative runs and a runbook explains safe recovery.

### S40. A job is green, but business KPIs are wrong. How do you structure the investigation?

Separate technical execution success from data-quality acceptance and business-semantic correctness. Verify input scope, timezone and date boundaries, source freshness, deduplication precedence, joins and cardinality, null handling, currency or unit conversions, and aggregation grain. Trace a small set of business keys from bronze through silver into gold and compare intermediate values with the authoritative source or agreed business rule.

Add layered tests for schema, uniqueness, referential integrity, accepted ranges, reconciliation and KPI invariants. Make publication conditional on critical checks and record rule versions and rejected-record reasons. After correcting the transformation, replay the affected scope idempotently, quantify the downstream impact, and obtain business-owner sign-off when the metric definition itself—not just the code—was ambiguous.

## H. Additional practical implementation exercises — P06–P10

### P06. Implement deterministic latest-record selection before a Delta MERGE.

Create a sample change DataFrame containing repeated business keys, source sequence numbers, change timestamps and event IDs. Use a window ordered by the authoritative source sequence and a stable tie-breaker to select exactly one winning row per key. Assert uniqueness before the merge and include a test where two records have the same timestamp but different sequence numbers.

Merge the winning records into a Delta target using explicit insert/update conditions and test that replaying the same batch does not change the final state. Include a delete event if the source contract supports deletes. Produce reconciliation counts for incoming rows, duplicate rows, selected rows and target changes, and document the ordering rule used to resolve conflicts.

### P07. Build a batch reconciliation and quality-gate report.

For one logical batch, calculate source count, transformed count, rejected count, duplicate count and target count at the same data grain and source boundary. Add required-column checks, business-key uniqueness and a representative business-total reconciliation. Persist the batch ID, input boundary, rule version, table version where available, check result and failure reason in an operational table or report.

Define which checks are blocking and which are warning-only with explicit thresholds approved by the data owner. Simulate a missing partition and a duplicate source key, then verify that the pipeline prevents publication and exposes actionable diagnostics. A green job status should be separate from a passed data-quality status so orchestration cannot mistake successful execution for accepted data.

### P08. Create a safe Structured Streaming recovery drill.

Build a small stream with a stable checkpoint, explicit schema, event ID and event timestamp, then write to a Delta target using a design appropriate to the output semantics. Stop and restart the query using the same checkpoint, inject duplicate and late events, and record source progress, batch durations, target counts and accepted event IDs. Include a controlled failure during sink processing if your environment allows it.

Verify that restart resumes from saved progress and that duplicate delivery does not create duplicate business effects. Document when a checkpoint must be preserved, when a new checkpoint represents a new query, and how raw source data can support replay. Do not delete or reuse checkpoints casually; include the recovery boundary and reconciliation steps in the exercise report.

### P09. Diagnose and improve a skewed join using measured evidence.

Generate or select a dataset with a deliberately skewed join key and record a baseline: stage duration, task-duration distribution, shuffle volume, spill and the slowest partitions. Inspect the physical plan and determine whether a broadcast join is feasible based on the measured size of the smaller side and executor memory, rather than forcing a strategy based only on row count.

Compare an appropriate fix such as AQE skew handling, pre-aggregation, salting or a revised join strategy. Validate row counts and aggregate equality against the baseline, then compare runtime and compute cost over repeated runs. Document the data characteristics that made the chosen strategy effective and the conditions under which it would not be appropriate.

### P10. Prepare a production release and rollback package for a Databricks job.

Create a release checklist containing source commit, notebook/package version, runtime, job parameters, compute configuration, permissions, source/target contracts, smoke-test results and the rollback artifact. Deploy to a test environment with production-like identity and representative data volume, then execute a limited-scope run and verify the accepted output and downstream compatibility before full promotion.

For rollback, identify both code/configuration reversal and data recovery requirements. Demonstrate what happens if a bad release writes one Delta commit or publishes several gold tables before failing. Record the last accepted version, safe replay scope, approver and communication plan; a code rollback alone is insufficient if incorrect data has already been exposed to consumers.
