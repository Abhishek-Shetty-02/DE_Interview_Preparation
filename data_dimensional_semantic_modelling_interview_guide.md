# Data Modelling, Dimensional Modelling & Semantic Modelling — Complete Interview Question Guide

**50 questions with detailed answers:** fundamentals, advanced design, production scenarios, practical exercises, and architecture/project discussion.

**Focus:** Designing trustworthy data structures from source systems through analytical models and governed business consumption.

**Audience:** Data engineering, analytics engineering, BI, Azure Databricks, Microsoft Fabric, Synapse, Snowflake, and enterprise data platform interviews.

**How to use this guide:** The three modelling disciplines are covered separately first, then connected through practical and end-to-end scenarios. Each answer has two detailed paragraphs: the first explains the concept and design decision; the second describes implementation, validation, trade-offs, or evidence. Examples are illustrative. Adapt them to your actual source systems, business definitions, platform capabilities, and project experience.

## Contents

- A. Data modelling — D01–D12
- B. Dimensional modelling — DM01–DM18
- C. Semantic modelling — SM01–SM12
- D. Integrated production scenarios — S01–S05
- E. Practical implementation exercises — P01–P03
- F. Architecture and project discussion — A01–A03
- Revision checklist, self-assessment, and references

## End-to-end relationship between the three disciplines

```text
Business requirements + source systems
              |
              v
      DATA MODELLING
Entities, attributes, relationships, keys, constraints
              |
              v
  DIMENSIONAL MODELLING
Facts, dimensions, grain, surrogate keys, SCD history
              |
              v
     SEMANTIC MODELLING
Business measures, relationships, hierarchies, security
              |
              v
Reports / dashboards / self-service analytics / APIs
```

The disciplines are related but not interchangeable. Data modelling establishes how data entities and relationships are represented. Dimensional modelling shapes analytical data around measurable business events and descriptive context. Semantic modelling exposes consistent business concepts, calculations, relationships, and access rules to consumers. A good semantic layer cannot compensate for ambiguous fact grain, and a well-designed star schema cannot compensate for conflicting metric definitions.

---

# A. Data modelling — 12 questions

## D01. What is data modelling, and why is it important?

Data modelling is the process of representing business concepts, their attributes, relationships, and rules in a structured form. It translates business requirements into a design that engineers can implement and consumers can understand. A model helps clarify what a customer, account, order, product, or transaction means; which identifiers distinguish records; how entities relate; and which rules make a record valid. Without an explicit model, the same concept can be represented differently across source systems and downstream datasets.

In a production project, begin with business questions, source profiling, terminology, data ownership, and expected use cases. Define entities, attributes, primary and foreign keys, optionality, cardinality, and business constraints, then validate the model with source owners and consumers. Keep a record of assumptions and unresolved definitions. Evidence of a good model includes traceable source-to-target mappings, consistent keys, documented constraints, fewer duplicated definitions, and downstream datasets that answer the intended questions without hidden interpretation.

## D02. What are conceptual, logical, and physical data models?

A conceptual model communicates the major business entities and their relationships without committing to implementation details. A logical model adds attributes, identifiers, relationships, cardinality, and business rules while remaining largely independent of a particular database engine. A physical model translates the logical design into platform-specific structures such as tables, columns, data types, indexes, partitions, constraints, storage layouts, and naming conventions. These are different levels of detail for the same problem, not competing modelling techniques.

For a banking domain, the conceptual model may show Customer, Account, Transaction, and Branch. The logical model defines identifiers, transaction amount, currency, posting timestamp, account ownership relationships, and cardinality. The physical model decides whether the tables are Delta tables, relational tables, or warehouse tables and specifies data types, distribution or partition strategy, and supported constraints. Review each level with its audience: business stakeholders validate meaning, architects validate structure, and engineers validate implementability and workload performance.

## D03. What is an entity, an attribute, and a relationship?

An entity represents a distinguishable business object or event, such as Customer, Product, Account, or Transaction. An attribute describes a property of that entity, such as customer status, product name, account currency, or transaction amount. A relationship states how entities are associated, such as a customer owning accounts or an order containing order lines. An entity is not necessarily identical to a source table: one source table may mix several business concepts, while one business entity may be assembled from multiple source tables.

When modelling an order process, Order and OrderLine should usually be distinct because one order can contain multiple lines, and line-level quantities and prices have a different meaning from order-level shipping status. Document relationship cardinality and optionality: one-to-many, one-to-one, or many-to-many; mandatory or optional. Validate the model against real source examples, including missing references, duplicate identifiers, and records that violate expected cardinality. Those cases often reveal hidden business rules or source-system exceptions.

## D04. What are primary keys, foreign keys, candidate keys, and natural keys?

A candidate key is an attribute or attribute combination that can uniquely identify an entity. A primary key is the candidate key selected as the main identifier. A foreign key expresses a relationship to a key in another entity. A natural key comes from the business or source domain, such as an account number or product code, while a surrogate key is an identifier generated for modelling and integration purposes. A natural key may change, be reused, have inconsistent formatting, or be unique only within a source system.

In a multi-source platform, use a source-qualified business key when source identifiers can overlap, for example `(source_system, customer_id)`, and consider a warehouse surrogate key for stable downstream relationships. Check uniqueness, nullability, key stability, and source-specific scope before choosing a key. A surrogate key does not remove the need to preserve business identifiers and source lineage. Keep both where required so records can be traced to source and relationships remain stable when source codes change.

## D05. How do you model one-to-one, one-to-many, and many-to-many relationships?

A one-to-one relationship means one record in an entity corresponds to at most one record in another entity under the stated rules. A one-to-many relationship means one parent may have multiple children, such as one customer having multiple accounts. A many-to-many relationship means records on both sides can relate to multiple records on the other side, such as students and courses or products and promotions. In a relational design, a many-to-many relationship is normally represented with an associative or bridge entity.

