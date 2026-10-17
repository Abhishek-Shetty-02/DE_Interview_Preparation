# Data Governance Interview Guide — In-Depth Questions and Production Scenarios

**Purpose:** A practical, interview-focused reference for data engineers, analytics engineers, data architects, platform engineers, and governance practitioners. It covers governance principles, operating models, metadata, data quality, lineage, privacy, security, retention, cloud governance, and real production scenarios.

> **How to use this guide:** For each question, start with the concise interview answer, then expand with the implementation details and trade-offs. Treat regulatory obligations as jurisdiction- and organization-specific: confirm the applicable law, contractual requirements, and internal policies with the privacy, security, legal, and compliance teams.

## Table of contents

1. Fundamentals and principles (DG01–DG15)
2. Governance operating model and ownership (DG16–DG27)
3. Metadata, catalog, lineage, and discovery (DG28–DG40)
4. Data quality, observability, and issue management (DG41–DG54)
5. Privacy, security, access, and responsible use (DG55–DG70)
6. Lifecycle, retention, archival, and deletion (DG71–DG80)
7. Cloud and platform implementation (DG81–DG95)
8. Governance metrics, controls, and audits (DG96–DG105)
9. Advanced architecture and cross-domain scenarios (DG106–DG120)
10. Production troubleshooting scenarios (S01–S15)
11. Practical exercises and sample answers
12. End-to-end implementation blueprint
13. Interview revision checklist
14. Further reading

---

# 1. Fundamentals and principles

## DG01. What is data governance, and why is it important?

**Interview answer:** Data governance is the operating framework of decision rights, policies, standards, roles, and controls that ensures data is trustworthy, secure, discoverable, appropriately used, and managed throughout its lifecycle. It makes accountability explicit and translates business and regulatory expectations into repeatable technical controls.

**In depth:** Governance is not just a catalog, a committee, or a collection of documents. It connects business meaning and ownership to operational implementation. For example, a bank's definition of *active customer* should have an accountable business owner, a documented definition, approved source systems, quality expectations, access rules, lineage to reports, and a process for resolving disputes.

Typical outcomes include:
- Consistent definitions and fewer conflicting reports.
- Clear ownership and faster issue resolution.
- Appropriate access to sensitive data.
- Evidence that controls operate as intended.
- Better reuse, discovery, quality, and confidence in analytics and AI.

**Strong interview point:** Governance defines what must be true and who is accountable; data management and engineering implement the processes and controls that make it true.

## DG02. How is data governance different from data management?

- **Data governance:** decision rights, policies, accountability, standards, risk appetite, and oversight.
- **Data management:** operational work such as ingestion, modeling, quality remediation, master data management, storage, integration, and lifecycle execution.
- **Data security:** safeguards for confidentiality, integrity, availability, and appropriate use; it is a key control area within the broader governance ecosystem.
- **Data architecture:** structures, integration patterns, platforms, and standards used to organize and move data.

Example: Governance establishes that customer identifiers must be protected and access approved. Data management standardizes identifiers; security implements role-based access and masking; engineering enforces the controls in pipelines and serving layers.

## DG03. What are the core principles of effective data governance?

1. **Accountability:** each critical data asset has an owner and operational steward.
2. **Business alignment:** controls focus on decisions, products, risk, and regulatory obligations.
3. **Defined meaning:** business terms, metrics, and classifications have documented definitions.
4. **Quality by design:** checks occur at the point where errors can be detected and contained.
5. **Least privilege:** access is limited to the minimum required.
6. **Privacy by design:** purpose, minimization, retention, and protection are considered early.
7. **Traceability:** lineage and change history make data movement and transformations explainable.
8. **Lifecycle management:** data is retained, archived, and deleted according to approved rules.
9. **Evidence and auditability:** controls produce reviewable logs and outcomes.
10. **Continuous improvement:** metrics reveal gaps and guide remediation.

## DG04. What is the difference between a policy, standard, procedure, and control?

- **Policy:** high-level rule or intent, e.g. sensitive personal data must be protected.
- **Standard:** mandatory measurable requirement, e.g. production access must be approved and reviewed periodically.
- **Procedure:** the steps to satisfy a requirement, e.g. how a steward requests a classification change.
- **Control:** a mechanism that prevents, detects, or corrects a violation, e.g. an access approval workflow, automated scan, or periodic access review.
- **Evidence:** records demonstrating the control operated, e.g. approval logs, scan results, review sign-offs, and remediation tickets.

A strong program links each policy to standards, controls, owners, evidence, exceptions, and review dates.

## DG05. What is a data domain?

A data domain is a business-oriented grouping of related data with coherent meaning and accountability. Examples include Customer, Account, Payments, Product, Supplier, Employee, and Claims. Domains should reflect business capabilities and ownership rather than merely database schemas or organizational reporting lines.

A domain may own data products and definitions while consuming shared enterprise standards for identity, classification, access, and interoperability. Cross-domain data needs clear contracts and dispute-resolution paths.

## DG06. What is a data asset, and how do you determine its criticality?

A data asset can be a table, file, stream, API, semantic model, dashboard, feature set, report, or data product. Criticality depends on business impact, regulatory significance, sensitivity, downstream dependencies, recovery needs, and the consequences of inaccurate or unavailable data.

A practical tiering model might define:
- **Tier 1:** financial, regulatory, safety-critical, or customer-impacting data; strict controls and recovery targets.
- **Tier 2:** important operational and management analytics.
- **Tier 3:** lower-impact exploratory or temporary data.

Do not classify an asset based only on its size or storage location. A small reference table used in regulatory reporting may be more critical than a large raw log table.

## DG07. What is the difference between data governance and data quality?

Data quality is one governance capability. Governance establishes who defines acceptable quality, which dimensions matter, what thresholds apply, who receives alerts, and how exceptions are approved. Quality engineering implements checks, profiling, monitoring, and remediation.

Common dimensions are accuracy, completeness, validity, consistency, uniqueness, timeliness, and integrity. The relevant dimensions and thresholds depend on the use case; 100% completeness is not always appropriate if a field is legitimately optional.

## DG08. What is data stewardship?

Data stewardship is the day-to-day coordination that keeps data definitions, quality, metadata, and usage aligned with policy. A steward typically maintains business glossary terms, reviews quality exceptions, coordinates issue resolution, supports classification, and validates changes with subject-matter experts.

A steward may not own the business risk or approve all access. Decision rights must be explicit: the data owner is accountable, the steward coordinates, the custodian/platform team operates controls, and security/privacy/legal specialists advise or approve where required.

## DG09. What is a business glossary?

A business glossary is a managed collection of business terms, definitions, synonyms, ownership, examples, and relationships. It reduces ambiguity in metrics and concepts such as *net revenue*, *active account*, *delinquent loan*, or *unique customer*.

A good term record includes a preferred name, definition, domain, owner, steward, status, applicable rules, related technical assets, and effective date/version. Definitions should be understandable to business users and precise enough to guide implementation.

## DG10. What is a data dictionary, and how does it differ from a business glossary?

A data dictionary documents technical structures: schemas, columns, types, nullability, keys, constraints, descriptions, and sometimes example values. A business glossary documents business concepts. The catalog can connect a glossary term such as *Customer Lifetime Value* to the tables, columns, transformations, and semantic measures that implement it.

Neither replaces the other. A column called `cust_stat_cd` may have a technical type of `STRING`, but its allowed values and business meaning require metadata and a controlled definition.

## DG11. What is data classification?

Classification labels assets according to sensitivity and handling requirements. A typical scheme might include Public, Internal, Confidential, and Restricted, but the exact labels and obligations must come from organizational policy.

Classification can be based on content, context, source, business purpose, and regulatory requirements. Automated scanning helps identify likely sensitive fields, but ambiguous cases need owner review. Classification should flow into access controls, masking, encryption, retention, sharing restrictions, and audit monitoring.

## DG12. What is a data contract?

A data contract is an explicit agreement between a producer and consumers covering schema, semantics, quality expectations, freshness, delivery behavior, ownership, compatibility, and change management. It can include schema definitions, required fields, enumerations, keys, service-level objectives, and notification rules.

A contract should be versioned and tested in CI/CD. Breaking changes should be detected before production deployment, and consumers should have a migration window. A schema alone is not a complete contract because it does not fully describe meaning, quality, or operational expectations.

## DG13. What is the difference between data governance and data democratization?

Data democratization enables appropriate users to find and use data without unnecessary bottlenecks. Governance makes that access safe and consistent through discovery metadata, clear definitions, approved access paths, quality indicators, and usage restrictions.

The goal is not unrestricted access. It is **easy access to the right data for the right purpose by the right people**, with appropriate safeguards and accountability.

## DG14. What is the difference between centralized, decentralized, and federated governance?

- **Centralized:** a central team owns most policies and decisions. It offers consistency but can become a bottleneck.
- **Decentralized:** domains make most decisions. It improves local responsiveness but risks inconsistent standards.
- **Federated:** central teams define enterprise guardrails while domains own data and execute agreed responsibilities. It balances consistency and autonomy.

For a large multi-domain organization, federated governance is often practical: central ownership of classification taxonomy, identity standards, glossary conventions, and audit requirements; domain ownership of definitions, data products, and quality remediation.

