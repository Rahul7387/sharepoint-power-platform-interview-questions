# Power BI — Interview Questions

⬅ Prev [Power Pages](./09-power-pages.md) &nbsp;|&nbsp; [⬅ Back to Index](../index.html) &nbsp;|&nbsp; [Repo Home](../README.md) &nbsp;|&nbsp; Next ➡ [Scenario-Based (Cross-Topic)](./11-scenario-based.md)

Covers data modeling, DAX, storage modes, security, and ALM for Power BI.

## Contents
- [🟢 Beginner](#-beginner)
- [🔵 Intermediate](#-intermediate)
- [🟠 Advanced](#-advanced)
- [🔴 Expert](#-expert)
- [⚡ Quick-Fire Round](#-quick-fire-round)
- [🧩 Scenario-Based](#-scenario-based)

---

## 🟢 Beginner

**Q1. What are the main components of Power BI?**
Power BI Desktop (the authoring tool for building data models and reports), the Power BI Service (the cloud platform for publishing, sharing, and collaborating on reports/dashboards), and Power BI Mobile (apps for viewing reports on phones/tablets).

**Q2. What is a Dataset (Semantic Model) in Power BI?**
A Dataset (now often called a Semantic Model) is the data model — tables, relationships, and calculations (measures/columns) — that powers one or more reports; a single dataset can be reused across multiple reports.

**Q3. What is DAX?**
DAX (Data Analysis Expressions) is the formula language used in Power BI (and Excel Power Pivot, Analysis Services) to define calculated columns, measures, and custom table expressions.

**Q4. What's the difference between a Report and a Dashboard in the Power BI Service?**
A Report is a multi-page collection of interactive visuals built from a single dataset, allowing filtering/drill-down. A Dashboard is a single-page collection of "pinned" visuals (tiles), which can be pinned from multiple different reports/datasets, offering a high-level summary view but with less interactivity than the underlying reports.

**Q5. What is Power Query, and what's it used for?**
Power Query is the data connection and transformation engine in Power BI (and Excel), used to import, clean, reshape, and combine data from various sources before it's loaded into the data model.

**Q6. What's the difference between a Measure and a Calculated Column at a basic level?**
A Measure is calculated dynamically based on the current filter/selection context and isn't stored as data; a Calculated Column is computed once per row at refresh time and stored as part of the table, usable as a filter/axis field.

---

## 🔵 Intermediate

**Q1. Explain Import, DirectQuery, and Composite storage modes, and their tradeoffs.**
Import mode loads a compressed copy of the data into Power BI's in-memory engine — fastest performance, but only as fresh as the last refresh. DirectQuery sends live queries to the source on every interaction — always current, but slower and constrained by the source's own query performance and DAX feature limitations. Composite models combine both within a single dataset (e.g., a large fact table in DirectQuery alongside smaller Import-mode dimension tables) to balance freshness needs against performance.

**Q2. Explain Row Context versus Filter Context in DAX.**
Row context exists when a formula evaluates per row (e.g., inside a calculated column, or inside an iterator function like `SUMX`), giving direct access to that row's values. Filter context is the set of active filters (from slicers, visual axes, or explicit `CALCULATE`/`FILTER` modifiers) determining which rows are included when a measure is evaluated at query time.

**Q3. What does `CALCULATE` do, and what is "context transition"?**
`CALCULATE` evaluates an expression within a modified filter context — it can add, remove, or override existing filters. Context transition is what happens when `CALCULATE` is used inside a row context (e.g., inside an iterator): the current row's values are converted into an equivalent filter context, which is essential for many advanced DAX patterns.

**Q4. What is a Star Schema, and why is it recommended in Power BI data modeling?**
A Star Schema has a central Fact table (transactional/measurable data, like Sales) connected to surrounding Dimension tables (Date, Product, Customer) via single-direction, one-to-many relationships. It's recommended because the VertiPaq engine and DAX are optimized for this shape — smaller dimension tables efficiently filter large fact tables — and it avoids the ambiguity and complexity that comes from working against one large denormalized flat table.

**Q5. What's the difference between Single-directional and Bi-directional relationship filtering?**
Single-directional (the default/recommended pattern) means a filter on the dimension ("one" side) propagates to the fact table ("many" side) only. Bi-directional also lets filters on the fact table propagate back to the dimension, which can support certain many-to-many scenarios but risks ambiguous filter paths and unintended cross-filtering if applied broadly rather than scoped to a specific measure via `CROSSFILTER()`.

**Q6. How does Row-Level Security (RLS) work in Power BI?**
RLS is implemented by defining Roles in Power BI Desktop, each with a DAX filter expression applied to one or more tables (e.g., `[Region] = "West"`); users are then assigned to roles in the Power BI Service, and the platform automatically applies that role's filter whenever that user views the report — restricting what data they can see without needing separate reports per user group.

**Q7. What is the On-Premises Data Gateway, and why is it needed?**
The gateway securely bridges the cloud-based Power BI Service to on-premises data sources (SQL Server, file shares, on-prem SharePoint) for scheduled refresh or DirectQuery, without exposing the on-premises source directly to the internet.

---

## 🟠 Advanced

**Q1. How would you architect a dataset combining large historical data (5 years, rarely changing) with near-real-time recent data (updates every few minutes)?**
Build a Composite model: set the large historical fact/dimension tables to Import mode (fast, refreshed on a normal schedule like nightly), and set the near-real-time table to DirectQuery (or Import with a very frequent scheduled/incremental refresh if DirectQuery performance on that source is a concern); use Dual storage mode for shared dimension tables (like Date) so they work efficiently regardless of which mode a given fact table's visuals use, and apply Incremental Refresh on the large historical fact table so refreshes only reprocess recent partitions rather than the full 5-year history each time.

**Q2. Explain how Aggregations improve performance for very large DirectQuery-based datasets.**
Aggregations let you pre-summarize a large fact table (e.g., daily or monthly totals) into a much smaller, Import-mode aggregate table that Power BI automatically uses to answer high-level visual queries quickly, only falling back to the full, slower DirectQuery detail table when a visual genuinely needs row-level granularity — this is largely transparent to report authors and end users once configured, since Power BI handles the query routing between aggregate and detail tables automatically.

**Q3. What is Dynamic RLS, and how would you implement it based on a logged-in user's Dataverse/Dataset-mapped attribute (e.g., their region)?**
Dynamic RLS uses a function like `USERPRINCIPALNAME()` compared against a mapping table (e.g., an Employee-to-Region table) within a single Role's DAX filter expression, so that one role dynamically restricts data differently per logged-in user rather than needing a separate static role manually maintained for every individual user — critical for scaling RLS to large user populations without unmanageable role sprawl.

**Q4. How do Power BI Deployment Pipelines support ALM, and what are their limitations compared to a full Azure DevOps/CI-CD pipeline?**
Deployment Pipelines let you manage Dev → Test → Production stages for a workspace's content, compare differences between stages, deploy content forward, and rebind data source parameters per stage (e.g., pointing Test at a test database, Production at the production one) — all through the Power BI Service UI without needing external tooling. Limitations include less granular automation/scripting control than a full Azure DevOps pipeline (e.g., limited built-in automated testing of report correctness), and it's scoped specifically to Power BI content rather than being a general-purpose CI/CD solution spanning other Power Platform components.

---

## 🔴 Expert

**Q1. Design a Power BI architecture for an organization with 500+ reports, dozens of authors, and growing complaints about inconsistent metric definitions (different reports calculating "Revenue" slightly differently).**
Establish a small number of centrally-governed, certified Shared Datasets (or a Power BI dataflow-backed common data layer) containing standardized, single-source-of-truth measures for key business metrics, and require report authors to build "live connection" or Direct Lake/thin reports against these certified datasets rather than each author building their own independent dataset with its own re-implemented (and inevitably diverging) measure definitions; implement a dataset certification/endorsement workflow (Power BI's built-in "Certified"/"Promoted" endorsement levels) combined with governance review before a dataset earns certified status, and use the Power BI admin portal's tenant-wide inventory/usage metrics (or a Center of Excellence-style scanner) to identify and remediate report sprawl building on uncertified, duplicated datasets over time.

**Q2. A DirectQuery-based report against a large SQL Server data warehouse is slow, and the DBA team says query patterns from Power BI are inefficient compared to their own hand-written SQL. How would you investigate and improve this?**
Use SQL Server Profiler/Query Store (or Power BI's own Performance Analyzer, capturing the actual DAX-to-SQL generated queries) to see the exact SQL Power BI is generating per visual, checking for common inefficiency patterns like unnecessarily wide `SELECT *`-style joins across many columns, poorly-indexed filter columns being used in visual slicers, or overly complex DAX measures generating deeply nested subqueries; work with the DBA team to add or adjust indexes aligned to the actual filter/join patterns Power BI generates (rather than assuming the existing index design, optimized for their own hand-written queries, automatically suits Power BI's generated query shape), simplify overly complex measures where possible, and evaluate whether select high-traffic visuals would benefit from an Aggregation table to avoid hitting the DirectQuery source at all for common high-level views.

**Q3. How would you evaluate migrating a large existing DirectQuery/Import hybrid Power BI estate to Direct Lake mode (Microsoft Fabric), and what factors would make you cautious about adopting it immediately for a critical dataset?**
Evaluate Direct Lake's promise of combining near-Import-mode query performance with near-DirectQuery-mode data freshness (querying Delta tables directly in OneLake without a separate import/refresh step) against the organization's specific readiness: whether the data already lives in (or can reasonably be migrated to) a Fabric Lakehouse/OneLake in Delta format, whether the DAX/modeling patterns currently in use are fully compatible with Direct Lake's current feature support (some advanced modeling features may have caveats or fallback-to-DirectQuery behavior), and whether the team has evaluated Direct Lake's "fallback" behavior under certain conditions reverting to DirectQuery-like performance; for a genuinely critical, mission-sensitive dataset, pilot Direct Lake on a lower-stakes dataset first to validate real-world performance and feature compatibility before committing a business-critical report to a comparatively newer capability, given how quickly this specific area of the platform continues to evolve.

---

## ⚡ Quick-Fire Round

- **Q: What formula language is used for measures and calculated columns?** → DAX.
- **Q: What storage mode always queries the live source, never caching a copy?** → DirectQuery.
- **Q: What storage mode combines Import and DirectQuery in one model?** → Composite mode.
- **Q: What function modifies filter context in DAX?** → `CALCULATE`.
- **Q: What's the recommended data modeling shape for Power BI?** → Star Schema.
- **Q: What restricts which rows of data a specific user can see in a report?** → Row-Level Security (RLS).
- **Q: What bridges the Power BI Service to on-premises data sources?** → The On-Premises Data Gateway.
- **Q: What feature partitions a large table's refresh to only reprocess recent data?** → Incremental Refresh.
- **Q: What feature pre-summarizes a large fact table for faster query performance?** → Aggregations.
- **Q: What manages Dev/Test/Production promotion of Power BI content within the Service?** → Deployment Pipelines.

---

## 🧩 Scenario-Based

**Scenario 1. Two departments' reports show different total revenue numbers for the same period, and leadership has lost trust in the data. How do you diagnose and fix this?**
Compare each report's underlying dataset and measure definitions for "Revenue" — the discrepancy is almost always due to independently-built datasets calculating the metric slightly differently (different filter conditions, different handling of returns/refunds, or different date boundary logic); consolidate on a single certified, centrally-governed dataset with one authoritative "Revenue" measure definition that both departments' reports connect to (via live connection or a shared dataset), rather than each maintaining their own separate calculation — and communicate the fix along with the corrected historical numbers to rebuild leadership's trust in the data.

**Scenario 2. A report built on a large on-premises SQL Server DirectQuery source is fast for most users but painfully slow for users in a remote regional office. What would you investigate?**
Check whether the on-premises data gateway used for that DirectQuery connection is physically located far from the regional office's network path (network latency/routing through a distant gateway can dominate perceived slowness even if the actual database query itself is fast), and whether the regional office's own internet connectivity/bandwidth to the gateway/service is a factor; if gateway location is the root cause, consider deploying a gateway cluster closer to the regional office (or adjusting network routing) rather than assuming the issue is purely about DAX/query optimization, since a geographically distant gateway is a commonly overlooked cause of regional-only performance complaints.

**Scenario 3. Sales leadership wants each sales manager to see only their own team's data when opening the same shared report, without needing a separate report per manager.**
Implement Dynamic Row-Level Security: define a single Role with a DAX filter expression that compares `USERPRINCIPALNAME()` against a Manager-to-Team mapping table, so the same underlying report and dataset automatically restricts visible data per logged-in manager — avoiding the unmanageable alternative of maintaining dozens of near-identical static reports or roles as the sales team grows or reorganizes.

**Scenario 4. A dataset refresh that used to take 20 minutes now takes over 3 hours as the underlying fact table has grown to hundreds of millions of rows, and business users need same-day data.**
Implement Incremental Refresh on the large fact table, partitioning it by date so only recent partitions (e.g., the last few days) are reprocessed on each scheduled refresh instead of the entire historical table; if same-day/near-real-time freshness for the most recent data is still required beyond what a scheduled Import refresh can practically achieve, evaluate a Composite model with the most recent data window in DirectQuery (or a hybrid table configuration) while the bulk of history remains in fast, Import-mode partitions — restoring both refresh performance and the freshness business users need.