For a product-promotion model, a `product_promotion` bridge can hold product key, promotion key, effective dates, and optional allocation percentage. The bridge should define whether repeated pairs are allowed across time and how overlapping dates are handled. Validate referential integrity, duplicate pairs, and allocation rules. In analytical models, a bridge can introduce row multiplication, so downstream measure logic must define whether to count distinct entities, allocate amounts, or intentionally allow the many-to-many effect.

## D06. What is normalization? Explain 1NF, 2NF, and 3NF.

Normalization organizes relational data to reduce unnecessary duplication and prevent update, insertion, and deletion anomalies. First normal form generally requires atomic values and no repeating groups. Second normal form requires that non-key attributes depend on the whole candidate key, rather than only part of a composite key. Third normal form removes inappropriate transitive dependencies so non-key attributes depend on the key rather than on another non-key attribute. The exact design still depends on the business rules and the constraints of the target system.

For example, storing a customer address repeatedly on every transaction can cause inconsistent updates. Separating customer and transaction entities avoids repeating the customer attributes on each transaction. Normalize operational structures when updates and integrity matter, but do not mechanically normalize every analytical serving model: reporting often benefits from deliberate denormalization and dimensional structures. The important interview point is to explain the anomaly being prevented and why the chosen model suits the workload, not merely recite definitions.

## D07. When would you use a normalized model versus a denormalized model?

A normalized model is useful when entities need independent ownership, frequent updates, controlled redundancy, and clear transactional integrity. A denormalized model deliberately combines or repeats attributes to simplify reads and reduce joins for a known workload. Denormalization can improve analytical usability, but it creates responsibilities for synchronization, update propagation, and consistent definitions. Neither is universally superior; the correct choice follows the workload and the system’s correctness requirements.

For a transaction-processing application, separate Customer, Account, and Transaction structures may be appropriate. For a gold analytical dataset, a flattened customer segment and product category may make recurring analysis simpler, provided those attributes have clear historical semantics. Document the authoritative source for each repeated attribute, the refresh behavior, and what happens when it changes. Compare query complexity, update anomalies, storage cost, latency, and consumer comprehension before selecting the pattern.

## D08. What is a data contract, and how does it relate to a data model?

A data contract defines the expectations between a producer and its consumers. It can include schema, field meaning, keys, nullability, allowed values, freshness, ownership, classification, quality rules, and compatibility policy. A data model describes structure and relationships; the contract makes those expectations explicit and operational. A schema alone is not a complete contract because a field can retain the same type while its meaning or business interpretation changes.

For a transaction feed, define `transaction_id`, source system, account identifier, amount, currency, posting time, event type, and source sequence, together with uniqueness and late-arrival rules. Validate incoming data against the contract and classify changes as compatible, conditionally compatible, or breaking. On a breaking change, quarantine or hold the affected feed, alert the owner, and version or approve the migration before downstream publication. Capture contract version and run ID to make the result reproducible.

## D09. How do you handle data types, precision, time zones, and nullability in a model?

Choose data types based on business meaning and supported range, not just on how a source happens to serialize a value. Monetary values generally need an appropriate fixed-precision decimal rather than floating-point arithmetic; dates and timestamps need explicit time-zone and event-time semantics; identifiers should not be numeric merely because they contain digits. Nullability must distinguish unknown, not applicable, missing, and invalid values where the business requires those distinctions.

For a financial transaction, document amount precision and scale, currency, signed-amount convention, event timestamp, posting date, and timezone conversion policy. Test boundary values, negative amounts, high precision, daylight-saving transitions, and missing fields. Do not silently convert an invalid value to zero or interpret an unknown date as a valid default. Validate conversion counts and rejected records, and preserve the original representation when it is needed for audit or investigation.

## D10. How do you model history and changing attributes?

History modelling records how an entity changes over time. The required pattern depends on whether the consumer needs only the current value, an audit trail of changes, or the ability to reconstruct what was true at a past point. Common options include overwriting a current attribute, retaining an append-only change log, or storing effective-dated versions. Distinguish business-effective time from the time a platform received or processed the change.

For a customer segment change, capture the business key, effective start/end, current-row indicator where appropriate, source update timestamp or sequence, and processing metadata. Define how late and out-of-order changes are applied, how overlapping intervals are prevented, and how downstream facts are corrected. Test an ordinary update, a late correction, a duplicate event, and a replay. A history design is incomplete if it can store versions but cannot explain which version applies to a business event.

## D11. What is master data management, and how do you model a customer across multiple systems?

Master data management (MDM) addresses the consistent identification and governance of core entities such as customers, products, suppliers, and locations across systems. A single business entity can have multiple source identifiers and conflicting attributes. Modelling must distinguish source records, identity matching, survivorship rules, and the governed or golden representation. A master record should not erase provenance or pretend that uncertain matches are certain.

For a customer appearing in CRM, core banking, and digital channels, preserve source-system keys and matching evidence, then apply reviewed matching and survivorship rules to create a cross-reference or master identifier. Track confidence, merge/split decisions, effective dates, and source precedence where appropriate. Validate false matches as well as missed matches, since incorrectly combining two people can be more harmful than retaining two records pending review. Secure personally identifiable data and expose only the identifiers and attributes required by each consumer.

## D12. How do you validate a data model before implementation?

Validation checks that the model captures the required business concepts, supports the required questions, preserves source meaning, and can be implemented within performance and governance constraints. Review entity definitions, grain, keys, cardinality, optionality, history, data types, constraints, ownership, and sensitive-data handling. Walk through representative records and business scenarios instead of approving only an attractive diagram.