## DG15. How do you measure governance maturity?

Assess capabilities rather than merely counting tools or documents. A useful maturity scale is ad hoc, repeatable, defined, measured, and continuously improved. Evaluate ownership coverage, catalog completeness, classification coverage, quality-rule coverage, lineage availability, access-review completion, retention compliance, issue-resolution time, and control evidence.

Use a baseline, target state, accountable owners, and quarterly improvement plans. Avoid declaring maturity based only on a vendor implementation or a high number of cataloged assets.

---

# 2. Governance operating model and ownership

## DG16. Who should own data in an enterprise?

The business data owner is accountable for the meaning, quality expectations, permitted uses, and risk decisions for a domain or asset. The platform or IT custodian is accountable for technical operation and protection mechanisms. Stewards maintain definitions and coordinate quality. Security, privacy, legal, and compliance teams provide specialized requirements and oversight.

Ownership must be documented in a catalog and supported by escalation paths. A named person or accountable role is more useful than a generic team mailbox with no decision authority.

## DG17. Explain a RACI model for data governance.

- **Responsible:** performs the work, such as implementing a quality check.
- **Accountable:** owns the outcome and makes the decision; ideally one accountable role per decision.
- **Consulted:** provides expertise before the decision.
- **Informed:** receives updates after or during the decision.

For a sensitive-field classification change, the data owner may be accountable, the steward responsible for coordinating the review, security/privacy consulted, and downstream consumers informed. Exact allocation varies by policy and risk.

## DG18. What is a data governance council?

A governance council resolves cross-domain issues, approves enterprise standards, prioritizes remediation, manages exceptions, and tracks risk. It should have decision rights, a clear membership model, an agenda tied to measurable outcomes, and a path for decisions to reach implementation teams.

A council should not review every low-risk data issue. Routine issues should be resolved by domain owners and stewards; only cross-domain conflicts, high-risk exceptions, or enterprise policy changes should escalate.

## DG19. How do you assign ownership when multiple teams use the same dataset?

Separate producer accountability from consumer responsibilities. The producer owns the published contract and source-quality commitments; the domain owner owns business meaning and acceptable use; consumers own their derived transformations and downstream interpretation. Shared or mastered entities need a designated authoritative owner and conflict-resolution process.

Record ownership, consumers, dependencies, and change-notification requirements in the catalog or data-product registry.

## DG20. How would you implement governance in a new data platform?

1. Identify business goals, regulatory obligations, high-risk use cases, and stakeholders.
2. Inventory domains, source systems, critical assets, and data flows.
3. Define decision rights, ownership, classification, and minimum metadata.
4. Establish glossary, catalog, lineage, quality, access, and lifecycle standards.
5. Embed automated controls in ingestion, transformation, publishing, and access workflows.
6. Start with one high-value domain and demonstrate measurable outcomes.
7. Expand through reusable templates, automation, training, and governance metrics.
8. Review exceptions and incidents to improve the control framework.

Avoid attempting to catalog and perfect every asset before delivering any value.

## DG21. How do you prevent governance from becoming a bottleneck?

Use risk-based controls, delegated domain ownership, automated policy checks, reusable templates, standard access roles, self-service discovery, and predefined exception paths. Low-risk routine requests can be approved through policy-based workflows; high-risk or unusual requests receive human review.

Measure request lead time, percentage automated, exception volume, and control failures. Governance should reduce rework and uncertainty, not add approval steps without clear risk reduction.

## DG22. How do you resolve disagreements over a business definition?

Identify the decision and use case first. Compare source semantics, contractual definitions, accounting or regulatory meaning, and downstream consequences. Convene the relevant owner and stewards, document the selected definition and alternatives, version the glossary term, and assess impacted pipelines and reports.

If the definitions are legitimately different, do not force a false enterprise-wide definition. Publish separate terms such as *booked revenue* and *recognized revenue*, with clear distinctions.

## DG23. How do you govern a data product?

Define the product owner, consumers, intended use, schema, business definitions, quality SLOs, freshness SLOs, access policy, classification, lineage, support model, versioning, and retirement process. Publish a discoverable interface and consumer documentation.

Treat a data product as a supported product rather than a table with a friendly name. Measure adoption, reliability, quality, consumer satisfaction, and the cost of support.

## DG24. What is a governance exception, and how should it be handled?

An exception is a time-bound, explicitly approved deviation from a standard. Record the requirement being waived, business justification, affected assets, risk assessment, compensating controls, approver, expiration date, and remediation plan.

Exceptions should be searchable and monitored. Avoid permanent informal exceptions or treating a ticket closure as proof that risk has been accepted. Renewals should require review rather than happen automatically.

## DG25. How do you onboard a new data domain?

Identify the domain sponsor and owner; inventory critical sources and consumers; define terms and classification; map key entities and identifiers; assign stewards; document quality and freshness requirements; catalog assets and lineage; define access and retention; publish contracts; and agree on issue SLAs and escalation. Start with critical data and expand iteratively.

## DG26. How do you embed governance in agile delivery?

Include governance acceptance criteria in backlog items and definition of done. Examples include an owner, glossary mapping, classification, schema contract, quality tests, lineage update, access review, retention label, and documentation for every production data product. Use CI checks to block material violations and automated workflows to route reviews.

Governance should be part of design, pull requests, release gates, and operational monitoring—not a late-stage sign-off after a dataset has already spread.

## DG27. How do you handle governance across vendors and third parties?

Inventory the data shared, purpose, categories, destination, contractual restrictions, retention, subprocessors, residency constraints, and deletion obligations. Use approved transfer methods, least-privilege credentials, encryption, logging, periodic access review, and vendor risk assessment. Track data lineage into and out of the third party where feasible.

Contracts and technical controls must align; an agreement alone does not prove that data is actually protected or deleted.

---

# 3. Metadata, catalog, lineage, and discovery

## DG28. What is metadata? Explain its types.

- **Business metadata:** terms, definitions, owners, domain, criticality, and intended use.
- **Technical metadata:** schemas, types, keys, formats, storage locations, jobs, and platform details.
- **Operational metadata:** run status, row counts, freshness, duration, failures, and processing timestamps.
- **Governance metadata:** classification, access policy, retention, approval, certification, and policy status.
- **Lineage metadata:** relationships among sources, transformations, assets, and consumers.

The categories overlap in real catalogs. The important point is to capture metadata that supports a specific decision or control, keep it current, and assign responsibility for changes.

## DG29. What is a data catalog?

A data catalog is a searchable inventory that connects data assets with descriptions, ownership, technical metadata, classifications, lineage, quality indicators, and usage guidance. It helps users answer: what data exists, what does it mean, who owns it, can I use it, how trustworthy is it, and where did it come from?

A catalog becomes stale if ingestion and transformation pipelines do not update metadata or if owners are not accountable for business descriptions. Automate technical capture and use stewardship workflows for context that automation cannot reliably infer.

## DG30. How would you design a metadata management process?

1. Define required metadata fields by asset type and criticality.
2. Capture technical schemas and pipeline metadata automatically.
3. Scan or infer classifications, then route uncertain results for review.
4. Assign owners and stewards at registration time.
5. Connect glossary terms to tables, columns, measures, and reports.
6. Capture lineage from orchestration, SQL, notebooks, and BI tools.
7. Validate metadata completeness and freshness continuously.
8. Notify owners when schemas, sources, or downstream dependencies change.

Use APIs and event-driven updates where possible; avoid relying entirely on manual spreadsheet inventories.

## DG31. What is data lineage?

Data lineage describes how data moves and changes from origin through ingestion, transformations, storage, models, and consumption. It can exist at system, dataset, column, or business-rule level.

Lineage supports impact analysis, root-cause investigation, audit evidence, privacy deletion tracing, and change management. It is only as reliable as the metadata captured: dynamic SQL, opaque transformations, external APIs, and manually created extracts may leave gaps that should be labeled explicitly.

## DG32. Explain source-to-target lineage versus end-to-end lineage.

Source-to-target lineage describes a specific mapping from source fields to target fields, including transformations. End-to-end lineage follows dependencies across platforms and consumers—for example, operational account system → ADF ingestion → ADLS Bronze → Databricks Silver → curated Gold table → semantic model → executive report.

Both are useful. Dataset-level lineage helps platform impact analysis; column-level lineage is essential when a particular sensitive field or business metric must be traced.

## DG33. How do you implement lineage in an Azure data platform?

Use a combination of platform-native lineage integrations, catalog APIs, orchestration metadata, and explicit documentation for unsupported paths. Capture ADF pipeline and activity dependencies, Databricks job/notebook inputs and outputs, storage objects, SQL transformations, curated tables, and BI model dependencies where supported.

For custom Python, dynamic SQL, external services, and file handoffs, emit lineage events or maintain mapping metadata as part of the pipeline deployment. Validate lineage with representative end-to-end traces; do not assume that enabling a catalog automatically captures every transformation.

## DG34. How do you handle lineage gaps?

