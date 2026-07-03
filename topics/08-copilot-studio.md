# Copilot / Copilot Studio — Interview Questions

⬅ Prev [AI Builder](./07-ai-builder.md) &nbsp;|&nbsp; [⬅ Back to Index](../index.html) &nbsp;|&nbsp; [Repo Home](../README.md) &nbsp;|&nbsp; Next ➡ [Power Pages](./09-power-pages.md)

Covers Copilot Studio (formerly Power Virtual Agents), generative answers, actions, and governance.

## Contents
- [🟢 Beginner](#-beginner)
- [🔵 Intermediate](#-intermediate)
- [🟠 Advanced](#-advanced)
- [🔴 Expert](#-expert)
- [⚡ Quick-Fire Round](#-quick-fire-round)
- [🧩 Scenario-Based](#-scenario-based)

---

## 🟢 Beginner

**Q1. What is Copilot Studio?**
Copilot Studio (formerly Power Virtual Agents) is Microsoft's low-code platform for building custom conversational AI agents/copilots that can answer questions, follow guided conversation flows, and take actions, deployable to channels like Teams, websites, and other apps.

**Q2. What is a "Topic" in Copilot Studio?**
A Topic is an authored conversation flow with defined trigger phrases, a structured sequence of questions/messages, conditions, and actions — used for known, well-defined scenarios like "reset my password" or "check order status."

**Q3. What is the difference between Microsoft 365 Copilot and Copilot Studio?**
Microsoft 365 Copilot is the built-in AI assistant embedded across Word, Excel, Teams, and Outlook, working over the user's Microsoft Graph-accessible content. Copilot Studio is a separate platform for building your own custom conversational agents from scratch, tailored to specific business scenarios.

**Q4. What are Generative AI Answers?**
A capability where the agent uses a large language model, grounded on configured knowledge sources (like a SharePoint site or website), to dynamically generate answers to open-ended questions that aren't covered by a specific authored Topic.

**Q5. What channels can a Copilot Studio agent be published to?**
Common channels include Microsoft Teams, a website (via an embeddable web widget), and Direct Line for custom integration into other applications.

---

## 🔵 Intermediate

**Q1. How do Topics and Generative AI Answers work together in the same agent?**
Topics handle known, well-defined scenarios deterministically and predictably (specific trigger phrases lead to a controlled flow); Generative AI Answers act as a fallback/complement for open-ended questions that don't match any authored Topic, dynamically generating a grounded response from configured knowledge sources — a well-designed agent typically uses Topics for critical, must-be-consistent flows and generative answers for broader, less predictable coverage.

**Q2. How do you ground a Copilot Studio agent's answers in your organization's own data?**
Configure Knowledge Sources (SharePoint sites/document libraries, public websites, Dataverse tables, or uploaded files) under the generative AI settings; the agent retrieves relevant content from these sources at answer time (a retrieval-augmented generation, or RAG, approach) rather than relying purely on the base model's general training data.

**Q3. What are "Actions" in Copilot Studio, and what do they enable?**
Actions let a copilot invoke a Power Automate flow, a Connector, or a custom AI Builder Prompt mid-conversation to actually perform a task (like checking an order's status in a database or creating a support ticket) — turning the agent from a pure question-answering bot into one that can take real action on the user's behalf.

**Q4. How would you implement escalation to a human agent from within a Copilot Studio bot?**
Add an "Escalate" Topic/action that hands off the conversation to a live agent queue (e.g., via Omnichannel for Customer Service or a Teams handoff), typically triggered either by an explicit user request or a fallback pattern when the bot's confidence is low, ideally passing along the conversation transcript/context so the human agent isn't starting cold.

**Q5. What security consideration is critical when grounding a Copilot Studio agent on internal SharePoint content?**
The agent should run and retrieve knowledge using the *end user's* own identity context rather than a generic/elevated service account, so that Microsoft Search's security trimming applies at query time — ensuring the agent only surfaces and cites content the actual asking user already has permission to see in SharePoint, not everything in the configured knowledge source regardless of the user's own access.

---

## 🟠 Advanced

**Q1. How would you design a multi-topic agent to avoid Topics "colliding" (multiple Topics' trigger phrases overlapping ambiguously) as the number of Topics grows into the hundreds?**
Use distinct, well-differentiated trigger phrases per Topic and periodically review the built-in Topic overlap/analytics tooling that flags ambiguous trigger matches; group related Topics under a parent/hierarchical structure where a broader Topic first disambiguates intent (e.g., "I have a billing question" leads to a follow-up clarifying sub-Topic selection) rather than trying to have hundreds of flat, independently-triggered Topics all competing for the same broad phrasing space; and treat Topic authoring as an ongoing curation discipline, retiring or merging overlapping Topics as the agent's scope grows rather than only adding new ones indefinitely.

**Q2. What governance controls would you put in place for a Copilot Studio agent that has Actions calling Power Automate flows which write to production Dataverse tables?**
Apply DLP (Data Loss Prevention) policies restricting which connectors/actions the agent's underlying flows are permitted to use, build the agent and its flows within a properly governed Dataverse Solution (so ALM, Connection References, and environment separation apply just like any other Power Platform component), require any Action that performs a write/mutating operation to go through appropriate validation logic within the called flow (not blindly trusting whatever the LLM extracts from user conversation as clean input), and enforce a staged publishing process (Dev/Test agent instances validated before promoting to the Production-published channel) rather than editing and publishing directly against a live, customer-facing agent.

**Q3. How would you measure and improve a Copilot Studio agent's answer quality over time in production?**
Use Copilot Studio's built-in analytics (session counts, Topic engagement/completion rates, fallback rate — how often generative answers or "I don't understand" responses are triggered instead of a Topic) to identify gaps where users' actual questions aren't being well served; review transcripts of low-satisfaction or escalated conversations to identify missing Topics or knowledge source gaps, and iteratively expand knowledge sources or author new Topics based on real observed user intents rather than only the intents anticipated at initial design time; also periodically test edge-case and adversarial phrasing to ensure Generative AI Answers remain properly grounded and don't hallucinate outside the configured knowledge sources.

**Q4. Explain the tradeoffs between letting an agent rely heavily on Generative AI Answers versus authoring many explicit Topics, from a compliance/regulated-industry perspective.**
Explicit Topics are deterministic and fully auditable — you know exactly what the bot will say for a given trigger phrase, which matters in regulated contexts (financial services, healthcare) where consistent, compliance-reviewed wording may be legally required for certain disclosures. Generative AI Answers offer far broader coverage with much less authoring effort, but responses are inherently less predictable (the LLM synthesizes wording from grounded sources at runtime), which can be a compliance risk if there's no review step for a regulated disclosure-type question — in such contexts, it's common to explicitly author Topics for anything with compliance/legal wording requirements and reserve generative answers for lower-stakes, general-information questions only.

---

## 🔴 Expert

**Q1. Design an enterprise-wide Copilot Studio governance and rollout strategy for an organization where multiple business units want to build their own customer-facing agents, with legal/compliance requiring review of anything customer-facing.**
Establish a tiered environment/publishing model: an open "Sandbox" environment where business units can freely prototype internal-only agents for experimentation, but require any agent intended for a customer-facing channel (public website, external Teams guest access) to be built/promoted into a centrally governed environment with mandatory compliance review gates before publishing; standardize a shared knowledge-source governance process (e.g., a vetted, compliance-approved SharePoint site as the canonical knowledge source for customer-facing content, rather than each business unit pointing agents at ungoverned, potentially outdated internal documents); require DLP policies restricting connector/action usage in the customer-facing tier; and set up centralized monitoring/analytics rollups (via the Center of Excellence-style tooling or Copilot Studio's admin analytics) so leadership has visibility across all published customer-facing agents rather than each business unit operating and monitoring in isolation.

**Q2. A customer-facing Copilot Studio agent grounded on public website content occasionally produces an answer that sounds confident but is subtly incorrect (a hallucination not actually present in the grounded source). How would you systematically reduce this risk?**
Audit whether the knowledge source content itself is comprehensive and well-structured for retrieval (poorly organized or sparse source content increases the LLM's tendency to "fill gaps" with unsupported generation), tighten the agent's configuration toward stricter grounding behavior where the platform allows (favoring "answer only from sources" style settings over more open-ended generation), add explicit citations/source links in generated answers so users can verify the underlying source themselves rather than blindly trusting the synthesized answer, and establish an ongoing feedback loop (a thumbs up/down or "was this helpful" mechanism) to surface and manually review suspected hallucinations, feeding corrections back into either the knowledge source content or explicit Topic authoring to fill genuine gaps in a controlled, non-generative way once identified.

**Q3. How would you architect a Copilot Studio agent that must integrate with a legacy on-premises system (behind a firewall, no direct internet access) as an Action, while maintaining the same security posture the legacy system currently enforces?**
Route the Action's call through the on-premises data gateway (the same bridging technology used by Power Automate/Power BI for on-prem connectivity) rather than attempting any direct internet exposure of the legacy system, ensure the underlying Power Automate flow invoked by the Action authenticates using an appropriately scoped service account with the *minimum* necessary privilege on the legacy system (not a broad admin-level account, since the flow — and by extension, anything the LLM-driven conversation might attempt to trigger — should never have more access than the specific action genuinely requires), and add explicit input validation/business-rule checks within the flow itself before it touches the legacy system, since a conversational, LLM-mediated front end introduces a different (and less predictable) style of input than a traditional structured form, and the backend integration should never assume the LLM has already perfectly sanitized/validated what it extracted from free-text conversation.

---

## ⚡ Quick-Fire Round

- **Q: What was Copilot Studio formerly known as?** → Power Virtual Agents.
- **Q: What authored conversation flow handles a specific known scenario?** → A Topic.
- **Q: What lets an agent dynamically answer open-ended questions using an LLM grounded on your data?** → Generative AI Answers.
- **Q: What lets an agent actually perform a task (not just answer) mid-conversation?** → An Action (invoking a flow/connector/prompt).
- **Q: What must apply for an agent to only surface SharePoint content a user is authorized to see?** → Identity-aware retrieval / search security trimming (running as the end user, not a generic service account).
- **Q: What feature routes a conversation to a live human agent?** → Escalation (e.g., via Omnichannel for Customer Service).
- **Q: What Power Platform governance mechanism restricts which connectors an agent's flows can use?** → DLP (Data Loss Prevention) policies.
- **Q: What analytics metric indicates how often users' questions fall outside authored Topics?** → Fallback rate.
- **Q: What's a common channel for deploying a Copilot Studio agent internally?** → Microsoft Teams.
- **Q: What term describes an LLM confidently generating an unsupported/incorrect answer?** → Hallucination.

---

## 🧩 Scenario-Based

**Scenario 1. Your organization wants a Copilot Studio bot to answer HR policy questions using existing SharePoint HR documents, but must ensure an employee never sees a policy from a department they don't have SharePoint access to.**
Configure the SharePoint HR library as a Generative AI Knowledge Source, and ensure the agent queries using the end user's own identity so Microsoft Search's security trimming naturally restricts retrieval to only content that user is already permitted to see — then validate this explicitly with test accounts holding different SharePoint permission levels before broadly publishing the bot, since a misconfigured service-account-based retrieval would silently bypass this protection.

**Scenario 2. A customer service Copilot Studio bot needs to check a customer's order status by looking it up in an external order management system, then offer to escalate to a human if the order shows a problem (e.g., "delayed" status).**
Build an Action that calls a Power Automate flow, which authenticates to the order management system's API (via a Custom Connector or existing connector) using a properly scoped service account, retrieves the order status, and returns it to the conversation; add a condition within the Topic that checks the returned status — if it indicates a problem, proactively offer escalation to a human agent (via an Escalate Topic/Omnichannel handoff) rather than just stating the problem and ending the conversation, since surfacing a bad-news status without an immediate next step creates a poor customer experience.

**Scenario 3. Leadership wants to publish an external, public-website-facing Copilot Studio agent, but is concerned about cost if it goes viral and gets far more traffic than expected. How do you plan for this?**
Review and understand the licensing/consumption model (Copilot Studio agents typically consume message/session-based capacity that scales with usage), set up proactive monitoring/alerting on session volume so the team is notified well before hitting concerning usage thresholds rather than discovering a cost surprise after the fact, and design the agent's knowledge sources and Actions efficiently (e.g., avoiding unnecessarily chatty multi-turn flows for simple questions) since both message volume and the complexity of grounded/generative responses can influence cost — and have a clear plan (rate limiting, temporary Topic-only fallback disabling generative answers, or scaling the compliance review process) ready if traffic genuinely spikes beyond initial projections.