Create model acceptance checks such as unique-key tests, relationship orphan tests, valid domain tests, required-field tests, and reconciliation to source totals. Ask consumers to demonstrate that the proposed model answers the actual use cases without ambiguous joins or duplicated measures. Record decisions, rejected alternatives, assumptions, and change history. A model is accepted when its business meaning and expected behavior are agreed and testable—not merely when tables have been created successfully.

---

# B. Dimensional modelling — 18 questions

## DM01. What is dimensional modelling, and why is it used in analytics?

Dimensional modelling organizes analytical data around business processes, measurable facts, and descriptive dimensions. Facts represent events or periodic measurements; dimensions provide the context used to filter, group, and interpret those measurements. The pattern is designed to make business questions easier to express and to reduce ambiguity in common reporting. A star schema is a common implementation, although snowflaked dimensions and bridges may be useful for particular requirements.

For retail sales, a sales fact can store quantity, net sales, discount, and cost at a declared transaction-line grain, while Date, Product, Customer, Store, and Channel dimensions provide descriptive context. Before writing ETL, define what one fact row represents, how measures aggregate, and which dimensions apply. Validate totals against the source process and test queries at multiple grains. The main benefit is consistent analytical interpretation, not simply fewer joins.

## DM02. What is fact-table grain, and why must it be declared first?

Grain defines exactly what one row in a fact table represents. Examples include one transaction line, one payment event, one account balance per account per day, or one shipment item. Measures, keys, dimensions, and load logic must all agree with that grain. Mixing grains in one table can duplicate measures, make aggregation invalid, and produce results that look plausible but are wrong.

For a transaction-line fact, define whether returns and reversals are separate events, whether one source line can be corrected, and how duplicates are identified. Then choose the keys and measures that belong at that level. Validate grain with uniqueness checks over the declared business key and reconcile counts and amounts to the source. If requirements need both transaction-level and daily snapshot analysis, model separate fact tables instead of forcing unlike records into one structure.

## DM03. What are transaction, periodic snapshot, and accumulating snapshot fact tables?

A transaction fact captures individual business events at the moment they occur, such as a payment, order line, or claim event. A periodic snapshot captures the state or activity of a defined subject at regular intervals, such as daily account balance or monthly inventory. An accumulating snapshot represents a process that progresses through milestones, such as order placed, shipped, delivered, and closed; the same row is updated as milestones occur.

For banking, transaction events and daily account balances should generally be separate fact tables because their grains and aggregation rules differ. Transaction amounts can be summed across valid event rows under a defined sign convention, while daily balances should not be summed across dates to report an ending balance. For an accumulating snapshot, define how reopened or cancelled processes behave and whether milestone timestamps can be corrected. Test each fact type against its intended business questions and lifecycle.

## DM04. What is the difference between a fact table and a dimension table?

A fact table stores business events or measurements at a defined grain, typically with foreign keys to dimensions and numeric measures. A dimension table stores descriptive attributes used to filter, group, label, and segment facts. Facts are often large and frequently appended; dimensions are usually smaller and may require history management. These are common patterns, not absolute physical-size rules.

In a sales model, `fact_sales` might contain date key, product key, store key, quantity, sales amount, and discount amount. `dim_product` might contain product code, brand, category, and lifecycle attributes. Keep descriptive attributes in dimensions rather than repeatedly embedding them in every fact row, unless a deliberate design calls for a degenerate dimension or a specific snapshot attribute. Verify that each fact key resolves to the correct dimension row for the event’s business time.

## DM05. What is a star schema versus a snowflake schema?

A star schema connects a central fact table directly to denormalized dimensions. A snowflake schema normalizes some dimension attributes into related sub-dimensions, such as Product → Subcategory → Category. Star schemas are often easier for BI consumers and semantic models; snowflaking can reduce repeated attributes or represent independently governed hierarchies, but introduces more joins and complexity.

For a product dimension with a small stable category hierarchy, a denormalized star dimension may be the clearest choice. If categories are separately governed, reused, or require independent history, a snowflaked design may be justified. Evaluate query behavior, semantic-model support, hierarchy handling, and maintenance complexity. Do not claim that one pattern is always faster; test representative queries and keep the consumer-facing relationships understandable.

## DM06. What are conformed dimensions?

A conformed dimension has consistent definitions, keys, and attribute meanings across multiple fact tables or business processes. Conformance allows reports to compare metrics across processes using the same analytical context. A Date, Product, Customer, Region, or Currency dimension may be conformed when its definitions and key semantics are shared. Similar column names alone do not make dimensions conformed.

For an enterprise retail platform, Sales, Inventory, and Returns can share conformed Product, Store, and Date dimensions. Ensure that product identity, store hierarchy, date roles, and history behavior agree across facts. Document any intentional differences, such as a sales channel classification that is not equivalent to an inventory location. Test cross-process reports using the same dimension filters and verify that all facts resolve to the correct conformed members.

## DM07. What are surrogate keys, and why are they useful in dimensional models?

A surrogate key is a warehouse-generated identifier used to identify a dimension row independently of the source-system business key. It supports stable fact relationships, multi-source integration, and multiple historical versions of the same business entity. A source business key is still needed for matching, lineage, and change detection. Surrogate keys must be generated and managed consistently across loads.

For a slowly changing customer dimension, a single customer business key can have several historical rows, each with a different surrogate key and effective interval. A transaction fact resolves the customer surrogate key that was valid for the transaction’s business time. Define special members for unknown or not-yet-arrived dimension records where the architecture requires them. Test duplicate source keys, source-system key collisions, late-arriving dimensions, and retry behavior so facts do not receive inconsistent surrogate keys.

## DM08. Explain Slowly Changing Dimensions Type 1, Type 2, and Type 3.

Type 1 overwrites an attribute and does not retain its previous value in the dimension row. Type 2 creates a new version row with a new surrogate key and effective dates, preserving historical context. Type 3 stores a limited previous value or alternative attribute in additional columns; it is useful only when the required history is deliberately limited. These types can be applied selectively by attribute rather than requiring one universal policy for an entire dimension.