Classify gaps as unsupported connectors, dynamic logic, manual movement, missing permissions, or unregistered assets. Prioritize gaps affecting regulated reports, sensitive data, and critical products. Add instrumentation or mapping declarations, assign an owner, document confidence and last verification, and monitor for regressions after deployments.

A partial lineage graph should show uncertainty rather than imply that every edge is known.

## DG35. What is impact analysis, and how do you perform it before a schema change?

Identify direct and transitive consumers, critical reports, downstream data products, contracts, quality rules, and sensitive-data controls. Classify the change as compatible or breaking, run contract and regression tests, notify consumers, provide a version or migration period when needed, deploy in stages, and verify downstream outcomes.

A column rename can break a pipeline, semantic measure, access policy, and audit report even if the underlying values do not change. Impact analysis must include technical and business dependencies.

## DG36. What is a certified or trusted data asset?

Certification is a controlled status indicating that an asset meets defined criteria, such as an accountable owner, documented definition, acceptable quality and freshness, lineage, approved access, support contact, and validated use. The certification label should identify the criteria, scope, reviewer, date, and review/expiry conditions.

Certification does not mean data can never be wrong. It communicates that specific checks were completed and that ongoing monitoring exists.

## DG37. How do you distinguish authoritative, golden, and derived data?

- **Authoritative source:** the designated system or dataset for a particular attribute or process.
- **Golden record:** a reconciled master representation created by matching and survivorship rules across sources.
- **Derived data:** data calculated or transformed for a downstream purpose.

A single system is not necessarily authoritative for every attribute. For example, a CRM may be authoritative for sales contact preferences while a billing platform is authoritative for invoicing status. Document authority at the attribute or domain level where needed.

## DG38. How do you govern schema evolution?

Use explicit contracts, schema versioning, compatibility rules, automated checks, impact analysis, and consumer communication. Additive nullable columns may be compatible, but not always if consumers use positional parsing, strict schemas, or `SELECT *`. Type narrowing, field deletion, semantic changes, and key changes are generally higher risk.

In Delta Lake or other schema-aware storage, schema evolution settings can help execute an approved change; they do not replace governance approval, validation, or downstream compatibility testing.

## DG39. What is metadata-driven governance?

Metadata-driven governance stores rules and asset properties in configuration so reusable frameworks can apply them consistently. A metadata record might specify asset owner, classification, primary key, required columns, quality thresholds, retention class, sensitivity tags, and approved consumer groups.

Pipelines can read configuration to run validation, produce quality reports, apply masking, route exceptions, and register lineage. Validate metadata itself: malformed or stale configuration can cause controls to be skipped or incorrectly applied.

## DG40. How do you keep catalog information current?

Automate technical metadata capture during deployments and pipeline runs; detect new assets and schema changes; require owners and classifications before production publication; run periodic completeness checks; alert on stale ownership or definitions; and review critical assets on a defined cadence. Track metadata freshness and coverage as metrics.

---

# 4. Data quality, observability, and issue management

## DG41. How do you establish data quality rules?

Start from business decisions and failure consequences. Identify critical data elements, define dimensions and measurable thresholds, assign a rule owner, choose the validation point, set severity and response time, and document permitted exceptions. Validate thresholds using historical distributions and business expectations.

Examples: unique transaction identifier, nonnegative transaction amount where business rules require it, valid currency code, account reference exists, event timestamp within an acceptable range, and expected delivery by a deadline.

## DG42. What is a critical data element (CDE)?

A CDE is a field or data element whose correctness materially affects a business process, regulatory obligation, customer outcome, or important metric. Examples may include account identifier, transaction amount, customer consent, risk classification, and reporting date.

CDE selection should be business-led and risk-based. For each CDE, define owner, meaning, source, permitted values, quality dimensions, thresholds, lineage, sensitivity, and remediation process.

## DG43. How do you implement data quality checks across Bronze, Silver, and Gold layers?

- **Bronze/raw:** preserve source evidence; check file readability, ingestion metadata, schema expectations, source counts, and corruption. Quarantine unreadable or structurally invalid records while preserving replay capability.
- **Silver/standardized:** validate types, required fields, keys, duplicates, referential integrity, domain rules, timestamp normalization, and standardization. Separate valid and invalid records with reason codes.
- **Gold/curated:** validate business rules, aggregate reconciliation, dimensional integrity, metric consistency, freshness, and consumer-facing SLAs.

Do not silently discard bad records. Define whether a failure blocks the batch, quarantines a subset, or permits a documented exception.

## DG44. What is the difference between validation, monitoring, and remediation?

- **Validation:** checks whether a record or dataset meets a rule at a point in the pipeline.
- **Monitoring:** tracks quality and operational signals over time and alerts on deviations.
- **Remediation:** investigates and corrects the underlying issue, repairs affected data, and prevents recurrence.

A dashboard showing a 4% null rate is monitoring; a pipeline assertion that rejects invalid records is validation; identifying an upstream API regression and replaying affected partitions is remediation.

## DG45. How do you manage duplicate records?

Define the business key and duplicate semantics first. Exact duplicates, repeated delivery, and multiple legitimate events for one entity are different cases. For event data, use a stable event identifier or idempotency key. For entity snapshots, define survivorship rules such as source priority, effective timestamp, and deterministic tie-breakers.

Use deterministic deduplication and preserve audit information about removed or superseded rows. Do not use `dropDuplicates()` blindly when the requirement is to keep the latest record; use a window ordered by update timestamp and a stable tie-breaker, then merge into the target idempotently.

## DG46. How do you handle late-arriving or out-of-order data?

Define event time versus processing time, allowed lateness, watermark behavior, replay window, and correction policy. For batch loads, use a source watermark plus overlap or change tracking where appropriate; for streams, use event-time windows and state retention based on the business lateness requirement.

Late records may require updating historical aggregates or dimensions. Track arrival delay, reprocessing counts, and reconciliation outcomes. A watermark must not be treated as proof that no later event can arrive.

## DG47. How do you investigate a source-to-target record-count mismatch?

1. Confirm the same business date, filter conditions, snapshot time, and load scope.
2. Compare source extract counts, ingested raw counts, valid/invalid counts, deduplication counts, and target insert/update/delete counts.
3. Check incremental watermarks, time zones, pagination, CDC lag, retries, and overlapping batches.
4. Compare distinct business keys and use anti-joins to identify missing or extra records.
5. Inspect transformation filters, join cardinality, null keys, and late data.
6. Reconcile monetary or other additive measures, not only row counts.
7. Correct the root cause, replay safely, and record evidence.

Counts may legitimately differ because of filtering or aggregation; every difference should be explainable by a documented transformation or rule.

## DG48. What is data observability?

Data observability is continuous visibility into data health through freshness, volume, schema, distribution, lineage, quality, and pipeline signals. It combines technical telemetry with business-impact context so teams can detect, localize, and resolve incidents quickly.

A useful alert states which asset is affected, which expectation failed, the observed value, expected range, downstream impact, owner, and recommended response. Alert fatigue can be reduced by severity thresholds, grouping related failures, and routing to accountable owners.

## DG49. How do you design a data quality score?

Define the score's purpose and dimensions first. One possible weighted score is:

`Score = Σ(weight_i × pass_rate_i) / Σ(weight_i)`

where weights reflect business risk. Publish both the aggregate score and the underlying dimension results. A high average must not hide a critical failed rule; hard-stop controls for critical data elements should remain independent of the score.

Document how missing measurements, low sample sizes, exceptions, and changing thresholds affect the score. Avoid comparing domains without normalizing for different risk and use cases.

## DG50. How do you prevent bad data from reaching reports?

Use layered controls: source validation, schema contracts, quality gates, quarantines, reconciliation, certified publishing, and consumer-facing quality status. Critical defects should block publication or mark the asset unavailable according to the agreed policy. Noncritical defects may be published with a visible warning if the owner has approved the behavior.

Maintain the ability to replay or repair affected partitions and notify downstream consumers when already-published results need correction.

## DG51. What is a data quality SLA or SLO?

A service-level objective (SLO) defines a measurable target such as 99.5% of expected records arriving by a deadline, or at least 99.9% of key values meeting a validity rule. An SLA may be a formal commitment with consequences or escalation requirements. Use precise measurement windows, scope, exclusions, and reporting rules.

Freshness, completeness, validity, availability, and issue-resolution time may have separate objectives. Avoid setting a target without a baseline and a response plan for misses.

## DG52. How do you prioritize data quality incidents?

Assess severity using business impact, criticality, sensitivity, downstream reach, regulatory deadlines, and whether incorrect data has already been consumed. A simple matrix can classify incidents as critical, high, medium, or low. Define response times and escalation for each tier.

For critical issues, contain the impact first—pause publication, restrict access, or mark outputs invalid—then investigate and repair. Communicate status and scope to affected consumers; do not wait for a perfect root-cause analysis before containment.

## DG53. How do you handle a recurring quality issue?

Perform root-cause analysis across source behavior, contract gaps, transformation logic, reference data, process ownership, and monitoring. Quantify recurrence and business impact. Add a preventive control at the earliest reliable point, a detection control downstream, a regression test, and an accountable owner. Verify that the issue does not reappear after remediation.

Repeated manual corrections without fixing the upstream cause are a governance and reliability smell.

