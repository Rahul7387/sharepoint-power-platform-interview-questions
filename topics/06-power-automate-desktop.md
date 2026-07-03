# Power Automate Desktop (RPA) — Interview Questions

⬅ Prev [Power Automate – Cloud](./05-power-automate-cloud.md) &nbsp;|&nbsp; [⬅ Back to Index](../index.html) &nbsp;|&nbsp; [Repo Home](../README.md) &nbsp;|&nbsp; Next ➡ [AI Builder](./07-ai-builder.md)

Covers UI automation, selectors, attended/unattended RPA, error recovery, and hybrid cloud+desktop patterns.

## Contents
- [🟢 Beginner](#-beginner)
- [🔵 Intermediate](#-intermediate)
- [🟠 Advanced](#-advanced)
- [🔴 Expert](#-expert)
- [⚡ Quick-Fire Round](#-quick-fire-round)
- [🧩 Scenario-Based](#-scenario-based)

---

## 🟢 Beginner

**Q1. What is Power Automate Desktop, and how is it different from Cloud Flows?**
Power Automate Desktop (PAD) is Microsoft's Robotic Process Automation (RPA) tool that automates interactions with desktop applications and legacy systems by simulating mouse clicks, keystrokes, and reading/writing UI elements — unlike Cloud Flows, which orchestrate cloud/API-based connectors rather than simulating a human using a screen.

**Q2. What is a "Selector" in Power Automate Desktop?**
A Selector is the set of properties (like Name, AutomationId, ControlType) used to uniquely identify and locate a specific UI element on screen so the automation can interact with it reliably.

**Q3. What's the difference between Attended and Unattended automation?**
Attended automation runs on a user's machine with the user present (triggered manually or from a cloud flow while logged in); Unattended automation runs without a human present, typically scheduled or triggered remotely via a registered machine and the on-premises data gateway.

**Q4. What's a common use case for Power Automate Desktop that Cloud Flows can't handle?**
Automating a legacy desktop application or a system with no API/connector — e.g., entering data into an old Windows accounting application that only has a UI, no exposed API.

**Q5. What action would you use to pause an automation until a specific window/element appears?**
"Wait for window" or "Wait for element" actions, which pause execution until the target UI element/window is detected, rather than using a fixed, potentially too-short or too-long, `Delay`.

---

## 🔵 Intermediate

**Q1. How do you handle errors and recovery in a Desktop Flow?**
Use "On Block Error" (the desktop-flow equivalent of try/catch) around risky steps to define retry logic, fallback paths, or graceful termination instead of letting the whole automation crash; combine this with "Wait for element/window" (rather than fixed delays) so the flow adapts to variable application load times instead of failing due to timing assumptions.

**Q2. Why do UI automations commonly break, and how do you make them more resilient?**
They break most often when the target application's UI changes — a new version, different screen resolution, or dynamically-generated control IDs invalidate previously recorded selectors. Resilience comes from favoring stable UI Automation (UIA) framework properties over brittle image-based recognition, using relative/anchor-based selectors where the framework supports it, and building in "Wait for element" patience rather than assuming instant UI readiness.

**Q3. What licensing considerations apply to Attended vs. Unattended automation?**
Attended automation typically runs under the license of the signed-in user triggering it (often included with standard Power Automate per-user plans). Unattended automation requires a machine to be registered (often via the on-premises data gateway) and a dedicated unattended RPA add-on/license tied to that machine, since it runs without a human actively present to "consume" a per-user session.

**Q4. When would you choose Power Automate Desktop over building a Cloud Flow with an API connector for the same integration?**
Choose Desktop when the target system genuinely has no accessible API or Power Platform connector (legacy/green-screen systems, certain older desktop software), or the process fundamentally requires simulating a human's interaction across multiple desktop windows. If an API or first-class connector exists, a Cloud Flow is almost always more reliable and maintainable than UI automation, which should be treated as a fallback, not a first choice.

**Q5. How can you combine a Cloud Flow with a Desktop Flow in a single automation?**
A Cloud Flow acts as the orchestrator/trigger (e.g., a new item in SharePoint or a scheduled recurrence) and invokes a registered Desktop Flow as an action via the Machine/Process Automation connector, optionally passing input parameters and receiving outputs back — a common "cloud triggers legacy desktop task" hybrid pattern.

**Q6. What's the difference between using image recognition and UI Automation (UIA)-based selectors in PAD?**
UIA-based selectors interact with an application's accessibility/automation framework properties directly, which is faster and more resilient to minor visual changes (like a resized window or slightly different theme). Image recognition matches a screenshot pattern on screen, which is more fragile (breaks with resolution changes, font rendering differences, or theme changes) but is sometimes the only option for applications that don't expose proper UIA elements (e.g., some legacy or virtualized/remote desktop apps).

---

## 🟠 Advanced

**Q1. Design a resilient error-recovery strategy for an unattended nightly Desktop Flow that occasionally encounters a frozen/unresponsive target application.**
Wrap the interaction with the target app in "On Block Error," and within the error-handling block, implement a detect-and-recover routine: check if the process is still running but unresponsive (e.g., via a "Run application"/process-check action), forcibly terminate and relaunch the application if needed, and retry the specific step a limited number of times before ultimately failing gracefully with a detailed log/screenshot and an alert (e.g., emailing the automation owner) rather than leaving the automation hung indefinitely or silently failing with no record of what happened.

**Q2. How would you approach maintaining dozens of Desktop Flows dependent on a legacy application that gets updated periodically by its vendor, without a full rebuild each time?**
Centralize commonly-reused selector logic and interaction sequences into Subflows (reusable Desktop Flow modules) so a UI change only requires updating the shared subflow once rather than every flow that touches that screen independently; maintain a test/staging copy of the legacy application (or a scheduled canary run against a non-production instance) to catch breaking UI changes proactively after vendor updates rather than discovering them via a failed production run; and favor relative/property-based selectors over exact coordinate or fragile absolute-path selectors to reduce the blast radius of minor UI tweaks.

**Q3. What governance considerations apply specifically to unattended RPA at scale (dozens of bots running on shared or dedicated machines), that don't apply to Cloud Flows?**
Unattended RPA requires managing a fleet of registered machines (patching, OS updates, application version consistency across machines to avoid "works on one machine, fails on another"), credential management for whatever accounts the bots log in as (secure storage, rotation, and least-privilege scoping since these are often powerful service accounts), scheduling conflicts (ensuring two unattended flows don't compete for the same machine/application session simultaneously), and machine capacity planning (a single machine can typically only run one interactive unattended session at a time, unlike Cloud Flows which scale more transparently in the cloud).

**Q4. Describe how you would decide, for a specific legacy process automation candidate, whether Power Automate Desktop RPA is actually the right tool versus pushing for a proper API-based integration instead.**
Evaluate: (1) whether the target system's vendor has any API roadmap or existing but underused API that could be adopted instead of UI automation, (2) the expected lifespan of the legacy system (if it's being replaced in 12 months, investing heavily in a fragile RPA solution may not be worth it versus a lighter, more disposable automation), (3) the true frequency/volume of the process (a rarely-run manual process may not justify RPA's fragility risk at all versus a documented manual procedure), and (4) the criticality/tolerance for occasional failure — RPA is a good fit for high-volume, stable-UI, no-API-alternative processes where occasional selector maintenance is an acceptable ongoing cost, but a poor long-term fit for a system on its way out or one that already has a viable API path.

---

## 🔴 Expert

**Q1. You're tasked with designing an enterprise-wide unattended RPA operating model spanning 50+ bots across multiple business units, with strict uptime SLAs for several finance-critical processes. What operational architecture would you propose?**
Establish a dedicated, monitored machine pool (physical or VM-based) with redundancy (spare capacity to fail over if a primary machine becomes unavailable), a centralized scheduling/orchestration layer (potentially combining Cloud Flows as the trigger/orchestration layer calling Desktop Flows, so scheduling logic, retries, and alerting live in the more observable and resilient cloud layer rather than solely within Desktop Flow's own scheduling), robust health-check automation (a lightweight "canary" flow verifying each critical target application is reachable/responsive before the main automation runs, failing fast with an early alert rather than mid-process), a credential vault (e.g., Azure Key Vault integration) for service account management rather than hardcoded credentials, and a clear escalation/on-call process for finance-critical bot failures given the SLA commitments — since unattended RPA, unlike cloud-native automation, still depends on physical/virtual machine health that must be actively managed.

**Q2. How would you evaluate and mitigate the long-term technical debt risk of an organization becoming heavily reliant on RPA for dozens of critical processes against legacy systems?**
RPA's core risk is that it automates the *symptom* (a UI-only legacy system) rather than fixing the *root cause* (lack of an API), meaning technical debt compounds as more processes come to depend on UI stability that the automation team doesn't control; mitigate by maintaining a living inventory of all RPA processes with their business criticality and underlying system's replacement roadmap, treating RPA explicitly as a *bridge* solution with a periodic review cadence (e.g., annually re-assessing whether an API alternative has become available), building strong regression testing/monitoring so breakage is caught immediately rather than silently causing downstream data issues, and advocating architecturally for API-first integration wherever a system replacement or major upgrade creates the opportunity, rather than simply re-recording a new RPA flow against the new UI out of habit.

**Q3. A finance-critical unattended Desktop Flow silently produced incorrect output for several days (it "succeeded" each run, but extracted the wrong column of data after a vendor UI update shifted a table's layout) before anyone noticed. How would you redesign to catch this class of failure faster?**
This is a business-outcome validation gap, not a flow-execution failure — the automation technically ran and "succeeded" by RPA's own definition (it read *some* value and completed) even though the extracted value was wrong; add explicit output validation steps within the flow (e.g., sanity-check extracted values against expected data types/ranges, or cross-validate against a secondary independent data point) rather than trusting extraction success alone, and add a downstream reconciliation check (a lightweight scheduled job comparing RPA-extracted totals against another source of truth, like an end-of-day summary report from the source system) so discrepancies surface within a day rather than persisting silently — treating "the automation ran without an error" and "the automation produced correct data" as two distinct claims that must both be actively verified.

---

## ⚡ Quick-Fire Round

- **Q: What action pauses execution until a UI element appears, instead of a fixed delay?** → "Wait for element" (or "Wait for window").
- **Q: What's the RPA equivalent of a try/catch block in Power Automate Desktop?** → "On Block Error."
- **Q: What component lets you reuse a common sequence of steps across multiple Desktop Flows?** → A Subflow.
- **Q: What connects Power Automate Desktop machines to the cloud for orchestration?** → The on-premises data gateway / machine registration.
- **Q: What type of automation runs without a human physically present?** → Unattended automation.
- **Q: What's more resilient to minor UI changes — image recognition or UIA-based selectors?** → UIA-based selectors.
- **Q: What's the term for uniquely identifying a UI element for automation to interact with?** → A Selector.
- **Q: What should you check first when a previously-working Desktop Flow suddenly fails?** → Whether the target application's UI/version changed (broken selector).
- **Q: What connector lets a Cloud Flow invoke a Desktop Flow?** → The Machine / Process Automation connector (formerly Desktop Flows connector).
- **Q: What's a key limitation of running unattended automation on a single machine?** → Typically only one interactive unattended session can run at a time per machine.

---

## 🧩 Scenario-Based

**Scenario 1. An unattended Desktop Flow that extracts data from a legacy inventory system starts failing intermittently after the vendor pushed a UI update. What's your fix, both immediate and long-term?**
Immediately, re-record/update the broken selectors against the new UI to restore functionality quickly. Long-term, refactor to use relative/property-based UIA selectors rather than exact coordinates or fragile absolute paths, wrap the interaction in "On Block Error" with retry logic and screenshot capture on failure for faster future diagnosis, and evaluate whether the vendor has since introduced an API that could replace this specific fragile UI dependency altogether.

**Scenario 2. A Cloud Flow needs to trigger a Desktop Flow to generate a report in an old Windows accounting application, then email the report — combining cloud orchestration with legacy desktop automation.**
Use a Cloud Flow (triggered on a schedule or a SharePoint event) to call the registered Desktop Flow via the Machine connector, passing any needed input parameters (e.g., the reporting date range); the Desktop Flow opens the legacy application, generates and saves the report file, and returns the file path/confirmation as output; the Cloud Flow then picks up the resulting file (e.g., from a shared network location or by receiving it as an output parameter) and completes the process by emailing it via Outlook — keeping orchestration, scheduling, and notification logic in the more maintainable cloud layer while confining fragile UI interaction to only the piece that truly requires it.

**Scenario 3. Leadership asks whether a manual, twice-a-year data reconciliation process (done by one person, taking about 2 hours) is a good candidate for RPA investment. How do you respond?**
Recommend against investing in RPA here — the volume/frequency (twice a year) doesn't justify the ongoing maintenance cost of a UI automation that will likely need selector updates whenever the underlying systems change, and the two-hour manual effort is a low-risk, low-cost process to simply document well instead; RPA delivers the best ROI for high-frequency, high-volume, stable-UI processes, not infrequent, low-effort ones — a better use of automation effort would be prioritizing daily/weekly processes with real cumulative time savings.

**Scenario 4. Multiple unattended bots need to run on the same shared machine, but occasionally two are scheduled to start at the same time, causing conflicts. How do you resolve this at an architectural level?**
Move scheduling/orchestration logic to the cloud layer (a Cloud Flow-based scheduler or a proper RPA orchestration queue) that enforces mutual exclusion — e.g., checking a "machine busy" flag/semaphore (a Dataverse or SharePoint record representing machine availability) before dispatching a run, and queuing/deferring a bot's start time if the machine is already occupied — rather than relying on independently-scheduled Desktop Flow triggers that have no awareness of each other's execution state on the same shared machine.