For customer address, a Type 2 design is appropriate if historical sales must report the address or region valid at transaction time. A spelling correction may be Type 1 if prior spelling is not analytically relevant. Define `effective_start`, `effective_end`, and current-row logic, and prevent overlapping intervals. When a late correction arrives, reconstruct affected history and decide whether facts must be re-keyed. Validate both current-state queries and as-of historical queries.

## DM09. How do you implement a Type 2 dimension load?

First identify the business key, compare incoming attributes with the current dimension version, and separate new entities from changed entities and unchanged rows. For a change that requires history, close the existing current row and insert a new version with a new surrogate key and the correct effective interval. Source sequence and effective time must determine precedence; arrival order alone may be incorrect when events arrive late.

Use a staging set that has one authoritative change per business key and deterministic tie-breaking. Validate that the dimension has no more than one current row per business key and that historical intervals do not overlap. Check insert/update counts, preserve source lineage, and test replaying the same batch. Avoid using an unreviewed current timestamp as the only effective-time rule if the source provides a more authoritative business-effective timestamp.

## DM10. What are additive, semi-additive, and non-additive measures?

Additive measures can be summed across all relevant dimensions, such as sales quantity or transaction amount when event and sign semantics are valid. Semi-additive measures can be summed across some dimensions but not others; account balance can often be summed across accounts for a given date but should not be summed across dates. Non-additive measures, such as ratios, percentages, and many rates, should generally be recomputed from their components rather than summed or averaged blindly.

For gross margin percentage, store or calculate revenue and margin components and define the measure as total margin divided by total revenue for the selected filter context. Do not average row-level percentages unless that is the intended weighting. For inventory or account balance snapshots, define whether the report wants end-of-period, average, minimum, or another time aggregation. Validate calculations using small hand-checkable examples and totals at different groupings.

## DM11. How do you handle late-arriving facts and late-arriving dimensions?

A late-arriving fact is an event received after its business event time or expected processing window. A late-arriving dimension is descriptive information that arrives after a related fact needs to be loaded. The model must distinguish event time from processing time, retain source ordering where available, and define whether the system can temporarily use an unknown member or must hold the fact until dimension data arrives.

For an account transaction received before the account master record, the pipeline can assign an approved unknown member and later repair the fact’s dimension key, or delay publication until the account is resolved. Choose based on reporting latency and correctness needs. Preserve unresolved counts and make the temporary status visible. Test late arrival, replay, correction, and duplicate delivery; ensure that a repair does not double-count the fact or rewrite unrelated history.

## DM12. What are degenerate dimensions, role-playing dimensions, and junk dimensions?

A degenerate dimension is a business identifier stored directly in the fact table, such as an invoice number, when a separate dimension with useful descriptive attributes is unnecessary. A role-playing dimension is one physical dimension used in multiple logical roles, such as Order Date, Ship Date, and Delivery Date referencing the Date dimension. A junk dimension combines low-cardinality flags and indicators that would otherwise create many small dimensions.

For a shipment fact, order date key and delivery date key can both reference the same Date dimension but must be exposed as clearly named roles. Keep invoice number on the fact if it is useful for drill-through but has no meaningful dimension attributes. Group small indicators into a junk dimension only when the combination remains manageable and understandable. Validate that semantic relationships use the intended date role and that measures are not accidentally filtered by the wrong date.

## DM13. How do you model many-to-many relationships in a dimensional model?

A many-to-many relationship occurs when a fact or entity can be associated with multiple dimension members and vice versa. A bridge table can represent the associations, often with effective dates and allocation weights. The design must state whether the analytical result should duplicate measures across associations, count distinct entities, or allocate measures so totals remain conserved. Without an explicit rule, joining a fact to a bridge can multiply rows and inflate sums.

For a sales fact associated with several promotions, a promotion bridge can connect the transaction or line to applicable promotions. If revenue must be attributed across promotions, store or calculate allocation weights whose sum follows a defined rule. If the report only asks which promotions touched a sale, distinct counts may be more appropriate. Test totals before and after bridge joins and document the intended aggregation behavior in the semantic layer.

## DM14. How do you design a date dimension and handle fiscal calendars?

A date dimension stores reusable calendar attributes such as date, day, week, month, quarter, year, weekday, holiday, and business-day flags. Fiscal calendars may have non-calendar fiscal years, 4-4-5 periods, custom week definitions, or regional holidays. A dedicated dimension makes these rules consistent and prevents every report from independently calculating fiscal periods.

Generate the calendar across the full reporting horizon and validate leap years, week boundaries, fiscal-year transitions, and regional variations. Keep the date key convention consistent and use explicit date roles in facts. If business calendars vary by entity or country, model the calendar identifier or appropriate attributes rather than assuming one global fiscal calendar. Confirm which date—event, posting, settlement, or snapshot—is used for each measure.

## DM15. How do you model balances and transactions for banking analytics?

Transactions represent events such as debit, credit, fee, payment, reversal, and adjustment. Daily balances represent the state of an account at a point in time and usually need a separate periodic snapshot fact. The transaction model should define signed amount conventions, transaction status, posting date, event date, currency, reversal linkage, and business key. The balance model should define whether it represents ledger balance, available balance, closing balance, or another specific measure.

Use `fact_transaction` at transaction-event grain and `fact_daily_balance` at account-by-date-by-balance-type grain, with appropriate dimensions such as Account, Customer, Date, Currency, Product, and Branch. Do not derive every balance by summing all transaction rows unless opening balances, pending items, reversals, and cutoff rules are fully accounted for. Reconcile transaction movements and ending balances to authoritative control totals by date and currency. Keep the two fact grains separate and expose distinct measure semantics to consumers.

