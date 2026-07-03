# Power Pages — Interview Questions

⬅ Prev [Copilot Studio](./08-copilot-studio.md) &nbsp;|&nbsp; [⬅ Back to Index](../index.html) &nbsp;|&nbsp; [Repo Home](../README.md) &nbsp;|&nbsp; Next ➡ [Power BI](./10-power-bi.md)

Covers external-facing site design, authentication, table permissions, and security for Power Pages (formerly Power Apps Portals).

## Contents
- [🟢 Beginner](#-beginner)
- [🔵 Intermediate](#-intermediate)
- [🟠 Advanced](#-advanced)
- [🔴 Expert](#-expert)
- [⚡ Quick-Fire Round](#-quick-fire-round)
- [🧩 Scenario-Based](#-scenario-based)

---

## 🟢 Beginner

**Q1. What is Power Pages?**
Power Pages is Microsoft's low-code platform for building secure, external-facing websites backed by Dataverse — used for customer, partner, or citizen self-service portals, form submissions, and knowledge-base style sites.

**Q2. How does Power Pages relate to the older "Power Apps Portals"?**
Power Pages is the evolved, standalone version of what was previously called Power Apps Portals — re-architected with its own dedicated design studio (Pages, Styling, Data, Security workspaces), though existing Portals apps continue to run and are manageable from the same modern studio.

**Q3. What is a Web Role in Power Pages?**
A Web Role is a group assigned to portal visitors (e.g., "Authenticated Users," "Customer," "Partner") that determines what they can access on the site, similar in spirit to security roles in Dataverse but scoped to the portal.

**Q4. What is a Table Permission?**
A Table Permission defines what CRUD (Create, Read, Update, Delete) access a given Web Role has on a specific Dataverse table, and at what scope (e.g., only their own records, or globally).

**Q5. What authentication options does Power Pages support for external users?**
Local Authentication (username/password managed within Dataverse) or external Identity Providers (Microsoft Entra ID, Entra External ID/B2C, or other OpenID Connect/SAML providers), letting customers or partners sign in with their own credentials.

---

## 🔵 Intermediate

**Q1. Explain how Web Roles and Table Permissions work together to control access.**
A visitor is assigned one or more Web Roles upon authentication (or defaults to an "Anonymous Users" role if not signed in); Table Permissions are then defined per Web Role, specifying exactly which tables that role can access and with what CRUD operations, at a defined scope (Global, Contact, Account, Parent, or a custom relationship-based scope) — this is the primary access-control layer for portal visitors, layered on top of (not replacing) Dataverse's own table-level security.

**Q2. What's the difference between "Global," "Contact," and "Parent" scope in a Table Permission?**
Global scope grants access to all records of that table regardless of ownership; Contact scope restricts access to only records directly related to the authenticated visitor's own Contact record; Parent scope grants access to records related through a specified parent relationship (e.g., all cases belonging to the visitor's associated Account), useful for B2B portal scenarios where a partner user should see their whole organization's records, not just their own.

**Q3. How would you expose a business process (like a support ticket approval) to external users while keeping sensitive logic secure?**
Build the data capture as a Basic Form or Advanced Form on a Power Pages page bound to a Dataverse table, then use a Power Automate flow (triggered by the Dataverse record's creation) to run the actual business logic (internal notifications, approvals, integrations) server-side rather than in any client-facing code, and reflect status back to the portal user via a status field the page displays — keeping sensitive logic in the more secure, centrally-governed Dataverse+Flow layer rather than exposed to the public-facing site.

**Q4. What is Liquid templating in Power Pages, and what is it used for?**
Liquid is an open-source templating language used within Power Pages page/web templates to dynamically render content — pulling in Dataverse data, conditionally showing/hiding sections, and looping over lists of records directly within the page's HTML, giving fine-grained control over rendered output beyond the drag-and-drop Studio components.

**Q5. What are Basic Forms, Advanced Forms, and Lists in Power Pages, and how do they differ?**
Basic Forms render a single Dataverse form (create/edit/read-only) as a portal page. Advanced Forms support a multi-step wizard-like experience across multiple forms/steps for more complex data capture. Lists render a filtered/sorted grid of Dataverse records, similar to a Model-Driven App view, but for external portal visitors.

---

## 🟠 Advanced

**Q1. A customer-facing Power Pages site accidentally exposed internal-only records to anonymous visitors. What likely went wrong, and how do you prevent recurrence?**
Almost certainly a Table Permission was scoped too broadly (e.g., "Global" scope granted to an overly permissive Web Role like "Anonymous Users") for a table that should have been Contact- or Account-scoped; correct the specific permission immediately, then audit every other table's permissions for similarly broad scopes. To prevent recurrence, adopt a least-privilege-by-default posture (explicitly grant only what's needed per Web Role, deny by default), require peer review of any Table Permission change before production deployment, and periodically run a security audit against the full permission model as part of routine change management.

**Q2. What performance and security considerations are unique to Power Pages compared to internal Model-Driven or Canvas apps?**
Since Power Pages sites are internet-facing, you must actively defend against anonymous abuse — enabling a Web Application Firewall (via Azure Front Door/CDN integration), CAPTCHA on public forms, and rate limiting on form submissions to prevent spam/scraping; you must be far more deliberate with Table Permissions since a misconfiguration can expose tenant data to the public internet rather than just internal staff; and you should consider output caching for largely static public content pages, since traffic patterns (potentially thousands of anonymous visitors) differ significantly from internal LOB apps with a known, limited user base.

**Q3. How would you design multi-tenant B2B partner access, where each partner organization should see only their own cases/orders, but users within a partner organization should see their whole company's records (not just their own)?**
Model each partner as an Account in Dataverse, with each partner user's Contact record related to that Account; configure Table Permissions with "Parent" scope on the relevant tables (Cases, Orders) so a partner user's access follows the relationship up to their Account rather than being limited to records they personally created — this correctly reflects the "see your whole company's data" requirement without needing per-user record sharing, while still fully isolating one partner Account's data from another's.

**Q4. How do you approach securing custom JavaScript/client-side logic added to Power Pages pages, given that all client-side code is visible to any visitor?**
Treat any client-side JavaScript as fundamentally public and untrusted from a security perspective — never embed secrets, API keys, or sensitive business logic that must not be tamperable directly in page scripts; use client-side code only for genuine UX enhancements (validation feedback, dynamic show/hide), and route any actual data mutation or sensitive logic through server-side-enforced mechanisms (Table Permissions, Dataverse plugins, or a Power Automate flow triggered by the resulting Dataverse record) so that even a malicious visitor manipulating the client-side script or bypassing it entirely cannot achieve anything the server-side permission model wouldn't already allow.

---

## 🔴 Expert

**Q1. Design a Power Pages architecture for a government citizen-services portal expected to handle high-traffic public form submissions (e.g., permit applications) during predictable peak periods (e.g., a filing deadline), with strict availability requirements.**
Plan capacity around Power Pages' page view/login-based licensing model and Dataverse API request limits well ahead of the known peak (a filing deadline creating a predictable traffic spike, unlike unpredictable viral traffic), and load-test the site against realistic peak volume beforehand; put a CDN/WAF (Azure Front Door) in front of the site both for security and to absorb/cache static asset load, reducing direct hits to the Dataverse backend for non-dynamic content; design form submissions to be resilient to backend slowness (e.g., queuing submissions via an asynchronous Power Automate flow rather than a synchronous, potentially slow direct Dataverse write blocking the user's browser during peak load), and have a clear incident/communication plan (a maintenance/status page) in case availability is impacted despite preparation, given the reputational and citizen-service impact of downtime during a filing deadline.

**Q2. A Power Pages site needs to integrate with a legacy on-premises payment/records system that has no cloud API, while maintaining PCI-relevant security boundaries for any payment data involved.**
Route any required on-premises connectivity through the on-premises data gateway from a Power Automate flow (never directly from client-side portal code, which would expose connectivity details/credentials publicly), and critically, avoid having Power Pages or its backing Dataverse tables directly store or transit raw payment card data at all — instead integrate with a PCI-compliant payment gateway/processor (tokenization-based) for the actual card handling, with Power Pages/Dataverse only storing the resulting transaction reference/token, keeping the portal and Power Platform components entirely out of PCI DSS's most stringent scope (cardholder data environment) by design rather than attempting to secure raw card data within a low-code platform not built or certified for that purpose.

**Q3. How would you approach a security review/penetration test finding that a Power Pages site's Table Permissions correctly restrict data access via the standard portal UI, but a direct call to the underlying Web API endpoint (used by the portal's own JavaScript for AJAX operations) returns data beyond what the UI displays?**
Recognize that Table Permissions are enforced at the API/data layer, not just the UI layer, in a properly configured site — so a genuine discrepancy like this typically indicates either a permission scope misconfiguration (broader than intended) that happens to not be exposed through the normal UI flow, or a custom Liquid/JavaScript component making an API call with elevated implicit context (e.g., inadvertently querying with a scope or filter that bypasses the intended restriction); systematically test the actual Web API endpoints directly (not just through the rendered UI) against every Web Role during security review, since UI-only testing can miss server-side/API-level permission gaps that are just as exploitable by a sufficiently technical malicious visitor inspecting network calls.

---

## ⚡ Quick-Fire Round

- **Q: What was Power Pages formerly called?** → Power Apps Portals.
- **Q: What defines which Dataverse tables a portal Web Role can access?** → Table Permissions.
- **Q: What scope would you use so a partner user sees their whole company's records, not just their own?** → Parent scope.
- **Q: What templating language is used for dynamic page content in Power Pages?** → Liquid.
- **Q: What component supports a multi-step, wizard-like form experience?** → An Advanced Form.
- **Q: What should never be embedded in client-side JavaScript on a Power Pages site?** → Secrets/API keys or logic that must not be tamperable.
- **Q: What's used to defend a public Power Pages site against bots/abuse?** → A Web Application Firewall (e.g., via Azure Front Door) and CAPTCHA.
- **Q: What role do unauthenticated visitors default to?** → Anonymous Users.
- **Q: What authentication modes does Power Pages support besides Local Authentication?** → External Identity Providers (Entra ID, Entra External ID/B2C, OpenID Connect, SAML).
- **Q: What component renders a filtered/sorted grid of Dataverse records to external visitors?** → A List.

---

## 🧩 Scenario-Based

**Scenario 1. A customer-facing Power Pages site accidentally exposed internal-only Dataverse records to anonymous visitors. What went wrong and how do you prevent recurrence?**
A Table Permission was almost certainly scoped too broadly (e.g., "Global" instead of "Contact"/"Account") for an overly permissive Web Role; correct the scope immediately and audit all other tables' permissions for the same pattern. Prevent recurrence with a least-privilege-by-default policy, mandatory peer review of permission changes before production deployment, and periodic security audits of the full Table Permission model.

**Scenario 2. A portal used by external auditors needs to let them view (read-only) specific financial records tied to the audit engagement, but never modify anything, and access must automatically expire when the engagement ends.**
Grant the auditors a dedicated "Auditor" Web Role with Table Permissions scoped to Read-only on only the relevant tables, filtered to records related to their specific engagement (e.g., via a custom "Engagement" lookup field used in the permission's scope/relationship); manage the time-bound nature of access through a scheduled Power Automate flow (or a manual/process-driven review) that deactivates the auditor's portal contact/Web Role assignment on the engagement's end date, rather than relying on manual, easily-forgotten offboarding.

**Scenario 3. Leadership wants to expose a complex multi-stage loan application process to external applicants via Power Pages, with each stage requiring different information and some stages requiring internal review before the applicant can proceed.**
Use an Advanced Form to guide applicants through the multi-step data capture (personal info, financial details, document upload) as sequential steps; upon each stage's submission, trigger a Power Automate flow to run the actual internal review/validation logic server-side and update a status field on the underlying Dataverse record; gate the applicant's ability to proceed to the next stage based on that status field (e.g., the portal page conditionally shows "waiting for review" versus the next step's form) so the internal review logic and any sensitive underwriting criteria never need to be exposed to the client-facing portal code.

