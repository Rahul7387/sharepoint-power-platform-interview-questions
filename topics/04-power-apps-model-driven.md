# Power Apps – Model-Driven Apps & Dataverse — Interview Questions

⬅ Prev [Power Apps – Canvas](./03-power-apps-canvas.md) &nbsp;|&nbsp; [⬅ Back to Index](../index.html) &nbsp;|&nbsp; [Repo Home](../README.md) &nbsp;|&nbsp; Next ➡ [Power Automate – Cloud](./05-power-automate-cloud.md)

Covers Dataverse data modeling, security, plugins, business logic, and Model-Driven App design.

## Contents
- [🟢 Beginner](#-beginner)
- [🔵 Intermediate](#-intermediate)
- [🟠 Advanced](#-advanced)
- [🔴 Expert](#-expert)
- [⚡ Quick-Fire Round](#-quick-fire-round)
- [🧩 Scenario-Based](#-scenario-based)

---

## 🟢 Beginner

**Q1. What is Dataverse?**
Dataverse is Microsoft's underlying data platform for the Power Platform and Dynamics 365 — a fully relational, secure, cloud-hosted data store with built-in business logic (business rules, plugins), auditing, and a rich metadata-driven security model.

**Q2. What is a Model-Driven App?**
A Model-Driven App is a Power Apps app type whose UI is largely auto-generated from your Dataverse data model (tables, forms, views, business rules) rather than hand-designed pixel-by-pixel like a Canvas App — ideal for structured, data-centric line-of-business applications.

**Q3. What is a Table (Entity) in Dataverse?**
A Table is a structured set of rows and columns (similar to a database table) that stores data of a specific type, such as "Account," "Contact," or a custom table like "Inspection."

**Q4. What is a Security Role in Dataverse?**
A Security Role defines what privileges (Create, Read, Write, Delete, Append, Assign, Share) a user has over each table, and at what access level (User, Business Unit, Parent-Child Business Unit, or Organization).

**Q5. What's the difference between a Form and a View in Model-Driven Apps?**
A Form is the data-entry/detail layout shown when opening a single record; a View is a list/grid layout showing multiple records with selected columns, filters, and sorting.

**Q6. What is a Business Rule?**
A Business Rule is a no-code way to apply simple logic to a form — like showing/hiding a field, setting a default value, or validating a value — that runs on both the client (form) and, where applicable, the server.

---

## 🔵 Intermediate

**Q1. Compare Dataverse as a backend to SharePoint Lists — when would you choose each?**
Dataverse offers a truly relational model (proper 1:N and N:N relationships), a granular row-level security model, server-side business logic via plugins, real-time and async processing, and much higher scalability/API limits, making it the better choice for complex, secure, process-heavy line-of-business apps. SharePoint Lists are simpler and quicker to stand up, have lower delegation/threshold limits, and lack native relational integrity or plugin-level extensibility — fine for lightweight tracking apps but not for enterprise-scale LOB systems.

**Q2. Explain Business Units, Teams, and how effective user access is calculated.**
A Business Unit is an organizational security boundary that records/users belong to. Teams (Owner Teams that can own records directly, or Access Teams for ad hoc sharing) let you grant a security role or specific record access to a group without assigning roles individually to each user. A user's effective privileges are the union of their own directly-assigned security role(s) plus any roles inherited via team membership, evaluated against a record's owning business unit/owner at each access level.

**Q3. Differentiate Plugins, Business Rules, Classic Workflows, and Power Automate Cloud Flows as automation options in Dataverse.**
Plugins are compiled .NET code executing synchronously or asynchronously in the server-side event pipeline — best for complex logic requiring full control and top performance. Business Rules are simple, no-code, field-level logic running on the form (client) and/or server. Classic Workflows are the legacy background/real-time automation engine (largely superseded). Power Automate Cloud Flows triggered by Dataverse events are the modern low-code automation/integration choice, running asynchronously in the Power Automate service (with some newer synchronous trigger capability), ideal for multi-step business processes and external integrations without writing code.

**Q4. What are the plugin execution stages, and why does the choice of stage matter?**
Pre-validation (stage 10, outside the database transaction) is used for early validation/security checks; Pre-operation (stage 20, inside the transaction, before commit) is used to modify the incoming data or cancel the operation before it's saved; Post-operation (stage 40, after commit) is used for side effects like sending notifications or updating related records once the change is guaranteed saved. Choosing the wrong stage risks trying to "undo" an already-committed change or missing the chance to alter data before it's persisted.

**Q5. What's the difference between Managed and Unmanaged solutions, and how does that fit into ALM?**
Unmanaged solutions are editable "development" containers used while actively building/customizing (typically in a Dev environment). Managed solutions are locked/compiled packages exported for deployment to Test/UAT/Production, preventing direct editing in the target environment and supporting clean, controlled uninstall — the standard ALM pattern is develop unmanaged in Dev, export as managed, and promote through environments.

**Q6. What is a Calculated Column versus a Rollup Column, and what are their limitations?**
A Calculated Column computes its value using a formula evaluated at read/save time based on the same or related record's fields (real-time-ish, simple logic, no aggregation across many child rows). A Rollup Column aggregates values (sum, count, min, max) from related child records, but only recalculates on a schedule or on-demand refresh, not instantly — meaning a rollup value can be momentarily stale immediately after a related record changes.

**Q7. How do Duplicate Detection Rules work, and are they enforced automatically?**
Duplicate Detection Rules define matching criteria (e.g., same email + same name) to flag potential duplicate records on create/update; by default they are advisory — they warn the user with a dialog they can override — not a hard block, so if true prevention is required you'd need additional enforcement (e.g., a plugin or Alternate Key constraint) rather than relying on duplicate detection alone.

---

## 🟠 Advanced

**Q1. Design the security model for a multi-department organization in Dataverse where department managers should see only their department's records, and executives should see everything.**
Use Business Units mapped to departments (each department as its own Business Unit, or use a hierarchical structure if departments roll up under divisions), assign managers a Security Role scoped at the "Business Unit" access level (sees their own BU's records) and executives a role scoped at "Organization" level (sees everything); if departments need cross-visibility for shared records without a strict BU hierarchy fit, consider Access Teams for specific shared record scenarios instead of restructuring the whole BU model, since BU changes are heavier/harder to reorganize later than adding teams.

**Q2. When would you choose a synchronous plugin over an asynchronous Power Automate flow triggered on the same Dataverse event, given both can technically achieve similar outcomes?**
Choose a synchronous plugin when the logic must complete and be reflected before the user's save operation returns (e.g., calculating and displaying a derived value immediately, enforcing a hard validation that must block the save, or maintaining strict transactional consistency where a related record must be created/updated atomically with the triggering record). Choose a Power Automate flow when the logic can run asynchronously without blocking the user (sending an email, calling an external API, running a multi-step approval), since flows execute outside the immediate save transaction and are easier for non-developers to build/maintain, at the cost of not being instantly reflected back to the user's current screen.

**Q3. Explain how Alternate Keys enable reliable data integration patterns, and a scenario where skipping them causes problems.**
An Alternate Key lets external systems/integration jobs uniquely identify a Dataverse row using a business-meaningful value (e.g., an ERP's "Customer Number") instead of Dataverse's internal GUID, which is essential for `Upsert` operations during recurring data syncs — without it, a naive integration re-running an import might create duplicate records every run instead of updating the existing one, since it has no reliable way to check "does this record already exist" without the key.

**Q4. How would you design a solution to safely enforce a hard uniqueness/duplicate-prevention rule that Duplicate Detection Rules (advisory-only) can't guarantee?**
Combine an Alternate Key (a genuine database-level uniqueness constraint) for straightforward single/composite field uniqueness, and for more complex cross-field business logic that a simple key can't express (e.g., "no two open cases for the same customer and product within 30 days"), implement a synchronous Pre-operation plugin that explicitly queries for conflicting records and throws an `InvalidPluginExecutionException` to hard-block the save if a violation is found, rather than relying on the softer, user-overridable Duplicate Detection Rule dialog.

**Q5. What's the impact of registering too many synchronous plugins on a high-volume table (e.g., a table receiving thousands of updates per minute from an integration), and how would you mitigate it?**
Each synchronous plugin adds directly to the save/transaction latency the caller experiences, and with many plugins stacked on the same event, cumulative latency can become severe, potentially causing integration timeouts or throttling under high volume. Mitigation includes moving non-essential logic to asynchronous plugins or Power Automate flows (only keeping genuinely must-be-synchronous validation/calculation logic synchronous), consolidating multiple small plugins into fewer, more efficient ones to reduce per-plugin overhead, and profiling with Plugin Trace Logs to identify which specific plugin(s) contribute the most latency.

---

## 🔴 Expert

**Q1. Design a Dataverse data model and security architecture for a multi-tenant SaaS-style scenario where a single Model-Driven App instance serves multiple external customer organizations, each of whom must never see another's data, using a shared Dataverse environment.**
Add a "Customer Organization" lookup/owning reference on every relevant table and use Business Units to represent each customer organization as its own isolated BU (or use a custom row-level filtering approach via a plugin-enforced column check if BU-per-customer doesn't scale to the number of tenants); assign each external user's Security Role scoped strictly to their own BU/organization with no cross-BU visibility, and critically, audit every custom plugin, Power Automate flow, and report/view to ensure none inadvertently queries or aggregates across the tenant boundary (a very common real-world failure mode is a "global" dashboard or rollup that unintentionally bypasses row-level security by running under a privileged service account). For very large tenant counts, evaluate whether a single shared environment is even appropriate versus a dedicated environment or dedicated Dataverse database per major customer for stronger isolation guarantees.

**Q2. Explain how you would approach performance-tuning a Model-Driven App where a specific entity's list view (with 500,000+ records) is slow to load and filter for end users.**
Review whether the view's underlying FetchXML includes unnecessary joined/linked entities or unindexed filter columns (add appropriate database indexes via table column configuration where supported), reduce the number of columns rendered in the default view to only what's essential (each additional column can add join/query cost), consider whether Quick Find search configuration (which columns are searchable) is overly broad and causing expensive full-text-style scans, and evaluate whether the specific department/team actually needs a personal or team-scoped view (naturally filtered smaller) rather than defaulting everyone to the full unfiltered organization view. For extreme scale scenarios, evaluate Elastic Tables (Cosmos DB-backed) if the access pattern is more key-value/time-series in nature than relational, since they behave very differently at scale from standard Dataverse tables.

**Q3. A critical Post-operation plugin that updates related records asynchronously is occasionally not firing at all under high system load, and no error appears in the plugin trace logs. How would you investigate and design around this?**
Investigate Dataverse's Async Service/System Job queue health during the load window — under extreme load, asynchronous plugin/flow execution can be delayed or, in rare misconfigurations, jobs can fail silently if exception handling within the plugin swallows errors without re-throwing (check the plugin's own try/catch blocks for silent failure patterns, not just the platform's trace log); check System Jobs (Settings → System Jobs) directly for failed/waiting async jobs correlated with the missing updates rather than assuming trace logs capture everything. For resilience, consider adding explicit idempotent retry/reconciliation logic (e.g., a scheduled Power Automate flow or nightly job that detects and repairs records where the expected related update didn't occur) as a safety net rather than relying solely on a single async plugin execution with no fallback.

**Q4. How would you architect a hybrid solution where some business logic must run as compiled Dataverse plugins for performance/security reasons, but the organization also wants business analysts to be able to adjust certain rule thresholds without needing a developer to redeploy code?**
Externalize the adjustable thresholds/parameters into Dataverse configuration records (a custom "Business Rule Configuration" table that analysts can edit through a simple Model-Driven App view/form) rather than hardcoding them in the plugin's compiled logic; have the plugin read these configuration values at runtime (with appropriate caching, e.g., using the Plugin Execution Context's shared variables or a short-lived in-memory cache to avoid a config lookup on every single execution) so analysts can tune thresholds/behavior through familiar Dataverse UI without any code deployment, while the core logic/algorithm itself remains protected, tested, and versioned as compiled code.

---

## ⚡ Quick-Fire Round

- **Q: What's the maximum access level a Security Role can grant?** → Organization.
- **Q: What plugin stage runs before the database transaction commits?** → Pre-operation (stage 20).
- **Q: What type of solution can't be directly edited in a target environment?** → Managed solution.
- **Q: What Dataverse feature enforces true field/combination-level uniqueness?** → Alternate Keys.
- **Q: What column type aggregates values from related child records on a schedule?** → Rollup Column.
- **Q: What's the modern, Power Fx-based alternative to Calculated Columns?** → Formula Columns.
- **Q: What table type is backed by Azure Cosmos DB for high-throughput/time-series data?** → Elastic Tables.
- **Q: What lets a group of users share access to records without individual role assignment?** → Teams (Owner or Access Teams).
- **Q: What tool helps design/customize the Model-Driven App command bar?** → Command Designer (or classic Ribbon Workbench).
- **Q: What UI feature shows a stage-by-stage guided process directly on a form?** → Business Process Flow.

---

## 🧩 Scenario-Based

**Scenario 1. A Model-Driven App form is slow to load, particularly one tab containing a sub-grid. How do you troubleshoot?**
Use browser DevTools' Network tab (filtering `api/data` calls) alongside Dataverse's form performance diagnostics to check whether a synchronous Retrieve plugin, an overly broad/unfiltered sub-grid view (too many columns or unindexed sort/filter), or an inefficient synchronous JavaScript `OnLoad` script is the bottleneck; fixes typically include adding server-side filtering to the sub-grid view, converting unnecessary synchronous `Xrm.WebApi` calls in form scripts to async, and reviewing plugin registration steps for unfiltered, unnecessarily synchronous triggers on that table.

**Scenario 2. HR wants managers to approve employee expense claims, but only claims from employees who report to them (potentially several levels down through the org chart), without manually configuring access per employee.**
Enable Dataverse's Hierarchical Security (Manager Hierarchy or Position Hierarchy model) on the relevant table, which automatically grants a manager read/write access (configurable depth) to records owned by users beneath them in the reporting hierarchy — avoiding the need to manually maintain Access Teams or role assignments as the org chart changes, since it dynamically follows the "reports to" relationship already maintained in the user/position records.

**Scenario 3. An integration job that bulk-imports 50,000 records nightly via the Dataverse Web API is intermittently timing out and occasionally creating duplicate records on retry. How do you fix this?**
Use `Upsert` operations keyed on an Alternate Key (a business-meaningful unique identifier from the source system) instead of blind `Create` calls, so a retried/re-run batch updates existing records instead of duplicating them; batch requests using `$batch`/`ExecuteMultiple` to reduce total round-trips and improve throughput within Dataverse's API limits, and implement exponential backoff on `429`/`503` responses rather than an immediate naive retry that could compound the load causing further timeouts.

**Scenario 4. Leadership wants a "one-click" way for a support agent to escalate a case to a specialist team, notify them via Teams, and update the case's priority — all from within the case form.**
Add a custom command bar button (via the Command Designer) visible only when appropriate (e.g., case status = "Open" and priority is below "High"), wired to call a Power Automate flow (via the "Run a flow" command action or a Custom API) that performs the multi-step logic — updating the case's priority field, reassigning the case's owning team via `Xrm.WebApi`/flow action, and posting an adaptive card notification to the specialist team's Teams channel — keeping the actual multi-step orchestration in the flow rather than complex JavaScript on the form, for easier maintenance.