## DM16. How do you model events, reversals, corrections, and duplicate facts?

First distinguish a duplicate delivery from a legitimate repeated business event. Define the business event identifier, source system, version or sequence, event type, correction linkage, and the authoritative precedence rule. Reversals are often new business events that offset prior events; they should not automatically be deleted as duplicates. Corrections may supersede a previous version or require an adjustment record depending on source semantics and audit rules.

Stage and validate incoming records, resolve duplicates deterministically, and apply only the latest authoritative version where the source contract says versions supersede one another. Preserve rejected or superseded records for traceability. Test duplicate messages, conflicting ties, reversal pairs, late corrections, and repeated processing. Reconcile row counts and signed amounts before and after transformation so a deduplication rule cannot silently remove valid business activity.

## DM17. What is a factless fact table, and when is it useful?

A factless fact table records that an event or relationship occurred without requiring a numeric measure. It can represent attendance, eligibility, coverage, promotion participation, or a customer’s presence at a service point. Counts can be derived by counting rows or distinct business keys, but the grain still needs to be declared so repeated events are handled correctly.

For an eligibility model, one row may represent one customer-product-eligibility-period combination. Define whether multiple assessments within a period create separate rows and whether the table records eligibility status at a point in time or an event. If a numeric measure is needed later, it can be introduced in a separate fact or as an appropriate event attribute. Validate counts against the source eligibility process and test how expired or overlapping eligibility intervals are represented.

## DM18. How do you optimize a dimensional model for a lakehouse or cloud warehouse?

Begin with the grain, access patterns, cardinality, volume, and required freshness. Use appropriate data types, stable keys, efficient joins, and physical layout features supported by the platform. Partitioning, clustering, file compaction, statistics, caching, and materialization are platform-specific optimization choices; they should be measured against representative workloads rather than applied automatically. A logical star schema and physical storage layout solve different problems.

In a Delta lakehouse, consider write frequency, incremental update patterns, small-file creation, query filters, and supported layout optimizations. In a cloud warehouse, assess distribution, clustering, indexes, materialized views, or other engine-specific options as appropriate. Compare query plans, scan volume, shuffle, concurrency, refresh duration, and cost before and after a change. Preserve correctness and access controls, and document the workload under which the optimization is beneficial.

---

# C. Semantic modelling — 12 questions

## SM01. What is semantic modelling, and how is it different from dimensional modelling?

Semantic modelling defines how business users and analytical tools interpret data: business entities, relationships, measures, dimensions, hierarchies, labels, formats, and security behavior. Dimensional modelling structures facts and dimensions for analysis; semantic modelling builds a consistent consumption layer on top of one or more datasets. A semantic model can use a star schema, but its job is not merely to mirror physical table names.

For a sales model, the dimensional layer may provide sales facts and Product, Date, and Store dimensions. The semantic layer defines measures such as Net Sales, Gross Margin %, and Distinct Customers, names them in business language, specifies formats and hierarchies, and controls which relationships filter which measures. Validate that two reports using the same measure return consistent results under the same filter context. A semantic model is successful when consumers can answer questions without recreating business logic in every report.

## SM02. What is a measure, a calculated column, and a calculated table?

A measure is evaluated in the current query or filter context and is commonly used for aggregations such as sales, margin, counts, and ratios. A calculated column is evaluated for each row and stores or exposes a row-level expression according to the engine’s behavior. A calculated table creates a table from an expression or transformation. These options differ in evaluation context, storage, refresh behavior, and performance, so they should not be chosen interchangeably.

For example, `Net Sales` is usually a measure because it should aggregate over the selected dates, stores, and products. A stable row-level classification may be a source/warehouse column or calculated column when appropriate, while a reusable date table may be generated upstream or by the semantic engine. Prefer transformations upstream when they are shared, auditable, or expensive to repeat, and use semantic calculations for consumer-facing business logic. Test refresh behavior and results under multiple filter combinations.

## SM03. Explain filter context and row context in Power BI/DAX-style semantic modelling.

Row context describes the current row when an expression is evaluated row by row, such as in a calculated column or an iterator. Filter context is the set of filters applied by the report, slicers, relationships, and expression. Measures are evaluated in filter context; iterators can create row context and evaluate an expression for each row. Understanding the distinction is essential because a correct-looking formula can change behavior when placed in a visual or used inside another calculation.

A measure such as total sales typically sums the fact amount over the current filter context. An iterator such as `SUMX` evaluates an expression for each row of a table and then aggregates the results, which may differ from summing a stored column when row-level logic is involved. Explain what filters are active and how relationships propagate them before debugging a measure. Use a small dataset with known totals to verify totals, subtotals, and cross-filter behavior.

## SM04. What is the difference between an explicit measure and an implicit aggregation?

An explicit measure is a named calculation deliberately defined in the semantic model. An implicit aggregation is generated by the tool when a numeric column is placed in a visual and aggregated automatically. Explicit measures make business definitions reusable, reviewable, documented, and testable; implicit aggregations can be convenient for exploration but may create inconsistent logic across reports.

For enterprise reporting, define named measures for Net Sales, Average Balance, Distinct Customers, and Margin Percentage rather than letting each report author choose a sum or average independently. Hide raw columns from consumer-facing field lists when direct aggregation could be misleading, and provide descriptions, display folders, formats, and certification where supported. Compare measure output across reports and validate that totals follow the approved business definition.

## SM05. How do relationships and filter direction affect semantic model results?

Relationships tell the semantic engine how tables are connected and how filters propagate. Cardinality, active/inactive relationships, and filter direction affect which rows participate in a calculation. Incorrect relationships can cause duplicated totals, ambiguous paths, unexpected blanks, or filters that do not reach the intended fact table. Relationship design must follow model grain and key uniqueness rather than being adjusted only until a visual looks correct.