## DG54. How should invalid data be quarantined?

Store rejected records or a secure reference to them with batch ID, source, ingestion timestamp, rule ID, reason code, processing version, and remediation status. Restrict access because rejected records may contain sensitive data. Set retention and deletion rules for the quarantine area.

Support controlled correction and replay, prevent duplicate reprocessing, and reconcile quarantined, corrected, discarded, and accepted counts. Avoid copying full sensitive payloads into logs or ticket descriptions.

---

# 5. Privacy, security, access, and responsible use

## DG55. What is the difference between data governance and data security?

Governance defines acceptable use, accountability, classification, and required controls. Security implements and operates mechanisms such as identity, authorization, encryption, network controls, monitoring, and incident response. They are interdependent: governance without technical enforcement is weak, while security controls without business context may be inconsistent or excessive.

## DG56. Explain least privilege and separation of duties.

**Least privilege** gives a user, service principal, or workload only the permissions needed for its role and task. **Separation of duties** prevents one identity from performing incompatible actions end to end, such as authoring and independently approving a high-risk access change.

Use role groups, workload identities, scoped permissions, time-bound elevation, approval workflows, access reviews, and audit logs. Avoid broad shared credentials and permanent administrator access for routine jobs.

## DG57. How do you implement role-based access control (RBAC)?

Map business roles to groups and permissions; scope permissions to the smallest appropriate resource; separate development, test, and production; use managed identities or service principals for workloads; and review membership periodically. Keep role definitions version-controlled and apply them through repeatable deployment processes.

RBAC is not always sufficient for row- or column-level restrictions. Combine it with table, view, row-level, column-level, masking, or attribute-based controls where supported and appropriate.

## DG58. What is attribute-based access control (ABAC)?

ABAC evaluates attributes of the subject, resource, action, and context—for example, department, data classification, purpose, region, or environment—to decide whether access is allowed. It can express more granular policies than static roles, but policy complexity and attribute quality become important risks.

Use explainable policy rules, default-deny behavior, test cases, policy versioning, and monitoring. Do not rely on unverified user-supplied attributes for sensitive decisions.

## DG59. How do you protect PII and sensitive data in a lakehouse?

Discover and classify sensitive fields; minimize collection; encrypt data in transit and at rest; control identities and network paths; restrict access by role and purpose; use masking or tokenization for appropriate consumer layers; prevent sensitive values from leaking into logs; govern extracts and nonproduction copies; monitor access; and apply retention/deletion requirements.

Protection must extend across Bronze, Silver, Gold, notebooks, caches, exports, backups, and downstream BI—not just the curated table. Tokenization and hashing are not automatically anonymization; re-identification risk depends on the method and context.

## DG60. What is the difference between masking, tokenization, pseudonymization, and anonymization?

- **Masking:** obscures selected values for a user or use case; it may be static or dynamic.
- **Tokenization:** replaces a sensitive value with a token, usually with a controlled mapping or service.
- **Pseudonymization:** replaces direct identifiers but may permit re-linking using additional information kept separately.
- **Anonymization:** aims to make identification infeasible under a defined risk model; it is difficult to guarantee, especially when datasets can be joined.

Choose based on the use case, threat model, and applicable legal definitions. Do not claim a dataset is anonymous simply because names were removed.

## DG61. What is data minimization?

Collect and retain only the data reasonably needed for a defined, legitimate purpose. It applies to fields, records, precision, collection frequency, access, copies, and retention duration. For example, analytics may need an age band rather than exact date of birth, or a tokenized customer identifier rather than direct identity details.

Record purpose and necessity, review whether each field is used, and remove or transform unnecessary data. Minimization reduces privacy risk, storage cost, and the impact of a breach.

## DG62. How do you govern consent and purpose limitation?

Capture the applicable permission or lawful basis and its scope, purpose, source, timestamp, and status where relevant. Make downstream systems respect that status for collection, use, sharing, and activation. Consent is not the only possible legal basis in every jurisdiction, so involve privacy/legal specialists to determine the applicable requirement.

A change in permission may require suppressing future use, updating derived audiences, stopping exports, or processing deletion requests. Keep evidence of enforcement and define how quickly changes propagate.

## DG63. How do you manage data subject or customer deletion requests?

Verify the request through the approved process; identify relevant records and systems using identity resolution and lineage; assess legal holds and retention obligations; execute deletion or approved restriction across sources, lakehouse layers, derived tables, indexes, extracts, and downstream processors; handle backups according to the approved policy; and retain minimal evidence of completion.

Some obligations require exceptions or different handling, so do not promise immediate deletion from every immutable backup without checking policy. Design systems for traceable requests and repeatable propagation rather than relying on ad hoc searches.

## DG64. How do you implement row-level and column-level security?

Define policies from approved business attributes and classification. Row-level security restricts which records a user can see; column-level security restricts which fields are visible. Apply controls in the serving layer and test them through the actual consumer identities, including BI tools, notebooks, APIs, exports, and service accounts.

Verify that alternate paths cannot bypass the policy. Dynamic filters in a report alone are not a security boundary if users can query the underlying table directly.

## DG65. What is data residency, and how do you govern cross-region transfers?

Data residency concerns where data is stored or processed; cross-border transfer rules may also regulate who can access it and under what safeguards. Inventory the data categories, regions, processors, backup locations, support access, and downstream copies. Validate the applicable legal, contractual, and internal requirements before designing replication or sharing.

Architecture options may include regional storage, regional processing, approved transfer mechanisms, minimization, and access restrictions. The correct answer depends on the data and jurisdictions involved.

## DG66. How do you govern nonproduction data?

Development and test environments often have weaker access and more copies, so they need explicit controls. Prefer synthetic or properly de-identified data; if production-derived data is necessary, minimize it, restrict access, mask sensitive fields, log use, define retention, and prevent uncontrolled downloads.

A production database backup should not automatically become a developer's test dataset. Verify that masking preserves useful statistical properties without exposing identities.

## DG67. How do you prevent sensitive data leakage into logs and observability tools?

Use structured logging with allowlisted fields; redact or tokenize sensitive values; avoid dumping full payloads and exception objects; scan logs for sensitive patterns; restrict log access; define log retention; and test failure paths where raw records are often accidentally emitted. Include notebooks, Spark driver logs, pipeline error messages, dead-letter queues, and support tickets in the review.

Log identifiers needed for tracing rather than full customer details. If leakage occurs, treat it as a security/privacy incident according to the approved process.

## DG68. How do you govern AI and machine-learning data?

Track training and evaluation datasets, provenance, collection purpose, consent or other applicable basis, sensitive attributes, label quality, representativeness, known limitations, transformations, feature definitions, and access. Version datasets and feature logic so model results can be reproduced. Establish controls for data leakage, bias assessment, retention, and downstream use.

For generated or inferred attributes, document that they are derived, their intended use, confidence or limitations, and restrictions. Governance requirements depend on model impact and applicable policies.

## DG69. What is a data breach or security incident from a governance perspective?

It is an event involving unauthorized access, disclosure, alteration, loss, or unavailability of data. Governance contributes asset ownership, classification, lineage, data-subject impact analysis, retention evidence, and decision accountability. Security/incident-response teams lead technical containment and investigation; privacy/legal teams determine notification and regulatory obligations.

Preserve evidence, contain access, determine affected data and consumers, document decisions, remediate, and validate corrective actions. Avoid making notification commitments without the authorized incident process.

## DG70. How do you audit data access?

Log the identity, resource, action, timestamp, result, and relevant context, while minimizing sensitive content in logs. Centralize and protect audit logs, synchronize timestamps, define retention and access restrictions, and alert on high-risk patterns such as unusual exports, privilege escalation, or access to restricted data outside normal behavior.

Periodic access reviews should verify business need, group membership, service identities, dormant accounts, and exceptions. Logs need integrity protection and enough context to support investigation.

---

# 6. Lifecycle, retention, archival, and deletion

## DG71. Explain the data lifecycle from creation to deletion.

A typical lifecycle includes collection, ingestion, validation, storage, transformation, publication, consumption, archival, legal hold where applicable, and deletion. Governance defines permitted purpose, ownership, quality expectations, protection, retention, and exit criteria at each stage.

Map all copies and derivatives. A source record may create copies in raw files, Delta tables, caches, extracts, BI models, feature stores, backups, and vendor systems. Lifecycle governance must account for those paths.

## DG72. How do you define a retention policy?

Determine the purpose, applicable laws and contracts, operational need, business owner, data class, start event, retention duration, archive conditions, legal-hold behavior, and deletion method. Use an approved retention schedule with clear scope and review dates.

Do not keep data indefinitely “just in case.” Conversely, do not delete data subject to a valid hold or mandatory retention requirement. When obligations conflict, escalate to the authorized legal/compliance decision-maker.

## DG73. How do you implement retention in a data lake?

Tag datasets and files with retention class and relevant dates; apply lifecycle policies for eligible storage objects; use table-aware deletion and vacuum/compaction behavior appropriate to the format; manage snapshots, versions, backups, replicas, and extracts; and generate evidence of execution. Test policies on nonproduction data before rollout.

