# Azure Data Lake Storage Gen2 (ADLS Gen2) — 49 Detailed Interview Questions

**Audience:** Azure Data Engineers, Databricks Engineers, Cloud Data Engineers  
**Focus:** Architecture, security, ingestion, performance, governance, reliability, and production scenarios  
**Last reviewed:** October 2026

> **Interview approach:** Explain the concept, describe how you would implement it, identify the production trade-offs, and give a realistic data-engineering example. Azure features, regional availability, SKUs, and pricing can change; verify current Microsoft documentation before implementing a production design.

## Table of Contents

1. [What is ADLS Gen2?](#q1-what-is-azure-data-lake-storage-gen2-adls-gen2)
2. [ADLS Gen2 vs Blob Storage](#q2-how-does-adls-gen2-differ-from-azure-blob-storage)
3. [Hierarchical namespace](#q3-what-is-hierarchical-namespace-in-adls-gen2)
4. [Big data analytics](#q4-how-does-adls-gen2-support-big-data-analytics)
5. [Key features](#q5-what-are-the-key-features-of-adls-gen2)
6. [Security and access control](#q6-how-does-adls-gen2-handle-security-and-access-control)
7. [ACLs](#q7-what-is-the-role-of-acls-in-adls-gen2)
8. [Managing permissions](#q8-how-do-you-manage-permissions-in-adls-gen2)
9. [Configuring RBAC](#q9-how-do-you-set-up-role-based-access-control-rbac-in-adls-gen2)
10. [Supported data types](#q10-what-types-of-data-can-be-stored-in-adls-gen2)
11. [Data organization](#q11-how-is-data-organized-in-adls-gen2)
12. [Soft delete vs hard delete](#q12-what-is-the-difference-between-soft-delete-and-hard-delete-in-adls-gen2)
13. [Snapshots](#q13-how-do-you-manage-snapshots-in-adls-gen2)
14. [Lifecycle management](#q14-what-is-data-lifecycle-management-in-adls-gen2)
15. [Versioning](#q15-how-do-you-manage-versioning-in-adls-gen2)
16. [Pricing tiers](#q16-what-are-the-different-pricing-tiers-available-for-adls-gen2)
17. [Redundancy](#q17-how-do-you-configure-data-redundancy-in-adls-gen2)
18. [Encryption](#q18-what-is-the-purpose-of-data-encryption-in-adls-gen2)
19. [ADF integration](#q19-how-do-you-use-adls-gen2-with-azure-data-factory)
20. [Performance optimization](#q20-what-are-the-performance-optimization-techniques-for-adls-gen2)
21. [Databricks integration](#q21-how-does-adls-gen2-integrate-with-azure-databricks)
22. [Synapse integration](#q22-what-is-azure-synapse-analytics-and-how-does-it-work-with-adls-gen2)
23. [On-premises migration](#q23-how-do-you-migrate-data-to-adls-gen2-from-on-premises-systems)
24. [Blob API compatibility](#q24-how-do-you-use-azure-blob-api-with-adls-gen2)
25. [Large datasets](#q25-how-do-you-manage-large-datasets-in-adls-gen2)
26. [Monitoring and logs](#q26-how-do-you-monitor-and-log-activities-in-adls-gen2)
27. [Diagnostic logging](#q27-how-do-you-configure-diagnostic-logging-for-adls-gen2)
28. [Big data use cases](#q28-what-are-the-use-cases-for-adls-gen2-in-big-data-scenarios)
29. [Partitioning](#q29-how-do-you-implement-data-partitioning-in-adls-gen2)
30. [Schema evolution](#q30-how-do-you-handle-schema-evolution-in-adls-gen2)
31. [Microsoft Entra ID](#q31-what-is-the-role-of-azure-active-directory-in-adls-gen2)
32. [Access policies](#q32-how-do-you-configure-access-policies-for-adls-gen2)
33. [Stream Analytics](#q33-how-do-you-integrate-adls-gen2-with-azure-stream-analytics)
34. [SAS tokens](#q34-what-is-the-role-of-sas-tokens-in-adls-gen2)
35. [Object replication](#q35-how-do-you-manage-object-replication-in-adls-gen2)
36. [ADLS Gen2 vs Amazon S3](#q36-what-are-the-differences-between-adls-gen2-and-amazon-s3)
37. [Data governance](#q37-how-does-adls-gen2-support-data-governance)
38. [Managed identity](#q38-what-is-a-managed-identity-and-how-do-you-use-it-with-adls-gen2)
39. [Cost monitoring](#q39-how-do-you-monitor-storage-costs-in-adls-gen2)
40. [File systems](#q40-how-do-you-create-and-manage-file-systems-in-adls-gen2)
41. [Key security features](#q41-what-are-the-key-security-features-of-adls-gen2)
42. [Encryption at rest and in transit](#q42-how-do-you-enforce-encryption-at-rest-and-in-transit-for-adls-gen2)
43. [Data audit](#q43-how-do-you-perform-a-data-audit-in-adls-gen2)
44. [Compliance](#q44-how-do-you-manage-compliance-requirements-in-adls-gen2)
45. [VNet service endpoints](#q45-how-do-you-configure-vnet-service-endpoints-for-adls-gen2)
46. [Machine learning](#q46-what-are-the-benefits-of-using-adls-gen2-for-machine-learning-workloads)
47. [File access optimization](#q47-how-do-you-optimize-file-access-in-adls-gen2)
48. [Large-scale ingestion](#q48-how-do-you-handle-large-scale-ingestion-into-adls-gen2)
49. [Azure Data Lake Explorer](#q49-what-is-the-role-of-the-azure-data-lake-explorer-tool)
50. [Cross-region replication](#q50-how-do-you-configure-cross-region-replication-in-adls-gen2)

---

## Q1. What is Azure Data Lake Storage Gen2 (ADLS Gen2)?

ADLS Gen2 is Azure's data-lake capability built on Azure Blob Storage. It combines Blob Storage's scalable object-storage foundation with a **hierarchical namespace (HNS)**, filesystem-style directories, and directory/file-level access-control lists (ACLs). It is designed for analytics workloads that process large volumes of structured, semi-structured, and unstructured data.

### Core concepts
- **Storage account:** Azure resource that controls endpoints, security, networking, redundancy, and account-level settings.
- **Container / file system:** Top-level grouping of data in a storage account. In ADLS Gen2, the filesystem terminology is common in DFS APIs and analytics tools.
- **Directory and file:** HNS gives directories and files real namespace semantics rather than relying only on slash characters in object names.
- **Data lake layers:** Commonly raw/Bronze, cleansed/Silver, and curated/Gold, though this is an architectural convention rather than a storage requirement.

```text
Source systems (DBs, APIs, files, events)
                 |
                 v
       Azure Data Factory
                 |
                 v
       ADLS Gen2 — Bronze
                 |
                 v
     Azure Databricks / Spark
                 |
          +------+------+
          |             |
          v             v
    Silver Delta    Quarantine
          |
          v
    Gold / curated
          |
          v
 Synapse / Fabric / BI / ML
```

### Why data engineers use it
- Durable, elastic storage for large data volumes.
- Native integration with ADF, Azure Databricks, Synapse, and other analytics services.
- HNS for directory operations and ACLs.
- Multiple redundancy, encryption, network-security, and lifecycle options.
- Storage and compute can scale independently.

**Interview answer:** “I use ADLS Gen2 as the scalable storage foundation for a medallion data platform. ADF lands source extracts in Bronze, Databricks validates and transforms them into Silver Delta tables, and Gold datasets serve reporting and downstream analytics. I secure access with managed identities, RBAC, ACLs, and network controls.”

**Microsoft docs:** [Introduction to Azure Data Lake Storage Gen2](https://learn.microsoft.com/en-us/azure/storage/blobs/data-lake-storage-introduction)

## Q2. How does ADLS Gen2 differ from Azure Blob Storage?

ADLS Gen2 is not a completely separate storage engine. It is Blob Storage with the hierarchical namespace capability enabled, plus data-lake-oriented semantics and integrations.

| Capability | Blob Storage without HNS | ADLS Gen2 with HNS |
|---|---|---|
| Storage foundation | Azure Blob Storage | Azure Blob Storage |
| Namespace | Flat object namespace; slash prefixes can imitate folders | Real hierarchical namespace |
| Directory rename/delete | Often requires operations over individual objects/prefixes | Filesystem directory operations are supported |
| POSIX-style ACLs | Not the same HNS ACL model | Directory/file ACLs supported |
| Analytics | Broad object-storage integrations | Optimized semantics for many analytics workloads |
| APIs | Blob APIs | Blob APIs and Data Lake Storage DFS APIs |
| Best fit | General-purpose object storage | Data lakes and analytics requiring HNS |

### Important design point
HNS is an account-level capability and enabling it has historically had restrictions and migration implications. Validate current conversion support and feature compatibility before creating or changing an existing production account. Not every Blob feature or workload behaves identically in every configuration.

**Scenario:** A platform repeatedly renames date-partitioned directories and needs path-based ACLs for analytics teams. HNS can make directory operations and access control more natural than a flat namespace.

**Microsoft docs:** [Introduction to ADLS Gen2](https://learn.microsoft.com/en-us/azure/storage/blobs/data-lake-storage-introduction), [Hierarchical namespace](https://learn.microsoft.com/en-us/azure/storage/blobs/data-lake-storage-namespace)

## Q3. What is hierarchical namespace in ADLS Gen2?

Hierarchical namespace (HNS) gives the storage account filesystem-like organization. Directories are first-class namespace entities and operations such as rename and delete can operate at directory level instead of treating every path as an unrelated blob name.

### Benefits
- Efficient directory rename and delete semantics for analytics directory structures.
- Directory and file ACLs.
- Familiar filesystem organization for Spark and data engineering tools.
- Easier separation of data by domain, environment, layer, or sensitivity.

Example:
```text
/pharma/
  /bronze/
    /batch_events/ingest_date=2026-10-18/
  /silver/
    /batch_quality/
  /gold/
    /batch_traceability/
```

### How to enable it
1. Create a supported general-purpose v2 storage account.
2. During account creation, enable **Hierarchical namespace**.
3. Select redundancy, region, networking, and security settings.
4. Create a filesystem/container and directories.
5. Test API compatibility and permissions with the intended workloads.

Do not enable HNS casually on an existing account: verify current migration/conversion support, service limitations, and required feature compatibility first.

**Microsoft docs:** [Upgrade Blob Storage with ADLS Gen2 capabilities](https://learn.microsoft.com/en-us/azure/storage/blobs/upgrade-to-data-lake-storage-gen2)

## Q4. How does ADLS Gen2 support big data analytics?

ADLS Gen2 separates storage from compute. Data can remain in the lake while different compute engines—such as Databricks, Synapse, and supported Fabric workloads—read it as needed. This avoids binding data to a single processing cluster.

### Typical processing pattern
1. Ingest source data with ADF, AzCopy, SDKs, or streaming services.
2. Preserve raw source files in Bronze for traceability and replay.
3. Validate schema, timestamps, keys, and data quality.
4. Transform into partitioned Parquet or Delta tables.
5. Publish curated Gold data for BI, SQL, and ML consumers.
6. Monitor pipeline completion, data quality, freshness, and cost.

### Performance considerations
- Use columnar formats such as Parquet for analytical scans.
- Avoid millions of tiny files.
- Partition by columns frequently used for filtering, while avoiding excessive cardinality.
- Use Delta Lake for transactional table operations where appropriate; ADLS stores the files, while Delta's transaction log supplies table-level transaction semantics.
- Tune compute, parallelism, and network connectivity as well as storage layout.

**Scenario:** A retail lake receives 80 GB/day. ADF lands daily source extracts; Databricks transforms them into Delta tables partitioned sensibly by business date; Gold aggregates serve sales and inventory reporting.

**Microsoft docs:** [ADLS Gen2 overview](https://learn.microsoft.com/en-us/azure/storage/blobs/data-lake-storage-introduction), [Azure Databricks documentation](https://learn.microsoft.com/en-us/azure/databricks/)

## Q5. What are the key features of ADLS Gen2?

| Feature | Why it matters |
|---|---|
| Hierarchical namespace | Real directory and file semantics |
| Fine-grained ACLs | Path-level access control |
| Azure RBAC | Role assignments at supported Azure resource scopes |
| Entra ID / managed identities | Identity-based authentication without embedded secrets |
| Encryption at rest | Protects stored data |
| HTTPS/TLS | Protects data in transit |
| Redundancy options | Durability and disaster-recovery choices |
| Lifecycle management | Automates tiering/deletion for supported data |
| Diagnostic logs and metrics | Operations, troubleshooting, and audit evidence |
| ADF / Databricks / Synapse integration | Common data engineering ecosystem |
| Blob and DFS APIs | Broad application compatibility |
| Private endpoints / firewall rules | Network exposure control |

Availability of individual features can depend on account type, region, protocol, and current service support. Check the feature matrix before relying on a specific combination.

**Microsoft docs:** [ADLS Gen2 documentation](https://learn.microsoft.com/en-us/azure/storage/blobs/data-lake-storage-introduction)

## Q6. How does ADLS Gen2 handle security and access control?

Use defense in depth. Authentication establishes the caller's identity; authorization decides what that identity may do; network controls restrict where connections originate; encryption protects data; logging supports investigation.

### Main controls
- **Microsoft Entra ID:** User, service principal, and managed identity authentication.
- **Azure RBAC:** Role assignments at subscription, resource group, account, and supported data scopes.
- **ACLs:** Directory/file-level permissions and inheritance for HNS-enabled accounts.
- **SAS:** Time-limited delegated access, when appropriate.
- **Storage account keys:** Powerful shared secrets; avoid for routine application access where identity-based access is feasible.
- **Network controls:** Firewall rules, selected networks, private endpoints, and appropriate DNS.
- **Encryption:** Encryption at rest and HTTPS/TLS in transit.
- **Monitoring:** Diagnostic settings, logs, metrics, alerts, and periodic access reviews.

### Example production design
- ADF uses a managed identity.
- Databricks accesses the lake using supported identity-based configuration and Unity Catalog external locations where applicable.
- Analysts receive read-only access to approved curated paths.
- Raw patient or safety data is isolated from general analyst access.
- Production identities have only the permissions they need.
- Secrets and keys are not hardcoded in notebooks or pipeline JSON.

**Microsoft docs:** [Authorize access to Azure Storage](https://learn.microsoft.com/en-us/azure/storage/common/authorize-data-access), [ADLS access control](https://learn.microsoft.com/en-us/azure/storage/blobs/data-lake-storage-access-control)

## Q7. What is the role of ACLs in ADLS Gen2?

Access-control lists provide directory/file-level permissions on HNS-enabled accounts. ACLs are useful when teams need access to a specific path without granting broad access to all data in the storage account.

### ACL concepts
- **Access ACL:** Permissions on a particular file or directory.
- **Default ACL:** A directory template used to establish default permissions for newly created child items.
- **User/group/other entries:** POSIX-style permission categories, including named users and groups.
- **Permissions:** Read (`r`), write (`w`), and execute (`x`). For directories, execute is important for traversing the path; write and execute together are generally needed to create or delete children.

### Example
A data-science group may need read access to `/gold/forecasting/`, while only the pipeline identity can write there. The group should not automatically gain access to `/bronze/restricted_patient_data/`.

### Critical nuance
RBAC and ACLs work together. Broad data-plane RBAC roles can grant access that makes restrictive ACLs ineffective for that principal. When troubleshooting access, check both RBAC role assignments and every relevant ACL along the path. Default ACLs do not retroactively repair permissions on existing children.

**Microsoft docs:** [Access control lists in ADLS Gen2](https://learn.microsoft.com/en-us/azure/storage/blobs/data-lake-storage-access-control)

## Q8. How do you manage permissions in ADLS Gen2?

A practical permission workflow is:

1. Define data owners, producers, consumers, and sensitivity levels.
2. Create Microsoft Entra security groups for roles such as platform operators, ingestion identities, analysts, and data scientists.
3. Assign the narrowest suitable Azure RBAC roles at the narrowest practical scope.
4. Use ACLs for path-specific permissions where needed.
5. Configure default ACLs for new child items in directories that need consistent inheritance.
6. Test using the actual application identity, not only an administrator account.
7. Review access regularly and remove stale assignments.
8. Log and investigate authorization failures.

### Troubleshooting checklist
- Is the identity authenticating as the expected tenant/principal?
- Does it have a suitable data-plane RBAC role?
- Does the role apply at the expected scope?
- Is the role assignment fully propagated?
- Does each parent directory allow traversal?
- Do file/directory ACLs grant the required operation?
- Is a firewall or private endpoint blocking the connection?
- Is the client using the correct endpoint/protocol?

Use least privilege. Avoid assigning broad Storage Account Contributor or account-key access simply to fix a file-read error.

**Microsoft docs:** [Access control model for ADLS Gen2](https://learn.microsoft.com/en-us/azure/storage/blobs/data-lake-storage-access-control)

## Q9. How do you set up role-based access control (RBAC) in ADLS Gen2?

RBAC assigns Azure roles to users, groups, service principals, or managed identities at a defined scope. For data access, choose a role intended for the required data-plane operation rather than assuming every management-plane role grants file contents.

### Typical setup
1. Open the storage account in Azure portal.
2. Select **Access control (IAM)**.
3. Add a role assignment.
4. Select an appropriate built-in or custom role.
5. Select the user, group, service principal, or managed identity.
6. Choose the narrowest supported scope and validate access.

### Common role distinctions
- **Storage Blob Data Reader:** Read/list blob data.
- **Storage Blob Data Contributor:** Read, write, and delete blob data within its assigned scope.
- **Storage Blob Data Owner:** Broad data access and permission-management capabilities, including ACL management where supported.
- **Contributor:** Manages Azure resources but should not be assumed to grant direct access to stored data.
- **Owner:** Broad management-plane permissions; do not grant casually.

Exact permissions and scope behavior should be checked in the current role definition.

### Example with Azure CLI
```bash
az role assignment create \
  --assignee "<managed-identity-or-group-object-id>" \
  --role "Storage Blob Data Reader" \
  --scope "/subscriptions/<subscription-id>/resourceGroups/<rg>/providers/Microsoft.Storage/storageAccounts/<account>"
```

Use ACLs for more granular path-level access where required. Validate the result with a real test read and a negative test against a path the principal should not access.

**Microsoft docs:** [Assign an Azure role for blob data access](https://learn.microsoft.com/en-us/azure/storage/blobs/assign-azure-role-data-access)

## Q10. What types of data can be stored in ADLS Gen2?

ADLS Gen2 can store almost any file content; the storage service does not impose a relational schema on the file payload.

Examples:
- Structured extracts: CSV, TSV, fixed-width files.
- Semi-structured: JSON, XML, Avro.
- Analytical columnar files: Parquet and ORC.
- Delta Lake tables: Parquet data files plus a `_delta_log` transaction log.
- Documents, images, audio, video, and binary data.
- Application logs, telemetry, IoT events, and archived extracts.
- ML features, training data, model artifacts, and evaluation outputs.

The file format should match the use case. Parquet is often a good choice for large analytical scans; JSON is useful for raw API payloads and flexible records; Delta adds transaction and table-management features on top of supported data files.

**Microsoft docs:** [ADLS Gen2 overview](https://learn.microsoft.com/en-us/azure/storage/blobs/data-lake-storage-introduction)

## Q11. How is data organized in ADLS Gen2?

A lake should be organized around data domains, processing stages, access boundaries, retention, and consumers—not just arbitrary folders.

Example:
```text
abfss://lake@companydatalake.dfs.core.windows.net/
├── bronze/
│   ├── pharma/
│   │   ├── batches/ingest_date=2026-10-18/
│   │   └── quality_results/ingest_date=2026-10-18/
├── silver/
│   ├── pharma/batch/
│   └── pharma/quality_result/
├── gold/
│   ├── batch_traceability/
│   └── quality_dashboard/
├── quarantine/
└── reference/
```

### Organization guidelines
- Separate environments (dev/test/prod) and sensitive domains.
- Keep raw data immutable where replay/audit needs justify it.
- Use consistent naming, timestamps, source identifiers, and ingestion metadata.
- Use partitions for common query filters, not every column.
- Define retention and ownership.
- Avoid putting thousands of unrelated files in one operational directory if listing and processing become inefficient.
- Document schema, lineage, and data-quality expectations in a catalog.

Directories are physical organization; the medallion architecture is a data-processing design. They are related but not the same thing.

## Q12. What is the difference between soft delete and hard delete in ADLS Gen2?

A **hard delete** removes the item from its active namespace. A **soft delete** retains recoverable data for a configured retention window, when the relevant protection feature is supported and enabled.

| Aspect | Soft delete | Hard delete |
|---|---|---|
| Recovery | Possible during retention window | Generally not recoverable through ordinary storage controls |
| Storage cost | Deleted data may continue to incur storage charges | Usually stops charging for the removed active data, subject to retained copies/versions |
| Use | Accidental deletion recovery | Intentional permanent removal |
| Governance | Helps recovery but is not a complete backup | Must respect retention, legal hold, and deletion obligations |

### Implementation considerations
- Enable the appropriate blob/container soft-delete protections where supported.
- Set a retention period based on recovery objectives and compliance needs.
- Test restoration procedures.
- Understand interactions with HNS, versions, snapshots, lifecycle policies, and account configuration.
- For sensitive data, confirm that soft delete, immutable retention, and legal obligations do not conflict with the intended deletion workflow.

Do not assume that enabling one recovery feature automatically protects every object type or every deletion path. Verify current feature support for the exact ADLS Gen2 configuration.

**Microsoft docs:** [Soft delete for blobs](https://learn.microsoft.com/en-us/azure/storage/blobs/soft-delete-blob-overview), [Soft delete for containers](https://learn.microsoft.com/en-us/azure/storage/blobs/soft-delete-container-overview)

## Q13. How do you manage snapshots in ADLS Gen2?

A snapshot is a point-in-time, read-only representation of a blob. Snapshots are useful for recovery and comparison, but they are not a substitute for a complete backup or cross-region disaster-recovery plan. Feature availability and behavior depend on HNS and the specific storage configuration.

### Practical workflow
1. Confirm snapshots are supported for the target account and data type.
2. Define which data needs point-in-time protection.
3. Create snapshots using a supported API, SDK, or tool.
4. Restrict who can create/delete snapshots.
5. Define retention and cleanup policies.
6. Test recovery, including metadata and application consistency.

### Important distinction
- **Snapshot:** Point-in-time copy/reference for a blob.
- **Versioning:** Maintains versions when supported and enabled.
- **Soft delete:** Retains deleted data for a retention window.
- **Backup/DR:** Broader recovery design that may include separate copies, regions, and application dependencies.

For Delta tables, do not restore isolated data files casually; the transaction log and table consistency must be considered. Use Delta-aware recovery procedures for table-level recovery.

**Microsoft docs:** [Blob snapshots](https://learn.microsoft.com/en-us/azure/storage/blobs/snapshots-overview)

## Q14. What is data lifecycle management in ADLS Gen2?

Lifecycle management automates tiering or deletion of eligible data based on rules such as age, last modification, or supported object attributes. It is useful for controlling the long-term cost of raw files, logs, and historical datasets.

### Example policy
- Keep recently ingested files in Hot for active processing.
- Move older, rarely read eligible data to Cool or Cold.
- Delete transient landing files after the approved retention period.
- Preserve regulated datasets according to the records-retention policy.

### Design process
1. Classify data by business value, access frequency, and retention.
2. Confirm which access tiers and operations are supported for the account and object type.
3. Write and test lifecycle rules on a non-production prefix.
4. Monitor tier transitions and deletions.
5. Account for retrieval charges, minimum-retention charges, and rehydration requirements where applicable.

Lifecycle policies should not delete files that are still referenced by Delta transaction logs or needed for replay, audit, or downstream consumers. Coordinate retention with table maintenance and data governance.

**Microsoft docs:** [Azure Blob Storage lifecycle management](https://learn.microsoft.com/en-us/azure/storage/blobs/lifecycle-management-overview)

## Q15. How do you manage versioning in ADLS Gen2?

Blob versioning can preserve earlier states of blobs when enabled and supported. It helps recover from overwrites and supports investigation of unintended changes. HNS feature compatibility and restrictions must be checked before relying on versioning in an ADLS Gen2 account.

### Versioning workflow
1. Check current account feature support and limitations.
2. Enable versioning if available for the required account configuration.
3. Define which data needs versions and for how long.
4. Test overwrite and restore operations.
5. Set lifecycle cleanup for old versions where supported.
6. Monitor storage growth.

### Operational caution
Versions, snapshots, and soft-deleted data can all consume capacity. A versioning policy without lifecycle cleanup may substantially increase storage cost. Versioning a data file is also not the same as maintaining a consistent historical version of a multi-file Delta table.

**Microsoft docs:** [Blob versioning overview](https://learn.microsoft.com/en-us/azure/storage/blobs/versioning-overview)

## Q16. What are the different pricing tiers available for ADLS Gen2?

ADLS Gen2 uses Azure Blob Storage pricing models. Depending on account type, region, and feature support, online access tiers can include Hot, Cool, and Cold, while Archive is an offline tier with rehydration requirements. Confirm current support for the exact HNS-enabled account.

| Cost driver | What to estimate |
|---|---|
| Capacity | GB stored by tier and redundancy option |
| Operations | Read, write, list, create, rename, and other requests |
| Retrieval | Data read from cooler tiers where charged |
| Data transfer | Outbound or cross-region transfer |
| Retention features | Versions, soft-deleted data, snapshots where applicable |
| Redundancy | LRS/ZRS/GRS/GZRS and supported variants |
| Related services | Private Link, monitoring ingestion, compute, orchestration |

### Cost-control process
- Measure access patterns before choosing tiers.
- Keep frequently processed data online.
- Apply lifecycle policies to eligible older data.
- Compact tiny files to reduce request and compute overhead.
- Use the Azure Pricing Calculator and actual cost analysis data.
- Include Databricks/Synapse/ADF compute costs; storage savings can be outweighed by processing costs.

**Microsoft docs:** [Azure Blob Storage pricing](https://azure.microsoft.com/en-us/pricing/details/storage/blobs/), [Access tiers](https://learn.microsoft.com/en-us/azure/storage/blobs/access-tiers-overview)

## Q17. How do you configure data redundancy in ADLS Gen2?

Redundancy determines how storage data is replicated for durability and resilience.

| Option | High-level protection |
|---|---|
| LRS | Multiple copies within one physical location in the primary region |
| ZRS | Replication across availability zones in the primary region, where supported |
| GRS | Local redundancy in the primary region plus asynchronous replication to a secondary region |
| RA-GRS | GRS with read access to the secondary region |
| GZRS | Zone redundancy in the primary region plus geo-replication |
| RA-GZRS | GZRS with read access to the secondary region |

### Configuration approach
1. Choose the region and supported account configuration.
2. Select the required redundancy option during creation or change it where supported.
3. Review regional availability, HNS compatibility, and cost.
4. Define the recovery point objective (RPO) and recovery time objective (RTO).
5. Test application recovery, not just storage replication.

Geo-replication is asynchronous, so it does not guarantee zero data loss. Redundancy is not a substitute for protection against accidental deletion, corrupt writes, compromised identities, or application-level mistakes.

**Microsoft docs:** [Azure Storage redundancy](https://learn.microsoft.com/en-us/azure/storage/common/storage-redundancy)

## Q18. What is the purpose of data encryption in ADLS Gen2?

Encryption protects data confidentiality if underlying storage media or network traffic is exposed. It is one part of a broader security design.

- **Encryption at rest:** Azure Storage encrypts stored data by default.
- **Microsoft-managed keys:** Azure manages the encryption keys.
- **Customer-managed keys (CMK):** The customer controls keys using supported Azure Key Vault or Managed HSM configurations.
- **Encryption in transit:** HTTPS/TLS protects data moving between clients and storage.
- **Infrastructure encryption:** An additional encryption layer may be available in supported configurations.

### CMK implementation outline
1. Create or select a suitable Key Vault key.
2. Configure the required identity and permissions.
3. Configure the storage account encryption setting.
4. Validate key rotation, recovery, monitoring, and access procedures.
5. Test operational behavior if the key becomes unavailable.

Encryption does not replace authorization, network isolation, data classification, or audit logging.

**Microsoft docs:** [Azure Storage encryption at rest](https://learn.microsoft.com/en-us/azure/storage/common/storage-service-encryption), [Require secure transfer](https://learn.microsoft.com/en-us/azure/storage/common/storage-require-secure-transfer)

## Q19. How do you use ADLS Gen2 with Azure Data Factory?

ADF orchestrates ingestion and movement of data between source systems and ADLS Gen2. ADF can use a linked service to connect to the storage account and datasets or inline settings to describe paths and formats.

### Typical implementation
1. Create or select an ADLS Gen2 storage account and filesystem.
2. Create an ADF linked service for ADLS Gen2.
3. Authenticate with managed identity, service principal, or another supported method.
4. Grant the identity appropriate RBAC and, where needed, ACL permissions.
5. Configure source and sink datasets / Copy Activity.
6. Parameterize filesystem, directory, file name, and date.
7. Add watermark/CDC logic for incremental ingestion.
8. Add retries, logging, alerts, and validation.
9. Test using the pipeline identity and publish through CI/CD.

### Example pipeline
```text
SQL Server / API / SFTP
          |
          v
ADF pipeline (parameters + watermark)
          |
          v
ADLS Gen2 Bronze
          |
          v
Databricks transformation
          |
          v
Silver / Gold Delta
```

**Common failures:** wrong DFS endpoint, missing RBAC/ACL permission, firewall/private endpoint/DNS issue, expired secret, incorrect path, unsupported format, or integration runtime network reachability.

**Microsoft docs:** [Copy data to/from ADLS Gen2](https://learn.microsoft.com/en-us/azure/data-factory/connector-azure-data-lake-storage), [ADF managed identity](https://learn.microsoft.com/en-us/azure/data-factory/data-factory-service-identity)

## Q20. What are the performance optimization techniques for ADLS Gen2?

Storage performance depends on request patterns, file sizes, concurrency, compute, network, and data layout.

### Key techniques
- **Right-size files:** Avoid excessive tiny files; compact them into files appropriate for the processing engine and workload.
- **Use columnar formats:** Parquet and Delta often reduce I/O for analytical queries.
- **Control concurrency:** Increase parallelism carefully; too many concurrent requests can cause throttling or overload upstream systems.
- **Partition thoughtfully:** Partition on common filter columns and avoid high-cardinality directory explosions.
- **Use incremental processing:** Process new/changed data instead of repeatedly scanning all history.
- **Reduce directory-listing overhead:** Avoid repeated broad recursive listings when manifests or incremental metadata can be used.
- **Tune compute:** Spark executor sizing, shuffle behavior, caching, and partition counts can matter more than raw storage throughput.
- **Use suitable networking:** Validate region proximity, private endpoint routing, DNS, and integration runtime location.
- **Monitor throttling and latency:** Use metrics and logs to identify request patterns and bottlenecks.

### Troubleshooting example
If a Databricks job that previously completed in 20 minutes now takes 90 minutes, compare input volume, file count and size, Spark stage metrics, shuffle spill, storage latency, throttling, and concurrent workloads. Do not assume the storage account alone is the bottleneck.

**Microsoft docs:** [ADLS performance and scalability](https://learn.microsoft.com/en-us/azure/storage/blobs/data-lake-storage-best-practices)

## Q21. How does ADLS Gen2 integrate with Azure Databricks?

ADLS Gen2 is a common durable storage layer for Databricks. Databricks reads and writes data using Spark connectors and cloud-storage APIs. For modern governance, use Unity Catalog external locations and storage credentials backed by supported identities instead of embedding storage keys in notebooks.

### Recommended pattern
1. Create an ADLS Gen2 account and filesystem.
2. Configure a managed identity or service principal according to the supported Unity Catalog design.
3. Grant storage RBAC and ACL permissions required by the identity.
4. Create a Unity Catalog storage credential and external location.
5. Grant catalog/schema/table or external-location privileges to the right groups.
6. Read and write Delta tables using governed paths or managed tables.
7. Monitor access and job performance.

Example Spark read for a supported authenticated configuration:
```python
df = (spark.read
      .format("parquet")
      .load("abfss://bronze@<account>.dfs.core.windows.net/sales/"))
```

The URI alone does not configure authentication. Avoid account keys in notebook source. Prefer Unity Catalog governance for production workloads; exact setup depends on workspace and cloud configuration.

### Common uses
- Bronze raw files and replay.
- Silver Delta transformations.
- Gold curated datasets.
- Structured Streaming checkpoints and outputs.
- ML training datasets and model artifacts.

**Microsoft docs:** [Azure Databricks and ADLS](https://learn.microsoft.com/en-us/azure/databricks/connect/storage/azure-storage), [Unity Catalog external locations](https://learn.microsoft.com/en-us/azure/databricks/connect/unity-catalog/cloud-storage/external-locations)

## Q22. What is Azure Synapse Analytics and how does it work with ADLS Gen2?

Azure Synapse Analytics is an analytics service that combines SQL analytics, Spark, and integration capabilities. ADLS Gen2 commonly stores source and curated data that Synapse serverless SQL, Spark, or dedicated SQL workloads can consume.

### Common integration patterns
- **Serverless SQL:** Query supported files in the lake without first loading them into dedicated tables.
- **Spark pools:** Use Spark for data preparation and analytics.
- **Dedicated SQL pool:** Load and model data for predictable, high-performance warehouse workloads.
- **Pipelines:** Orchestrate ingestion and movement, depending on the architecture.

### Design considerations
- Secure the lake with managed identity and appropriate RBAC/ACLs.
- Choose file format and partitioning for the query pattern.
- Avoid scanning unnecessary partitions and columns.
- Distinguish lake files from dedicated SQL pool storage: a dedicated pool typically loads/manages its own data.
- Keep semantic modeling and reporting requirements in mind.

**Scenario:** Gold Delta/Parquet datasets in ADLS are exposed to analytics consumers through a supported Synapse query pattern, while frequently used warehouse facts may be loaded into a modeled warehouse.

**Microsoft docs:** [Query ADLS Gen2 using serverless SQL pool](https://learn.microsoft.com/en-us/azure/synapse-analytics/sql/query-data-storage), [Synapse Analytics documentation](https://learn.microsoft.com/en-us/azure/synapse-analytics/)

## Q23. How do you migrate data to ADLS Gen2 from on-premises systems?

The best tool depends on volume, bandwidth, change rate, connectivity, and downtime tolerance.

### Common options
- **AzCopy:** High-throughput command-line transfer for supported storage operations.
- **Azure Data Factory:** Managed orchestration and connectors for supported source systems.
- **Azure Data Box:** Physical transfer for very large datasets or constrained network connectivity.
- **Self-hosted integration runtime:** ADF component for reaching on-premises data sources.
- **Custom SDK/API transfer:** For specialized metadata, transformations, or application integration.

### Migration plan
1. Inventory source systems, total data, file counts, permissions, formats, and retention.
2. Choose the transfer method and estimate bandwidth/duration.
3. Establish private connectivity or secure network access.
4. Create target paths and access controls.
5. Perform an initial bulk copy.
6. Run incremental/delta copies for changes since the baseline.
7. Validate counts, sizes, checksums where available, schema, and business totals.
8. Cut over consumers and keep a rollback plan.
9. Monitor post-cutover workloads and retire the old path only after approval.

### CLI illustration
```bash
azcopy copy "D:\exports\*" \
  "https://<account>.dfs.core.windows.net/<filesystem>/<path>?<SAS>" \
  --recursive=true
```

Use supported authentication and endpoint options for the chosen AzCopy version and account configuration. Treat SAS tokens as secrets; do not store them in shell history or logs.

**Microsoft docs:** [AzCopy](https://learn.microsoft.com/en-us/azure/storage/common/storage-use-azcopy-v10), [Azure Data Box](https://learn.microsoft.com/en-us/azure/databox/)

## Q24. How do you use Azure Blob API with ADLS Gen2?

ADLS Gen2 supports Data Lake Storage (DFS) APIs and Blob service APIs, but not every feature behaves identically across all HNS-enabled configurations. Use the API suited to the operation and verify compatibility.

- **DFS endpoint:** Commonly `https://<account>.dfs.core.windows.net`; supports filesystem-oriented operations such as directory/file operations.
- **Blob endpoint:** Commonly `https://<account>.blob.core.windows.net`; supports many Blob operations and integrations.
- **ABFS/ABFSS:** Common filesystem schemes used by Hadoop/Spark clients for Azure storage; `abfss` denotes TLS-secured access.

### Practical guidance
1. Confirm the operation is supported for HNS-enabled accounts.
2. Use the documented endpoint and API version.
3. Authenticate using Microsoft Entra ID or another approved method.
4. Grant appropriate RBAC and ACL permissions.
5. Test rename, delete, list, upload, and concurrency behavior.
6. Review feature-specific compatibility before combining Blob features with HNS.

Do not assume “Blob API support” means every Blob Storage feature is available without restrictions in every ADLS Gen2 configuration.

**Microsoft docs:** [ADLS Gen2 multi-protocol access](https://learn.microsoft.com/en-us/azure/storage/blobs/data-lake-storage-multi-protocol-access)

## Q25. How do you manage large datasets in ADLS Gen2?

Large datasets require storage-layout, metadata, processing, and operational discipline.

### Recommended practices
- Partition by useful low-to-moderate-cardinality columns such as event date or business date.
- Use Parquet or Delta for analytical workloads.
- Compact small files; avoid generating a new tiny file for every micro-batch or task.
- Use incremental loads and CDC rather than full reloads.
- Maintain a raw layer for replay and a curated layer for query efficiency.
- Use manifests or ingestion metadata where appropriate to avoid expensive repeated file discovery.
- Apply retention policies and archive only when retrieval expectations permit.
- Monitor file count, directory listing time, storage operations, and downstream Spark task counts.

### Example
Instead of writing 100,000 tiny files for a day's sales data, repartition the dataset based on the processing workload and write a manageable number of larger files. Avoid blindly using `coalesce(1)`, which can create a bottleneck and a single oversized file.

**Microsoft docs:** [ADLS Gen2 best practices](https://learn.microsoft.com/en-us/azure/storage/blobs/data-lake-storage-best-practices)

## Q26. How do you monitor and log activities in ADLS Gen2?

Use Azure Monitor metrics, diagnostic resource logs, Activity Log, and workload-specific logs. These serve different purposes.

| Signal | What it helps answer |
|---|---|
| Metrics | Is request volume, latency, availability, or capacity changing? |
| Resource logs | Which storage requests occurred, and did they succeed or fail? |
| Azure Activity Log | Who changed Azure resource configuration or role assignments? |
| ADF / Databricks logs | Which pipeline/job failed and at which stage? |
| Alerts | When should operations be notified? |
| Log Analytics | How can logs be queried and correlated over time? |

### Production approach
1. Enable diagnostic settings for the storage account and relevant services.
2. Route logs to Log Analytics, Storage, or Event Hubs according to retention and analysis requirements.
3. Build alerts for availability, latency, throttling, unexpected data growth, and authorization failures.
4. Correlate storage request times with ADF runs and Databricks job IDs.
5. Define dashboards and operational runbooks.
6. Review diagnostic cost and retention.

Logging configuration is not automatic for every log category; enable the categories required by your investigation and audit needs.

**Microsoft docs:** [Monitor Azure Blob Storage](https://learn.microsoft.com/en-us/azure/storage/blobs/monitor-blob-storage)

## Q27. How do you configure diagnostic logging for ADLS Gen2?

### Azure portal
1. Open the storage account.
2. Select **Monitoring > Diagnostic settings**.
3. Create a diagnostic setting.
4. Select the available log categories and metrics needed for the service.
5. Choose a destination: Log Analytics workspace, storage account, or Event Hubs.
6. Save and generate test requests.
7. Confirm records arrive and configure retention/access controls at the destination.

### Azure CLI illustration
The exact category names and available metrics can change, so inspect the account's supported categories first:
```bash
az monitor diagnostic-settings categories list \
  --resource "<storage-account-resource-id>"

az monitor diagnostic-settings create \
  --name "adls-diagnostics" \
  --resource "<storage-account-resource-id>" \
  --workspace "<log-analytics-workspace-resource-id>" \
  --logs '[{"category":"StorageRead","enabled":true},
           {"category":"StorageWrite","enabled":true},
           {"category":"StorageDelete","enabled":true}]' \
  --metrics '[{"category":"Transaction","enabled":true}]'
```

This is an illustrative template: use the exact categories returned for your resource and CLI/API version. Validate ingestion and permissions after deployment.

### Common mistake
Enabling metrics but not resource logs may show that errors increased without showing enough request-level detail to investigate. Conversely, sending every log to a high-volume workspace can create unnecessary cost. Select categories intentionally.

**Microsoft docs:** [Create diagnostic settings](https://learn.microsoft.com/en-us/azure/azure-monitor/essentials/diagnostic-settings)

## Q28. What are the use cases for ADLS Gen2 in big data scenarios?

- **Enterprise lakehouse:** Bronze/Silver/Gold layers with Delta tables.
- **Batch ingestion:** Daily exports from SQL Server, SAP, Oracle, APIs, and SaaS applications.
- **Streaming:** Persist raw events and curated outputs from Event Hubs, Kafka-based systems, and Stream Analytics.
- **IoT and telemetry:** High-volume time-series event landing and aggregation.
- **Healthcare and pharma:** Encounter, batch, quality, safety, and clinical-supply datasets with appropriate sensitive-data controls.
- **Retail:** Sales, inventory, web events, and customer interactions.
- **Machine learning:** Training data, features, experiments, and model artifacts.
- **Log analytics:** Retained operational logs and large-scale event files.
- **Regulatory records:** Retained datasets with approved retention and immutability controls.

### Design principle
ADLS Gen2 is the storage layer, not the entire data platform. ADF, Databricks, Synapse, governance tooling, monitoring, and downstream BI/ML services supply ingestion, processing, cataloging, and consumption capabilities.

## Q29. How do you implement data partitioning in ADLS Gen2?

Partitioning places records into separate directory paths based on selected column values. It can reduce the amount of data scanned when query filters match partition columns.

Example:
```text
sales/
  business_date=2026-10-16/region=west/part-000.parquet
  business_date=2026-10-17/region=west/part-000.parquet
```

### How to choose partition columns
1. Inspect common filters and join patterns.
2. Prefer useful columns such as event date, business date, or sometimes region.
3. Avoid partitioning on unique IDs or other high-cardinality columns.
4. Estimate rows and bytes per partition.
5. Test query performance and file counts.
6. Revisit the design as data volume and query patterns change.

Partitioning is not the same as sorting, clustering, or bucketing. Excessive partitioning can create many small directories/files and slow metadata operations.

### PySpark example
```python
(df.write
   .format("parquet")
   .mode("append")
   .partitionBy("business_date")
   .save("abfss://curated@<account>.dfs.core.windows.net/sales"))
```

For production Delta tables, consider Delta-aware layout optimization and table maintenance instead of assuming a generic Parquet write is sufficient.

**Microsoft docs:** [ADLS Gen2 best practices](https://learn.microsoft.com/en-us/azure/storage/blobs/data-lake-storage-best-practices)

## Q30. How do you handle schema evolution in ADLS Gen2?

ADLS Gen2 stores files; it does not enforce one universal schema across a directory. Schema enforcement and evolution are handled by the file format, processing engine, and table format.

### Robust approach
- Preserve source payloads in Bronze for replay and traceability.
- Maintain an expected schema contract for each source.
- Validate required columns, data types, nullability, and allowed values.
- Quarantine incompatible files or records instead of silently dropping them.
- Define whether new nullable columns are allowed automatically or require approval.
- Version schema definitions and log changes.
- Use Delta Lake schema enforcement/evolution intentionally for curated tables.
- Backfill historical data where new columns need consistent values.

### Delta example
```python
(df.write
   .format("delta")
   .mode("append")
   .option("mergeSchema", "true")
   .save(target_path))
```

Use `mergeSchema` only when the change is expected and governed. It is not a replacement for schema validation, compatibility checks, or change approval. Avoid broad automatic schema evolution that lets unexpected upstream fields silently alter production contracts.

**Microsoft docs:** [Schema evolution in Delta Lake](https://docs.delta.io/latest/delta-batch.html#schema-validation)

## Q31. What is the role of Azure Active Directory in ADLS Gen2?

Azure Active Directory is now called **Microsoft Entra ID**. It provides identity and authentication for users, groups, applications, and managed identities accessing storage.

### Typical authentication flow
1. A user or workload obtains a token from Microsoft Entra ID.
2. The client sends the token to the storage service over HTTPS.
3. Storage evaluates RBAC assignments and, where relevant, ACLs.
4. The request is allowed or denied and can be logged.

### Why use identity-based access?
- Avoid distributing account keys to every pipeline or notebook.
- Centralize group membership and access reviews.
- Support managed identity for Azure services.
- Improve traceability by identifying the principal making a request.
- Enable least-privilege assignments and automated credential rotation through the identity platform.

Authentication alone does not grant data access. The identity also needs appropriate authorization and network reachability.

**Microsoft docs:** [Authorize access to Azure Storage](https://learn.microsoft.com/en-us/azure/storage/common/authorize-data-access)

## Q32. How do you configure access policies for ADLS Gen2?

“Access policy” can refer to several distinct controls, so clarify which one is meant:

1. **RBAC role assignments:** Define access at supported Azure scopes.
2. **ACLs:** Define read/write/execute permissions on directories and files.
3. **SAS stored access policies:** Where supported for the relevant SAS type and configuration, provide a centrally managed policy for associated SAS tokens.
4. **Network rules:** Restrict which networks or endpoints can reach the account.
5. **Immutable retention/legal holds:** Restrict modification or deletion of protected data when supported and configured.

### Recommended configuration
- Use Entra ID and RBAC for broad role-based data access.
- Use ACLs for path-specific authorization on HNS-enabled accounts.
- Prefer user-delegation SAS where supported and appropriate rather than account-key-signed access.
- Keep SAS permissions, resources, and expiry as narrow as possible.
- Apply private endpoints/firewall controls when private access is required.
- Test both allowed and denied actions.

Not every access-policy mechanism is supported for every account type, endpoint, or SAS flavor. Verify the current compatibility matrix.

**Microsoft docs:** [ADLS access control](https://learn.microsoft.com/en-us/azure/storage/blobs/data-lake-storage-access-control), [Grant limited access with SAS](https://learn.microsoft.com/en-us/azure/storage/common/storage-sas-overview)

## Q33. How do you integrate ADLS Gen2 with Azure Stream Analytics?

Azure Stream Analytics can process streaming inputs and write supported outputs to ADLS Gen2 for durable event storage and downstream analytics, subject to the connector and account configuration supported by the service.

### Example architecture
```text
IoT / application events
          |
          v
      Event Hubs
          |
          v
 Azure Stream Analytics
          |
          v
       ADLS Gen2
          |
          v
 Databricks / Synapse / BI
```

### Implementation
1. Send events to a supported streaming input such as Event Hubs.
2. Create a Stream Analytics job and define input schema.
3. Configure time-windowed queries or filtering as required.
4. Configure an ADLS Gen2 output and identity/authentication.
5. Set output path patterns and file format.
6. Start with a controlled test stream and validate output files.
7. Monitor input/output events, late arrivals, errors, and job health.

For complex stateful processing, custom deduplication, and lakehouse transformations, Databricks Structured Streaming may be a better fit. Choose based on processing semantics, latency, supported connectors, and operational complexity.

**Microsoft docs:** [Stream Analytics outputs](https://learn.microsoft.com/en-us/azure/stream-analytics/stream-analytics-define-outputs)

## Q34. What is the role of SAS tokens in ADLS Gen2?

A Shared Access Signature (SAS) grants delegated, time-limited access to specified storage resources and operations. It can be useful when a consumer needs temporary access without receiving the account key or a full identity role.

### Secure SAS practices
- Grant only required permissions, such as read-only when possible.
- Scope to the smallest resource and operation set.
- Use short expiry periods.
- Require HTTPS.
- Prefer user-delegation SAS backed by Microsoft Entra ID where supported.
- Store tokens in a secret manager; never commit them to source control or print them in logs.
- Rotate/revoke access when possible and investigate unexpected use.

A SAS is a bearer credential: anyone who obtains it can use the permissions it grants until expiry or revocation conditions apply. SAS does not replace network controls, audit logging, or least privilege.

**Microsoft docs:** [SAS overview](https://learn.microsoft.com/en-us/azure/storage/common/storage-sas-overview)

## Q35. How do you manage object replication in ADLS Gen2?

Blob object replication is a feature for asynchronous replication of eligible blobs between source and destination accounts. It is not the same as storage-account geo-redundancy. Feature eligibility and compatibility with HNS-enabled accounts and the required blob features must be checked carefully before designing around it.

### Design steps
1. Define whether the goal is disaster recovery, regional analytics, distribution, or compliance.
2. Confirm object replication is supported for both account configurations and required features.
3. Configure source/destination accounts and authorization prerequisites.
4. Define replication rules and eligible containers/prefixes.
5. Monitor replication status and lag.
6. Test destination readability and recovery procedures.

### Distinguish these options
- **LRS/ZRS/GRS/GZRS:** Storage-service redundancy.
- **Object replication:** Asynchronous replication of eligible objects according to rules.
- **AzCopy/ADF copy:** Application-managed data copy.
- **Backup:** Recovery mechanism with retention and restore capabilities.

Do not assume object replication supports every ADLS Gen2 feature or every type of existing data. Validate the current limitations first.

**Microsoft docs:** [Object replication for block blobs](https://learn.microsoft.com/en-us/azure/storage/blobs/object-replication-overview)

## Q36. What are the differences between ADLS Gen2 and Amazon S3?

Both are scalable cloud object-storage foundations for data lakes. Their access models and ecosystem integrations differ.

| Area | ADLS Gen2 | Amazon S3 |
|---|---|---|
| Cloud | Microsoft Azure | AWS |
| Namespace | HNS optional at account creation/configuration | Object-key namespace; prefixes commonly represent folders |
| Fine-grained path permissions | HNS ACLs plus Azure RBAC | IAM, bucket policies, access points, and other AWS controls |
| Identity | Microsoft Entra ID, managed identities, SAS, keys | IAM roles/policies, temporary credentials, presigned URLs |
| Analytics ecosystem | ADF, Azure Databricks, Synapse, Fabric | Glue, EMR, Athena, Redshift, SageMaker |
| Redundancy | LRS/ZRS/GRS/GZRS variants, depending on support | S3 service durability and regional/replication options |
| Access tiers | Hot/Cool/Cold/Archive subject to configuration | S3 storage classes and lifecycle policies |
| Network controls | Firewalls, service endpoints, Private Link | VPC endpoints, bucket policies and related network controls |

### Interview summary
Choose based on cloud platform, analytics ecosystem, identity model, network architecture, regional needs, cost, and governance requirements. Avoid declaring one universally faster or cheaper without a workload-specific benchmark and current pricing analysis.

**Docs:** [ADLS Gen2](https://learn.microsoft.com/en-us/azure/storage/blobs/data-lake-storage-introduction), [Amazon S3](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html)

## Q37. How does ADLS Gen2 support data governance?

ADLS Gen2 provides storage-level controls, but enterprise governance usually spans the storage layer and separate catalog/governance services.

### Governance building blocks
- **Classification:** Identify personal, financial, clinical, safety, and confidential data.
- **Access control:** RBAC, ACLs, managed identities, and network isolation.
- **Catalog and lineage:** Use supported catalog/governance tooling to document owners, schemas, lineage, and usage.
- **Data quality:** Validate completeness, uniqueness, referential integrity, freshness, and business rules in processing pipelines.
- **Retention:** Lifecycle policies, approved retention schedules, and immutable storage where required.
- **Audit:** Diagnostic logs, identity events, access reviews, and evidence retention.
- **Environment separation:** Isolate dev/test/prod and sensitive domains.
- **Data contracts:** Document schema and semantic expectations for producers and consumers.

### Example
For pharma batch-quality data, raw source records may be restricted to a quality operations group. Curated batch traceability data can be exposed to approved analytics users, with lineage and quality checks retained for audit.

ADLS does not automatically discover every sensitive field or enforce every business policy. Governance processes and complementary services must be configured.

**Microsoft docs:** [Microsoft Purview documentation](https://learn.microsoft.com/en-us/purview/), [ADLS access control](https://learn.microsoft.com/en-us/azure/storage/blobs/data-lake-storage-access-control)

## Q38. What is a managed identity and how do you use it with ADLS Gen2?

A managed identity is an identity managed by Azure for a supported Azure resource. It allows a workload to authenticate to services such as storage without storing a password or client secret in application configuration.

### Types
- **System-assigned:** Lifecycle is tied to the Azure resource.
- **User-assigned:** Separate Azure resource that can be attached to multiple supported workloads.

### Setup outline
1. Enable or create a managed identity for the workload.
2. Assign an appropriate storage data role.
3. Add ACL permissions if path-level permissions are required.
4. Configure the client/service to use that identity.
5. Test the exact read/write operation.
6. Monitor access and review permissions.

### ADF example
Enable the ADF managed identity, grant it required access to the filesystem/path, then configure the ADLS Gen2 linked service to use the factory identity.

### Why it is preferred
- No embedded long-lived secret.
- Centralized permission management.
- Easier rotation and lifecycle management.
- Clearer audit trail than shared keys.

**Microsoft docs:** [Managed identities for Azure resources](https://learn.microsoft.com/en-us/entra/identity/managed-identities-azure-resources/overview), [ADF service identity](https://learn.microsoft.com/en-us/azure/data-factory/data-factory-service-identity)

## Q39. How do you monitor storage costs in ADLS Gen2?

### Cost-monitoring workflow
1. Use Azure Cost Management to break down cost by subscription, resource group, storage account, meter, and tag where available.
2. Compare actual usage with forecast and budget.
3. Inspect capacity, transactions, data retrieval, transfer, and redundancy costs.
4. Review growth from retained versions, snapshots, soft-deleted data, and duplicated copies where applicable.
5. Correlate storage spend with ADF, Databricks, Synapse, monitoring, and networking costs.
6. Configure budgets and alerts.
7. Test lifecycle and compaction changes in a controlled way.

### Typical cost surprises
- Millions of small files creating high request and compute overhead.
- Moving data to cooler tiers that is still read frequently.
- Keeping redundant historical copies indefinitely.
- Log ingestion into Log Analytics without appropriate retention or filtering.
- Repeated full scans when incremental processing was possible.
- Cross-region transfer or unnecessary duplicate pipelines.

Do not optimize only for cost per GB. Optimize total workload cost while meeting performance, recovery, and compliance objectives.

**Microsoft docs:** [Azure Cost Management](https://learn.microsoft.com/en-us/azure/cost-management-billing/costs/overview), [Blob Storage pricing](https://azure.microsoft.com/en-us/pricing/details/storage/blobs/)

## Q40. How do you create and manage file systems in ADLS Gen2?

A filesystem is the top-level container for data accessed through the Data Lake Storage endpoint. In many Azure tools it is equivalent to the Blob container concept.

### Portal workflow
1. Open the storage account.
2. Open **Data storage > Containers**.
3. Create a container/filesystem.
4. Configure access through RBAC and ACLs; do not rely on public access for private enterprise data.
5. Create directories and upload test files.
6. Validate access with the intended workload identity.

### Azure CLI illustration
```bash
az storage fs create \
  --name bronze \
  --account-name <storage-account> \
  --auth-mode login
```

List filesystems:
```bash
az storage fs list \
  --account-name <storage-account> \
  --auth-mode login \
  --output table
```

Create directories using supported Data Lake commands or SDKs. Exact CLI command groups and options can vary by version; check `az storage fs --help`.

### Operational guidance
Use consistent filesystem naming, isolate environments and sensitive domains, define retention, document ownership, and avoid granting broad access simply because a directory is convenient.

**Microsoft docs:** [Azure CLI storage filesystem commands](https://learn.microsoft.com/en-us/cli/azure/storage/fs)

## Q41. What are the key security features of ADLS Gen2?

A layered security design includes:

1. **Identity:** Microsoft Entra ID and managed identities.
2. **Authorization:** Azure RBAC and HNS ACLs.
3. **Secret control:** Avoid account keys in notebooks; store required secrets in Key Vault.
4. **Network:** Firewalls, selected networks, private endpoints, DNS, and secure routing.
5. **Encryption:** Encryption at rest and TLS in transit.
6. **Temporary delegation:** Narrow SAS tokens only when needed.
7. **Monitoring:** Diagnostic logs, Azure Activity Log, alerts, and access reviews.
8. **Governance:** Classification, retention, lineage, and separation of duties.
9. **Recovery:** Soft delete, supported versioning/protection features, backups, and tested DR.
10. **Immutability:** Where supported, use time-based retention or legal holds for records that must not be changed.

### Production rule
Security is not just “turn on encryption.” A correctly encrypted account can still expose sensitive data if identities have excessive permissions or the network is too permissive.

**Microsoft docs:** [Azure Storage security recommendations](https://learn.microsoft.com/en-us/azure/storage/blobs/security-recommendations)

## Q42. How do you enforce encryption at rest and in transit for ADLS Gen2?

### At rest
Azure Storage encrypts data at rest by default. If policy requires customer-managed keys, configure a supported CMK integration with Key Vault or Managed HSM, assign required identity permissions, and plan key rotation and recovery.

### In transit
- Require secure transfer for the storage account.
- Use HTTPS/TLS endpoints.
- Ensure applications and SDKs use secure protocols.
- Avoid transmitting SAS tokens or secrets through insecure channels.
- Review clients that might still attempt HTTP and test that requests are rejected as expected.

### Verification checklist
- Confirm account encryption settings.
- Confirm secure transfer is required.
- Confirm clients use HTTPS.
- Validate key access and rotation processes if CMK is used.
- Monitor policy compliance and configuration drift.

Encryption settings, identity controls, and network rules should be validated together.

**Microsoft docs:** [Require secure transfer](https://learn.microsoft.com/en-us/azure/storage/common/storage-require-secure-transfer), [Storage encryption](https://learn.microsoft.com/en-us/azure/storage/common/storage-service-encryption)

## Q43. How do you perform a data audit in ADLS Gen2?

An audit should answer: who accessed or changed data, what operation occurred, when it happened, whether it succeeded, and which identity/resource was involved.

### Audit approach
1. Enable relevant storage diagnostic resource logs.
2. Capture Azure Activity Log for control-plane changes such as configuration and role assignments.
3. Route logs to a protected destination such as Log Analytics or an archive storage account.
4. Restrict access to audit records and define retention.
5. Correlate storage requests with ADF pipeline runs, Databricks jobs, and identity sign-in records where available.
6. Alert on unusual deletes, access denials, permission changes, or unexpected access patterns.
7. Review audit evidence periodically and test retrieval.

### Data-quality audit trail
For pipelines, record source name, batch/run ID, ingestion timestamp, row/file counts, schema version, watermark, rejected record count, target table, and outcome. Storage request logs alone do not replace business-level lineage and data-quality metadata.

**Microsoft docs:** [Monitor Blob Storage](https://learn.microsoft.com/en-us/azure/storage/blobs/monitor-blob-storage), [Azure Activity Log](https://learn.microsoft.com/en-us/azure/azure-monitor/essentials/activity-log)

## Q44. How do you manage compliance requirements in ADLS Gen2?

Compliance is a system of controls and evidence, not a single storage setting. Requirements vary by industry, region, data type, and legal obligations.

### Practical framework
- Identify applicable standards and obligations with security/legal/compliance teams.
- Classify data and document ownership.
- Apply least-privilege RBAC/ACLs and separation of duties.
- Encrypt data and restrict network access.
- Define retention, deletion, and legal hold processes.
- Use immutable storage where the applicable scenario and feature support it.
- Enable audit logs and protect them from unauthorized changes.
- Document data lineage and approved processing purposes.
- Review access periodically and test recovery.
- Use governance/compliance tooling to map controls and collect evidence.

For health/pharma or financial data, avoid placing sensitive data in broadly accessible raw paths. Ensure retention and deletion decisions reflect applicable legal requirements; do not invent a universal retention duration.

**Microsoft docs:** [Microsoft Purview](https://learn.microsoft.com/en-us/purview/), [Azure Storage security recommendations](https://learn.microsoft.com/en-us/azure/storage/blobs/security-recommendations)

## Q45. How do you configure VNet service endpoints for ADLS Gen2?

A virtual network service endpoint extends a VNet subnet's identity to a supported Azure service over the Azure backbone. For private connectivity to a storage account, Azure Private Endpoint is often the alternative considered. Choose based on network requirements and current service guidance.

### Service endpoint approach
1. Confirm the storage account and network design support the required endpoint type.
2. Enable the appropriate storage service endpoint on the intended subnet.
3. Configure the storage account network rules to allow the selected VNet/subnet.
4. Verify routing, firewall policy, and client access.
5. Test from the intended subnet and confirm public/other-network access is denied as expected.

### Service endpoint vs Private Endpoint
- **Service endpoint:** Uses the storage service's public endpoint, with subnet-based network restriction.
- **Private Endpoint:** Gives the service a private IP in a VNet and uses private DNS/routing.

Private endpoints are often preferred when private IP-based connectivity and reduced public endpoint exposure are required. Neither network mechanism grants data authorization; RBAC/ACLs are still required.

**Microsoft docs:** [Virtual network service endpoints](https://learn.microsoft.com/en-us/azure/virtual-network/virtual-network-service-endpoints-overview), [Private endpoints for Storage](https://learn.microsoft.com/en-us/azure/storage/common/storage-private-endpoints)

## Q46. What are the benefits of using ADLS Gen2 for machine learning workloads?

ADLS Gen2 can store large, durable datasets and artifacts separately from compute. ML workflows can use the same governed storage foundation for training, validation, feature preparation, and model artifacts.

### Benefits
- Scalable storage for raw and curated training data.
- Integration with Azure Databricks and Azure Machine Learning.
- Centralized permissions and access governance.
- Support for structured, semi-structured, and unstructured data.
- Data versioning and lineage patterns when combined with appropriate tooling.
- Separate storage and compute lifecycles.

### Recommended design
1. Preserve raw source data.
2. Create reproducible curated training datasets.
3. Record data version, feature definitions, label window, and preprocessing code version.
4. Split train/validation/test data correctly to prevent leakage.
5. Use managed identities and least-privilege access.
6. Track experiment and model artifacts in suitable ML tooling.
7. Monitor drift and data-quality issues.

ADLS is not itself an experiment tracker or feature store. Combine it with the relevant ML platform and governance process.

**Microsoft docs:** [Azure Machine Learning](https://learn.microsoft.com/en-us/azure/machine-learning/), [ADLS Gen2](https://learn.microsoft.com/en-us/azure/storage/blobs/data-lake-storage-introduction)

## Q47. How do you optimize file access in ADLS Gen2?

### Optimize the access pattern
- Prefer targeted reads over repeated recursive listing of large paths.
- Use date/domain prefixes that align with processing and retention.
- Use columnar files and predicate/partition pruning for analytics.
- Compact tiny files and avoid excessively large single files.
- Avoid writing one file per row or one file per tiny Spark task.
- Use incremental ingestion and metadata tables to identify new data.
- Tune Spark partitions and parallelism based on actual file sizes and compute.
- Cache reusable datasets in compute only when it helps and memory cost is acceptable.
- Avoid unnecessary cross-region access.
- Monitor request rates, latency, throttling, and job-level file counts.

### Interview scenario
If a Spark job spends most of its time listing files before processing, reduce broad path discovery, improve folder layout, and maintain incremental ingestion metadata. If the job reads data quickly but spends time in shuffle, optimize Spark joins and partitioning rather than changing storage settings blindly.

**Microsoft docs:** [ADLS Gen2 performance best practices](https://learn.microsoft.com/en-us/azure/storage/blobs/data-lake-storage-best-practices)

## Q48. How do you handle large-scale ingestion into ADLS Gen2?

Large-scale ingestion requires parallelism, idempotency, controlled retries, validation, and observability.

### Reference architecture
```text
Databases / APIs / ERP / files / events
                 |
                 v
        ADF / AzCopy / Event Hubs
                 |
                 v
         ADLS Gen2 Bronze
                 |
       schema + count + DQ checks
                 |
          +------+------+
          |             |
          v             v
     Valid records   Quarantine
          |
          v
    Databricks / Delta
          |
          v
      Silver / Gold
```

### Engineering controls
- Use parallel copy only within source and destination capacity limits.
- Use incremental extraction with watermarks or CDC for databases.
- Use Auto Loader or a supported incremental file-discovery approach for new files.
- Use unique batch/run IDs and deterministic target paths.
- Make retries idempotent; avoid duplicate output after retry.
- Validate record counts, schema, checksums where feasible, and business totals.
- Quarantine invalid records and preserve error context.
- Track throughput, latency, lag, failure rate, and cost.
- Separate transient errors from data errors.
- Define replay and backfill procedures.

### Example
For a daily 200 GB source extract, land files into a run-specific Bronze path, validate completion and schema, then merge changed records into Delta using stable keys and source sequence/timestamp. Publish a completion record only after all required checks pass.

## Q49. What is the role of the Azure Data Lake Explorer tool?

“Azure Data Lake Explorer” may refer to different experiences, so clarify the tool intended. In many data-engineering workflows, users mean **Azure Storage Explorer**, the desktop tool for browsing and managing storage resources, or the data explorer within a portal/analytics product.

### Azure Storage Explorer capabilities
- Connect to Azure subscriptions and storage accounts.
- Browse filesystems, directories, and files.
- Upload and download files.
- Inspect paths and metadata.
- Troubleshoot missing or misplaced files.
- Use supported authentication methods such as Microsoft Entra ID or SAS.

### Troubleshooting example
An ADF pipeline reports that a source file is missing. Browse the expected filesystem/path, confirm exact casing and folder names, verify file size and arrival time, and compare the path with pipeline parameters. Then check identity permissions and network connectivity.

Use least-privilege access even for desktop tools. Avoid distributing account keys to every developer.

**Microsoft docs:** [Azure Storage Explorer](https://azure.microsoft.com/en-us/products/storage/storage-explorer)

## Q50. How do you configure cross-region replication in ADLS Gen2?

First clarify whether the requirement is storage-managed geo-redundancy, application-managed copying, or object replication. These approaches have different prerequisites and recovery semantics.

### Option A: Geo-redundant storage
- Select GRS/RA-GRS or GZRS/RA-GZRS where supported.
- Azure replicates data asynchronously to a secondary region.
- Define failover requirements and understand RPO implications.
- Test the end-to-end application recovery process.

### Option B: Application-managed copy
- Copy selected data to a separate destination account/region using supported ADF, AzCopy, SDK, or pipeline workflows.
- Track copy status and validate data completeness.
- Define deletion propagation, conflict handling, and retention.

### Option C: Object replication
- Use only when the account configuration and required features are supported.
- Configure source/destination accounts and replication rules.
- Monitor lag and validate destination data.

### Disaster-recovery checklist
1. Define RPO and RTO.
2. Identify critical datasets and dependent compute/services.
3. Verify region and feature support.
4. Document DNS, identity, network, and key dependencies.
5. Test recovery, including data consistency and application restart.
6. Monitor replication lag and periodically rehearse failover.

Geo-replication is asynchronous and does not guarantee zero data loss. A second copy is useful only if access, keys, networking, permissions, and consumers can also recover.

**Microsoft docs:** [Azure Storage redundancy](https://learn.microsoft.com/en-us/azure/storage/common/storage-redundancy), [Object replication](https://learn.microsoft.com/en-us/azure/storage/blobs/object-replication-overview)

---

## High-value production scenarios

### Scenario 1: ADF can connect, but a pipeline gets “AuthorizationPermissionMismatch”
Check which identity the linked service actually uses. Confirm the correct data-plane RBAC role, role scope, ACLs on the file and every parent directory, and any network restrictions. Do not solve the issue by giving the identity Owner or exposing account keys.

### Scenario 2: Databricks read succeeds but write fails
Check write permission on the target directory, ACL `w` and `x` permissions, RBAC, external-location grants if using Unity Catalog, and whether the destination path is correct. Inspect the precise error and test with the same compute identity.

### Scenario 3: A daily pipeline is suddenly slow
Compare input bytes and file counts, list operations, request latency/throttling, Spark stage metrics, shuffle spill, partition skew, concurrency, and network path. Identify whether the bottleneck is discovery, storage I/O, transformation, or output commit.

### Scenario 4: Destination contains fewer records than the source
Compare source and destination counts at each stage. Check filters, incremental watermark boundaries, duplicate handling, schema parsing, null-key behavior, rejected/quarantined rows, merge conditions, and retry/idempotency. Record counts and reconciliation totals should be stored per pipeline run.

### Scenario 5: A team needs access to one curated directory only
Use a security group, least-privilege data-plane role, and path-level ACLs as appropriate. Check broad inherited or account-level RBAC assignments that might bypass the intended ACL restriction. Test both the permitted path and a restricted path.

### Scenario 6: Storage costs grow even though data volume is stable
Investigate transaction counts, tiny files, retained versions/soft-deleted data, duplicate copies, log ingestion, lifecycle rules, cool-tier retrieval, and data transfer. Compare cost by meter and test a change before applying it broadly.

### Scenario 7: A region outage occurs
Confirm the configured redundancy option and replication status. Follow the tested recovery plan for compute, identity, network, DNS, secrets/keys, and downstream consumers. Do not assume geo-replicated storage automatically fails over the whole data platform with no data loss.

---

## Quick revision table

| Topic | Key interview point |
|---|---|
| ADLS Gen2 | Blob Storage foundation + hierarchical namespace |
| HNS | Real directory/file semantics and ACL support |
| RBAC vs ACL | RBAC for broad scoped roles; ACL for path-level access |
| Authentication | Prefer Microsoft Entra ID and managed identities |
| ADF | Linked service, identity, copy/ingestion, parameters, retries |
| Databricks | Prefer governed identity-based access and Unity Catalog |
| Synapse | Query or load lake data according to workload |
| Partitioning | Align with filters; avoid high cardinality and tiny files |
| Schema evolution | Govern changes; do not blindly enable automatic evolution |
| Monitoring | Metrics + resource logs + Activity Log + workload logs |
| Lifecycle | Tier/delete eligible data with retention and retrieval costs in mind |
| Redundancy | Choose by availability, RPO/RTO, region, and cost |
| Cross-region | Geo-redundancy is asynchronous; test full application recovery |
| Security | Identity + authorization + network + encryption + audit |
| Cost | Capacity + operations + retrieval + transfer + related services |

## Recommended Microsoft documentation

1. [ADLS Gen2 introduction](https://learn.microsoft.com/en-us/azure/storage/blobs/data-lake-storage-introduction)
2. [ADLS Gen2 access control](https://learn.microsoft.com/en-us/azure/storage/blobs/data-lake-storage-access-control)
3. [ADLS Gen2 best practices](https://learn.microsoft.com/en-us/azure/storage/blobs/data-lake-storage-best-practices)
4. [Authorize access to Azure Storage](https://learn.microsoft.com/en-us/azure/storage/common/authorize-data-access)
5. [Azure Storage redundancy](https://learn.microsoft.com/en-us/azure/storage/common/storage-redundancy)
6. [Azure Storage security recommendations](https://learn.microsoft.com/en-us/azure/storage/blobs/security-recommendations)
7. [Monitor Blob Storage](https://learn.microsoft.com/en-us/azure/storage/blobs/monitor-blob-storage)
8. [Azure Data Factory ADLS Gen2 connector](https://learn.microsoft.com/en-us/azure/data-factory/connector-azure-data-lake-storage)
9. [Azure Databricks storage access](https://learn.microsoft.com/en-us/azure/databricks/connect/storage/azure-storage)
10. [Azure Storage pricing](https://azure.microsoft.com/en-us/pricing/details/storage/blobs/)

---

**Final interview tip:** For every answer, connect the service feature to a production decision: identity and authorization, performance, reliability, cost, monitoring, and recovery. For Azure features that depend on HNS, region, account type, or API version, explicitly say you would verify current Microsoft support before committing to the design.