In a typical star schema, dimensions filter facts through one-to-many relationships from unique dimension keys to fact foreign keys. Avoid unnecessary bidirectional filtering because it can create ambiguity and performance issues; use it only when the business behavior and engine semantics justify it. Test a dimension filter against each relevant measure, inspect unmatched keys, and verify totals with and without slicers. If two date roles exist, make the intended active or explicitly invoked role clear.

## SM06. How do you define and govern business metrics in a semantic layer?

A metric needs a business definition, formula, grain, included/excluded statuses, time basis, currency basis where relevant, owner, refresh expectation, and known limitations. Metric governance ensures that terms such as Active Customer, Revenue, Churn, or Average Balance mean the same thing across reports. A field name is not enough: two teams may use the same word with different populations or timing rules.

Create a metric catalogue and map each metric to approved fact columns and dimensions. For example, define whether Revenue includes reversals, taxes, pending transactions, or foreign-exchange conversions, and which date determines reporting period. Version changes to definitions and communicate their impact on dashboards. Validate the metric against source or finance-approved control totals and include test cases for boundary conditions. Where definitions differ by jurisdiction or business line, name and document the variants rather than hiding them.

## SM07. How do you model multiple date relationships in a semantic model?

A fact may have several meaningful dates, such as Order Date, Ship Date, Settlement Date, and Posting Date. A single Date dimension can be connected to multiple date columns, but many semantic engines allow only one active relationship between the same pair of tables at a time. The design must make the intended date role explicit in the measure or expose separate role-playing date dimensions where that is clearer for consumers.

For a transaction fact, reporting by Posting Date should not accidentally use Transaction Initiation Date. Name date relationships clearly and define which role each default measure uses. Test the same measure by each date role and compare expected counts and totals. If separate date dimensions are used, ensure they remain conformed in calendar definitions and that consumers understand why two date slicers may behave independently.

## SM08. What are hierarchies, drill-down paths, and perspectives?

A hierarchy defines a sequence of related levels for navigation, such as Year → Quarter → Month → Date or Country → Region → Branch. Drill-down allows users to move from summary to detail along that path. Perspectives or curated views, where supported, expose a focused subset of a larger semantic model for a particular audience. These features improve usability but do not replace correct keys, relationships, or metric definitions.

Build hierarchies that match how the business naturally analyzes data and confirm that each level has a stable relationship to the next. A geographic hierarchy should not mix incompatible parent-child structures. Use friendly names, sort columns where needed, hide technical keys, and group related measures. Test drill-down at leaf levels and check whether subtotals match the intended aggregation. Avoid overloading a model with many redundant hierarchies that create confusion.

## SM09. How do row-level security and object-level security differ?

Row-level security (RLS) restricts which rows a user can see based on role or identity rules. Object-level security (OLS), where supported, restricts visibility of model objects such as tables or columns. Neither should be confused with source-system permissions, workspace access, or the security of an exported file. Security must be designed end to end and tested using actual user identities and all supported access paths.

For a regional banking report, RLS may restrict a regional analyst to assigned branches, while sensitive account identifiers may need object-level restrictions or exclusion from the consumer model. Define the identity-to-entitlement mapping, default-deny behavior, and rules for users with multiple roles. Test allowed and denied rows, drill-through, exports, underlying data access, and alternate connections. Do not assume report-level filters alone protect the underlying dataset or every downstream consumer.

## SM10. What are import, DirectQuery/live connection, and composite semantic model patterns?

Import-style models store a refreshed copy of data in the semantic engine, often supporting responsive interactive queries at the cost of refresh and storage considerations. DirectQuery or live patterns query an underlying source or governed model, trading some local materialization for dependency on source performance, availability, and supported functionality. Composite patterns combine modes or sources where the platform permits them. Exact names, limits, and behavior depend on the product and configuration.

Choose a mode based on freshness, data size, concurrency, source capacity, modelling features, security, and user experience. A finance dashboard with controlled refresh windows may benefit from import, while a near-real-time operational view may require a different approach. Benchmark realistic visuals and concurrency, test source outages and refresh failures, and document freshness expectations. Avoid promising real-time results simply because the connection is live; the source itself may be delayed or cached.

## SM11. How do you optimize a semantic model?

Start with model shape and query behavior: reduce unnecessary columns, use sensible data types, keep dimension keys unique, remove unused objects, and avoid ambiguous relationships. Optimize expensive measures and iterators, reduce high-cardinality text where appropriate, and review visual-level query patterns. Storage mode, engine, relationships, calculated columns, and measure complexity all influence performance; one universal optimization rule does not fit every model.

Capture baseline query duration, refresh time, model size, memory pressure, and representative concurrent usage. Test changes with the same queries and filter context, then confirm values remain unchanged. Use platform-supported performance analyzers, query diagnostics, and engine-specific tools to locate the actual bottleneck. Do not hide slow measures by pre-aggregating without checking whether the aggregate preserves required filter behavior and detail-level questions.

## SM12. How do you test, deploy, and govern semantic models in production?

Semantic-model testing should cover schema compatibility, relationships, measure values, filter behavior, security, refresh, and performance. Deployment should promote reviewed artifacts through environments, preserve environment-specific connections and parameters, and maintain a rollback or correction path. Governance includes ownership, documentation, certified or endorsed status where supported, lineage, access review, metric versioning, and change communication.

Build a regression suite with known datasets and expected values for key measures, totals, time roles, security roles, and edge cases. Use CI/CD or the platform’s supported deployment workflow to promote the model, then validate the production connection and refresh under the intended identity. Monitor refresh failures, query latency, stale data, and security changes. A successful deployment is not only an artifact being published: it is a correct, accessible, refreshed, and accepted business model.

---

# D. Integrated production scenarios — 5 scenarios

