# SharePoint — Interview Questions

[⬅ Back to Index](../index.html) &nbsp;|&nbsp; [Repo Home](../README.md) &nbsp;|&nbsp; Next ➡ [SPFx](./02-spfx.md)

Covers SharePoint Online architecture, administration, governance, and out-of-box customization — organized by difficulty.

## Contents
- [🟢 Beginner](#-beginner)
- [🔵 Intermediate](#-intermediate)
- [🟠 Advanced](#-advanced)
- [🔴 Expert](#-expert)
- [⚡ Quick-Fire Round](#-quick-fire-round)
- [🧩 Scenario-Based](#-scenario-based)

---

## 🟢 Beginner

**Q1. What is SharePoint Online, and how is it different from SharePoint Server (on-premises)?**
SharePoint Online is the cloud-hosted, multi-tenant version of SharePoint delivered as part of Microsoft 365, managed and patched by Microsoft. SharePoint Server is installed and maintained on-premises (or in a customer's own Azure VMs), giving full infrastructure control but requiring the organization to handle patching, scaling, and hardware.

**Q2. What is a Site Collection, and how does it relate to a Site?**
A Site Collection is the top-level container with its own storage quota, unique permissions boundary, and unique URL root; it contains one root Site and can contain subsites beneath it. In modern SharePoint, most new sites are created as their own Site Collection rather than nested subsites, to avoid inheritance and migration complexity.

**Q3. What's the difference between a List and a Library?**
A List stores structured rows of data (like a spreadsheet), while a Library is a specialized list optimized for storing files, with additional file-specific features like check-in/check-out, versioning, and co-authoring.

**Q4. What is a View in SharePoint, and why would you create multiple views on the same list?**
A View is a saved configuration of columns, filters, sorting, and grouping applied to a list/library. Multiple views let different users or purposes see the same underlying data differently — e.g., "My Items," "Overdue Tasks," "By Department" — without duplicating data.

**Q5. What is versioning, and what's the difference between Major and Minor versions?**
Versioning keeps a history of changes to a list item or file so you can view or restore earlier versions. Major versions (1.0, 2.0) represent published/final states, while Minor versions (0.1, 0.2) represent drafts — minor versioning is only available when content approval or Major/Minor versioning is explicitly enabled on the library.

**Q6. What is a SharePoint Group, and how does it relate to permission levels?**
A SharePoint Group (e.g., "Site Members," "Site Owners," "Site Visitors") is a collection of users to which you assign one or more Permission Levels (like Read, Edit, Full Control). Managing access through groups instead of assigning permissions to individuals directly is best practice for maintainability.

**Q7. What is the OneDrive for Business relationship to SharePoint?**
OneDrive for Business is technically built on the same SharePoint platform — each user's OneDrive is provisioned as its own personal Site Collection, using the same list/library engine, permissions model, and APIs as any other SharePoint site.

---

## 🔵 Intermediate

**Q1. Explain the difference between a Communication Site and a Team Site, and when to use each.**
A Team Site is tied to a Microsoft 365 Group (shared mailbox, calendar, Planner) and is built for collaborative work by a defined team. A Communication Site has no Group by default and is designed for one-to-many broadcast — news, reports, dashboards — to a broad audience who mostly consume rather than co-author content.

**Q2. What is permission inheritance, and what happens when you break it?**
By default, subsites/libraries/items inherit their parent's permissions. Breaking inheritance creates an independent permission set for that object going forward — useful for restricting a specific folder or item — but it increases governance overhead since that object no longer receives future changes made at the parent level automatically.

**Q3. What are Site Columns and Content Types, and why prefer them over ad hoc local columns?**
Site Columns are centrally defined, reusable field definitions; Content Types bundle site columns (plus behaviors) into a reusable schema, like "Contract" or "Invoice." Using them ensures consistent metadata across multiple lists/libraries, and changes made at the source propagate to everywhere the column/content type is used.

**Q4. What is a Hub Site, and what problem does it solve?**
A Hub Site associates multiple independent site collections under shared navigation, branding, and search scope, without changing their underlying security/storage boundaries. It solves the problem of wanting a "family" of related sites (e.g., all HR sites) to feel connected in navigation and search without forcing them into a rigid subsite hierarchy.

**Q5. How does SharePoint search index custom metadata, and what's the difference between crawled and managed properties?**
Crawled properties are the raw metadata SharePoint discovers during a crawl (e.g., from a column). Managed properties are the indexed, query-ready mapping of a crawled property (or several) that you can search on, sort by, or use as a refiner — you must map a crawled property to a managed property before it becomes searchable/query-able.

**Q6. What are the List View Threshold and why does it matter for large lists?**
The List View Threshold (default 5,000 items) limits how many items a single query/view can process at once for non-admin operations; queries against unindexed columns on lists larger than this threshold will fail or be blocked. Designing around it means adding indexed columns, using filtered views, and avoiding "Show all items" views on large lists.

**Q7. What's the difference between SharePoint REST API and Microsoft Graph API, and when would you use each?**
SharePoint REST (`_api/web/...`) exposes deep, SharePoint-specific object model features. Microsoft Graph is the unified, cross-product API (Teams, Outlook, OneDrive, SharePoint) and is Microsoft's strategic direction for new development. Use Graph for cross-service scenarios and modern app registrations; use SharePoint REST when you need a SharePoint-only capability not yet available in Graph.

**Q8. How do retention labels and retention policies differ from sensitivity labels?**
Retention labels/policies (from Microsoft Purview) govern how long content is kept and whether it's automatically deleted or preserved (records management, compliance holds). Sensitivity labels govern access protection (encryption, watermarking, access restriction based on classification like "Confidential"). They serve different governance goals and can be applied together.

---

## 🟠 Advanced

**Q1. Design a governance model for a global organization migrating from a flat, ungoverned SharePoint tenant to a hub-based structure. What decisions do you need to make?**
Key decisions: hub taxonomy (by department, region, or business unit — usually department/function works better for navigation than region), site provisioning process (self-service vs. approval workflow via Power Automate/PnP provisioning), naming conventions and URL standards, default sharing settings (external sharing allowed per hub or tenant-wide), storage quota policy, and a lifecycle policy (site retention/expiration via Microsoft 365 Groups expiration policies) to avoid unbounded sprawl.

**Q2. How would you diagnose and resolve API throttling (429/503 errors) affecting a custom integration hitting SharePoint at scale?**
Inspect response headers (`Retry-After`) to implement proper exponential backoff rather than fixed retry intervals; batch requests using `$batch` (REST) or Graph batching to reduce total call count; review whether the integration is making per-item calls in a loop that could be replaced with a single filtered/paged query; and check the Microsoft 365 admin center / Graph API reporting for patterns (a specific app ID or user consistently triggering throttling) to isolate the root cause versus general tenant load.

**Q3. What's the architectural difference between Modern and Classic experiences, and what does that mean for migrating legacy Full Trust Solutions/Farm Solutions?**
Modern pages/web parts use a client-rendered, Fluent UI-based, SPFx-compatible model that runs in the page's own context without server-side custom code deployment; Classic relies on server-side rendering, master pages, and Farm Solutions (compiled, deployed to the SharePoint farm/tenant with elevated trust). Since Farm Solutions and Sandbox Solutions are largely unsupported/deprecated in SharePoint Online, migrating them requires re-architecting the logic into SPFx (client-side), Power Automate (workflow logic), or Azure Functions (backend logic) — there's no direct lift-and-shift path.

**Q4. Explain how eDiscovery and Litigation Hold interact with SharePoint content, and the performance/storage implications.**
When a site or specific content is placed under Litigation Hold or an eDiscovery hold, SharePoint preserves a copy of any version of a document even if a user tries to edit or delete it (using a hidden "preservation hold library"), which can significantly increase storage consumption in high-churn document libraries. Admins need to account for this in storage capacity planning and understand that hold-related preserved copies don't count against the same quotas as visible content in some scenarios, but do consume tenant storage overall.

**Q5. How would you design a multi-geo SharePoint tenant, and what are the key constraints?**
Multi-Geo lets an organization keep specific users'/sites' data resident in specific geographic regions (data residency compliance) while remaining part of a single tenant; you assign users a "Preferred Data Location" (PDL) and provision satellite geos. Key constraints include: some cross-geo scenarios have limitations (e.g., certain search and profile sync behaviors), it requires a Multi-Geo Capabilities add-on license, and site provisioning tooling/automation must be geo-aware to place new sites in the correct location.

---

## 🔴 Expert

**Q1. You're asked to design a tenant-wide search architecture where certain highly sensitive sites must be fully excluded from tenant-wide search results, but still searchable within their own site by authorized users. How do you achieve this?**
Configure a Result Source or a Search Schema-based approach isn't sufficient alone since default tenant search normally includes everything the querying user has permission to see; instead, use the "Exclude from search" site setting (or a search-based content organizer policy) at the site level combined with proper permission trimming so unauthorized users never see the content regardless, and validate using a Result Source/Verified custom query scope for the site's own search web part that explicitly targets that site collection ID even though it's excluded tenant-wide. Additionally, review whether Microsoft Search "acronyms/bookmarks/Q&A" admin-curated results could inadvertently surface hints about excluded content and adjust accordingly.

**Q2. How does the underlying SharePoint content database sharding and site collection storage model affect very large document repositories (multi-terabyte), and what design patterns mitigate performance degradation?**
SharePoint Online abstracts storage away from customers, but large single libraries still suffer from folder-level item count thresholds, view rendering costs, and search crawl freshness lag at scale; mitigation patterns include partitioning content across multiple site collections/libraries by business unit or year rather than one monolithic library, using metadata-driven navigation instead of deep folder hierararchies, applying indexed columns aggressively, and offloading historical/rarely-accessed content to a dedicated "Archive" site collection with tighter storage/retention policies.

**Q3. Walk through how Conditional Access Policies, SharePoint Online's "Unmanaged Devices" access control, and Microsoft Purview DLP policies interact to protect data on a highly sensitive site, and where the enforcement boundaries differ.**
Conditional Access (Entra ID) controls *whether a session is established at all* (e.g., block or require MFA/compliant device before login); SharePoint's site-level "Unmanaged Devices" access control (Limited Access / Block Download) governs what an already-authenticated session on an unmanaged device can *do* within SharePoint specifically (e.g., web-only access, block download); Purview DLP policies scan *content* for sensitive info types and can block sharing/uploading/downloading actions regardless of device state, acting as a content-aware layer on top of the identity- and device-aware layers. Designing defense-in-depth means layering all three rather than relying on any single control, since each addresses a different threat vector (identity, device posture, content sensitivity).

**Q4. In a merger/acquisition scenario, how would you architect a strategy to migrate one organization's SharePoint tenant content into another tenant with minimal disruption, addressing permissions, external sharing history, and customizations?**
Use a tenant-to-tenant migration tool (e.g., ShareGate, Sharegate/Metalogix, or Microsoft's Mailbox/SharePoint Migration Tool where applicable) with a phased approach: first migrate a pilot site collection to validate mapping of permissions (since user identities differ across tenants and must be remapped via a user/group mapping file), re-create or migrate SPFx customizations and Power Platform solutions separately (they aren't part of standard content migration), decide on external sharing link handling (most tools cannot preserve anonymous/guest sharing links as-is and require re-sharing post migration), and communicate a cutover plan (read-only freeze window, delta sync, DNS/redirect strategy for old URLs) to minimize business disruption.

---

## ⚡ Quick-Fire Round

- **Q: What does "SPO" commonly stand for?** → SharePoint Online.
- **Q: What is the maximum file size typically supported in SharePoint Online document libraries?** → 250 GB per file (subject to change; always verify current tenant limits).
- **Q: What feature lets multiple users edit a document simultaneously?** → Co-authoring.
- **Q: What's the tool used to move users/sites' data between tenants?** → A third-party migration tool (e.g., ShareGate) or Microsoft's SharePoint Migration Tool (SPMT) for content-only moves.
- **Q: What identifies a unique SharePoint site collection internally?** → Its Site Collection ID (a GUID), separate from its URL.
- **Q: Which feature automatically deletes/archives inactive Microsoft 365 Groups (and their connected Team Sites)?** → Microsoft 365 Groups Expiration Policy.
- **Q: What's the term for a document library feature that prevents two users from overwriting each other's edits when co-authoring isn't supported (e.g., legacy file types)?** → Check-out / Check-in.
- **Q: What tool lets admins see tenant-wide sharing and site activity reports?** → SharePoint Admin Center reports / Microsoft 365 usage reports.
- **Q: What's the default storage quota model for SharePoint Online in a Microsoft 365 tenant?** → Pooled storage across the tenant, calculated from licensed seats (base tenant storage + per-license allocation).
- **Q: What is "Site Design" combined with a "Site Script" used for?** → Automating the provisioning of consistent lists, libraries, branding, and content on new sites at creation time.

---

## 🧩 Scenario-Based

**Scenario 1. A department complains their document library becomes unusable ("This view is too large") once it passes a few thousand files organized in deep nested folders. How do you redesign it?**
Move from deep folder hierarchies to a flatter structure using indexed metadata columns (e.g., Department, Year, Document Type) combined with filtered/grouped views instead of folders, since folder depth doesn't scale well for browsing or bulk operations at that volume; add indexed columns on the fields used for filtering to stay under the List View Threshold, and consider Content Types with unique views per type (e.g., "Contracts" view, "Invoices" view) rather than one giant flat view of everything.

**Scenario 2. Legal has just issued a hold notice for a specific project's SharePoint site as part of litigation. What's your immediate action plan, and what should you communicate to the business?**
Apply a Litigation Hold (via Microsoft Purview eDiscovery) or a specific retention label/policy scoped to that site collection so that all versions of documents — even ones users try to edit or delete — are preserved in the hidden preservation hold library; communicate to the business that users can continue working normally (the hold is largely invisible to end users) but should not attempt to bulk-delete or "clean up" the site, and flag that storage consumption on that site may increase due to preserved historical versions.

**Scenario 3. A newly onboarded external auditing firm needs read-only access to a specific set of folders across three different site collections, without being able to browse anything else in the tenant. Design the access model.**
Create a dedicated SharePoint Group (or use Entra ID B2B guest accounts added to a Team/Group tied to that access) scoped only to the specific folders (breaking inheritance at the folder level in each of the three site collections and granting Read-only permission to that group), rather than granting site-level or tenant-level access; ensure tenant external sharing settings and Conditional Access policies for guest accounts (e.g., requiring MFA) are configured to secure the access appropriately, and periodically review/expire the access once the audit engagement ends.

**Scenario 4. Users report search results are stale — a document updated yesterday still shows old content in search previews and summaries. How do you troubleshoot?**
Distinguish between "the document isn't found at all" (a crawl/indexing issue, check the site's Search crawl log/health if using hybrid, though SPO usually auto-crawls near-real-time) versus "the document is found but shows old content" (usually a caching issue in the search results rendering, or a delay in reflecting changes given SharePoint Online's near-real-time but not instantaneous re-indexing SLA); verify actual index freshness using Microsoft Search & Intelligence reporting if available, check whether a Result Source/customization is caching old results, and if genuinely delayed beyond expected SLA, escalate via a Microsoft 365 admin support ticket since customers can't directly force a re-crawl in SharePoint Online.

