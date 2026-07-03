# SharePoint & Power Platform Interview Questions (Intermediate → Expert)

A curated set of interview questions and answers covering **SharePoint**, **SPFx**, **Power Apps (Canvas & Model-Driven)**, **Power Automate (Cloud & Desktop)**, **AI Builder**, **Copilot / Copilot Studio**, **Power Pages**, and **Power BI** — plus real-world **scenario-based** questions.

---

## Table of Contents
1. [SharePoint (Online / Admin / Customization)](#1-sharepoint-online--admin--customization)
2. [SPFx (SharePoint Framework)](#2-spfx-sharepoint-framework)
3. [Power Apps – Canvas Apps](#3-power-apps--canvas-apps)
4. [Power Apps – Model-Driven Apps & Dataverse](#4-power-apps--model-driven-apps--dataverse)
5. [Power Automate – Cloud Flows](#5-power-automate--cloud-flows)
6. [Power Automate – Desktop (RPA)](#6-power-automate--desktop-rpa)
7. [AI Builder](#7-ai-builder)
8. [Copilot / Copilot Studio](#8-copilot--copilot-studio)
9. [Power Pages](#9-power-pages)
10. [Power BI](#10-power-bi)
11. [Scenario-Based Questions (Cross-Topic)](#11-scenario-based-questions-cross-topic)

---

## 1. SharePoint (Online / Admin / Customization)

**Q1. What is the difference between a SharePoint Communication Site and a Team Site?**
A Team Site is connected to an Microsoft 365 Group, is collaboration-focused (document libraries, lists, shared calendar/mailbox), and is used by a project team. A Communication Site has no default Group, is designed for one-to-many broadcast of information (news, reports, status), and offers different out-of-box page designs (Topic, Showcase, Blank).

**Q2. Explain the SharePoint permission inheritance model and when you would break it.**
By default, sites, libraries, and items inherit permissions from their parent. You break inheritance when a subsite/library/item needs unique access (e.g., an HR folder within a general Team Site). Breaking inheritance creates a permission "silo" that must be maintained independently going forward, so it should be used sparingly — item-level breaking especially causes governance and performance overhead at scale.

**Q3. What are Site Columns and Content Types, and why use them over local columns?**
Site Columns are reusable field definitions created at the site (or hub) level; Content Types bundle site columns, workflows, and behavior into a reusable "template" for list items/documents (e.g., "Invoice", "Contract"). Using them centrally ensures consistent metadata schema across multiple lists/libraries and site collections, and updates propagate downstream when columns are changed at the source.

**Q4. What is a Hub Site and how does it differ from a classic Site Collection hierarchy?**
A Hub Site associates multiple independent site collections (each remains its own security/storage boundary) under a common navigation, branding, and search scope, without a parent-child structural dependency. Unlike the old subsite model, associated sites can be moved in/out of a hub without re-platforming content, and search/news roll-up works across the hub.

**Q5. How does SharePoint search work with managed properties and crawled properties?**
Crawled properties are raw metadata discovered during crawl (e.g., from list columns, documents). Managed properties are the mapped, indexed, and queryable/refinable representation used in search queries and Result Sources/KQL. You map a crawled property to a managed property (or create a custom one) to make a field searchable, sortable, or usable in refiners.

**Q6. What's the difference between a SharePoint List and a Document Library, and when would you choose one?**
Both are built on the same list infrastructure, but a Document Library is optimized for file storage (versioning, check-in/out, co-authoring) while a List is optimized for structured data rows without an attached file as the primary artifact. Choose a Library when the file itself is the record; choose a List (or Dataverse for scale) for structured tracking data.

**Q7. What are the storage/throttling considerations you must design around in SharePoint Online?**
List view thresholds (5,000 items per query without indexed columns), API throttling (429/503 responses under heavy load requiring exponential backoff), and tenant storage quotas. Design mitigation includes indexed columns, filtered views, folder-based partitioning, and batching Graph/REST calls with retry logic.

**Q8. Explain Microsoft Graph vs. SharePoint REST API — when do you pick one over the other?**
SharePoint REST (`_api/web/...`) is SharePoint-specific and exposes deep SharePoint object model features (webs, lists, content types). Microsoft Graph is the unified API across Microsoft 365 (Teams, Outlook, OneDrive, SharePoint) and is Microsoft's strategic direction, better for cross-service scenarios and app registrations using modern auth. Use Graph for cross-product scenarios and new development; use SharePoint REST for SharePoint-only features not yet exposed in Graph.

**Q9. What is the difference between "Modern" and "Classic" SharePoint experience, and what are the migration implications?**
Modern pages/lists use responsive, Fluent UI–based web parts and are mobile-friendly, faster, and support SPFx; Classic uses the older master page/web part infrastructure and full trust code (e.g., Farm Solutions/Web Parts). Migration from Classic often requires re-platforming customizations (Farm Solutions, InfoPath, classic workflows) into SPFx, Power Apps, and Power Automate, since full trust solutions are largely deprecated in SPO.

**Q10. How do sensitivity labels and DLP policies interact with SharePoint document libraries?**
Sensitivity labels (from Microsoft Purview) can be applied to sites/libraries/files to enforce encryption, watermarking, and access restrictions, and can flow down as default labels for new content. DLP policies scan content in libraries for sensitive info types (e.g., credit card numbers) and can block sharing, show policy tips, or notify admins/users — both work together for information protection without needing custom code.

---

## 2. SPFx (SharePoint Framework)

**Q1. What is SPFx and how is it different from the old SharePoint Add-in model?**
SPFx is a client-side development model that runs inside the SharePoint page context (same DOM, same authentication session), giving direct access to the page and SharePoint objects, and works across Modern pages, Teams, and Outlook. The Add-in model (Provider-Hosted/SharePoint-Hosted) ran in an iframe and required separate hosting/auth (OAuth), leading to slower load and context-switch overhead — SPFx removed the iframe boundary and unified auth via the SharePoint context.

**Q2. Walk through the SPFx web part lifecycle.**
`onInit()` runs once when the web part loads (good for fetching context, initializing services); `render()` is called to paint the DOM and again whenever a property changes; `onDispose()` cleans up (unmount React trees, remove event listeners); property pane changes trigger `onPropertyPaneFieldChanged()` and then `render()` again. Understanding this avoids memory leaks (not disposing) and redundant API calls (not caching in `onInit`).

**Q3. What is the difference between a Web Part, an Extension (Application Customizer, Field Customizer, Command Set), and a Library Component in SPFx?**
A Web Part is a page-embeddable UI component added via the page/Web Part gallery. Extensions inject behavior without living inside the page canvas: Application Customizers add placeholders (header/footer) or logic tenant/site-wide, Field Customizers change how a column renders in list views, and Command Sets add custom ribbon/context-menu commands. Library Components are reusable non-UI SPFx components (shared logic/services) consumed by other SPFx solutions.

**Q4. How does SPFx handle authentication when calling Microsoft Graph or an external API?**
For Graph/SharePoint calls, SPFx uses the `AadHttpClient`/`MSGraphClientV3` obtained via the web part context, which silently uses the logged-in user's SSO token (no manual OAuth flow needed) provided the API permissions are approved in the SharePoint Admin Center (API Access). For a fully external API, you register an Azure AD app, request a custom AadHttpClient audience, and still route through the same context-based token acquisition rather than storing secrets client-side.

**Q5. What is the SharePoint Framework "bundle and package" process (`gulp bundle` / `gulp package-solution`), and what does the `.sppkg` contain?**
`gulp bundle --ship` produces minified JS/CSS bundles (often uploaded to a CDN like Azure Storage or Office 365 CDN); `gulp package-solution --ship` creates the `.sppkg` app package containing the manifest, referencing the CDN paths for the bundles, feature XML, and permission requests. The `.sppkg` is uploaded to the App Catalog and then can be deployed tenant-wide or per-site.

**Q6. Explain "tenant-wide deployment" vs. per-site deployment of an SPFx solution, and API permission approval.**
Tenant-wide deployment (checkbox in App Catalog, "Make this solution available to all sites") auto-installs the app on every site without individual admin approval per site, useful for org-wide customizers. Per-site deployment requires an admin/owner to explicitly add the app from the site's "Add an app" experience. Either way, any Graph/AAD API permission requested by the package must be explicitly approved by a Global/SharePoint Admin under "API access" before calls succeed.

**Q7. How would you version and safely update an SPFx solution already deployed to hundreds of sites?**
Bump the `version` in `package-solution.json`, keep the `solution.id` (GUID) unchanged, re-bundle/re-package, and upload the new `.sppkg` to the App Catalog, choosing "Replace" — SharePoint auto-propagates the update to sites where it's installed (for tenant-wide, near-immediately; for per-site, on next page-cache refresh). It's best practice to test in a pilot Hub/site collection first and use feature flags/property pane toggles to control rollout of risky changes.

**Q8. What is the difference between `React` state management approaches commonly used in SPFx (local state, PnPjs caching, Redux, React Context)?**
Local `useState`/`this.state` suits component-scoped, short-lived UI state; PnPjs has a built-in caching layer (`usingCaching()`) to reduce duplicate SharePoint/Graph calls across renders; React Context avoids prop-drilling for shared read-mostly data (current user, theme); Redux/Zustand is used for complex, cross-component, mutable app state in larger SPFx solutions (e.g., a full custom list-management app). Choice depends on complexity — over-engineering a simple web part with Redux is a common anti-pattern.

**Q9. How does SPFx theming work, and how do you support dark/light/custom SharePoint themes?**
SPFx exposes a `ThemeProvider`/`this.context.microsoftTeams` or `SPFx ThemeProvider` (via `@microsoft/sp-component-base`) which reads the active SharePoint/Teams theme's semantic color slots (`theme.palette.themePrimary`, etc.). Components should style using theme tokens/CSS variables rather than hardcoded hex values so the web part automatically adapts when a site's theme changes or a user switches to dark mode.

**Q10. What are common performance pitfalls in SPFx solutions and how do you mitigate them?**
Pitfalls include: loading heavy third-party libraries (e.g., full lodash/moment) instead of tree-shakable alternatives, making unbatched sequential REST calls in a loop, not memoizing expensive renders, and bundling large bundles that aren't code-split/lazy-loaded. Mitigations: use `React.lazy`/dynamic `import()`, PnPjs batching, `externals` in `config.json` to load shared libraries from CDN, and Lighthouse/Page Diagnostics tooling to validate load impact.

**Q11. How do you debug an SPFx solution running on a real SharePoint page (not just the local workbench)?**
Use `gulp serve --nobrowser`, then navigate to the real modern page and append `?debug=true&noredir=true&debugManifestsFile=https://localhost:4321/temp/manifests.js`, which lets the live page load your locally-served component code with source maps in DevTools. This validates real page context, theme, and permissions that the hosted workbench can't fully replicate.

**Q12. What is the role of the App Catalog, and what's the difference between the tenant App Catalog and a site collection App Catalog?**
The App Catalog is a special site that stores `.sppkg` packages for distribution. The Tenant App Catalog (one per tenant) is the primary distribution point and required for tenant-wide deployment; a Site Collection App Catalog is scoped to a single site collection, useful for delegated/departmental admins who need to deploy custom apps to their own sites without Global Admin/SharePoint Admin rights on the tenant catalog.

---

## 3. Power Apps – Canvas Apps

**Q1. What is "delegation" in Canvas Apps, and why does it matter?**
Delegation means the data-processing (filtering, sorting, aggregating) is pushed down to the data source (e.g., Dataverse, SQL) instead of being pulled into the app and processed locally, which is capped (default 500, max 2,000 records). Non-delegable functions/operators on large data sources silently truncate results, so understanding each connector's delegable function list (and using `Filter`/`Sort` with delegable operators instead of `Search` with unsupported functions like `IsBlank` inside `If`) is critical for apps working over large tables.

**Q2. Explain the difference between `Patch`, `SubmitForm`, and `Collect`/`ClearCollect` for writing data.**
`SubmitForm` commits the data from a Form control's bound fields to its `Item`/data source, respecting form validation and `DataCard` bindings. `Patch` directly creates/updates a record on a data source (or a local collection) with explicit field-value pairs, offering finer control (e.g., patching without a Form control, bulk patches). `Collect`/`ClearCollect` create/populate an in-memory collection (not a data-source write) useful for local state, caching, or offline scenarios.

**Q3. How do you implement offline capability in a Canvas App?**
Use `LoadData`/`SaveData` to persist a local collection to the device, combined with `Connection.Connected` to detect connectivity, queuing changes locally when offline and syncing (via `Patch`/`Collect` loops or Power Automate) once back online; conflict handling (last-write-wins vs. custom merge logic) must be explicitly designed since Canvas Apps don't provide automatic offline sync like model-driven apps' offline profiles.

**Q4. What's the difference between a Canvas App and a Model-Driven App, and how do you decide which to build?**
Canvas Apps give pixel-level UI control and can bind to 100+ connectors (SharePoint, SQL, Excel, custom APIs) but require manual layout/logic; Model-Driven Apps are schema/Dataverse-driven, auto-generate responsive UI from entity metadata, and get built-in features (business rules, workflows, security roles, timeline) with less design effort but less UI flexibility. Choose Canvas for highly custom, task-specific, cross-data-source apps; choose Model-Driven for structured, data-model-centric, process-heavy line-of-business apps built primarily on Dataverse.

**Q5. How does component-based design (Canvas Components / Component Libraries) improve maintainability?**
Components let you build reusable, parameterized UI blocks (custom properties, custom output properties, and now events) that can be shared across multiple screens or even multiple apps via a Component Library, similar to a UI component framework. This avoids copy-pasting control logic across screens, centralizes bug fixes/updates, and supports a design-system approach across a portfolio of apps.

**Q6. What is `Concurrent()` and how does it help app performance?**
`Concurrent()` runs multiple otherwise-sequential formulas (e.g., several data source calls in `OnStart`/`OnVisible`) in parallel instead of one after another, reducing perceived load time. It's commonly used to parallelize independent `Set`/`Collect`/`Lookup` calls that don't depend on each other's results.

**Q7. How do you secure a Canvas App beyond basic Microsoft Entra ID authentication (e.g., row-level security)?**
Row-level/data-level security should be enforced at the data source layer (Dataverse security roles/business units, SQL row-level security, SharePoint item permissions) — not solely with app formulas like `If(User().Email = ...)`, since that logic can be bypassed by someone inspecting the app or via direct API/data-source access. The app can additionally use conditional visibility/DLP policies and Entra Conditional Access to control who can even open the app.

**Q8. What are Environment Variables and Connection References, and why are they important for ALM?**
Environment Variables externalize configuration values (URLs, SharePoint site addresses, thresholds) from hardcoded formulas so the same solution can point to Dev/Test/Prod resources without code changes. Connection References abstract the specific connection used by a connector so that when a solution is imported into a new environment, admins just re-map the reference to an existing connection instead of every maker re-authenticating inside formulas — both are essential for solution-based ALM (Application Lifecycle Management) and CI/CD pipelines.

**Q9. Explain `Filter` vs. `LookUp` vs. `Search` and their delegation implications.**
`Filter` returns all matching records and is delegable on most standard operators (`=`, `>`, `<`, `StartsWith`); `LookUp` returns the first matching single record and is also delegable under similar conditions; `Search` performs a text-contains search across specified columns and its delegation support is more limited and data-source dependent (e.g., not delegable on SharePoint for most cases). Choosing the wrong one on a >2000-row table silently returns incomplete results, so testing with the "delegation warning" (blue underline) in Power Apps Studio is essential.

**Q10. How would you optimize a Canvas App that is loading slowly on `OnStart`/first screen?**
Move non-critical data loads out of `App.OnStart` into the relevant screen's `OnVisible` (lazy loading), use `Concurrent()` for parallelizable calls, avoid loading entire large tables into collections unnecessarily, minimize expensive image/media assets, and use `Set()` for scalar app-level variables instead of `Collect()`ing into large collections just to read one value. Also enable "Explicit Column Selection" for data sources to fetch only needed columns instead of full rows.

---

## 4. Power Apps – Model-Driven Apps & Dataverse

**Q1. What is Dataverse and how does it differ from SharePoint Lists as a backend?**
Dataverse is a fully relational, enterprise-grade data platform with built-in security roles/row-level security, business rules, plugins, real-time and async processing, auditing, and native relationships (1:N, N:N) — purpose-built for line-of-business apps. SharePoint Lists are simpler, quota/threshold-limited, and lack native relational integrity, complex security modeling, and server-side business logic (plugins), making Dataverse the better choice for complex, scalable, secure LOB solutions.

**Q2. Explain Dataverse Security Roles, Business Units, and Teams, and how access is evaluated.**
A Business Unit is an organizational security boundary; Security Roles define privilege levels (Create/Read/Write/Delete/Append/Share/Assign) at four access levels (User, Business Unit, Parent:Child BU, Organization); Teams (Owner or Access Teams) let you grant shared access/roles to a group of users without assigning each user individually. Effective access for a user is the union of privileges from their own roles plus any team roles they belong to, evaluated against the record's owning business unit/owner.

**Q3. What's the difference between a Plugin, a Business Rule, a Classic Workflow, and a Power Automate Cloud Flow triggered from Dataverse?**
Plugins are compiled C# code executed synchronously or asynchronously on the server in response to Dataverse events (pre/post-operation), offering full control and best performance for complex logic. Business Rules are no-code, form/data-level logic (field visibility, default values, simple validation) that run on both client and server. Classic Workflows are the legacy background/real-time process automation (largely being replaced). Cloud Flows triggered by Dataverse connector events are the modern low-code equivalent for integrations and multi-step automation but run asynchronously in the Power Automate service, not in the transaction pipeline (except with newer "synchronous" trigger options).

**Q4. What are the Dataverse plugin execution stages, and why does stage matter?**
The pipeline stages are: Pre-validation (stage 10, outside the DB transaction), Pre-operation (stage 20, inside transaction, before the change is committed — used to modify incoming data or cancel the operation), and Post-operation (stage 40, after commit — used for side effects like notifications or related-record updates). Choosing the wrong stage can cause partial data issues (e.g., trying to roll back after commit) or missed validation opportunities (e.g., validating after the record is already saved).

**Q5. What is the difference between synchronous and asynchronous plugin/flow execution, and when would you choose each?**
Synchronous execution blocks the user's save operation until it completes — used when the logic must finish before the user proceeds (e.g., validation, calculated field that must be visible immediately). Asynchronous execution queues the operation to run in the background without blocking the UI — used for long-running or non-blocking tasks (sending emails, calling external APIs, bulk updates) where immediate confirmation to the user isn't required.

**Q6. Explain Dataverse solutions — Managed vs. Unmanaged — and their role in ALM.**
Unmanaged solutions are editable "development" containers used while building/customizing in a Dev environment. Managed solutions are compiled/locked packages exported for deployment into Test/UAT/Production — they can't be directly edited in the target environment (protecting production integrity) and support clean uninstall/rollback. Standard ALM practice: develop in unmanaged solutions in Dev, export as managed, and promote through environments via pipelines (Power Platform Pipelines, Azure DevOps, or GitHub Actions).

**Q7. How do Alternate Keys and Elastic Tables function in Dataverse, and when would you use each?**
Alternate Keys let you uniquely identify a Dataverse row using a non-primary-key column (or combination), essential for `Upsert` operations during data integration (avoiding duplicate creation on re-import). Elastic Tables are a Dataverse table type built on Azure Cosmos DB, optimized for high-throughput, schema-flexible, and time-series/IoT-style data (e.g., telemetry, logs) where standard relational tables would be inefficient at scale.

**Q8. What is the Dataverse "Power Fx" / formula columns feature, and how does it compare to calculated/rollup columns?**
Formula columns use Power Fx expressions evaluated in real time (similar to Excel-like formulas) and can reference related tables, offering more expressive logic than classic Calculated Columns. Rollup Columns aggregate values from related child records (sum, count, min/max) but only recalculate on a schedule or on demand (not instantly), whereas formula/calculated columns are computed at read/query time (or on save, for calculated).

**Q9. How would you design a solution to prevent duplicate records being created by multiple users simultaneously in a Model-Driven App?**
Combine Dataverse Duplicate Detection Rules (server-side, checks on create/update against defined match criteria) with Alternate Keys for hard uniqueness constraints on integration scenarios, and consider a synchronous pre-operation plugin for complex cross-field duplicate logic that OOB duplicate detection can't express. Duplicate Detection Rules alone are advisory (warn but don't block by default) unless you enforce blocking via a plugin or by disabling save on detection in the client.

**Q10. What is the ribbon/command bar customization approach in Model-Driven Apps, and how do you add custom logic to a button?**
Use the modern Command Designer (Power Apps maker portal) or classic Ribbon Workbench to add custom command bar buttons, define visibility rules (JavaScript/Power Fx-based "Show Rule" or via enable/display rules), and attach an action calling a Power Automate flow, a Custom API, or a JavaScript web resource function via Client API (`Xrm.WebApi`). This is the standard extensibility point for adding business-specific actions beyond OOB Save/Deactivate/Delete.

---

## 5. Power Automate – Cloud Flows

**Q1. What is the difference between Automated, Instant, Scheduled, and Business Process flows?**
Automated flows trigger on an event (new item, HTTP request, Dataverse row change); Instant flows are manually triggered by a user (button in app/Teams, "Run now"); Scheduled flows run on a recurrence (daily, hourly); Business Process Flows are a Dataverse/Model-Driven-specific guided, stage-based UI process shown directly on the entity form, not a background automation. Choosing correctly avoids building an automated flow when a guided user process (BPF) is actually what's needed, or vice versa.

**Q2. How do you handle errors robustly in a Cloud Flow (beyond the default "Configure run after")?**
Use "Configure run after" on each critical action to branch on `is successful`/`has failed`/`is skipped`/`has timed out`, wrap risky sections in a `Scope` container with a matching "on failure" Scope that runs cleanup/notification/logging, and use `Terminate` with a custom error message for controlled failure exits. For production flows, centralize error notification (e.g., post to Teams/email with `@{workflow()}` run details) so failures are actionable, not silent.

**Q3. What is the difference between a Standard Connector and a Premium Connector, and how does that affect licensing?**
Standard connectors (SharePoint, Outlook, Teams) are included in base Microsoft 365 licensing; Premium connectors (Dataverse, HTTP, SQL Server, on-prem gateway connectors, many third-party connectors) require a Power Automate per-user/per-flow premium license or Power Apps premium license. Using even one premium connector or Custom Connector in a flow makes the *entire flow* require premium licensing for the account running it.

**Q4. Explain concurrency control and "Degree of Parallelism" in Apply to Each loops — why would you change it?**
By default, `Apply to Each` runs iterations sequentially (concurrency of 1) unless "Concurrency Control" is turned on, in which case you can set parallelism up to 50 to process items faster. However, increasing concurrency can cause race conditions (e.g., two iterations updating a shared counter/record simultaneously) or hit API throttling limits faster, so it should be used carefully with idempotent, independent operations.

**Q5. What are the licensing/execution implications of using the HTTP connector vs. a Custom Connector wrapping the same API?**
The raw HTTP action is a premium connector and calls any REST endpoint directly but lacks reusable schema/authentication management across flows. A Custom Connector wraps an API with a defined OpenAPI schema, reusable authentication (API key/OAuth), and can be shared across the organization/environment as a first-class connector — better for governance, reuse, and DLP policy classification, though it also counts as premium.

**Q6. How do you design a flow to avoid hitting Dataverse or SharePoint API throttling limits at scale?**
Batch operations where possible (e.g., Dataverse's batch/`ExecuteMultiple` via HTTP action, or SharePoint batch REST calls), add a `Delay`/exponential backoff pattern on 429/503 responses, use "trigger conditions" to reduce unnecessary flow runs, and split heavy per-item logic into a child flow invoked via `Child flow` action to parallelize sensibly without single-flow API call ceilings. Reviewing the Power Platform admin center's API request analytics helps detect and pre-empt throttling before it happens in production.

**Q7. What is a Trigger Condition and why use it instead of a Condition action right after the trigger?**
A trigger condition (`@equals(...)` expression set in the trigger's settings) prevents the flow run from even starting if the condition isn't met, saving a full flow run (and its consumption/quota) versus running the entire trigger + first Condition action and then exiting. This is a key performance and cost optimization for high-volume triggers like "When an item is created or modified."

**Q8. Explain the difference between "Solution-Aware" (Dataverse-based) flows and flows created outside a solution, from an ALM perspective.**
Flows built inside a Dataverse solution can be exported/imported as part of solution ALM (Dev → Test → Prod), support Connection References and Environment Variables for portability, and are versioned with the rest of the solution's components. Flows created outside a solution ("My Flows") are tied to the personal environment/maker and are much harder to migrate, back up, or manage in Application Lifecycle Management — production business flows should always be solution-aware.

**Q9. How does a Child Flow differ from calling another flow via HTTP, and when would you use each?**
A Child Flow (Power Automate "trigger: when called by another flow" — a Dataverse solution capability) can be directly invoked with input/output parameters from a parent flow in the same environment/solution, is easy to reuse and version together. Calling another flow via an HTTP-triggered flow ("When an HTTP request is received") works across environments/tenants and doesn't require solution awareness, useful for more decoupled or cross-boundary integration, but requires managing the HTTP endpoint URL/security (SAS-like) separately.

**Q10. What are Approvals in Power Automate, and how would you design a multi-stage sequential/parallel approval process?**
The Approvals connector provides built-in actions ("Start and wait for an approval") supporting first-to-respond or everyone-must-approve patterns; for multi-stage processes (e.g., Manager approval, then Finance approval), you chain sequential "Start and wait for approval" actions with conditional branching based on each stage's outcome, and can use parallel branches for simultaneous multi-approver stages. Approval history and status are tracked centrally, and outcomes can drive subsequent actions (e.g., update a Dataverse record's status field).

---

## 6. Power Automate – Desktop (RPA)

**Q1. What's the fundamental difference between Power Automate Cloud Flows and Power Automate Desktop flows?**
Cloud flows orchestrate SaaS/API-based integrations across connectors in the cloud; Desktop flows automate UI-level interactions on Windows applications and legacy/desktop apps that don't have APIs (screen scraping, mouse/keyboard simulation, Excel/legacy app automation) — true RPA. Desktop flows require a machine (attended or unattended via a Gateway/machine registration) to actually execute the automation against the target application UI.

**Q2. Explain "Attended" vs. "Unattended" RPA in Power Automate Desktop, and licensing implications.**
Attended automation runs on a user's machine with the user present/triggering it (e.g., from the tray icon or a Cloud Flow calling it while the user is logged in), typically usable with a standard Power Automate license tied to that user. Unattended automation runs on a machine without a human present (scheduled or triggered from the cloud, using a machine registered via the on-premises data gateway and a dedicated unattended RPA license/machine profile) — critical for back-office, after-hours processing.

**Q3. What are "Selectors" in Power Automate Desktop, and why do UI automations break?**
A Selector is the mechanism used to uniquely identify a UI element (via a tree of properties like `Name`, `AutomationId`, `ControlType`) so the desktop flow can locate and interact with it reliably. Automations commonly break when the target application's UI changes (updated version, different screen resolution, dynamic/generated IDs), so best practice is to use image-based recognition or anchor-based/relative selectors sparingly as a fallback, and to prefer stable UI Automation (UIA) framework elements over pure image matching.

**Q4. How do you handle error recovery in a Desktop Flow when a target application becomes unresponsive?**
Wrap risky action blocks in "On Block Error" (equivalent of try/catch) to define retry logic, alternate paths, or graceful termination; use "Wait for element/window" actions with configurable timeouts instead of fixed `Delay` steps to make the flow resilient to variable app load times; and design flows to detect and restart/kill-and-relaunch a hung application as a recovery strategy before failing the whole automation.

**Q5. When would you choose Power Automate Desktop over a Cloud Flow with an API-based connector for the same integration?**
Choose Desktop when the target system has no API/connector (legacy green-screen/mainframe apps, older desktop software, or a SaaS tool without a Power Platform connector), or when the process fundamentally requires simulating human interaction (multi-window desktop workflows). If an API or a first-class connector exists, a Cloud Flow is almost always more reliable, maintainable, and scalable than UI automation.

**Q6. How can a Cloud Flow and a Desktop Flow be combined in a single automation, and what's a typical pattern?**
A Cloud Flow acts as the orchestrator/trigger (e.g., on a new item in SharePoint or a scheduled recurrence) and calls a Desktop Flow (registered via the Machine/Process Automation connector) as an action, optionally passing input parameters and receiving outputs back; this hybrid pattern is common for "cloud triggers legacy desktop task" scenarios like generating a report in an old Windows app and emailing it via a Cloud Flow afterward.

---

## 7. AI Builder

**Q1. What is AI Builder, and what are the two categories of models it offers?**
AI Builder is a Power Platform capability that brings AI/ML models into apps and flows without needing data science expertise. It offers Prebuilt models (ready to use out of the box — e.g., Business Card Reader, Text Recognition/OCR, Sentiment Analysis, Object Detection for common objects) and Custom models (trained on your own data — Form Processing, Object Detection for custom objects, Prediction models, Category Classification).

**Q2. How does the AI Builder "Form Processing" model differ from a generic OCR/Text Recognition model?**
Text Recognition/OCR extracts raw text from an image/document without structural understanding. Form Processing is trained on sample documents of a specific layout (e.g., a specific invoice template) to extract structured key-value fields and table data (e.g., "Invoice Number", "Total", line items) with the same layout, requiring you to label 5+ sample documents per variation during training.

**Q3. What is the "Prediction" model type in AI Builder, and what's a real-world use case?**
A Prediction model is a binary classification model trained on your historical Dataverse data (with a known outcome column) to predict a yes/no outcome for new records — e.g., predicting whether a sales lead will convert, or whether an equipment maintenance ticket will result in an escalation — and can be run directly against Dataverse tables or invoked from a Cloud Flow/Canvas app for real-time scoring.

**Q4. How are AI Builder credits consumed, and what should you consider for cost governance?**
AI Builder operations (each prediction, document scan, image tag, etc.) consume "AI Builder credits" from a capacity add-on pool assigned at the tenant or environment level; heavy usage (e.g., processing thousands of documents monthly) requires purchasing additional AI Builder capacity packs. For cost governance, monitor consumption in the Power Platform admin center's Capacity view and design flows to avoid unnecessary re-processing (e.g., don't re-run OCR on the same document on every flow retry).

**Q5. How would you integrate a Custom Object Detection model into a Canvas App for a quality-inspection scenario?**
Train and publish the Object Detection model in AI Builder using labeled images of the objects/defects to detect, then add the built-in "AI Builder – Object Detector" component to a Canvas App screen, bind it to the trained model, let a user capture/upload a photo, and use the returned bounding boxes/confidence scores to branch logic (e.g., flag "defect detected" and `Patch` a Dataverse inspection record).

---

## 8. Copilot / Copilot Studio

**Q1. What is the difference between "Microsoft 365 Copilot," "Copilot Studio," and "Copilot in Power Apps/Power Automate" (the maker-assist features)?**
Microsoft 365 Copilot is the end-user AI assistant embedded in Word/Excel/Teams/Outlook working over the user's Microsoft Graph-accessible content. Copilot Studio (formerly Power Virtual Agents) is the platform for building custom conversational copilots/agents (with generative answers, topics, and now autonomous agent actions) that can be deployed to Teams, websites, or embedded in apps. Copilot *in* Power Apps/Power Automate refers to the in-product AI assistant that helps makers generate app screens, formulas, or flow steps from natural-language prompts during development.

**Q2. In Copilot Studio, what is the difference between a "Topic" and "Generative AI answers"?**
A Topic is an authored, deterministic conversation flow with defined trigger phrases, conditions, and actions (classic Power Virtual Agents style) — predictable and controllable for known scenarios (e.g., "reset my password"). Generative AI answers use an LLM (grounded on a configured knowledge source — SharePoint, websites, Dataverse) to dynamically generate responses for open-ended questions not covered by a specific Topic, trading some determinism for broader coverage.

**Q3. How do you ground a Copilot Studio agent's answers in your organization's own data securely?**
Configure Knowledge sources (SharePoint sites/document libraries, public websites, Dataverse tables, or uploaded files) under Generative AI settings, which the agent retrieves-and-grounds against at answer time (RAG-style) rather than relying purely on the base LLM's training data; access is scoped so the agent only surfaces content the *end user* (not just the maker) has permission to see when SharePoint/Dataverse security trimming is properly configured, which is critical to avoid inadvertent data leakage.

**Q4. What are "Actions" (plugins/tools) in Copilot Studio, and how do they extend a copilot beyond conversation?**
Actions let a copilot invoke Power Automate flows, Connectors, or custom Prompts (AI Builder GPT-based prompts) mid-conversation to perform real tasks — e.g., checking order status via a flow that queries Dataverse/SQL, or creating a ticket — turning the copilot from a pure Q&A bot into an "agent" that can take action, not just answer.

**Q5. How would you implement escalation to a human agent (e.g., Omnichannel for Customer Service) from a Copilot Studio bot?**
Add an "Escalate" action/topic that transfers the conversation to a live agent queue via the Omnichannel for Customer Service integration (or a Teams/Dynamics 365 handoff), typically triggered by a fallback Topic (when confidence is low or the user explicitly asks for a human), passing conversation context/transcript along so the human agent isn't starting cold.

**Q6. What governance controls exist for Copilot Studio agents in an enterprise (DLP, publishing, environment strategy)?**
Copilot Studio agents are governed like other Power Platform components: DLP policies can restrict which connectors/actions an agent's flows can use, agents live within an Environment (so Dev/Test/Prod separation and solution-based ALM apply), and admins can control publishing channels (Teams, website, Direct Line) and require approval before an agent goes broadly available, alongside monitoring via the Power Platform admin center / Copilot Studio analytics.

---

## 9. Power Pages

**Q1. What is Power Pages, and how does it relate to the older "Portals" capability?**
Power Pages is the evolved, standalone low-code platform (rebranded/re-architected from Power Apps Portals in 2023) for building secure, external-facing websites backed by Dataverse, with a dedicated design studio (Pages, Styling, Data, Security workspaces) rather than portals being just an app type inside Power Apps. Existing Portals apps continue to run and can be managed/edited from the same modern Power Pages design studio.

**Q2. How does authentication work in Power Pages for external users (customers/partners)?**
Power Pages supports Local Authentication (username/password managed within Dataverse's contact-based identity), or Enterprise/external Identity Providers (Microsoft Entra ID, Entra External ID/B2C, or other OpenID Connect/SAML providers), allowing customers, partners, or citizens to sign in with their own credentials while still mapping to a Dataverse Contact record that drives personalization and security.

**Q3. Explain Table Permissions and Web Roles in Power Pages security.**
Web Roles are groups assigned to portal users (e.g., "Authenticated Users," "Customer," "Partner") similar in spirit to Dataverse security roles but scoped to the portal. Table Permissions define, per Web Role, what CRUD access is allowed on a given Dataverse table and at what scope (Global, Contact, Account, Parent, or a custom relationship-based scope) — this is the primary mechanism controlling what data an anonymous or authenticated portal visitor can see/edit, layered on top of (not replacing) Dataverse's own table-level security.

**Q4. How would you expose a complex approval or business process to external users via Power Pages while keeping core logic in Dataverse/Power Automate?**
Build the data capture as a Basic Form/Advanced Form (or List) on a Power Pages page bound to a Dataverse table, use a Power Automate flow triggered on record creation (via Dataverse trigger) to run the actual business/approval logic (internal notifications, approvals, integrations) server-side, and reflect status back to the portal user by updating a status field the page displays — keeping sensitive business logic off the public-facing site and in the more secure/managed Dataverse+Flow layer.

**Q5. What performance/security considerations are unique to Power Pages compared to internal Model-Driven/Canvas apps?**
Since Power Pages sites are internet-facing, you must actively defend against anonymous abuse (enable Web Application Firewall via Azure Front Door/CDN, CAPTCHA on forms, rate limiting), be very deliberate with Table Permissions (a misconfigured "Global" scope can expose all tenant records to anonymous users), and consider caching/output caching for public content pages since traffic patterns and scale (potentially thousands of anonymous visitors) differ greatly from internal LOB apps.

---

## 10. Power BI

**Q1. Explain the difference between Import, DirectQuery, and Composite (Dual) storage modes.**
Import mode loads a compressed copy of data into Power BI's in-memory VertiPaq engine — fastest query/DAX performance but data is only as fresh as the last scheduled refresh. DirectQuery sends live queries to the source on every interaction — always up to date but slower and constrained by source performance/DAX limitations. Composite models mix both (e.g., a large fact table in DirectQuery, small dimension tables set to Dual so they can be used efficiently with either mode) to balance freshness and performance.

**Q2. What is the difference between a Calculated Column and a Measure in DAX, and when do you use each?**
A Calculated Column is computed row-by-row at data refresh time and stored in the model (consuming memory, usable as a slicer/axis field), evaluated in row context. A Measure is computed on the fly at query time based on the current filter context (visuals, slicers) and is not stored — used for aggregations that must dynamically respond to user interaction (e.g., "Total Sales," "YoY Growth %"). Overusing calculated columns where a measure would work bloats model size and hurts performance.

**Q3. Explain Row Context vs. Filter Context in DAX, and how `CALCULATE` interacts with them.**
Row context exists when a formula is evaluated per-row (e.g., inside a calculated column or an iterator function like `SUMX`), giving access to that row's field values directly. Filter context is the set of filters applied by slicers, visual axes, or explicit `CALCULATE`/`FILTER` modifiers that determines which rows are included in an aggregation at query time. `CALCULATE` is the key function that can modify filter context (add, remove, or override filters) and can also convert row context into filter context via context transition when used inside a row-context iteration.

**Q4. What is a Star Schema, and why is it recommended over a single flat/wide table in Power BI data modeling?**
A Star Schema has a central Fact table (transactional/measurable data, e.g., Sales) surrounded by denormalized Dimension tables (Date, Product, Customer) connected by single-direction, one-to-many relationships. It's recommended because VertiPaq's compression and DAX engine are optimized for this shape (smaller dimension tables filter large fact tables efficiently), it avoids ambiguous many-to-many relationship issues, and it's far easier to build correct, performant measures than against one denormalized flat table with repeated dimension attributes.

**Q5. What's the difference between Bi-directional and Single-directional relationship filtering, and what risk does bi-directional introduce?**
Single-directional filtering means a filter on the "one" side (dimension) propagates to the "many" side (fact), which is the standard/recommended default. Bi-directional (both-ways) filtering also lets a filter on the fact table propagate back up to the dimension, which can enable certain many-to-many or role-playing scenarios but risks ambiguous filter paths, circular dependency errors, and unintended cross-filtering between unrelated visuals if applied broadly instead of scoped via `CROSSFILTER()` in a specific measure.

**Q6. How does Row-Level Security (RLS) work in Power BI, and how do you implement dynamic RLS based on the logged-in user?**
RLS is implemented via Roles defined in Power BI Desktop with a DAX filter expression applied to a table (e.g., `[Region] = "West"`), and users are assigned to roles in the Power BI Service. Dynamic RLS uses a function like `USERPRINCIPALNAME()` compared against a table mapping users to allowed dimension values (e.g., an Employee-to-Region mapping table), so a single role's filter expression dynamically restricts data per logged-in user without needing a separate static role per user.

**Q7. What is the role of an On-Premises Data Gateway, and what's the difference between Personal and Standard mode?**
The gateway securely bridges Power BI Service (cloud) to on-premises data sources (SQL Server, file shares, on-prem SharePoint) for scheduled refresh and DirectQuery without exposing the data source directly to the internet. Personal mode gateways support only that single user's scheduled refresh for their own reports and can't be shared; Standard mode gateways are installed centrally, can be shared across the organization for multiple users/reports/datasets, and support DirectQuery/live connections, which Personal mode does not.

**Q8. Explain Aggregations and Incremental Refresh, and why they matter for large datasets.**
Aggregations let you pre-summarize a large fact table (e.g., daily/monthly totals) into a smaller in-memory table that Power BI automatically uses to answer high-level queries faster, falling back to the detailed DirectQuery table only when a visual needs row-level detail. Incremental Refresh partitions a large table by date ranges so only recent partitions (e.g., last 7 days) are refreshed each time instead of reprocessing the entire history, drastically reducing refresh duration and gateway/source load for very large fact tables.

**Q9. What are Power BI Deployment Pipelines, and how do they support ALM for BI content?**
Deployment Pipelines let you manage Dev → Test → Production stages for a Power BI workspace's content (reports, datasets, dataflows), allowing you to compare differences between stages, deploy content forward with a click, and optionally rebind data sources/parameters per stage (e.g., pointing Test to a test SQL database and Production to the production one) — bringing structured release management to BI artifacts similar to how solutions manage Power Apps/Automate ALM.

**Q10. What is the difference between a Dataflow (Power Query Online) and a standard Power BI dataset, and when would you use a Dataflow?**
A Dataflow is a reusable, centrally managed ETL layer (built with Power Query Online, stored in Dataverse/CDM-format storage in a workspace or Dataverse) that multiple Power BI datasets/reports (or even Power Apps) can consume as a common source, avoiding each report re-implementing the same transformation logic. Use Dataflows when multiple reports/authors need the same cleaned/shaped data (e.g., a shared "Cleaned Sales" table) to ensure consistency and reduce duplicated refresh load on the source system.

---

## 11. Scenario-Based Questions (Cross-Topic)

**Scenario 1. Your Canvas App works fine in testing with 200 rows in a SharePoint list, but in production (12,000 items) users report that filters return incomplete/wrong results. What's happening and how do you fix it?**
This is a delegation issue — SharePoint's non-delegable query limit truncates the working set at ~500/2000 records before your `Filter`/`Search` formula ever sees the rest of the data. Fix by: rewriting formulas to use only delegable operators/functions for the connector, indexing the filtered columns in SharePoint, considering migrating the list to Dataverse (much higher delegation limits and richer querying), or, if the list must stay in SharePoint, pre-filtering server-side via a Power Automate flow/scheduled sync into a smaller working table.

**Scenario 2. A tenant-wide SPFx Application Customizer that adds a compliance banner is suddenly causing multiple sites to load slowly after an update. How do you diagnose and roll back safely?**
Diagnose using browser DevTools Performance tab / Page Diagnostics tool on an affected page to identify whether the new version is blocking on a slow API call in `onInit()`, then check the App Catalog for the deployed `.sppkg` version and use "Replace" to redeploy the previous known-good package version (keeping the same solution ID) as an immediate rollback, buying time to fix and re-test the new version (e.g., moving the slow call off the critical render path, adding caching) before redeploying tenant-wide, ideally validating first against a pilot site collection.

**Scenario 3. Finance wants an approval process where an invoice over $10,000 needs both Manager and Finance Director sign-off (in sequence), but under $10,000 only needs Manager approval — built on a SharePoint list with a Power Automate flow. Design the flow.**
Trigger on item create/modify in the SharePoint list; add a `Condition` checking `Amount >= 10000`; in the "Yes" branch, run a sequential "Start and wait for an approval" to the Manager, then (only if approved) a second "Start and wait for an approval" to the Finance Director, updating the SharePoint status column after each stage; in the "No" branch, run just the Manager approval. Use "Configure run after" so a Manager rejection short-circuits to an "Rejected" status update without proceeding to the Finance Director step, and log each decision (approver, comments, timestamp) back to the list or a related "Approval History" list for auditability.

**Scenario 4. You need to migrate a legacy InfoPath form + SharePoint 2013 workflow (multi-stage HR onboarding) to the modern platform. What's your target architecture?**
Replace InfoPath with a Power Apps Canvas App (embedded in the SharePoint list via the "PowerApps" customized form option) or a Model-Driven App if migrating the data to Dataverse for richer relational modeling of onboarding tasks; replace the SharePoint 2013 workflow with a solution-aware Power Automate Cloud Flow handling the multi-stage approvals/notifications (using Approvals connector, Teams notifications, and conditional branching by department); if the process needs a guided step-by-step experience for HR staff, add a Dataverse Business Process Flow on top. Package everything (app, flow, Dataverse tables/connection references) into a Dataverse Solution for proper Dev→Test→Prod ALM going forward.

**Scenario 5. Users report that a Model-Driven App form is slow to load, especially a specific tab with a sub-grid. How do you troubleshoot?**
Use the browser's Performance tooling combined with Dataverse's built-in Form/Performance diagnostics (`?flags=RealtimeaudioOff` style diagnostics or F12/Network tab filtering `api/data` calls) to see if a synchronous plugin on Retrieve, an unindexed/overly broad sub-grid view (fetching too many columns/related records without a filter), or an inefficient JavaScript form `OnLoad` script (e.g., unnecessary `Xrm.WebApi` calls run synchronously) is the culprit; common fixes include adding server-side filtering/paging to the sub-grid view, converting synchronous form-load web-resource calls to async, and reviewing plugin registration steps for unnecessary synchronous, unfiltered attribute-triggered registrations.

**Scenario 6. A customer-facing Power Pages site accidentally exposed internal-only Dataverse records to anonymous visitors. What went wrong and how do you prevent recurrence?**
Almost certainly a Table Permission was scoped to "Global" (or a Web Role/permission was granted to an overly broad role like "Anonymous Users") for a table that should have been scoped to "Contact"/"Account"-owned records only; immediately correct the Table Permission scope and audit all other tables' permissions for similarly broad scopes. To prevent recurrence, adopt a least-privilege default (deny by default, explicitly grant only what's needed per Web Role), require peer review/testing of Table Permission changes before production deployment, and periodically run a security audit/checklist against the Power Pages security model as part of change management.

**Scenario 7. Leadership wants a single Power BI report combining near-real-time sales data (changes every few minutes) with a large historical warehouse (5 years of data, rarely changes). How do you architect the dataset?**
Build a Composite model: set the large, slow-changing historical fact/dimension tables to Import mode (fast queries, refreshed nightly), and set the near-real-time sales table to DirectQuery (or Import with a very frequent scheduled/incremental refresh if DirectQuery performance on the source is a concern); use Dual storage mode on shared dimension tables so they work efficiently regardless of which mode a given visual's fact table uses, and apply Incremental Refresh on the historical fact table so nightly refreshes only touch recent partitions rather than reprocessing 5 years of history.

**Scenario 8. A Power Automate Desktop unattended flow that used to work reliably starts failing intermittently after a target application's UI was updated. How do you make the automation more resilient long-term?**
Short-term, update/re-record the broken Selectors against the new UI. Long-term, reduce reliance on brittle, absolute UI selectors: use UIA-based relative/anchor selectors where possible, add "Wait for element" with reasonable timeouts instead of hardcoded `Delay`s, wrap steps in "On Block Error" with retry logic and screenshot-on-failure for diagnostics, and where feasible, evaluate whether the target system has since exposed an API — if so, migrate that portion of the automation to a Cloud Flow connector instead of continuing to depend on fragile UI automation.

**Scenario 9. Your organization wants a Copilot Studio bot to answer HR policy questions using existing SharePoint HR documents, but must ensure an employee never sees a policy document from a department they don't have SharePoint access to. How do you design this?**
Configure the SharePoint site/document library as a Generative AI Knowledge source in Copilot Studio, and critically ensure the agent is set up to run with the *end user's* identity context (not a generic service account) so that Microsoft Search's security trimming applies at query/grounding time — meaning the LLM can only retrieve and cite content the actual asking user is already permitted to see in SharePoint. Additionally, test with multiple test accounts holding different SharePoint permissions to validate no over-exposure occurs before broadly publishing the bot, and apply DLP policies restricting which connectors/actions the bot's underlying flows can use.

**Scenario 10. A SPFx web part needs to call three different SharePoint/Graph endpoints, and the current implementation calls them sequentially inside `componentDidMount`, causing a 3–4 second load delay. How do you optimize it?**
Refactor to fire all three independent calls concurrently (e.g., `Promise.all([...])` instead of sequential `await`s), move the calls into `onInit()`/constructor level where appropriate to start fetching before render rather than waiting for mount, add loading skeletons so perceived performance improves even if total time doesn't drop to zero, and consider PnPjs's caching (`usingCaching()`) if any of the three calls return relatively static data that doesn't need to be re-fetched on every render/navigation.

**Scenario 11. A Power Automate cloud flow that processes uploaded invoices via AI Builder Form Processing occasionally extracts wrong totals for a new vendor's invoice layout. How do you fix and prevent this?**
The custom Form Processing model likely wasn't trained on this vendor's specific layout — add several labeled sample documents of the new layout to the model's training set and retrain/republish it (Form Processing models are layout-specific and need representative samples per distinct template). For prevention, add a confidence-threshold check in the flow (AI Builder returns confidence scores per field) routing low-confidence extractions to a human-review step (e.g., a Power Apps approval screen) instead of blindly trusting every extraction, and establish a process to periodically retrain the model as new vendor layouts are onboarded.

**Scenario 12. You're asked to design an ALM strategy for a Power Platform solution spanning a Model-Driven App, three Cloud Flows, and a Canvas App shared across Dev, Test, and Production environments. Outline your approach.**
Build everything inside a single (or logically grouped) Dataverse Solution in the Dev environment, using Environment Variables for all environment-specific config (URLs, thresholds) and Connection References for every connector so no hardcoded/personal connections are embedded; export as a Managed Solution and promote via Power Platform Pipelines (or Azure DevOps/GitHub Actions using the Power Platform Build Tools) into Test, running automated or manual validation, then promote the same managed solution package into Production; maintain source control of the unpacked solution (`pac solution unpack`) in a Git repo for versioning, code review, and rollback capability, rather than relying solely on the maker portal's export history.

---

## Contributing
Found an issue or want to add more questions? Feel free to open a pull request or an issue.

## License
This content is provided for educational and interview-preparation purposes.
