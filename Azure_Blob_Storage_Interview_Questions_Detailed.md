# Azure Blob Storage Interview Questions — Detailed Guide

**Focus:** Microsoft Azure Data Engineering  
**Topics:** Blob fundamentals, access tiers, security, redundancy, ingestion, monitoring, APIs, event-driven processing, analytics, and ADLS Gen2.

> This guide is for interview preparation. Azure features, account compatibility, regional availability, and pricing can change, so verify implementation details against the linked Microsoft documentation before deploying to production.

## Table of Contents

1. [What is Azure Blob Storage?](#q1-what-is-azure-blob-storage)
2. [How do you use Blob Storage for unstructured data?](#q2-how-do-you-use-azure-blob-storage-for-unstructured-data)
3. [What blob types are supported?](#q3-what-blob-types-are-supported)
4. [Block blobs vs. append blobs vs. page blobs](#q4-block-blobs-vs-append-blobs-vs-page-blobs)
5. [Create a blob container](#q5-how-do-you-create-a-blob-container)
6. [Blob Storage vs. General Purpose v2 accounts](#q6-blob-storage-vs-general-purpose-v2-accounts)
7. [Blob snapshots](#q7-how-do-you-manage-blob-snapshots)
8. [Lifecycle policies](#q8-how-do-you-set-up-blob-lifecycle-policies)
9. [Blob versioning](#q9-how-do-you-configure-blob-versioning)
10. [SAS tokens](#q10-what-is-the-role-of-sas-tokens)
11. [Redundancy: LRS, ZRS, GRS](#q11-how-do-you-implement-data-redundancy-lrs-zrs-grs)
12. [Large file uploads](#q12-how-does-blob-storage-handle-large-file-uploads)
13. [Archive tier](#q13-what-is-the-blob-storage-archive-tier)
14. [Moving between access tiers](#q14-how-do-you-move-blobs-between-hot-cool-cold-and-archive-tiers)
15. [Azure Storage Explorer](#q15-how-do-you-access-blob-data-using-azure-storage-explorer)
16. [Encryption](#q16-how-do-you-manage-data-encryption)
17. [CDN and edge delivery](#q17-how-does-blob-storage-integrate-with-a-cdn)
18. [Pricing](#q18-what-is-the-pricing-structure)
19. [Large-scale ingestion](#q19-how-do-you-manage-large-scale-data-ingestion)
20. [Private endpoints and VNet connectivity](#q20-how-do-you-secure-blob-data-using-private-endpoints-and-vnet-connectivity)
21. [Performance and usage monitoring](#q21-how-do-you-monitor-performance-and-usage)
22. [Cost optimization](#q22-how-do-you-optimize-cost-management)
23. [REST API](#q23-what-is-the-blob-storage-rest-api)
24. [Event-based triggers](#q24-how-do-you-set-up-event-based-triggers)
25. [Soft delete](#q25-how-do-you-configure-blob-soft-delete)
26. [Immutable storage](#q26-what-are-immutable-blob-storage-policies)
27. [Automated data movement](#q27-how-do-you-automate-data-movement)
28. [Static website hosting](#q28-how-do-you-create-a-blob-storage-static-website)
29. [Azure Functions integration](#q29-how-do-you-integrate-blob-storage-with-azure-functions)
30. [Cross-region replication](#q30-how-do-you-perform-data-replication-across-regions)
31. [Power BI analytics](#q31-how-do-you-use-blob-storage-with-power-bi)
32. [Data pipeline use cases](#q32-what-are-common-use-cases-in-data-pipelines)
33. [Logging and monitoring](#q33-how-do-you-implement-logging-and-monitoring)
34. [Hierarchical namespace](#q34-what-is-the-impact-of-hierarchical-namespaces)

---

## Q1. What is Azure Blob Storage?

Azure Blob Storage is Microsoft's scalable object storage service for unstructured data, including CSV, JSON, XML, Parquet, images, videos, PDFs, logs, and binary files.

### Core components

- **Storage account:** Top-level Azure resource that provides storage endpoints, security settings, redundancy, and service configuration.
- **Container:** Logical grouping of blobs.
- **Blob:** The stored object itself.

A common data engineering architecture is:

```text
Source systems
    |
    v
Azure Data Factory (ingestion and orchestration)
    |
    v
Blob Storage / ADLS Gen2 (raw landing)
    |
    v
Azure Databricks (validation and transformation)
    |
    v
Delta Lake / curated datasets
    |
    v
Synapse / Fabric / Power BI / downstream consumers
```

### Key features

- Scalable object storage.
- Hot, Cool, Cold, and Archive access tiers, subject to account and blob-type support.
- Encryption at rest and secure network access options.
- Local, zone, and geo-redundancy options.
- Lifecycle management, versioning, soft delete, and immutable storage.
- Integration with ADF, Databricks, Event Grid, Functions, and analytics services.

**Interview answer:** I use Blob Storage as a scalable landing and storage layer for source files, API responses, logs, and analytical datasets. ADF can ingest data into the storage account, while Databricks transforms raw files into validated and curated Delta or Parquet datasets.

**Microsoft docs:** [Introduction to Azure Blob Storage](https://learn.microsoft.com/en-us/azure/storage/blobs/storage-blobs-introduction)

## Q2. How do you use Azure Blob Storage for unstructured data?

Unstructured data does not require a predefined relational table schema. Examples include images, videos, PDFs, audio, raw logs, and JSON or XML documents.

### Implementation

1. Create a StorageV2 storage account.
2. Create containers for data domains or processing stages.
3. Upload data through ADF, SDKs, REST APIs, Storage Explorer, or applications.
4. Organize objects with blob names and prefixes.
5. Configure authorization, encryption, redundancy, and retention.
6. Process files with Databricks, Azure Functions, or other consumers.

Example logical layout:

```text
healthcare-storage/
├── raw/
│   ├── patient/
│   ├── encounters/
│   └── medical-images/
├── validated/
├── curated/
└── archive/
```

In a flat-namespace account, names such as `raw/encounters/file.json` generally use a prefix convention rather than a real directory hierarchy. ADLS Gen2 with hierarchical namespace adds filesystem-style directory semantics.

Blob Storage stores objects; other applications interpret their content.

**Scenario:** A source API produces hundreds of gigabytes of JSON daily. ADF lands the files, and Databricks validates schemas, removes duplicates, standardizes fields, and writes curated datasets.

**Microsoft docs:** [Introduction to Azure Blob Storage](https://learn.microsoft.com/en-us/azure/storage/blobs/storage-blobs-introduction)

## Q3. What blob types are supported?

Azure Blob Storage supports three principal blob types:

| Type | Purpose | Examples |
|---|---|---|
| Block blob | General file and object storage | CSV, JSON, Parquet, images, videos |
| Append blob | Append-oriented writes | Certain application log patterns |
| Page blob | Random reads and writes at page offsets | Virtual hard disk workloads |

**Block blobs:** Data is uploaded in blocks and committed as a complete blob. This is the common choice for data files.

**Append blobs:** Optimized for adding blocks at the end. They can suit certain logging patterns, although Event Hubs or other streaming services are often better for high-volume event ingestion.

**Page blobs:** Support random reads and writes over page ranges and are associated with VHD scenarios.

For analytical data files such as Parquet and Delta Lake data files, block blobs are generally relevant. ADLS Gen2 is built on the Blob Storage platform.

**Microsoft docs:** [Blob Storage introduction](https://learn.microsoft.com/en-us/azure/storage/blobs/storage-blobs-introduction)

## Q4. Block blobs vs. append blobs vs. page blobs

| Feature | Block blob | Append blob | Page blob |
|---|---|---|---|
| Access pattern | Upload/replace objects | Append to end | Random reads/writes |
| Structure | Blocks | Append blocks | Fixed-size pages |
| Common use | Data files and media | Append-oriented logs | VHD and random-access workloads |
| Typical data engineering use | High | Specific cases | Limited |

**Scenario:** For 80 GB of daily retail data, store CSV or Parquet files as block blobs. Use append blobs only where the application genuinely needs append semantics. Use page blobs for workloads that require page-based random access.

Do not confuse *incremental ingestion* with appending to one physical blob. Incremental ingestion usually means processing only new or changed records/files.

## Q5. How do you create a blob container?

### Azure portal

1. Open the [Azure portal](https://portal.azure.com/).
2. Create or open a StorageV2 storage account.
3. Go to **Data storage → Containers**.
4. Select **+ Container**.
5. Enter a name, such as `raw-data`.
6. Set anonymous access to **Private** unless public access is explicitly required.
7. Create the container.

### Azure CLI example

```bash
az login

az storage account create \
  --name mydatalake123 \
  --resource-group my-resource-group \
  --location westeurope \
  --sku Standard_LRS \
  --kind StorageV2

az storage container create \
  --name raw-data \
  --account-name mydatalake123 \
  --auth-mode login
```

Replace the sample account and resource group names. Storage account names must be globally unique and use supported lowercase characters and numbers.

### Production considerations

- Prefer Microsoft Entra ID and managed identities over sharing account keys.
- Keep containers private by default.
- Apply least-privilege data-plane permissions.
- Use logical containers or prefixes for domains and processing stages.

**Microsoft docs:** [Create a blob container](https://learn.microsoft.com/en-us/azure/storage/blobs/blob-containers-portal)

## Q6. Blob Storage vs. General Purpose v2 accounts

Blob Storage is a storage service; **General Purpose v2 (GPv2) is a storage account type**. They are not direct alternatives.

| Feature | Legacy Blob Storage account | GPv2 StorageV2 account |
|---|---|---|
| Blob service | Supported | Supported |
| Azure Files | Not its general-purpose focus | Supported |
| Queue and Table services | Not its general-purpose focus | Supported |
| Broad current storage capabilities | More limited by account type | Broad general-purpose support |

For new general-purpose storage workloads, GPv2 is usually the account type to evaluate.

ADLS Gen2 is different: it builds on Blob Storage and adds a hierarchical namespace and filesystem-oriented operations for analytics workloads.

**Interview answer:** I generally choose GPv2 unless a specific workload requires a different account type. For a lakehouse requiring directory semantics and analytics integration, I evaluate ADLS Gen2.

**Microsoft docs:** [Azure Storage account overview](https://learn.microsoft.com/en-us/azure/storage/common/storage-account-overview)

## Q7. How do you manage blob snapshots?

A snapshot is a read-only point-in-time representation of a blob. It can help recover earlier contents after an accidental change.

### Example with Python

```python
from azure.identity import DefaultAzureCredential
from azure.storage.blob import BlobServiceClient

account_url = (
    "https://mydatalake123.blob.core.windows.net"
)

service = BlobServiceClient(
    account_url=account_url,
    credential=DefaultAzureCredential()
)

blob = service.get_blob_client(
    container="raw-data",
    blob="customer_master.csv"
)

snapshot = blob.create_snapshot()
print("Snapshot:", snapshot["snapshot"])
```

Install dependencies using `pip install azure-identity azure-storage-blob` and configure the identity with suitable data-plane permissions.

### Recovery and considerations

- The base blob can change while a snapshot remains associated with its earlier state.
- Recovery typically involves copying the snapshot contents back to the base blob or a recovery blob.
- Snapshots are not a complete storage-account backup.
- Changed blocks can consume additional capacity.
- Versioning and soft delete provide different recovery mechanisms.

| Feature | Purpose |
|---|---|
| Snapshot | Explicit point-in-time copy |
| Versioning | Automatically maintains earlier versions after changes |
| Soft delete | Retains deleted/overwritten data for a configured period |

**Microsoft docs:** [Blob snapshots overview](https://learn.microsoft.com/en-us/azure/storage/blobs/snapshots-overview)

## Q8. How do you set up blob lifecycle policies?

Lifecycle management automates supported actions such as changing access tiers and deleting eligible blobs after a defined period.

Example policy:

- First 30 days: Hot.
- After 30 days: Cool.
- After 90 days: Archive, if offline access is acceptable.
- Delete expired data only when retention requirements permit.

These are examples, not universal recommendations. The standard Archive tier has a 180-day minimum retention period; cooler online tiers also have minimum-duration and cost considerations.

### Portal steps

1. Open the storage account.
2. Go to **Data management → Lifecycle management**.
3. Add a rule and define blob-type and prefix filters.
4. Configure age-based tier transitions or deletion.
5. Save and monitor the policy.

### Illustrative policy fragment

```json
{
  "rules": [
    {
      "enabled": true,
      "name": "archive-old-raw-files",
      "type": "Lifecycle",
      "definition": {
        "filters": {
          "blobTypes": ["blockBlob"],
          "prefixMatch": ["raw-data/"]
        },
        "actions": {
          "baseBlob": {
            "tierToCool": {
              "daysAfterModificationGreaterThan": 30
            },
            "tierToArchive": {
              "daysAfterModificationGreaterThan": 90
            }
          }
        }
      }
    }
  ]
}
```

This is an illustrative fragment; configure and validate the complete policy against supported account features.

### Limitations

- Archive blobs must be rehydrated before they can be read online.
- Tiering restrictions apply to some blob types.
- Immutable retention policies can block deletion.
- Policy evaluation is asynchronous rather than immediate.

**Microsoft docs:** [Lifecycle management overview](https://learn.microsoft.com/en-us/azure/storage/blobs/lifecycle-management-overview)

## Q9. How do you configure blob versioning?

Versioning automatically retains previous versions of a blob after supported modifications.

### Portal steps

1. Open the storage account.
2. Select **Data protection**.
3. Enable **Blob versioning** if supported by the account configuration.
4. Save the setting.
5. Test recovery and establish a lifecycle policy for old versions if appropriate.

For example, if `sales.csv` is overwritten with new values, versioning can preserve an earlier version so it can be retrieved.

### Considerations

- Previous versions consume storage.
- Versioning is not a substitute for a tested backup strategy.
- Configure lifecycle management to remove old versions only when permitted.
- Check compatibility with hierarchical namespace and other enabled features before implementing it.

**Microsoft docs:** [Blob versioning overview](https://learn.microsoft.com/en-us/azure/storage/blobs/versioning-overview)

## Q10. What is the role of SAS tokens?

A Shared Access Signature (SAS) is a signed token that grants restricted access to Azure Storage resources.

A SAS can limit the resource, permitted operations, validity period, protocol, and—in supported configurations—source IP address.

### SAS types

- **User delegation SAS:** Signed using a Microsoft Entra ID user delegation key; generally preferred for Blob Storage when suitable.
- **Service SAS:** Grants access to specified service resources.
- **Account SAS:** Can delegate access across supported storage services and resource types.

### Example

An external partner needs to download one report for 15 minutes. Issue a read-only, HTTPS-only SAS for that blob rather than sharing the storage account key.

### Security practices

- Prefer managed identities for service-to-service authentication.
- Grant the minimum required permissions.
- Use short expiry times.
- Avoid embedding SAS tokens in source code or logs.
- Treat a SAS as a bearer credential: anyone possessing a valid token can use its granted permissions.

**Microsoft docs:** [SAS overview](https://learn.microsoft.com/en-us/azure/storage/common/storage-sas-overview)

## Q11. How do you implement redundancy (LRS, ZRS, GRS)?

Azure Storage redundancy maintains copies of data to protect against failures.

| Option | Protection |
|---|---|
| LRS | Multiple copies in one physical location in the primary region |
| ZRS | Synchronous replication across availability zones in the primary region, where supported |
| GRS | Local redundancy in the primary region plus asynchronous replication to a secondary region |
| RA-GRS | GRS plus read access to the secondary region |
| GZRS | Zone redundancy in the primary region plus geo-replication |
| RA-GZRS | GZRS plus read access to the secondary region |

### How to configure

1. Create a supported storage account.
2. Choose a redundancy option during creation or change it where supported.
3. Verify the selected configuration and regional availability.
4. For disaster recovery, test application recovery and data consistency.

GRS does not guarantee zero data loss: replication to the secondary region is asynchronous. Application failover, DNS, dependent services, and recovery procedures must be planned.

**Microsoft docs:** [Azure Storage redundancy](https://learn.microsoft.com/en-us/azure/storage/common/storage-redundancy)

## Q12. How does Blob Storage handle large file uploads?

Clients can upload block blobs in blocks, often concurrently, and commit the blocks into a complete blob.

For a 50 GB Parquet file, a robust upload design includes:

1. Divide the file into manageable blocks.
2. Upload with controlled concurrency.
3. Retry transient failures.
4. Commit the blocks.
5. Validate size and integrity where appropriate.

### Supported tools

- Azure Storage SDKs.
- AzCopy.
- Azure Storage Explorer.
- Azure Data Factory Copy Activity.
- Blob REST APIs.

### Performance practices

- Tune block size and concurrency.
- Avoid overloading the source, destination, network, or integration runtime.
- Use retries with backoff.
- Monitor throughput, latency, throttling, and failed requests.
- Validate data after transfer.

If ADF frequently fails while transferring 100 GB files, investigate integration runtime capacity, network stability, source limitations, parallelism, and retry configuration.

**Microsoft docs:** [Upload blobs](https://learn.microsoft.com/en-us/azure/storage/blobs/storage-blob-upload)

## Q13. What is the Archive tier?

Archive is an offline tier for rarely accessed data that must be retained at low capacity cost.

Typical uses include regulatory retention, historical extracts, old logs, and long-term backups.

| Property | Hot | Cool | Cold | Archive |
|---|---|---|---|---|
| Access | Frequent | Infrequent | Rare | Very rare |
| Online access | Yes | Yes | Yes | No |
| Retrieval latency | Milliseconds | Milliseconds | Milliseconds | Typically hours |
| Storage capacity cost | Higher | Lower | Lower | Lowest |

Archived blobs must be rehydrated to an online tier before reading. Standard-priority rehydration can take many hours; higher-priority options may cost more and depend on service conditions.

In the standard GPv2 tier model, Archive has a 180-day minimum retention period. Early deletion, overwriting, or tier changes can incur charges.

**Microsoft docs:** [Blob access tiers](https://learn.microsoft.com/en-us/azure/storage/blobs/access-tiers-overview)

## Q14. How do you move blobs between Hot, Cool, Cold, and Archive tiers?

Tier changes can be made manually or automated through lifecycle management.

### Azure CLI examples

```bash
# Move a block blob to Cool
az storage blob set-tier \
  --account-name mydatalake123 \
  --container-name raw-data \
  --name sales.csv \
  --tier Cool \
  --auth-mode login

# Archive the same blob
az storage blob set-tier \
  --account-name mydatalake123 \
  --container-name raw-data \
  --name sales.csv \
  --tier Archive \
  --auth-mode login
```

### Moving data out of Archive

Archive is offline. Rehydrate the blob to an online tier using a supported operation, or copy it to an online destination. Retrieval may take hours.

For historical audit data, Archive may be suitable if access can be delayed. If Databricks must read the data nightly, keep it online.

**Microsoft docs:** [Manage access tiers](https://learn.microsoft.com/en-us/azure/storage/blobs/access-tiers-online-manage)

## Q15. How do you access blob data using Azure Storage Explorer?

Azure Storage Explorer is a desktop application for browsing and managing Azure Storage.

### Steps

1. Install [Azure Storage Explorer](https://azure.microsoft.com/en-us/products/storage/storage-explorer).
2. Sign in with Microsoft Entra ID.
3. Select the subscription and storage account.
4. Expand **Blob Containers**.
5. Select a container.
6. Upload, download, browse, or inspect blobs according to your permissions.

Supported authentication methods include Microsoft Entra ID, account keys/connection strings, and SAS-based access.

**Troubleshooting example:** If ADF reports that a file is missing, use Storage Explorer to confirm the container, blob path, file size, and whether the expected file arrived.

**Microsoft docs:** [Manage storage with Storage Explorer](https://learn.microsoft.com/en-us/azure/vs-azure-tools-storage-manage-with-storage-explorer)

## Q16. How do you manage data encryption?

Azure Storage encrypts data at rest by default. Additional options cover customer-managed keys and encryption in transit.

| Option | Explanation |
|---|---|
| Microsoft-managed keys | Azure manages encryption keys |
| Customer-managed keys | You manage keys using supported Key Vault or Managed HSM configurations |
| Infrastructure encryption | Additional encryption layer in supported configurations |
| HTTPS/TLS | Protects data while in transit |

### Customer-managed key setup

1. Configure an appropriate Key Vault key.
2. Configure the required managed identity and permissions.
3. Open the storage account's encryption settings.
4. Select the supported customer-managed key configuration.
5. Validate access and recovery procedures.

Encryption does not replace authorization. Combine encryption with RBAC, network restrictions, auditing, and secret management.

**Microsoft docs:** [Azure Storage encryption at rest](https://learn.microsoft.com/en-us/azure/storage/common/storage-service-encryption)

## Q17. How does Blob Storage integrate with a CDN?

A content delivery network caches content at distributed edge locations so users can retrieve it from a nearby location rather than repeatedly fetching it from the origin.

Typical use cases include images, video, downloadable files, and static website assets.

```text
Blob Storage (origin)
        |
        v
Azure Front Door / supported edge delivery
        |
        v
Users around the world
```

### Implementation

1. Store static content in Blob Storage.
2. Configure Azure Front Door or a suitable supported edge delivery service.
3. Set caching rules and HTTPS.
4. Restrict origin access as appropriate.
5. Test cache updates and invalidation.

Azure's CDN offerings and migration paths have changed over time. For new designs, check current service and SKU availability rather than assuming a legacy CDN offering is available.

**Microsoft docs:** [Azure Front Door](https://learn.microsoft.com/en-us/azure/frontdoor/)

## Q18. What is the pricing structure?

Blob Storage costs depend on several factors:

| Cost component | Main driver |
|---|---|
| Capacity | Data stored and tier |
| Transactions | Read, write, list, and other operations |
| Retrieval | Reads from certain cooler tiers |
| Data transfer | Outbound and some cross-region transfers |
| Redundancy | Selected replication option |
| Data protection | Retained versions, snapshots, and soft-deleted data |
| Related services | Monitoring, Private Link, edge delivery, and other services |

### Cost estimation process

1. Estimate monthly data volume by tier.
2. Estimate read/write/list operations.
3. Estimate retrieval and outbound data.
4. Include redundancy and data protection.
5. Include associated processing and monitoring costs.
6. Use the [Azure Pricing Calculator](https://azure.microsoft.com/en-us/pricing/calculator/) and current regional rates.

Keeping frequently processed data in Hot and older data in cooler tiers may reduce costs. But retrieval, transitions, early-deletion charges, and compute can offset savings.

**Official pricing:** [Azure Blob Storage pricing](https://azure.microsoft.com/en-us/pricing/details/storage/blobs/)

## Q19. How do you manage large-scale data ingestion?

Large-scale ingestion needs orchestration, controlled parallelism, incremental processing, validation, and monitoring.

```text
Sources (databases, APIs, files, events)
                 |
                 v
Azure Data Factory
(metadata-driven ingestion and parallel copy)
                 |
                 v
Blob Storage / ADLS Gen2 — Raw
                 |
                 v
Databricks — validation and transformation
                 |
                 v
Curated Delta datasets
```

### Best practices

- Use ADF Copy Activity for supported batch ingestion.
- Use CDC and incremental loads instead of repeatedly reloading complete datasets.
- Partition by useful attributes such as business date or source system.
- Avoid excessive small files.
- Tune parallelism to source capacity, network, integration runtime, and destination throughput.
- Implement retries, idempotency, quarantine, and ingestion manifests.
- Record source counts, destination counts, batch IDs, and watermarks.
- Use Event Hubs or a suitable streaming service for sustained event ingestion.

**Scenario:** For 80 GB of daily retail data, ADF ingests tables and files into partitioned raw storage. Databricks validates and transforms the data. Control tables track watermarks and batch status.

**Microsoft docs:** [ADF Blob Storage connector](https://learn.microsoft.com/en-us/azure/data-factory/connector-azure-blob-storage)

## Q20. How do you secure Blob Storage with private endpoints and VNet connectivity?

A private endpoint uses Azure Private Link to provide a private IP address from a virtual network for a supported Storage service.

### Setup

1. Select a virtual network and subnet.
2. Create a private endpoint for the Blob service.
3. Configure private DNS so the storage hostname resolves to the private IP from the intended network.
4. Ensure the workload has network connectivity to the endpoint.
5. Restrict or disable public network access as required.
6. Verify that only approved workloads can connect.

A private endpoint is not the same as VNet integration. The endpoint provides private access to a resource; VNet integration enables supported application services to connect from a virtual network. ADF also supports managed virtual networks and managed private endpoints in applicable configurations.

### Troubleshooting checklist

- Is the private endpoint approved?
- Does DNS resolve to the expected private IP?
- Does the workload have network connectivity?
- Are required Blob and DFS endpoints configured for the workload?
- Does the identity have the correct data-plane permissions?

**Microsoft docs:** [Private endpoints for Azure Storage](https://learn.microsoft.com/en-us/azure/storage/common/storage-private-endpoints)

## Q21. How do you monitor performance and usage?

Azure Monitor provides platform metrics and diagnostic capabilities for Storage accounts and Blob services.

| Signal | Purpose |
|---|---|
| Availability | Service-level request success |
| Transactions | Storage operation volume |
| Success/failure responses | Request outcomes |
| Latency | Request duration |
| Ingress/egress | Data transferred |
| Capacity | Stored data volume |
| Throttling | Requests encountering service limits |

### Setup

1. Open the storage account and select **Monitoring → Metrics**.
2. Configure **Diagnostic settings** to send supported logs to Log Analytics, Storage, or Event Hubs.
3. Create alerts for relevant failure rates, latency, or availability conditions.
4. Use Azure dashboards and workbooks to visualize trends.
5. Correlate storage diagnostics with ADF and Databricks job logs.

If an ADF copy activity slows down, compare storage latency and request volume with source response time, integration runtime capacity, and pipeline concurrency.

**Microsoft docs:** [Blob service metrics](https://learn.microsoft.com/en-us/azure/azure-monitor/reference/supported-metrics/microsoft-storage-storageaccounts-blobservices-metrics)

## Q22. How do you optimize cost?

1. Keep frequently accessed datasets in Hot.
2. Move eligible older data to Cool or Cold.
3. Use Archive only when offline retrieval is acceptable.
4. Apply lifecycle policies.
5. Set sensible retention periods for versions and snapshots.
6. Use Parquet or another suitable format when it improves analytical efficiency.
7. Avoid unnecessary scans, list operations, and redundant copies.
8. Use Azure Cost Management, budgets, and tags to track spending.

Evaluate total cost of ownership: capacity + transactions + retrieval + data transfer + retained versions + compute. The cheapest storage tier is not necessarily the cheapest end-to-end design.

**Microsoft docs:** [Blob access tiers](https://learn.microsoft.com/en-us/azure/storage/blobs/access-tiers-overview)

## Q23. What is the Blob Storage REST API?

The REST API lets applications interact with Blob Storage using HTTP requests.

| Operation | Purpose |
|---|---|
| Put Blob | Create a blob or upload content |
| Get Blob | Download contents |
| Delete Blob | Delete a blob |
| List Blobs | List objects |
| Set Blob Metadata | Set metadata |
| Set Blob Tier | Change access tier |
| Get Blob Properties | Retrieve properties such as size and ETag |

Example request:

```http
GET https://mydatalake123.blob.core.windows.net/raw-data/sales.csv
```

A real request must include valid authentication and authorization, such as Microsoft Entra ID OAuth headers, Shared Key authorization, or a valid SAS. Required API-version headers depend on the operation.

Prefer Microsoft Entra ID and managed identities where supported instead of distributing account keys.

**Microsoft docs:** [Blob Service REST API](https://learn.microsoft.com/en-us/rest/api/storageservices/blob-service-rest-api)

## Q24. How do you set up event-based triggers?

Event Grid can route Blob Storage events to Azure Functions, webhooks, and other supported event handlers.

```text
Blob uploaded
     |
     v
Azure Event Grid
     |
     v
Azure Function
     |
     v
ADF / Databricks processing
```

### Implementation

1. Create a storage account and container.
2. Create an Event Grid subscription for the required event types.
3. Configure the destination.
4. Filter by container or blob prefix where appropriate.
5. Implement the event handler.
6. Monitor failures and configure retry/dead-letter handling as appropriate.

Events can be delivered more than once. Make the handler idempotent, verify that the file is ready, and maintain a durable processing ledger. For large files, orchestrate processing rather than processing the entire dataset synchronously inside the event handler.

**Microsoft docs:** [Blob Storage events](https://learn.microsoft.com/en-us/azure/event-grid/event-schema-blob-storage) · [Azure Functions and Blob events](https://learn.microsoft.com/en-us/azure/azure-functions/scenario-blob-storage-events)

## Q25. How do you configure blob soft delete?

Soft delete retains deleted or overwritten data for a configured recovery period.

### Setup

1. Open the storage account.
2. Select **Data protection**.
3. Enable blob soft delete.
4. Configure a retention period.
5. Enable container soft delete as appropriate.
6. Test recovery.

Soft-deleted data remains recoverable only during the applicable retention window and consumes storage. Soft delete alone does not protect an entire deleted storage account.

Combine soft delete with suitable versioning, access controls, and tested recovery procedures. Verify compatibility with the account's features.

**Microsoft docs:** [Soft delete for blobs](https://learn.microsoft.com/en-us/azure/storage/blobs/soft-delete-blob-overview)

## Q26. What are immutable blob storage policies?

Immutable storage implements a Write Once, Read Many (WORM) model. It can prevent modifications or deletions that violate a configured retention restriction.

### Main policy types

- **Time-based retention:** Protects data for a defined retention period.
- **Legal hold:** Protects data until an authorized party clears the hold.

### Example

A financial organization retains audit files in an immutable container so ordinary pipeline identities cannot delete protected records before the required retention period ends.

### Implementation outline

1. Identify the data subject to retention requirements.
2. Configure a supported immutability policy at the appropriate scope.
3. Set retention duration or legal hold.
4. Apply policy-locking controls where required.
5. Test write/delete behavior.
6. Audit access and retention.

A locked policy can be difficult or impossible to reverse during the protected period. Validate it with compliance and security teams before enforcement.

**Microsoft docs:** [Immutable storage overview](https://learn.microsoft.com/en-us/azure/storage/blobs/immutable-storage-overview)

## Q27. How do you automate data movement?

Choose the service based on the task.

| Requirement | Suitable option |
|---|---|
| Scheduled source ingestion | Azure Data Factory |
| Large-scale file copy | AzCopy |
| Programmatic operations | Storage SDK |
| Event-driven processing | Event Grid and Azure Functions |
| Tier transitions and expiration | Lifecycle management |
| Distributed transformations | Azure Databricks |

### ADF process

1. Create linked services for source and destination.
2. Create datasets.
3. Configure Copy Activity and file filters.
4. Add validation and failure handling.
5. Configure a schedule or event trigger.
6. Monitor runs and record ingestion metadata.

If files move from `incoming/` to `processed/`, validate the copy and track each file or batch. Retrying a pipeline should not create duplicate downstream records.

Lifecycle management handles supported tier transitions and deletion; use a copy tool or pipeline for moving objects between locations or accounts.

**Microsoft docs:** [ADF Blob Storage connector](https://learn.microsoft.com/en-us/azure/data-factory/connector-azure-blob-storage)

## Q28. How do you create a static website?

Blob Storage can host static HTML, CSS, JavaScript, and image assets.

### Steps

1. Open a supported storage account.
2. Select **Data management → Static website**.
3. Enable static website hosting.
4. Specify `index.html` and optionally an error document.
5. Save the settings.
6. Upload files to the special `$web` container.
7. Test the generated website endpoint.

Example layout:

```text
$web/
├── index.html
├── 404.html
├── css/styles.css
├── js/app.js
└── images/logo.png
```

Static hosting does not execute server-side code. Use Azure Functions or another backend service for dynamic APIs. The static website endpoint is intended for website delivery, not as a private document repository.

**Microsoft docs:** [Static website hosting](https://learn.microsoft.com/en-us/azure/storage/blobs/storage-blob-static-website)

## Q29. How do you integrate Blob Storage with Azure Functions?

Azure Functions runs code in response to events, HTTP requests, timers, and other triggers. Blob integration can automate file validation and routing.

### Common patterns

- Blob trigger.
- Blob input binding.
- Blob output binding.
- Event Grid trigger.

### Illustrative Python Blob-triggered function

```python
import logging
import azure.functions as func

app = func.FunctionApp()

@app.blob_trigger(
    arg_name="input_blob",
    path="incoming/{name}",
    connection="AzureWebJobsStorage"
)
def process_blob(input_blob: func.InputStream):
    logging.info(
        "File: %s, size: %s bytes",
        input_blob.name,
        input_blob.length
    )
```

The Function App needs the correct runtime, deployment dependencies, and storage configuration. The example's connection setting must be configured appropriately; for production, managed identity can be used where supported and configured.

### Production practices

- Design for duplicate delivery and retries.
- Validate file format and metadata.
- Quarantine malformed files.
- Avoid long-running synchronous processing in event handlers.
- Use ADF or Databricks to orchestrate larger workloads.
- Monitor using Azure Monitor and Application Insights.

**Microsoft docs:** [Process Blob Storage events with Azure Functions](https://learn.microsoft.com/en-us/azure/azure-functions/scenario-blob-storage-events)

## Q30. How do you replicate data across regions?

Geo-redundant storage options replicate data to a secondary region.

- **GRS:** Local redundancy in the primary region plus asynchronous replication.
- **RA-GRS:** GRS with secondary read access.
- **GZRS:** Zone redundancy in the primary region plus geo-replication.
- **RA-GZRS:** GZRS with secondary read access.

### Disaster recovery plan

1. Select a supported redundancy option.
2. Monitor replication status where available.
3. Define the application recovery procedure.
4. Test dependent services, credentials, networking, and compute in the recovery region.
5. Validate data consistency before resuming processing.

Geo-replication is asynchronous, so recent writes may not yet exist in the secondary. Storage redundancy is not a complete backup of all application configuration and dependencies.

**Microsoft docs:** [Azure Storage redundancy](https://learn.microsoft.com/en-us/azure/storage/common/storage-redundancy)

## Q31. How do you use Blob Storage with Power BI?

Power BI can consume supported files from Blob Storage or ADLS Gen2 through appropriate connectors and authentication methods. ADLS Gen2 is often useful for analytics because it supports hierarchical namespaces.

```text
ADF / Databricks
       |
       v
Blob Storage / ADLS Gen2
       |
       v
Power BI connector / Power Query
       |
       v
Semantic model and reports
```

### Implementation

1. Write curated CSV, Parquet, or other supported datasets to storage.
2. Organize and partition data appropriately.
3. In Power BI Desktop, select the relevant Azure Storage connector.
4. Authenticate and select files or data.
5. Transform and model the data.
6. Publish the model and configure refresh or another suitable access pattern.

Power BI does not automatically treat every arbitrary file as a relational table. Large workloads may benefit from a warehouse or curated serving layer rather than direct scans of raw files.

**Microsoft docs:** [Analyze ADLS Gen2 data with Power BI](https://learn.microsoft.com/en-us/power-query/connectors/analyze-data-in-adls-gen2)

## Q32. What are common Blob Storage data pipeline use cases?

| Use case | Role of Blob Storage |
|---|---|
| Raw landing | Stores source extracts and API responses |
| Data lake | Holds raw and curated datasets |
| Batch ingestion | Receives CSV, JSON, XML, and Parquet files |
| Streaming output | Stores files produced by streaming services |
| Backup and archive | Retains historical data |
| Machine learning | Stores training data and artifacts |
| Logging | Holds exported logs and diagnostics |
| Static websites | Hosts website assets |
| Documents and media | Stores PDFs, images, audio, and video |
| Data exchange | Supports controlled file transfers |

### Example Azure data platform

For a manufacturing or pharma platform:

- ADF ingests production, quality, distribution, and safety data.
- Blob Storage or ADLS Gen2 stores raw source files.
- Databricks validates, standardizes, deduplicates, and transforms records.
- Delta tables hold curated datasets.
- Synapse, Fabric, or Power BI supports analytical consumption.

For lakehouse workloads, evaluate ADLS Gen2 and Delta Lake rather than treating object storage as a complete analytical database.

## Q33. How do you implement logging and monitoring?

Use Azure Monitor metrics, diagnostic logs, alerts, application telemetry, and pipeline-level audit records.

### Implementation

1. Review storage service metrics.
2. Configure diagnostic settings for supported log categories.
3. Send logs to Log Analytics, Storage, or Event Hubs as appropriate.
4. Create alerts for failures, latency, availability, or unusual usage.
5. Correlate events with ADF run IDs, Databricks job IDs, batch IDs, timestamps, and source filenames.
6. Track data-quality metrics separately.

Example audit record:

| Field | Example |
|---|---|
| Pipeline | `pl_ingest_sales` |
| Batch ID | `BATCH_20261018_001` |
| Source | SQL Server |
| Destination | `raw-data/sales/` |
| Expected records | 1,000,000 |
| Loaded records | 1,000,000 |
| Status | Success |
| Duration | 12 minutes |

The values are illustrative. Populate production audit records from actual runs and validation results.

Storage diagnostics do not replace data-quality checks for record counts, duplicates, schema changes, rejected records, and business rules.

**Microsoft docs:** [Blob service metrics](https://learn.microsoft.com/en-us/azure/azure-monitor/reference/supported-metrics/microsoft-storage-storageaccounts-blobservices-metrics) · [Azure Monitor diagnostic settings](https://learn.microsoft.com/en-us/azure/azure-monitor/data-collection/diagnostic-settings)

## Q34. What is the impact of hierarchical namespaces?

Hierarchical namespace (HNS) provides directory and filesystem-style semantics and is the defining storage feature behind ADLS Gen2.

In a flat-namespace account, a name such as `raw/sales/2026/data.parquet` is generally a blob name containing slashes. With HNS, directories have filesystem semantics and supported directory operations are available.

| Flat namespace | Hierarchical namespace |
|---|---|
| Folder-like names use prefixes | Supports real directory semantics |
| Directory-like operations may require object-level work | Supports filesystem operations and atomic directory manipulation for supported operations |
| General object storage | Particularly useful for analytics and data lake workloads |

### Benefits

- Efficient supported directory rename/move operations.
- Filesystem-style organization.
- Analytics integration.
- Directory and file access-control options.
- Potentially lower end-to-end processing time for directory-heavy workloads.

### Considerations

- Feature compatibility differs by account configuration.
- Some Blob Storage features, including versioning, have compatibility constraints with HNS.
- Enabling HNS on an existing account is a significant decision; it cannot simply be reverted to a flat namespace.
- Validate migration, access control, cost, and integration requirements before choosing.

For a Databricks lakehouse with Bronze, Silver, and Gold datasets, ADLS Gen2 with HNS is usually worth evaluating. For a simple repository of images or backups, a flat-namespace Blob Storage account may be sufficient.

**Microsoft docs:** [Hierarchical namespace in Azure Data Lake Storage](https://learn.microsoft.com/en-us/azure/storage/blobs/data-lake-storage-namespace)

---

## Final interview revision: important distinctions

| Concept | Key distinction |
|---|---|
| Blob Storage vs. ADLS Gen2 | ADLS Gen2 builds on Blob Storage and adds hierarchical namespace capabilities |
| Block vs. append vs. page blobs | General objects vs. append-oriented writes vs. random-access pages |
| Snapshot vs. versioning | Explicit point-in-time copy vs. automatic version history |
| Soft delete vs. immutability | Recovery from deletion vs. enforced retention restrictions |
| Hot/Cool/Cold/Archive | Balance capacity cost, access cost, retrieval needs, and retention |
| LRS/ZRS/GRS | Local redundancy vs. zone resilience vs. geo-replication |
| SAS vs. managed identity | Delegated time-limited token vs. application identity |
| Private endpoint vs. public endpoint | Private network connectivity vs. public service endpoint access |
| Event Grid vs. ADF schedule | Event-driven notification vs. scheduled orchestration |
| Azure Monitor vs. data-quality checks | Service health vs. correctness of the dataset |

## Recommended Microsoft documentation

- [Azure Blob Storage documentation](https://learn.microsoft.com/en-us/azure/storage/blobs/)
- [Azure Storage security guide](https://learn.microsoft.com/en-us/azure/storage/common/storage-security-guide)
- [Azure Blob Storage pricing](https://azure.microsoft.com/en-us/pricing/details/storage/blobs/)
- [Azure Data Factory Blob Storage connector](https://learn.microsoft.com/en-us/azure/data-factory/connector-azure-blob-storage)
- [Azure Storage redundancy](https://learn.microsoft.com/en-us/azure/storage/common/storage-redundancy)
