# SPFx (SharePoint Framework) — Interview Questions

⬅ Prev [SharePoint](./01-sharepoint.md) &nbsp;|&nbsp; [⬅ Back to Index](../index.html) &nbsp;|&nbsp; [Repo Home](../README.md) &nbsp;|&nbsp; Next ➡ [Power Apps – Canvas](./03-power-apps-canvas.md)

Covers client-side web parts, extensions, tooling, ALM, and performance for the SharePoint Framework.

## Contents
- [🟢 Beginner](#-beginner)
- [🔵 Intermediate](#-intermediate)
- [🟠 Advanced](#-advanced)
- [🔴 Expert](#-expert)
- [⚡ Quick-Fire Round](#-quick-fire-round)
- [🧩 Scenario-Based](#-scenario-based)

---

## 🟢 Beginner

**Q1. What is SPFx?**
The SharePoint Framework is Microsoft's client-side development model for building web parts and extensions that run directly inside the SharePoint page (and also in Teams/Outlook), using standard web technologies like TypeScript, React, and npm.

**Q2. What are the two main types of components you can build with SPFx?**
Web Parts (visible, page-embeddable UI blocks users add to a page) and Extensions (Application Customizers, Field Customizers, and Command Sets, which inject behavior rather than living inside the page canvas).

**Q3. What toolchain is used to scaffold a new SPFx project?**
The Yeoman generator for SharePoint (`@microsoft/generator-sharepoint`), run via `yo @microsoft/sharepoint`, which scaffolds the project structure, `package.json`, and boilerplate manifest/config files.

**Q4. What is the SharePoint Workbench, and what's the difference between local and hosted workbench?**
The Workbench is a canvas page for testing SPFx web parts outside a real SharePoint page during development. The Local Workbench (`gulp serve`, `localhost` workbench page) runs entirely locally with mocked context, while the Hosted Workbench runs on an actual SharePoint site (`/_layouts/15/workbench.aspx`) giving access to real SharePoint context and data.

**Q5. What is the `.sppkg` file, and where do you deploy it?**
The `.sppkg` is the packaged SPFx solution (manifest, feature definitions, permission requests) produced by `gulp package-solution`. It's uploaded to a SharePoint App Catalog (tenant-wide or site collection-scoped) for distribution and installation.

**Q6. What language and framework does SPFx primarily use?**
TypeScript is the primary language; React is the most common UI framework used (though SPFx is UI-framework agnostic and can be used with plain JS/HTML, Vue, or Angular).

---

## 🔵 Intermediate

**Q1. Walk through the SPFx web part lifecycle methods.**
`onInit()` runs once when the component loads (good for context setup, initial data fetch); `render()` paints the DOM and re-runs on every property pane change or re-render trigger; `onDispose()` runs on teardown to clean up subscriptions/DOM listeners; property-pane-specific hooks like `onPropertyPaneFieldChanged()` fire before `render()` is re-triggered by a property change.

**Q2. Differentiate an Application Customizer, a Field Customizer, and a Command Set.**
An Application Customizer injects UI or logic into predefined page placeholders (Top/Bottom) or runs page-wide logic without any visible placeholder, applied tenant/site-wide. A Field Customizer changes how a specific column renders within list/library views (e.g., color-coding a status column). A Command Set adds custom buttons to the command bar or context menu for list items, wiring custom actions to selected items.

**Q3. How does SPFx authenticate calls to Microsoft Graph or SharePoint REST without the developer managing OAuth tokens manually?**
SPFx components use the web part context's `AadHttpClientFactory`/`MSGraphClientFactory` (or `SPHttpClient` for SharePoint REST), which silently acquires a token for the already-signed-in user via the SharePoint page's SSO session, provided the required API permissions have been approved in the SharePoint Admin Center's "API access" panel.

**Q4. What is the purpose of the CDN configuration (`write-manifests.json` / Office 365 CDN or Azure Storage) in an SPFx solution?**
Production SPFx bundles (JS/CSS produced by `gulp bundle --ship`) are hosted on a CDN rather than embedded directly in the `.sppkg`, so the package manifest references CDN URLs; this improves load performance (CDN caching/edge delivery) and lets you update assets without repackaging/redeploying the `.sppkg` for minor asset changes when configured that way.

**Q5. What is the difference between tenant-wide deployment and per-site deployment of an SPFx app?**
Tenant-wide deployment (checked in the App Catalog) automatically installs the solution on every site in the tenant without requiring a site owner to add it manually — mainly used for global Extensions. Per-site deployment requires a site owner/admin to explicitly add the app from the site's "Add an app" page, giving more granular control over where a customization is active.

**Q6. How do you debug a locally running SPFx component against a real SharePoint page?**
Run `gulp serve --nobrowser`, then navigate to the target modern page and append `?debug=true&noredir=true&debugManifestsFile=https://localhost:4321/temp/manifests.js` to the URL, which tells the real page to load your locally-served (unbundled, source-mapped) component code for live debugging in DevTools.

**Q7. What is PnPjs, and why is it commonly used in SPFx projects instead of raw REST calls?**
PnPjs is a fluent, TypeScript-friendly JavaScript library (community-maintained under the Microsoft 365 & Power Platform Community) that wraps SharePoint REST and Microsoft Graph calls with a chainable, strongly-typed API, built-in caching, batching, and error handling — reducing boilerplate versus hand-writing raw `fetch`/`SPHttpClient` calls.

---

## 🟠 Advanced

**Q1. How would you safely roll out an update to a tenant-wide Application Customizer that touches every page in the tenant?**
Bump the version in `package-solution.json` while keeping the same solution ID, test thoroughly in a sandbox tenant or a pilot site collection first (deploy the package there as a non-tenant-wide app to validate), use a feature flag or environment variable check inside the customizer's logic to allow toggling new behavior on/off without a full redeploy if something goes wrong, and monitor Page Diagnostics/telemetry after the tenant-wide "Replace" deployment to catch regressions quickly, keeping the prior `.sppkg` version on hand for immediate rollback.

**Q2. Explain how SPFx solutions can share code/state across multiple independently-deployed web parts on the same page.**
Options include: a Library Component (a non-visual SPFx component exposing shared services, published separately and consumed as a dependency by multiple web part solutions), the browser's own `localStorage`/`sessionStorage`/custom events (`window.dispatchEvent`) for loosely-coupled cross-component messaging on the same page, or a shared Redux/Context store if the web parts are built as part of the same solution and bundle. Cross-solution (separately packaged) sharing is trickier and usually relies on custom events or a Library Component rather than shared in-memory state.

**Q3. How do you handle a scenario where an SPFx web part needs to call a third-party external API (not Graph/SharePoint) securely, without exposing a secret client-side?**
Register the external API behind an Azure AD app registration and use SPFx's `AadHttpClient` with a custom resource/audience if the external API supports Azure AD auth; if it doesn't (e.g., needs an API key), proxy the call through an Azure Function or Azure API Management instance that holds the secret server-side, and have the SPFx component call that secured intermediary (authenticated via AAD) instead of embedding any credential in client-side code, since anything shipped to the browser is inherently visible to the end user.

**Q4. What governance/DLP-like controls exist for what SPFx solutions can do in a tenant, and how would you enforce a review process before tenant-wide deployment?**
SharePoint Admin Center's "API access" approval gate ensures no SPFx solution can call Graph/AAD-protected APIs without explicit admin approval of the requested permission scopes; beyond that, org governance is largely process-based — requiring code review, a staging App Catalog for UAT validation, and a documented approval workflow (e.g., a change advisory board) before a package is promoted to the production tenant App Catalog and marked tenant-wide, since SharePoint itself doesn't provide a built-in formal approval gate for the package content itself (only for its declared API permissions).

**Q5. What are common causes of memory leaks in long-lived SPFx Application Customizers, and how do you prevent them?**
Not removing event listeners or intervals/timeouts registered in `onInit()` when `onDispose()` is called (or not implementing `onDispose()` cleanup at all), holding references to DOM nodes/placeholders after the page has navigated away (SharePoint's PnP-style page navigation is often SPA-like, so customizers persist across page changes within a session), and subscribing to external State/Redux stores without unsubscribing. Prevention: always pair every subscription/listener/timer in `onInit()` with a matching teardown in `onDispose()`, and test by navigating across multiple pages in a session while watching the browser's memory profiler for growth.

---

## 🔴 Expert

**Q1. Design an SPFx-based extensibility platform for a large enterprise where dozens of teams need to build and deploy their own web parts, without a single team becoming a deployment bottleneck, while still enforcing security and brand consistency.**
Establish a shared "component library" solution (published once, tenant-wide) providing common UI primitives (branded buttons, headers, theming helpers) that all team solutions consume as a dependency, so visual consistency is enforced by reuse rather than review; delegate a Site Collection App Catalog to each major department/hub so their teams can deploy their own web parts to their own sites without needing Global/SharePoint Admin approval for every release, while reserving the tenant App Catalog and tenant-wide deployment specifically for cross-cutting Extensions (like a compliance banner) that require centralized governance; and enforce the security boundary through the "API access" approval process (still centrally gated) plus automated CI/CD linting/security scanning (e.g., checking for hardcoded secrets, disallowed external domains) before any package reaches even the delegated catalogs.

**Q2. How would you architect an SPFx solution that must function consistently across SharePoint pages, Microsoft Teams tabs, and Outlook, sharing a majority of the codebase?**
Build the core logic and UI as SPFx components using the cross-host-compatible APIs (Microsoft Graph via `MSGraphClientV3`, and SPFx's built-in Teams context detection via `this.context.sdks.microsoftTeams`), abstracting any host-specific behavior (e.g., detecting `sdks.microsoftTeams` presence to adjust layout/interactions for Teams' narrower tab real estate) behind a thin adapter layer, and configure the manifest's supported hosts (`SharePointWebPart`, `TeamsTab`, `TeamsPersonalApp`) appropriately; test each host context independently since subtle differences (theming tokens, available context properties, iframe sizing behavior in Teams) can cause regressions that don't appear when testing only in SharePoint.

**Q3. Explain the tradeoffs of building complex, stateful business logic entirely client-side in SPFx versus offloading it to Azure Functions/a backend service, from a security, performance, and maintainability perspective.**
Client-side-only logic in SPFx is simpler to deploy (no separate backend to manage) and has zero additional infrastructure cost, but exposes all logic and any embedded configuration to the browser (unsuitable for anything requiring secrets or where business rules must not be tamperable/visible), can't easily do heavy computation without impacting the user's browser performance, and can't be reused outside the SharePoint/Teams/Outlook host context. Offloading to Azure Functions centralizes and secures business logic/secrets, enables reuse across multiple front-ends (SPFx, Power Apps, mobile), and can scale compute independently, at the cost of added latency (network round-trip), additional infrastructure to secure/monitor/deploy, and a slightly more complex overall architecture requiring careful auth (Azure AD token validation) between the SPFx client and the Function.

**Q4. A tenant-wide SPFx Application Customizer intermittently fails to load only for a subset of users, seemingly at random, and only in Microsoft Teams (not directly in SharePoint). How would you systematically root-cause this?**
Start by confirming whether the affected users share a common trait (specific Teams client version, specific browser/WebView engine on desktop Teams, a specific Conditional Access policy applying only to them, or a specific network/proxy); check whether the customizer's `onInit()` makes any assumption about `this.context.sdks.microsoftTeams` being immediately available (Teams' context injection can have timing differences versus a native SharePoint page load) and add defensive null-checks/retry logic around Teams-specific context access; review whether a CDN geo-routing or caching inconsistency could be serving a stale/broken bundle version to a subset of users based on their region; and use Application Insights/telemetry (if instrumented) or ask affected users for browser console errors correlated with their Teams client build to narrow whether it's a timing race condition, a permission/consent gap for Teams specifically, or a CDN delivery issue.

---

## ⚡ Quick-Fire Round

- **Q: What command bundles an SPFx solution for production?** → `gulp bundle --ship`.
- **Q: What command packages the solution into a `.sppkg`?** → `gulp package-solution --ship`.
- **Q: What identifies an SPFx solution uniquely across versions/updates?** → The Solution ID (a GUID in `package-solution.json`), which must stay constant across updates.
- **Q: What client-side framework is most commonly paired with SPFx?** → React.
- **Q: What SPFx object type lets you add a custom button to the SharePoint command bar?** → A Command Set (Extension).
- **Q: What must be approved before an SPFx solution can call a Graph API requiring delegated permissions?** → The permission request in SharePoint Admin Center → API access.
- **Q: What is the local development test surface called before deploying to a real site?** → The SharePoint Workbench.
- **Q: What npm-based tool scaffolds a new SPFx project?** → Yeoman (`yo @microsoft/sharepoint`).
- **Q: True/False: SPFx solutions run inside an iframe like the old Add-in model.** → False — SPFx runs in the same page context/DOM, no iframe boundary.
- **Q: What library provides fluent, typed wrappers over SharePoint/Graph REST calls commonly used in SPFx?** → PnPjs.

---

## 🧩 Scenario-Based

**Scenario 1. A tenant-wide Application Customizer update caused multiple sites to load noticeably slower. How do you diagnose and safely roll back?**
Use browser DevTools' Performance/Network tab or the SharePoint Page Diagnostics Tool on an affected page to pinpoint whether the new version added a slow synchronous call in `onInit()`; if confirmed, redeploy the previous known-good `.sppkg` version via "Replace" in the App Catalog (same solution ID) as an immediate rollback, then fix the root cause (e.g., move the slow call off the critical path, add caching) and re-validate in a pilot site collection before redeploying tenant-wide again.

**Scenario 2. Your SPFx web part needs to display data from three different sources (SharePoint list, Graph calendar, and an external API) and currently takes 3–4 seconds to load because calls run sequentially in `componentDidMount`. How do you optimize?**
Fire all independent calls concurrently using `Promise.all([...])` instead of sequential `await`s, move data-fetch kickoff as early as possible (constructor/`onInit()` rather than waiting for full component mount), show a loading skeleton so perceived performance improves immediately, and apply PnPjs caching (`usingCaching()`) for any of the three sources that doesn't need to be freshly fetched on every render.

**Scenario 3. Business wants a custom "Approve/Reject" button added to the command bar only for documents with a specific Content Type, visible only to members of a specific SharePoint group. How do you build this with SPFx?**
Build a Command Set Extension, configure its manifest to bind to the target list(s), and implement `onListViewUpdated()` to dynamically show/hide the command based on the selected item's Content Type field and the current user's group membership (checked via a Graph/SharePoint REST call to the group's membership, cached per session to avoid repeated calls); wire the command's `onExecute()` to call a Power Automate flow (via HTTP trigger) or update the item directly via `SPHttpClient`/PnPjs to record the approval decision.

**Scenario 4. After deploying an SPFx solution tenant-wide, a small number of sites report the web part simply doesn't appear at all, while most sites work fine. What would you check?**
Check whether those specific sites are on a "Targeted Release"/different SharePoint update ring causing a temporary version mismatch, whether those sites have custom CSP (Content Security Policy) or tenant-level "restricted apps" settings blocking the CDN domain hosting the bundles, whether those particular sites have the app explicitly "removed" or blocked at the site level despite tenant-wide deployment (site owners can remove a tenant-wide-deployed app from their specific site), and whether those sites' users are hitting a stale browser cache still referencing an old, now-invalid bundle path — a hard refresh or cache-busting version bump in the manifest often resolves the latter.