Object lifecycle deletion alone may not satisfy requirements for Delta tables, replicas, external extracts, or backups. Align storage policies with table semantics and recovery obligations.

## DG74. What is the difference between archival and deletion?

Archival moves data to a lower-cost or restricted-access state while retaining it for a defined purpose and period. Deletion removes data from active use and, according to policy, from applicable replicas and storage after required controls and holds are considered.

Archived data still needs access control, encryption, retention limits, discoverability for approved needs, and eventual disposition. “Cold storage” is not a substitute for a retention decision.

## DG75. How do legal holds affect data deletion?

A legal hold suspends normal deletion for scoped data when preservation is required. The process should record scope, custodian, start date, authorizing role, affected systems, periodic review, and release conditions. Automated deletion should check hold status before execution.

When a hold is released, the normal retention process should resume in a controlled, auditable way. Legal teams determine the scope and duration; engineering ensures the control is enforced.

## DG76. How do you govern backup retention and recovery copies?

Define backup schedules, retention windows, encryption, access, geographic placement, restore testing, and deletion behavior. Document whether a deletion request is handled by immediate logical suppression, expiration through backup cycles, or a controlled restore-and-redelete procedure, consistent with policy and legal requirements.

A restore process must not silently reintroduce deleted or restricted data. Maintain reconciliation or tombstone mechanisms where needed.

## DG77. How do you handle data lineage during deletion?

Use lineage and identity mapping to locate source records, derived records, materialized views, aggregates, caches, extracts, and downstream recipients. Determine whether deletion, correction, suppression, or aggregation is appropriate for each derivative. Record completion and exceptions without retaining unnecessary sensitive values in the evidence.

Lineage may be incomplete, so include system inventories, owner attestations, and reconciliation checks. Report uncertainty rather than claiming global deletion without evidence.

## DG78. What is data archival in a medallion architecture?

Bronze may retain source evidence for replay or audit under an approved retention policy; Silver stores standardized and validated records; Gold stores curated business outputs. Each layer can have different access, quality, retention, and deletion requirements. Layer names alone do not define the retention period.

A practical policy can shorten retention for unnecessary raw copies while preserving legally required records in an approved repository. Consider Delta history, time travel, snapshots, and downstream extracts when defining effective deletion.

## DG79. How do you manage temporary data and intermediate files?

Classify temporary outputs, define TTLs, clean up staging directories and failed-run artifacts, restrict access, and prevent sensitive payloads from persisting in debug files or notebook checkpoints. Make cleanup idempotent and ensure it does not remove data still required for replay or incident investigation.

Track temporary asset creation and expiry through job metadata or storage lifecycle controls. Exceptions should be explicit and time-bound.

## DG80. How do you prove that a retention or deletion control worked?

Maintain the approved rule and version, execution logs, asset scope, record or object counts where appropriate, exception and legal-hold results, verification queries, downstream confirmations, and reviewer sign-off for high-risk actions. Protect evidence and avoid embedding sensitive record values unnecessarily.

A successful job status alone is insufficient; validate the expected scope and check that the data is not still accessible through another path.

---

# 7. Cloud and platform implementation

## DG81. How would you implement governance in Azure Data Factory, ADLS Gen2, Databricks, and Synapse?

A workable design combines identity, storage, processing, catalog, quality, and operational controls:

- **Azure Data Factory:** use managed identity, parameterized pipelines, controlled linked services, pipeline/run metadata, schema and count checks, and secure parameter handling.
- **ADLS Gen2:** use hierarchical namespaces where appropriate, scoped RBAC/ACLs, encryption, private networking, lifecycle policies, and separate zones or accounts based on risk and operational needs.
- **Azure Databricks / Delta Lake:** use governed catalogs and permissions, service identities, table/column controls where supported, schema contracts, quality checks, lineage capture, and controlled publishing.
- **Azure Synapse / serving:** expose approved curated views or models, enforce least privilege, audit access, and avoid bypass paths around governed tables.
- **Catalog and monitoring:** connect business glossary, owners, classifications, lineage, quality metrics, and operational alerts.

Exact features depend on the organization's Azure configuration, licensing, region, and service versions. Validate implementation details against current platform documentation.

## DG82. How do you govern ADLS Gen2 zones?

Define the purpose and contract of each zone—for example, landing/raw, standardized, curated, and consumer outputs—plus ownership, classification, access groups, naming, encryption, retention, and allowed transformations. Separate high-risk data where policy requires it and avoid broad access to the entire storage account merely for convenience.

Folder boundaries are not automatically security boundaries. Validate effective permissions, inherited ACLs, service identities, shortcuts, and alternate access routes.

## DG83. How do you govern Delta Lake tables?

Register tables in the governed catalog; assign owners; document schema and business definitions; set access policies; enforce data quality and contract checks; capture lineage; control schema evolution; monitor table health and freshness; and manage history, retention, and deletion. Use transactional merge patterns carefully for idempotent updates and CDC processing.

Delta transaction logs and time travel provide operational capabilities, not a complete governance program. Access, retention, lineage, and sensitive-data controls still need explicit design.

## DG84. How do you govern Databricks Unity Catalog or another centralized catalog?

Use governed objects and ownership, least-privilege grants, group-based access, external locations and storage credentials controlled by approved identities, lineage where supported, and environment separation. Define standards for catalogs, schemas, tables, functions, volumes, and external data access.

Automate grants and object creation through reviewed code; monitor privilege changes and unused permissions; and test access from actual identities. Avoid distributing long-lived storage keys or granting unrestricted workspace access to solve a single table-access issue.

## DG85. What is the role of Microsoft Purview in data governance?

Microsoft Purview capabilities can support cataloging, discovery, classification, lineage, glossary/metadata management, and governance workflows across supported sources. A typical implementation scans prioritized assets, assigns owners, connects business terms, classifies sensitive data, and uses lineage and search to improve discovery and impact analysis.

Do not assume every source, transformation, or column-level edge is automatically captured. Connector coverage, permissions, platform versions, and configuration matter; document unsupported paths and validate coverage against real workflows.

## DG86. How do you implement governance in CI/CD?

Store schemas, contracts, quality rules, policy definitions, and deployment configuration in version control. In pull requests, validate required metadata, naming conventions, schema compatibility, secrets handling, tests, and access changes. Deploy through separate environments with approvals for high-risk changes, then verify catalog and lineage updates.

Use automated gates proportionate to risk. A policy failure should explain what failed and how to fix it. Emergency changes need an auditable exception and post-implementation review.

## DG87. How do you govern secrets and connection credentials in pipelines?

Prefer managed identities or workload identity federation where supported. Store unavoidable secrets in a managed secret store, scope them narrowly, rotate them, prevent exposure in parameters and logs, and restrict who can read or change them. Separate credentials by environment and workload.

Never embed secrets in notebooks, source control, pipeline JSON, plain-text config, or error messages. Review secret access and rotation failures as part of operational monitoring.

## DG88. How do you separate development, test, and production data access?

Use distinct workspaces, identities, storage paths/accounts, access groups, and deployment approvals. Production writes should be restricted to approved workloads. Development should use synthetic or masked datasets where feasible, and promotion should move code and configuration—not uncontrolled copies of production data.

Test environment-specific settings, identity permissions, schema contracts, and data classification at each stage. A separate workspace alone is insufficient if identities can still read all production storage.

## DG89. How do you govern streaming data and event hubs?

Define event schemas, producer identity, topic/stream ownership, retention, allowed lateness, ordering assumptions, duplicate semantics, sensitive fields, consumer groups, replay rights, and schema compatibility. Monitor lag, volume, schema failures, dead-letter rates, and consumer health.

Version event contracts and test producer/consumer compatibility. For replay, ensure downstream processing is idempotent and does not double-count events.

## DG90. How do you govern APIs and semi-structured data?

Document API owner, purpose, schema, versioning, authentication, rate limits, pagination, retries, error handling, data classification, and retention. Validate JSON/XML against approved schemas where appropriate, preserve source metadata, and quarantine invalid payloads securely.

Schema validation alone does not guarantee semantic validity. Also check business rules, identity keys, timestamps, and expected completeness; monitor API changes and notify consumers.

## DG91. How do you govern CDC and incremental pipelines?

Document the source change mechanism, key, operation codes, sequence or commit ordering, delete semantics, watermark, overlap strategy, replay behavior, and reconciliation. Persist batch IDs and checkpoints, make processing idempotent, and monitor lag and missed-change risk.

A timestamp watermark can miss updates if timestamps are unreliable or multiple changes share a timestamp. Use source-supported CDC/sequence mechanisms where available and validate against source counts or periodic reconciliation.

## DG92. How do you govern master and reference data?

Define authoritative sources, allowed values, ownership, change approvals, effective dates, distribution channels, and conflict resolution. Master data management may match records and create golden records; reference data management controls enumerations such as country, currency, product status, and risk codes.

Downstream systems should consume governed versions rather than maintain divergent local mappings without oversight. Record effective dates and history where changes affect reporting or decisions.

## DG93. How do you govern a data warehouse or semantic model?

Assign owners to business measures, dimensions, source tables, and reports. Document metric formulas, grain, filters, relationships, refresh cadence, certified status, and access rules. Test aggregation behavior, duplicate counting, slowly changing dimensions, row-level security, and reconciliation to approved sources.