## S01. A report total is twice the finance-approved total. How do you investigate?

First verify the measure definition, source control total, fact grain, and filter context. Then inspect join cardinality, duplicate fact keys, bridge-table paths, and whether a dimension has duplicate keys or a many-to-many relationship that multiplies rows. Check whether the report combines transaction facts and snapshot facts or joins two facts before aggregation. Do not start by adding `DISTINCT` to the measure because that can hide the modelling defect or remove legitimate repeated events.

Reproduce the issue with a small slice and reconcile row counts and signed amounts at the same business grain. Correct the root cause in the source-to-target model or relationship design, then add regression tests for the affected dimensions and measure. Compare the report under multiple slicers and validate that a valid event repeated in the source is not incorrectly deleted. Record the accepted reconciliation evidence and notify consumers if a published metric changes.

## S02. A source adds a new customer status and changes the meaning of an existing field. What should happen?

Treat the new status and changed meaning as a data-contract change, not merely a schema addition. Determine whether the new value is compatible with the current domain rules and whether the existing field still means what downstream transformations and measures assume. Identify affected data models, dimension history, fact logic, semantic measures, security rules, and reports. Hold unsafe publication if the new value would be silently misclassified.

Update the governed mapping and documentation through review, preserve the source value and contract version, and test both historical and new records. If the meaning changed, consider a new attribute or versioned definition rather than retroactively changing historical meaning without approval. Reconcile counts by status before and after release and validate reports that depend on the field. Communicate the semantic impact, not just the physical schema change.

## S03. A Type 2 dimension receives a backdated correction after facts were published. What is the recovery plan?

Establish the authoritative effective time and source precedence, then reconstruct the affected dimension timeline. Detect overlapping or missing intervals and identify facts whose event time maps to the changed interval. Depending on the business rules, affected fact foreign keys may need to be re-resolved, or only current reporting attributes may need correction. Do not simply close the latest row and insert another row if that would leave historical intervals inconsistent.

Run the correction in a bounded and reproducible scope, record the source change and rule version, and rebuild affected facts or aggregates using idempotent logic. Reconcile counts and measures before and after correction, validate current-state and as-of queries, and verify downstream semantic measures. Retain an audit trail of the correction and its impact so users can explain why a historical report changed.

## S04. A business metric differs between two dashboards even though both use the same warehouse. What do you check?

Compare the semantic definitions, filters, relationship paths, date roles, security context, source freshness, and aggregation behavior. One report may be using an implicit aggregation, a different date relationship, a stale model refresh, or a measure that excludes a status the other includes. Also check whether the dashboards use the same semantic model version and whether one is connected to a different table or data mode.

Use a controlled filter context and a small expected dataset to reproduce the difference. Agree on a single business definition and publish a governed explicit measure or documented variant. Add regression tests for totals, subtotals, date roles, and security roles, and make the model version and freshness visible. Do not assume that matching field names guarantee matching metric semantics.

## S05. A model is correct but too slow for business users. How do you improve it safely?

Profile the full path from source and physical layout through fact/dimension joins and semantic queries. Measure query duration, scanned rows, relationship complexity, high-cardinality columns, expensive calculations, refresh behavior, and concurrency. Decide whether the bottleneck is in ingestion, transformation, storage layout, semantic calculation, or report design. Avoid changing the model’s grain or pre-aggregating data before understanding which questions and filter combinations must remain supported.

Create a representative benchmark and a correctness suite, then make one targeted change at a time: improve physical layout, remove unused columns, simplify a measure, create a valid aggregate, or change the serving mode if supported. Compare latency, cost, refresh, and result equality under realistic filters. Confirm that drill-through, historical semantics, and RLS/OLS behavior remain correct. Publish the optimization with measured evidence and a rollback plan.

---

# E. Practical implementation exercises — 3 exercises

## P01. Design a source-to-target model for customer, account, transaction, and daily balance data.

Start by collecting the reporting questions, source schemas, business identifiers, event-time definitions, currency rules, status codes, and historical requirements. Create a conceptual model showing Customer, Account, Transaction, and Daily Balance; then define a logical model with cardinalities, candidate keys, relationships, and constraints. Separate transaction-event grain from account-date snapshot grain and document which attributes are current-state versus historically effective.

Translate the design into physical tables and source-to-target mappings. Define deterministic deduplication, late-arriving dimension behavior, unknown-member handling, Type 2 history where required, and reconciliation rules. Validate key uniqueness, referential integrity, event and snapshot counts, signed transaction totals, and balance control totals by date and currency. Supply a model diagram, data dictionary, test cases, and sample queries as acceptance evidence.

## P02. Implement a Type 2 dimension and validate its history.

Prepare a staging dataset containing business key, tracked attributes, effective timestamp, source sequence, and batch/run ID. Resolve duplicate or conflicting changes deterministically before applying changes. Identify new business keys, unchanged current versions, and changes requiring a new version. Close the prior version and insert a new surrogate-keyed row with valid effective dates, following the approved policy for same-timestamp changes and late-arriving corrections.

Test a new entity, a changed entity, an unchanged replay, duplicate input, and a backdated correction. Assert that each business key has at most one current row and that effective intervals do not overlap. Verify that facts map to the correct dimension version for their business time. Compare source counts, inserted/closed rows, and downstream report values, and retain run-level evidence to make the process reproducible.

## P03. Build and test a governed semantic model for sales or banking analytics.

Begin with a validated star schema and document the grain and aggregation behavior of every fact. Define explicit measures such as Net Sales, Distinct Customers, Average Daily Balance, or Transaction Count; document inclusions, exclusions, currency handling, and date roles. Configure relationships, hierarchies, friendly names, formats, descriptions, and appropriate security. Hide technical fields that encourage incorrect aggregations and expose only approved consumer concepts.

