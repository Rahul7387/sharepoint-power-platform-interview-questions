# Power Apps – Canvas Apps — Interview Questions

⬅ Prev [SPFx](./02-spfx.md) &nbsp;|&nbsp; [⬅ Back to Index](../index.html) &nbsp;|&nbsp; [Repo Home](../README.md) &nbsp;|&nbsp; Next ➡ [Power Apps – Model-Driven](./04-power-apps-model-driven.md)

Covers Canvas App formulas, delegation, performance, offline design, and componentization.

## Contents
- [🟢 Beginner](#-beginner)
- [🔵 Intermediate](#-intermediate)
- [🟠 Advanced](#-advanced)
- [🔴 Expert](#-expert)
- [⚡ Quick-Fire Round](#-quick-fire-round)
- [🧩 Scenario-Based](#-scenario-based)

---

## 🟢 Beginner

**Q1. What is a Canvas App?**
A Canvas App is a Power Apps app type built on a blank canvas where the maker has pixel-level control over layout, connecting controls to data sources via Excel-like formulas (Power Fx), rather than the app being auto-generated from a data model.

**Q2. What is Power Fx?**
Power Fx is the low-code, Excel-formula-like expression language used throughout Canvas Apps (and increasingly across the Power Platform) to define control properties, logic, and data operations declaratively.

**Q3. What's the difference between `Screen.OnVisible` and `App.OnStart`?**
`App.OnStart` runs once when the entire app launches, often used for global variable initialization; `Screen.OnVisible` runs every time that specific screen becomes visible, useful for refreshing screen-specific data each time a user navigates to it.

**Q4. What are the three main ways to store data in memory within a Canvas App?**
`Set()` for a single global scalar/record variable, `UpdateContext()` for screen-scoped context variables, and `Collect()`/`ClearCollect()` for in-memory tables (collections).

**Q5. What is the Gallery control used for?**
A Gallery displays a repeating, scrollable list of records from a data source (or collection), where each item in the gallery can show multiple fields/controls per record — commonly used for list views, card layouts, and browsing data.

**Q6. What does "Delegable" mean when you see the blue underline warning in Power Apps Studio?**
It's a warning that the formula you've written can't be fully pushed down to the data source and may only process a subset of records (the delegation limit, default 500, max 2000), meaning it could silently return incomplete results on large data sources.

---

## 🔵 Intermediate

**Q1. Explain delegation in depth — why does it exist, and how do delegable operators differ per connector?**
Delegation exists because Canvas Apps don't pull an entire (potentially huge) data source locally before filtering — instead, delegable formulas get translated into the data source's native query language (OData for SharePoint/Dataverse, SQL for SQL Server) and executed server-side, returning only the relevant subset. Each connector supports a different subset of delegable functions/operators (e.g., SharePoint has more delegation restrictions than Dataverse or SQL Server), so the same formula might be fully delegable against Dataverse but only partially delegable against SharePoint.

**Q2. Compare `Patch`, `SubmitForm`, and `Collect` for writing data — when would you use each?**
`SubmitForm` writes the bound Form control's DataCards to the underlying record, respecting form-level validation rules — best for standard create/edit screens using a Form control. `Patch` directly creates/updates specific field-value pairs on a record (or a local collection), giving finer control when you're not using a Form control or need to patch multiple/partial fields programmatically. `Collect`/`ClearCollect` only affect in-memory collections, not a live data source — used for local caching, temporary staging data, or offline scenarios.

**Q3. How would you design offline support in a Canvas App?**
Use `SaveData`/`LoadData` to persist a collection to local device storage, detect connectivity with `Connection.Connected`, queue local changes (e.g., in a "PendingSync" collection) while offline, and process/sync them back to the data source once connectivity returns — with an explicit conflict-resolution strategy (e.g., last-write-wins, or flagging conflicts for manual review) since Canvas Apps don't provide automatic offline sync out of the box (unlike Model-Driven Apps' offline profiles).

**Q4. What is `Concurrent()`, and how does it improve app performance?**
`Concurrent()` executes multiple independent formulas in parallel instead of sequentially, commonly used in `App.OnStart`/`Screen.OnVisible` to fire off several unrelated data loads (e.g., `Set`, `Collect`, `Lookup` calls) at once rather than waiting for each to finish before starting the next, reducing total wait time.

**Q5. How do Canvas App Components and Component Libraries improve reusability, and what are their limitations?**
Components let you build a reusable, parameterized UI block (with custom input/output properties and, more recently, custom events) usable across multiple screens; a Component Library packages components for sharing across multiple separate apps. Limitations historically included restricted access to some native control behaviors and more complex debugging versus inline screen controls, though functionality has expanded significantly over recent releases (e.g., support for more native-like behaviors and code components/PCF integration).

**Q6. What are Environment Variables and Connection References, and why do they matter for solution portability?**
Environment Variables externalize configuration values (a SharePoint site URL, an API endpoint, a numeric threshold) so the same app can point to different resources across Dev/Test/Prod without editing formulas. Connection References abstract which specific connection instance a connector uses, so importing a solution into a new environment only requires re-mapping the reference to an existing connection rather than every maker re-authenticating inline — both are essential for clean ALM.

**Q7. What's the difference between `Filter`, `LookUp`, and `Search`, and how does delegation differ between them?**
`Filter` returns all matching records and supports delegation on most standard comparison operators for typical connectors. `LookUp` returns just the first matching record and follows similar delegation rules to `Filter`. `Search` performs a substring/contains-style text search across specified columns, but its delegation support is often more limited and connector-dependent (e.g., not delegable in several SharePoint scenarios), so it's the one most likely to silently truncate results on a large data source if you're not careful.

---

## 🟠 Advanced

**Q1. A Canvas App connected to SharePoint returns inconsistent filter results in production (12,000 items) despite working fine in a test list of 200 items. Diagnose and fix.**
This is a delegation warning being ignored — SharePoint's non-delegable operations only process the first ~500–2000 records before applying the local (client-side) part of the formula, silently dropping the rest. Fix by rewriting the formula using fully delegable functions/operators for SharePoint (e.g., replacing `Search` with a delegable `Filter` on an indexed column, avoiding functions like `IsBlank()` nested inside conditions that break delegation), indexing the filtered columns in the SharePoint list, or migrating the list to Dataverse where delegation limits and supported functions are considerably better.

**Q2. How would you secure a Canvas App so that a user genuinely cannot bypass app-level formulas to see records they shouldn't (e.g., other departments' data)?**
Never rely solely on app-level formulas (like `Filter(Data, Dept = User().Department)`) for security, since a technically capable user could inspect network calls or query the underlying data source directly via API and bypass the app's UI entirely; instead enforce security at the data source layer — Dataverse security roles/row-level security, SQL Server row-level security policies, or SharePoint item-level permissions — so the *data source itself* refuses to return unauthorized rows regardless of how it's queried, using the app's formulas only as a UX convenience layer on top of true server-side enforcement.

**Q3. Design a Canvas App architecture for a field-service scenario with unreliable connectivity, where technicians must capture inspection data (including photos) and have it reliably sync later.**
Use local collections with `SaveData`/`LoadData` to persist inspection records and photo attachments (as base64 or local URIs) to the device even without connectivity; implement a sync queue pattern where each locally-created record has a "Synced" flag and a locally-generated unique ID (e.g., GUID via `GUID()`), attempt to `Patch` records to Dataverse/SharePoint when `Connection.Connected` is true, and mark them synced only on confirmed success; handle partial failures (e.g., record synced but photo attachment upload failed) with granular status tracking per sub-component rather than one all-or-nothing sync flag, and build a "Sync Status" screen so technicians can visually confirm nothing was lost.

**Q4. How would you approach performance-optimizing a Canvas App with 15+ screens, several of which load slowly on first navigation?**
Move data loading out of `App.OnStart` and into each screen's `OnVisible` (lazy load only what's needed per screen instead of loading everything upfront at app launch), use `Concurrent()` to parallelize independent calls within a screen, apply Explicit Column Selection on data source calls to avoid pulling unused columns, minimize/optimize media assets (compress images, avoid autoplaying video where unnecessary), audit for unnecessary nested Galleries/complex conditional formatting formulas that re-evaluate expensively on every render, and use the Power Apps Monitor tool to profile actual network/formula evaluation timing rather than guessing at the bottleneck.

---

## 🔴 Expert

**Q1. You need to build a Canvas App that must support 5,000+ concurrent users against a Dataverse backend during a narrow daily peak window (e.g., a benefits open-enrollment app). What architectural considerations come into play beyond typical app design?**
Beyond standard delegation/performance practices, consider Dataverse API request limits per user/per tenant (each user has a rolling 24-hour API request allocation; a poorly optimized app making many redundant calls per session could hit limits under concurrent peak load), whether some Dataverse operations should be offloaded to asynchronous Power Automate flows (queued processing) rather than synchronous in-app writes during the peak window to smooth load, whether a "waiting room"/staggered access pattern is needed if the backend truly can't handle instantaneous full concurrency, and load-testing the app beforehand (e.g., using scripted multi-session simulation) since Power Apps Studio testing with a single maker session won't reveal concurrency-specific bottlenecks.

**Q2. Compare building a complex, highly custom UI in Canvas Apps versus using Power Apps Component Framework (PCF) code components, and explain when PCF is the right call.**
Canvas App native controls and Components cover the vast majority of business app UI needs with zero/low code, but PCF (custom code components written in TypeScript/React and registered as a control) is appropriate when you need a genuinely custom control behavior not achievable with native controls/formulas (e.g., a custom charting visualization with specific interaction patterns, a specialized barcode scanner integration, or complex drag-and-drop not supported natively) or when the same custom control needs to be reused consistently across both Canvas and Model-Driven apps (PCF works in both, whereas Canvas Components are Canvas-only). The tradeoff is added development/maintenance complexity (a real code project, build pipeline, and packaging) versus the simplicity of native low-code controls.

**Q3. How would you design a governance strategy to prevent "shadow IT" sprawl of ungoverned Canvas Apps built by business users directly against sensitive Dataverse/SQL data, without stifling citizen development?**
Use Data Loss Prevention (DLP) policies at the environment level to classify connectors (Business vs. Non-Business vs. Blocked), preventing risky combinations (e.g., an app that both reads from a sensitive SQL database and can post to a public Twitter/social connector); establish a tiered environment strategy (a broadly-accessible "Sandbox/Citizen Developer" environment with lower-risk data sources, versus a tightly governed "Managed" environment for apps touching sensitive systems, gated by a lightweight review/promotion process); and use Center of Excellence (CoE) Starter Kit tooling to inventory apps, flag apps with concerning connector combinations or high usage without formal review, and proactively reach out to app owners rather than only reactively auditing after an incident.

**Q4. A Canvas App exhibits severe performance degradation only for a specific subset of users in one region, while working fine for everyone else, despite identical app logic. How would you systematically root-cause this?**
First rule out user-side factors: device type/OS (older/low-powered devices struggling to render complex Galleries or run heavy formulas), network latency/quality specific to that region (especially relevant if the app calls an on-premises data source through a Data Gateway physically located far from that region, or a service with region-specific routing), and browser/Power Apps mobile app version differences. Use the Power Apps Monitor tool session captured from an affected user (or have them reproduce while sharing a monitor session) to see actual network call timings and formula evaluation duration versus a working user's session; if a Data Gateway is involved, check whether that specific gateway cluster is under-provisioned or geographically distant from the affected region's users, which is a common but easily overlooked root cause of regional-only performance issues.

---

## ⚡ Quick-Fire Round

- **Q: What function returns a single record matching a condition?** → `LookUp`.
- **Q: What function creates/adds a row to an in-memory collection?** → `Collect` (or `ClearCollect` to clear first, then add).
- **Q: What's the default delegation limit, and the maximum you can raise it to?** → Default 500, maximum 2,000 (via app settings).
- **Q: What function lets you run several independent formulas in parallel?** → `Concurrent()`.
- **Q: What control is typically used to build a repeating list/card layout?** → Gallery.
- **Q: What two functions persist/retrieve local device data for offline use?** → `SaveData` and `LoadData`.
- **Q: What's the formula-language name used across Canvas Apps?** → Power Fx.
- **Q: What object externalizes environment-specific config values (URLs, thresholds) in a solution?** → Environment Variables.
- **Q: What abstracts a specific connector's authenticated connection for ALM portability?** → Connection References.
- **Q: What tool profiles network calls and formula performance in a running Canvas App?** → Power Apps Monitor.

---

## 🧩 Scenario-Based

**Scenario 1. Sales reps in the field report the Canvas App "freezes" briefly every time they open the customer list screen, which pulls ~3,000 Dataverse rows into a Gallery. How do you fix it?**
Avoid loading all 3,000 rows into a collection upfront; instead bind the Gallery directly to a delegable `Filter`/`Sort` query (e.g., filtered to "My Accounts" or a search box-driven filter) so only the visible/needed subset is fetched, use `Concurrent()` if multiple related lookups happen on load, enable Explicit Column Selection to avoid pulling unused Dataverse columns, and consider a search-as-you-type pattern with a delegable `StartsWith` filter instead of loading the entire customer base by default.

**Scenario 2. Leadership wants the same "Expense Approval" business logic (validation rules, calculations) reused across three separate Canvas Apps built by different teams. How do you avoid duplicating logic three times?**
Build a Component Library containing the shared logic encapsulated in components (e.g., an "ExpenseValidator" component with custom input/output properties), publish it, and have each of the three apps import and reference the same Component Library so any future business rule change is made once and picked up by all three apps on their next component library update — rather than three teams maintaining three independent copies of the same formulas that will inevitably drift out of sync.

**Scenario 3. Users on a Canvas App occasionally submit duplicate expense claims when they double-tap "Submit" on a slow connection. How do you prevent this at the app level?**
Disable the Submit button immediately on first tap (e.g., toggle a `Set(varSubmitting, true)` variable bound to the button's `DisplayMode`) until the `Patch`/`SubmitForm` call completes and returns success or failure, re-enabling only on failure to allow retry; additionally, consider a server-side safeguard (a Dataverse duplicate detection rule or a uniqueness constraint on a natural key like claim reference + timestamp) since app-level UI disabling alone doesn't protect against retries from network-level issues (e.g., the request actually succeeded but the response was lost, causing a user-perceived "it didn't work" retry).

**Scenario 4. A Canvas App needs to let managers approve time-off requests, but managers should only ever see and act on requests from their own direct reports — enforced in a way that can't be bypassed.**
Enforce this at the data layer: if using Dataverse, configure security roles/business units so a manager's access is scoped via the "reports-to" hierarchy (Dataverse supports hierarchical security), so any query — whether from this app, a different app, or a direct API call — inherently only returns records the manager is authorized to see; the Canvas App's gallery/filter formulas then simply reflect what the security-trimmed query already returns, rather than the app being the only thing preventing a manager from seeing/approving someone else's report by manipulating the client.