A report may be technically correct yet misleading if metric definitions differ. Centralize shared measures where practical and communicate intentional differences explicitly.

## DG94. How do you handle governance in a multi-cloud or hybrid environment?

Establish common policies and metadata vocabulary while respecting platform-specific enforcement. Maintain a central inventory of critical assets and owners; integrate catalogs and lineage where supported; standardize identity, classification, retention, logging, and incident processes; and document gaps between cloud controls.

Avoid assuming the same control behaves identically across platforms. Test effective permissions and evidence in each environment, including on-premises systems and third-party SaaS.

## DG95. How do you govern cost and resource usage without compromising data controls?

Tag assets and workloads with domain, owner, environment, criticality, and cost center; monitor storage, compute, scans, and retention; remove orphaned assets after owner review; right-size workloads; and tier storage according to access and recovery needs. Cost optimization must respect retention, legal holds, audit evidence, and service-level commitments.

Do not delete apparently unused data until ownership, dependencies, legal obligations, and downstream access have been checked.

---

# 8. Governance metrics, controls, and audits

## DG96. What KPIs would you use for a data governance program?

Choose metrics tied to risk and outcomes, such as:
- Percentage of critical assets with an owner and steward.
- Percentage of critical data elements with definitions and quality rules.
- Classification coverage for high-risk assets.
- Lineage coverage for critical reports and sensitive fields.
- Percentage of production assets with documented contracts.
- Quality SLO attainment and recurring defect rate.
- Median time to acknowledge and resolve critical issues.
- Access-review completion and overdue exceptions.
- Retention/deletion control completion and exceptions.
- Catalog search success, reuse, and certified-asset adoption.

Define numerator, denominator, scope, measurement window, data source, owner, and target. A metric without a clear definition can create false confidence.

## DG97. What evidence would an auditor expect for a data control?

Depending on the control: approved policy, named owner, configuration, change history, execution logs, test results, access review, exception approvals, incident records, and evidence that exceptions were remediated. Evidence should be complete, time-stamped, attributable, protected from unauthorized modification, and retained according to policy.

Distinguish **control design** (the control could work) from **operating effectiveness** (it actually worked over the period tested).

## DG98. How do you conduct a governance control assessment?

Define scope and risks; map requirements to controls; identify control owners; inspect design and implementation; sample evidence over a relevant period; test exceptions and failure paths; document findings and severity; assign remediation owners and due dates; and retest closure. Include automated controls and manual processes.

Do not treat a screenshot of a configuration as proof of sustained operation if the control requires ongoing review or monitoring.

## DG99. What is the difference between preventive, detective, and corrective controls?

- **Preventive:** blocks a violation before it occurs, such as rejecting an incompatible schema or denying unauthorized access.
- **Detective:** identifies a violation, such as an alert for unusual exports or a quality threshold breach.
- **Corrective:** restores compliance or data correctness, such as revoking access, repairing a partition, or replaying a batch.

Critical risks often need more than one control type because no single control is perfect.

## DG100. How do you govern policy exceptions and control failures in reporting?

Track the requirement, control, asset, owner, severity, discovery time, impact, compensating controls, approver, target date, and closure evidence. Distinguish accepted risk from unresolved noncompliance. Report overdue high-risk findings to the authorized governance body.

Aggregate metrics should not hide severe individual failures. A single unapproved exposure of restricted data may matter more than a 99% pass rate across low-risk checks.

## DG101. How do you demonstrate compliance without collecting excessive evidence manually?

Automate evidence capture from identity systems, catalogs, CI/CD, quality jobs, storage policies, access reviews, and incident workflows. Normalize evidence into a controlled repository with timestamps, asset identifiers, control IDs, and version information. Preserve reviewer approval for decisions that cannot be automated.

Automation reduces effort but needs validation, access restrictions, retention, and monitoring. A broken evidence collector can create a false appearance of compliance.

## DG102. How do you prioritize governance investments?

Use a risk-and-value matrix considering regulatory exposure, customer impact, financial impact, criticality, number of downstream consumers, current control gaps, remediation cost, and feasibility. Prioritize foundational controls that protect many assets, but deliver in domain-based increments so benefits are visible early.

Examples might include cataloging and classifying a high-risk customer domain, improving lineage for regulatory reporting, or automating access reviews for restricted datasets.

## DG103. How do you calculate the business value of governance?

Estimate measurable changes such as reduced analyst search time, fewer reconciliation incidents, shorter issue-resolution cycles, reduced duplicate pipeline development, lower audit preparation effort, fewer unauthorized access paths, and improved report reliability. Establish a baseline and use conservative attribution.

Do not claim every improvement was caused by governance alone. Track both direct cost savings and risk reduction, and explain assumptions to leadership.

## DG104. What is a governance maturity roadmap?

A roadmap translates the current state into prioritized capability increments. Example phases:
1. **Discover:** inventory critical assets, risks, owners, and key consumers.
2. **Define:** establish policies, classification, roles, glossary, and control standards.
3. **Operationalize:** implement catalog, lineage, quality workflows, access review, and retention controls.
4. **Automate:** embed checks in CI/CD and pipeline execution; automate evidence and alerts.
5. **Optimize:** measure outcomes, improve coverage, reduce friction, and review exceptions.

Set outcomes and owners for each phase. Do not define maturity solely by the number of tools purchased.

## DG105. How do you report governance status to executives?

Lead with risk, business impact, trend, decisions required, and remediation—not technical activity counts. Show critical data assets without owners, material quality breaches, high-risk access exceptions, lineage gaps affecting important reports, overdue retention actions, and progress against agreed targets.

Separate enterprise-level risk from domain-level detail, and identify the accountable executive or decision-maker for unresolved issues.

---

# 9. Advanced architecture and cross-domain scenarios

## DG106. How would you govern an enterprise medallion architecture?

Define the purpose, ownership, quality expectations, classification, access, and retention of each layer. Bronze preserves source evidence within policy; Silver standardizes, validates, and resolves identities; Gold publishes governed business entities and metrics. Apply lineage and batch metadata across all transitions.

Governance decisions should be expressed as reusable rules and enforced in pipeline templates. Avoid assuming that all Bronze data must be unrestricted or that Gold is automatically safe for broad consumption.

## DG107. How do you govern a cross-domain customer 360 platform?

Agree on customer identity, source authority by attribute, matching rules, survivorship, consent and purpose, data quality, conflict resolution, access, and lineage. Keep direct identifiers separate or restricted where appropriate, and document uncertainty in identity matches.

Measure match precision/recall where possible, duplicate profiles, unresolved conflicts, update latency, and downstream impact. A unified view should not erase legitimate differences among sales, billing, support, and legal definitions.

## DG108. How do you govern financial or banking transactions?

Define transaction identity, event ordering, reversal and correction semantics, currency, amount precision, posting date versus event date, status lifecycle, source authority, reconciliation, retention, access, and auditability. Distinguish transaction events from account balance snapshots and derived balance calculations.

Controls can include uniqueness, valid account references, balanced ledger rules where applicable, source-to-target reconciliation, immutable audit history, and controlled restatement. Business and regulatory rules vary by product and jurisdiction and should be approved by domain experts.

## DG109. How do you govern healthcare data?

Define patient identity, source authority, clinical semantics, consent/permission requirements, minimum necessary access, sensitive-data classification, interoperability standards, lineage, correction history, retention, and audit logging. Validate coding systems and formats where applicable, but do not assume syntactic HL7/FHIR conformance guarantees clinical correctness.

Use qualified clinical, privacy, security, and compliance stakeholders to approve rules. Monitor patient matching errors, missing critical fields, invalid codes, and unauthorized access.

## DG110. How do you govern data used for regulatory reporting?

Establish a controlled definition of each reported measure, authoritative source, transformation logic, lineage, reconciliation, data-quality rules, approval workflow, versioning, and evidence retention. Changes require impact analysis and approval proportionate to risk. Reproducibility is important: retain enough version and run metadata to explain how a report was produced.

A final report total should reconcile to approved source totals or documented adjustments. Maintain a clear record of restatements and corrections.

## DG111. How do you govern real-time operational analytics?

Define latency objectives, event schemas, duplicate and ordering semantics, allowed lateness, quality gates, failure handling, retention, and replay. Decide which defects can be tolerated temporarily and which must block or suppress outputs. Make data freshness and quality status visible to consumers.

Governance cannot require every event to wait for a human approval. It should define automated controls and escalation for unusual or high-risk events.

## DG112. How do you govern unstructured documents and files?

Track source, owner, purpose, sensitivity, file type, access, retention, and processing history. Apply content scanning and classification where appropriate, validate file integrity, quarantine suspicious or malformed files, and control extracted text or embeddings derived from the files.

OCR output, document summaries, vector indexes, and extracted entities can preserve sensitive information even after the original file is deleted. Include derivatives in lineage, access, and lifecycle planning.

## DG113. How do you govern data shared with external analytics or AI services?

Review the data categories, purpose, contractual use, retention, training use, subprocessors, location, access, deletion, and incident obligations before sharing. Minimize and transform data where possible, use approved identities and transfer channels, and log what was shared and when.