Create a test dataset with known expected values and validate measure results at detail, subtotal, and grand-total levels. Test date-role behavior, unknown dimension members, security identities, refresh failure, and performance. Compare the published model with an approved source or finance control total, and deploy through the supported environment process. Deliver the model, metric catalogue, access matrix, regression evidence, and operating notes.

---

# F. Architecture and project discussion — 3 questions

## A01. Explain an end-to-end modelling architecture using Azure Databricks or Microsoft Fabric.

Start from source systems and business questions, then explain how raw data is landed and preserved, how conformed entities and quality rules are created, and how analytical facts and dimensions are published. Data modelling clarifies entities and relationships; dimensional modelling defines analytical grain and facts/dimensions; semantic modelling exposes governed measures, relationships, hierarchies, and security to consumers. Depending on the platform, transformations may run in Databricks notebooks, Fabric notebooks, SQL, or another supported engine, while the serving layer may be a warehouse, lakehouse SQL endpoint, or semantic model.

A practical flow is source feeds → raw/bronze → conformed/silver → dimensional gold → semantic model → reports. Each transition needs an explicit contract, owner, quality gates, and lineage. Use a run ID and source boundary to reproduce results, reconcile counts and measures, and control publication. Explain the actual services used in your project, how you handled late data and history, and what evidence proved the result was correct. Distinguish verified hands-on experience from a proposed reference design.

## A02. How would you explain a modelling issue you solved in production?

Choose one concrete issue such as duplicated totals from an incorrect many-to-many relationship, historical facts mapped to the wrong Type 2 dimension row, inconsistent customer identities, or a metric calculated differently across dashboards. Explain the business impact, the evidence you examined, the root cause, and why the existing model allowed the problem. Avoid presenting a tool name as the solution; show how the business semantics and model structure were corrected.

Describe the bounded fix and the regression controls added afterward. Evidence may include before/after reconciliation, key uniqueness tests, interval checks, query results at several grains, security tests, and consumer sign-off. State what you personally designed or implemented, what other teams owned, and which results you can substantiate. If the scenario is hypothetical, identify it as a design exercise rather than claiming it happened in production.

## A03. How do you govern model changes without breaking reports?

Treat a model change as a contract change across source schema, physical tables, dimensional structures, semantic measures, and report consumers. Inventory dependencies and classify the change as additive, compatible, or breaking. Review whether keys, grain, attribute meaning, historical behavior, measure definitions, relationships, or security rules change. Additive columns can still be breaking if consumers infer meaning or automatically aggregate them.

Deploy compatible prerequisites first, validate old and new consumer paths, and use versioned contracts or parallel measures where necessary. Test schema compatibility, source reconciliation, dimension history, semantic outputs, access rules, refresh, and performance. Provide a clear activation, communication, and rollback/correction plan. Keep lineage and release evidence so a future incident can identify which model and metric definition produced a result.

---

# Revision checklist and common mistakes

- Define business meaning and grain before choosing tables, keys, or measures.
- Distinguish conceptual, logical, and physical models.
- Declare primary/candidate/natural/surrogate keys and preserve source lineage.
- Do not confuse duplicate delivery with a legitimate repeated event or reversal.
- Keep transaction facts separate from periodic snapshots when their grains differ.
- Use Type 2 history only when historical attribute context is required; define effective-time rules.
- Prevent duplicate current dimension rows and overlapping effective intervals.
- Do not sum balances across dates or sum percentages without a valid aggregation rule.
- Check many-to-many bridges for row multiplication and define allocation/counting semantics.
- Use conformed dimensions only when their definitions and keys truly align.
- Define business metrics explicitly, including inclusions, exclusions, time basis, and currency.
- Test semantic relationships, filter context, date roles, totals, RLS, and OLS.
- Treat physical optimization and semantic-model optimization as measured workload decisions.
- Reconcile at the same business grain, date boundary, status scope, and currency.
- Keep assumptions, rejected alternatives, model versions, and acceptance evidence documented.
- Do not claim illustrative scenarios, scale, or results as verified project experience.

## Self-assessment

Score each answer from 0–3:

- **0:** Cannot explain the concept.
- **1:** Can define it and give a simple example.
- **2:** Can design an implementation and describe trade-offs.
- **3:** Can diagnose failure modes, prove correctness, and explain governance and recovery.

For a mock interview, select four data-modelling questions, five dimensional-modelling questions, four semantic-modelling questions, two production scenarios, and one practical exercise. Draw the model while explaining it. Expect follow-ups such as: “What is the grain?”, “How do you know the key is unique?”, “What happens when a change arrives late?”, “Can this measure be summed?”, and “What evidence proves the report is correct?”

## Further reading — official documentation

Use platform-specific documentation to verify current syntax, supported features, and operational limits for your target environment.

- [Microsoft Learn — Power BI guidance](https://learn.microsoft.com/en-us/power-bi/guidance/)
- [Microsoft Learn — Star schema guidance for Power BI](https://learn.microsoft.com/en-us/power-bi/guidance/star-schema)
- [Microsoft Learn — DAX overview](https://learn.microsoft.com/en-us/dax/dax-overview)
- [Microsoft Learn — Understand relationships in Power BI](https://learn.microsoft.com/en-us/power-bi/transform-model/desktop-relationships-understand)
- [Microsoft Learn — Row-level security (RLS) with Power BI](https://learn.microsoft.com/en-us/fabric/security/service-admin-row-level-security)
- [Microsoft Learn — Azure Databricks documentation](https://learn.microsoft.com/en-us/azure/databricks/)
- [Microsoft Learn — Microsoft Fabric documentation](https://learn.microsoft.com/en-us/fabric/)

The questions and answers in this guide are interview-preparation material. Adapt examples to your actual platform, business rules, and experience before using them as project claims.