Do not assume that a vendor's security certification or a successful API call alone proves the sharing is permitted. Align technical configuration with contractual and privacy review.

## DG114. How do you govern data products in a data mesh?

Use domain ownership for local meaning and delivery, combined with federated enterprise standards for interoperability, identity, classification, quality, discoverability, and policy enforcement. Data products should publish contracts and SLOs, register lineage, expose access workflows, and provide support contacts.

The platform should make compliance the easiest path through templates, automated tests, and shared infrastructure. Federated governance fails if domains can publish untracked assets without minimum metadata or control requirements.

## DG115. How do you handle conflicting source values?

First determine whether the fields have the same meaning and effective time. Then apply approved source-authority and survivorship rules, preserve provenance, and expose unresolved conflicts when a safe decision cannot be made automatically. Do not simply choose the most recent timestamp if clocks, source reliability, or business priority differ.

For master data, maintain match confidence, selected source, rule version, and history so the golden record can be explained and corrected.

## DG116. How do you govern metrics that disagree across dashboards?

Compare grain, filters, time zone, currency conversion, status definitions, deduplication, late-arriving data, refresh timing, and inclusion/exclusion rules. Identify the accountable metric owner and decide whether the definitions should converge or remain distinct with explicit names.

Publish a canonical measure and shared calculation when appropriate, version the definition, reconcile historical outputs, and notify consumers. A single label should not hide materially different business calculations.

## DG117. How do you govern data used by generative AI and retrieval-augmented generation (RAG)?

Inventory source documents and permissions; classify content; preserve source attribution and version; enforce access filters at retrieval time; govern chunking and embedding derivatives; monitor stale or deleted content; and test for sensitive-data leakage and unauthorized retrieval. The model must not gain broader access than the user has to source material.

Define how source updates propagate to indexes, how deletion requests affect embeddings and caches, and how citations or evidence are returned. Evaluate retrieval quality and access-policy enforcement separately.

## DG118. How do you prevent governance controls from being bypassed?

Map all access paths: cataloged tables, direct storage access, notebook exports, BI extracts, APIs, service identities, replicas, and third-party copies. Use least privilege, centralized policy enforcement where feasible, network restrictions, logging, automated drift detection, and periodic adversarial testing.

A governed catalog is not an enforcement boundary by itself. Test the effective permissions of real users and workloads, including alternate paths and inherited grants.

## DG119. How do you manage a major governance policy change?

Assess affected domains, assets, consumers, contracts, and technical controls. Define the target state, transition period, migration tasks, exception path, and communication plan. Version policy and configuration, deploy in stages, test control behavior, and monitor adoption. Retire old rules only after the transition criteria are met.

Track policy acknowledgements where required, but focus on whether controls and workflows actually changed.

## DG120. What would you do if governance metadata conflicts with the data itself?

Treat metadata as a claim to validate, not unquestionable truth. For example, if a catalog marks a field non-sensitive but profiling reveals likely personal information, restrict exposure if appropriate, raise a classification review, identify affected copies, and correct the metadata and controls. Preserve evidence and assess whether the mismatch created an incident.

Use automated drift checks between declared schema, observed schema, classification, lineage, and actual access. Resolve conflicts through accountable owners rather than silently overwriting metadata.

---

# 10. Production troubleshooting scenarios

## S01. A critical Gold table contains duplicate business keys. What do you do?

**Immediate response:** assess whether reports or downstream decisions are affected; pause publication or mark the table as degraded if necessary; identify affected partitions and consumers.

**Investigation:** inspect source duplication, replay behavior, CDC sequence handling, merge keys, join cardinality, late updates, and concurrent writes. Compare raw, standardized, and curated counts. Verify whether the requirement is one row per entity or multiple legitimate events.

**Remediation:** define deterministic survivorship, repair the affected scope, make ingestion and merges idempotent, add a uniqueness rule and regression test, reconcile results, and notify consumers. Record root cause and evidence.

## S02. The source has 10 million rows, but the target has 9.6 million. How do you investigate?

Reconcile equivalent scopes and snapshots first. Build a stage-by-stage count ledger: extracted, ingested, accepted, quarantined, deduplicated, inserted, updated, deleted, and published. Compare distinct keys and anti-join missing records. Inspect filters, pagination, watermark boundaries, timezone conversions, join behavior, rejected records, and retries.

If the difference is expected, document the rule and quantify it. If not, replay missing data safely and add an automated reconciliation control that alerts before publication.

## S03. A new source column appears unexpectedly in production. What governance controls should apply?

Determine whether the change is compatible, whether the column contains sensitive data, and whether downstream consumers are affected. Do not automatically expose the column to broad users. Compare the observed schema with the contract, notify the owner, scan/classify the field, assess lineage and access implications, and approve or reject the change through the change process.

After approval, version the contract, update schemas and catalog metadata, test downstream consumers, and deploy through controlled environments.

## S04. A report uses an incorrect definition of “active customer.”

Identify the report, metric owner, source logic, affected periods, and consumers. Compare the implemented definition with the approved glossary and any contractual or regulatory definition. Contain the impact by marking or withholding the metric if necessary, then correct the logic and reconcile historical outputs.

Version the definition, update lineage and semantic metadata, add a regression test, communicate the correction, and assess whether previous decisions or submissions require formal correction.

## S05. Sensitive data is found in a general-purpose storage folder.

Restrict exposure promptly in line with incident procedures; preserve necessary evidence; determine what data, identities, and copies are involved; inspect access logs and downstream lineage; and engage security/privacy teams. Reclassify the asset, remove unauthorized access, move or protect the data, and assess whether notification or other obligations apply.

Prevent recurrence with automated classification scans, secure landing templates, access-boundary tests, and controls that block publication when classification is missing or incompatible with the destination.

## S06. A deletion request is completed in the source but data remains in Gold and BI extracts.

Trace the identifier through lineage, keys, transformations, aggregates, extracts, caches, and external recipients. Determine the correct action for each derivative—delete, suppress, recompute, or retain under an approved exception. Check whether identity resolution is incomplete or keys changed.

Build a tracked deletion workflow with per-system status and verification. Add downstream propagation tests and ensure restored backups do not silently reintroduce the record.

## S07. Data quality alerts are firing too often and teams ignore them.

Analyze alert volume, false positives, duplicates, severity, and ownership. Separate hard business invariants from statistical anomaly warnings. Tune thresholds using baselines, group related failures, route by asset owner, and provide actionable context. Suppress noise only when a documented rule and expiry exist.

Measure alert acknowledgement, true-positive rate, recurring defects, and time to resolution. Never lower a critical threshold simply to improve dashboard appearance.

## S08. Lineage is missing between a Databricks notebook and a report.

Identify the missing edges: notebook inputs/outputs, table writes, SQL transformations, semantic model, and report dependency. Check connector support, permissions, dynamic paths, temporary views, external files, and whether the objects are registered. Add supported integration or explicit lineage metadata for unsupported transformations.

Validate with a known column-level trace and test again after deployment. Mark unresolved gaps and prioritize them based on report criticality and sensitive data.

## S09. A domain team refuses to assign an owner to a critical dataset.

Explain the business and operational consequences, identify the executive sponsor and consumers, and clarify the decision rights required. Escalate through the agreed governance council if no accountable owner is nominated. Until ownership is resolved, apply risk-based safeguards such as limiting certification or restricting high-risk publication.

Do not fabricate ownership or assign accountability to an unrelated platform engineer simply because they operate the storage.

## S10. A pipeline team wants to bypass quality checks to meet a deadline.

Assess the failed rule, severity, affected records, downstream consumers, and available containment. If policy permits, use a documented, time-bound exception with an authorized approver and compensating controls; otherwise do not publish the unsafe output. Record the decision, expiry, and repair plan.

After the deadline, fix the root cause and add tests. A deadline alone is not evidence that a control is safe to bypass.

## S11. A user can read restricted data through direct storage access even though the catalog says access is denied.

Treat this as a policy-enforcement gap. Reproduce using the affected identity, inspect effective RBAC/ACLs, inherited grants, service credentials, workspace permissions, external locations, and alternate paths. Contain access, review audit logs, and assess incident impact.

Align catalog metadata with actual enforcement, remove excessive grants, test least privilege, and automate drift detection. The catalog label alone does not deny access.

## S12. A vendor cannot confirm deletion of data received months ago.

Review the contract, transfer inventory, data-sharing logs, retention terms, and approved escalation path. Request evidence from the vendor and identify all copies or subprocessors covered by the agreement. Involve vendor risk, privacy, legal, and security owners to determine obligations and next actions.

Improve future sharing with documented deletion clauses, recipient attestations, periodic reviews, minimized transfers, and traceable data-sharing records.

## S13. A schema change passes CI but breaks a downstream dashboard.

Check whether CI tested only schema compatibility or also semantics, relationships, measures, and report behavior. Identify the breaking change—type, nullability, key, meaning, column order, or relationship cardinality—and restore service using rollback or a compatible version if safe.

Add consumer contract tests, semantic-model tests, and representative dashboard validation. Update impact analysis to include indirect BI consumers, not just immediate tables.

## S14. Data quality is good on average, but one critical field fails badly.

Do not rely on the aggregate score. Apply the critical-field hard-stop or escalation rule, assess affected decisions, and contain the output. Report the failed dimension and scope separately, repair data, and review why the weighting or dashboard did not make the risk visible.

Governance metrics should surface critical exceptions regardless of the overall average.

## S15. An audit finds access reviews were not completed on time.

Establish which systems, roles, and data assets were in scope; determine whether access remained appropriate; review logs for potentially risky use; and complete the overdue review with authorized owners. Remove access that is no longer needed and document the delay and remediation.

Identify the process failure—missing owner, poor notifications, unclear cadence, or manual tracking—and automate reminders, escalation, completion evidence, and overdue reporting.

---

# 11. Practical exercises and sample answers

## P01. Design a governance metadata record for a critical table

**Scenario:** A curated `gold.customer_account` table feeds financial reporting and customer analytics.

A useful minimum record could include:

```yaml
asset_name: gold.customer_account
asset_type: curated_table
domain: Customer_Account
business_owner: Customer_Account_Data_Owner
technical_custodian: Data_Platform_Team
steward: Customer_Account_Data_Steward
business_purpose: Approved customer-account analytics and reporting
criticality: Tier_1
classification: Confidential  # Example only; confirm the actual policy
primary_key: account_id
quality_rules:
  - account_id is not null
  - account_id is unique at the declared table grain
  - account_status belongs to the approved domain
freshness_slo: "Published by the agreed business deadline"
lineage_required: true
access_policy: Approved groups and purpose-specific controls
retention_class: Approved enterprise retention schedule
contract_version: "1.0"
last_reviewed_at: "YYYY-MM-DD"
exception_status: none
```

This is illustrative configuration, not a universal schema. Real systems should use approved taxonomies, stable identifiers, secret-free metadata, and controlled updates.

## P02. Write a quality rule and response policy

**Rule:** Every published payment record must have a transaction identifier, valid account reference, supported currency, and a parseable event timestamp.

**Design details:**
- Define uniqueness scope and whether reversals are separate events.
- Validate schema and required fields at ingestion.
- Validate reference integrity and business status in Silver.
- Reconcile totals and publish only after the critical gate passes.
- Quarantine invalid records with batch ID and reason code.
- Alert the payment data owner and pipeline support team based on severity.
- Define repair/replay, deadline, and escalation behavior.
- Measure invalid rate, late arrival, unresolved quarantine age, and recurrence.

## P03. Create a lineage and change-impact checklist

Before changing a source field, check:
1. Source owner and contract.
2. Ingestion pipeline and raw storage.
3. Standardized transformations and quality rules.
4. Curated tables, facts/dimensions, and semantic models.
5. Reports, exports, APIs, ML features, and external consumers.
6. Classification, masking, access rules, and retention.
7. Tests, deployment order, rollback, and communication.
8. Certification status and audit/reporting dependencies.

## P04. Design a simple governance scorecard

Pick 6–10 metrics that support action. Example categories:
- Ownership: percentage of Tier 1 assets with an owner.
- Metadata: percentage with complete definitions and classifications.
- Quality: critical-rule pass rate and unresolved high-severity defects.
- Lineage: percentage of critical reports with verified lineage.
- Access: overdue access reviews and unapproved privilege exceptions.
- Lifecycle: overdue retention actions and deletion propagation failures.
- Operations: mean time to acknowledge and resolve critical incidents.

For each metric, define the calculation, owner, target, source, reporting frequency, and escalation threshold. Keep raw counts alongside percentages.

---

# 12. End-to-end implementation blueprint

## Reference architecture

```text
Business objectives / legal and contractual obligations
                        |
                        v
       Governance council + enterprise policies
                        |
        +---------------+----------------+
        |                                |
        v                                v
 Domain owners and stewards       Security / Privacy / Legal
        |                                |
        +---------------+----------------+
                        v
       Catalog + glossary + asset inventory
       Classification + ownership + lineage
                        |
                        v
 Sources -> Ingestion -> Bronze -> Silver -> Gold -> BI / APIs / AI
    |          |          |        |        |             |
    |          v          v        v        v             v
    |       Contracts   Schema   Quality  Certification  Access
    |       & counts    checks   & rules  & SLO status   controls
    |____________________________________________________________|
                        |
                        v
          Monitoring, audit evidence, issue workflow
                        |
                        v
      Reconciliation, remediation, retention and deletion
```

## Suggested delivery phases

### Phase 1 — Establish the baseline
- Select a high-value business domain and identify critical assets.
- Record owners, stewards, consumers, data sensitivity, and major flows.
- Document current quality, lineage, access, and lifecycle gaps.
- Agree on measurable outcomes and decision rights.

### Phase 2 — Define minimum standards
- Publish classification and handling rules.
- Establish glossary and data contract templates.
- Define quality dimensions, critical data elements, and issue severity.
- Establish access approval, review, retention, and exception processes.

### Phase 3 — Implement reusable controls
- Automate schema and quality validation in pipelines.
- Capture operational metadata, lineage, and source-to-target reconciliation.
- Apply governed identity and access controls.
- Connect catalog entries to business definitions and owners.
- Implement secure quarantine and replay processes.

### Phase 4 — Operate and measure
- Alert the accountable owner with business impact and evidence.
- Track issue lifecycle, recurring defects, and overdue exceptions.
- Test access, deletion, retention, and recovery controls.
- Publish a governance scorecard and review it with domain leadership.

### Phase 5 — Scale and continuously improve
- Reuse templates across domains and platforms.
- Add automated policy checks to CI/CD and data-product publication.
- Improve lineage coverage and consumer discovery.
- Reassess risks and policies as sources, regulations, products, and uses change.

---

# 13. Interview revision checklist

Before an interview, be ready to explain:

- [ ] Governance versus data management, quality, architecture, and security.
- [ ] Owners, stewards, custodians, councils, RACI, and federated governance.
- [ ] Business glossary, data dictionary, catalog, metadata, classification, and critical data elements.
- [ ] Data contracts, schema evolution, lineage, impact analysis, and certification.
- [ ] Quality dimensions, layered validation, quarantine, reconciliation, and remediation.
- [ ] Least privilege, RBAC/ABAC, row/column controls, masking, tokenization, and purpose limitation.
- [ ] Retention, archival, legal holds, deletion propagation, backups, and evidence.
- [ ] Governance in ADF, ADLS Gen2, Databricks/Delta Lake, Synapse, BI, and catalogs.
- [ ] CI/CD policy gates, managed identities, secrets, environment separation, and access drift.
- [ ] KPIs, audit evidence, preventive/detective/corrective controls, maturity, and executive reporting.
- [ ] At least three production incidents where you can describe containment, root cause, remediation, prevention, and measurable result.

## Strong structure for answering scenario questions

Use this sequence:

1. **Clarify the business rule and scope.** What is the asset, grain, time window, and expected behavior?
2. **Assess impact and risk.** Which consumers, people, decisions, or obligations are affected?
3. **Contain safely.** Pause publication, restrict access, or mark results degraded where warranted.
4. **Trace evidence.** Use logs, lineage, counts, contracts, and quality metrics.
5. **Fix and reconcile.** Repair data or controls and validate the result.
6. **Prevent recurrence.** Add tests, automation, ownership, and monitoring.
7. **Communicate and document.** Record decisions, exceptions, evidence, and follow-up actions.

## Sample closing answer: “How do you make governance practical for data engineers?”

“I translate governance requirements into metadata and controls that are part of normal engineering workflows. For each critical data product, I want an owner, business definition, classification, contract, quality rules, lineage, access policy, and retention requirement. I automate schema checks, quality gates, reconciliation, metadata registration, and evidence capture in the pipeline and CI/CD process. When a control fails, I contain the impact, route the issue to the accountable owner, preserve rejected records securely, and support deterministic repair or replay. I track outcomes such as critical-data quality, lineage coverage, access-review completion, and time to resolve issues. That way, governance is measurable and enforceable rather than only documentation.”

---

# 14. Further reading

Use current official documentation to validate product-specific details and regulatory obligations. Product features and connector coverage can change over time.

- Microsoft Purview documentation: https://learn.microsoft.com/en-us/purview/
- Azure Data Factory documentation: https://learn.microsoft.com/en-us/azure/data-factory/
- Azure Data Lake Storage documentation: https://learn.microsoft.com/en-us/azure/storage/blobs/data-lake-storage-introduction
- Azure Databricks documentation: https://learn.microsoft.com/en-us/azure/databricks/
- Delta Lake documentation: https://docs.delta.io/
- Microsoft identity and access documentation: https://learn.microsoft.com/en-us/entra/identity/
- NIST Privacy Framework: https://www.nist.gov/privacy-framework
- NIST Cybersecurity Framework: https://www.nist.gov/cyberframework
- DAMA International (DMBOK resources): https://dama.org/

**Final reminder:** Governance is contextual. Use the organization's approved policy, applicable jurisdictional requirements, contracts, and risk decisions for production implementation. The examples in this guide are interview and design patterns, not legal advice or universal control requirements.
