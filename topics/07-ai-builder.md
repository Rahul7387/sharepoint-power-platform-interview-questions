# AI Builder — Interview Questions

⬅ Prev [Power Automate – Desktop](./06-power-automate-desktop.md) &nbsp;|&nbsp; [⬅ Back to Index](../index.html) &nbsp;|&nbsp; [Repo Home](../README.md) &nbsp;|&nbsp; Next ➡ [Copilot Studio](./08-copilot-studio.md)

Covers prebuilt and custom AI models, credits/licensing, and integration into apps and flows.

## Contents
- [🟢 Beginner](#-beginner)
- [🔵 Intermediate](#-intermediate)
- [🟠 Advanced](#-advanced)
- [🔴 Expert](#-expert)
- [⚡ Quick-Fire Round](#-quick-fire-round)
- [🧩 Scenario-Based](#-scenario-based)

---

## 🟢 Beginner

**Q1. What is AI Builder?**
AI Builder is a Power Platform capability that lets makers add AI/machine learning models — either prebuilt or custom-trained on their own data — into Canvas Apps, Model-Driven Apps, and Power Automate flows, without requiring data science expertise.

**Q2. What are the two broad categories of AI Builder models?**
Prebuilt models (ready to use immediately — e.g., Business Card Reader, Text Recognition/OCR, Sentiment Analysis) and Custom models (trained on your own labeled data — e.g., Form Processing, Object Detection, Prediction, Category Classification).

**Q3. What does the "Text Recognition" (OCR) prebuilt model do?**
It extracts raw text from an image or scanned document, without understanding the document's structure or specific field meaning.

**Q4. What does the "Sentiment Analysis" prebuilt model do?**
It analyzes a piece of text and classifies it as positive, negative, neutral, or mixed sentiment — commonly used for analyzing customer feedback or survey comments.

**Q5. What is required before you can use a custom AI Builder model like Form Processing?**
You must train the model on sample data specific to your use case (e.g., several labeled example documents for Form Processing) and then publish it before it can be consumed in an app or flow.

---

## 🔵 Intermediate

**Q1. How does the Form Processing model differ from generic Text Recognition/OCR?**
Text Recognition extracts raw text without structural understanding. Form Processing is trained on sample documents of a specific, consistent layout (e.g., a particular invoice template) to extract structured key-value fields and table data (like "Invoice Number," "Total," and line items), requiring several labeled sample documents per distinct layout during training.

**Q2. What is the "Prediction" model type, and what kind of business problems does it solve?**
A Prediction model is a binary classification model trained on your own historical Dataverse data (with a known outcome column) to predict a yes/no outcome for new records — common use cases include predicting whether a sales lead will convert, whether an invoice is likely to be paid late, or whether a support ticket will escalate.

**Q3. How are AI Builder credits consumed, and what governance considerations apply?**
Each AI Builder operation (a document scan, a prediction, an image classification, etc.) consumes credits drawn from a capacity pool allocated at the tenant or environment level; heavy usage requires purchasing additional AI Builder capacity add-on packs, so cost governance involves monitoring consumption via the Power Platform admin center's Capacity view and avoiding wasteful reprocessing (e.g., re-running OCR on the same document across every flow retry).

**Q4. How would you integrate a Custom Object Detection model into a Canvas App?**
Train and publish the Object Detection model using labeled sample images of the objects you want detected, then add the built-in "AI Builder – Object Detector" component to a Canvas App screen, bind it to the published model, allow the user to capture/upload an image, and use the returned bounding boxes/confidence scores to drive further app logic (e.g., flagging a detected defect and writing a Dataverse record).

**Q5. What's the difference between using AI Builder's prebuilt Business Card Reader model versus building a custom Form Processing model for scanning business cards?**
The prebuilt Business Card Reader model is already trained on the general structure/variety of business cards worldwide and works out of the box with no training required. A custom Form Processing model would only make sense if you needed to extract from a very specific, non-standard card layout the prebuilt model doesn't handle well — for standard business cards, the prebuilt model is almost always the better, zero-training-effort choice.

---

## 🟠 Advanced

**Q1. A custom Form Processing model performs well on most invoices but frequently misreads totals from a specific new vendor's invoice layout. How do you diagnose and fix this?**
This typically indicates the model wasn't trained on samples of that vendor's specific layout — Form Processing models are inherently layout-specific, so add several labeled sample documents of the new layout to the training set and retrain/republish the model; if the organization regularly onboards new vendors with varying layouts, consider whether a more general, layout-agnostic extraction approach (or maintaining a growing library of trained layouts) fits the long-term maintenance model better than a single rigid model.

**Q2. How would you design a Power Automate flow around an AI Builder Prediction or Form Processing model so that low-confidence results don't silently cause bad business outcomes?**
Check the confidence score returned alongside each prediction/extracted field, and add a Condition branching low-confidence results (below a defined threshold) to a human-review step (e.g., an approval task or a Power Apps review screen) instead of automatically trusting and acting on every AI output; log both the AI's original output and the human's correction where applicable, since that correction data can also inform future retraining/model improvement decisions.

**Q3. What are the cost and governance tradeoffs of using AI Builder's built-in models versus calling Azure AI Services (Azure Cognitive Services) directly from a flow via a Custom Connector, for a similar capability?**
AI Builder is more accessible to low-code makers (no separate Azure subscription/resource provisioning needed) and integrates natively as first-class actions in flows/apps, but consumes the org's shared AI Builder credit capacity and can be more expensive per-operation at high volume compared to provisioning your own Azure AI Services resource with its own (often more granular and cost-optimizable) pricing tier; calling Azure AI Services directly gives more control over model versions/regions and potentially lower cost at scale, but requires more setup, an Azure subscription, and typically a developer to build/maintain the integration versus a citizen-developer-friendly AI Builder action.

---

## 🔴 Expert

**Q1. Design an end-to-end architecture for a high-volume invoice processing pipeline (10,000+ invoices/month from 50+ different vendors) that uses AI Builder while remaining cost-effective and maintainable.**
Rather than a single Form Processing model attempting to handle all 50 vendor layouts, group vendors by layout similarity and maintain a small number of models (or evaluate whether a more general/unstructured document AI approach in Azure AI Document Intelligence — which underlies some of this capability — might scale better for high layout diversity than many discrete AI Builder models); route each incoming invoice to the correct model based on vendor identification (e.g., detected from the sender's email domain or a barcode/vendor code on the document) before extraction; implement confidence-threshold routing to human review for low-confidence extractions; monitor AI Builder credit consumption against the volume forecast to ensure the tenant's capacity add-ons are sized correctly ahead of scale, since 10,000+ monthly document operations at this volume will materially exceed typical default capacity allocations; and establish a periodic retraining cadence as new vendors or layout changes are onboarded, rather than treating the initial training as a one-time setup.

**Q2. How would you evaluate whether AI Builder is the right tool for a business problem versus building a custom machine learning solution in Azure Machine Learning?**
AI Builder is the right fit when the problem maps cleanly onto one of its supported model types (document extraction, binary prediction, classification, object detection) using structured/labeled data the platform can consume directly, and the team lacks dedicated data science resources — it trades some model flexibility and tuning control for dramatically faster time-to-value and low-code accessibility. A custom Azure ML solution is warranted when the problem requires a model type AI Builder doesn't support (e.g., complex multi-class regression, custom NLP beyond sentiment/text extraction, or a bespoke deep learning architecture), when you need fine-grained control over feature engineering, hyperparameter tuning, and model explainability beyond what AI Builder exposes, or when the expected volume/complexity makes a dedicated, independently-scaled ML pipeline more cost-effective than AI Builder's credit-based consumption model at extreme scale.

**Q3. A Prediction model deployed for 8 months starts showing declining accuracy in production (more false positives flagged by business users than at launch). How do you diagnose and address this?**
This is a classic model/data drift scenario — the real-world patterns the model was originally trained on may have shifted (e.g., business process changes, seasonal effects, or a change in the underlying population of records being scored) without the model being retrained to reflect the new reality; pull recent production outcomes (including the business-flagged false positives) and compare the distribution of key input features against the original training data to confirm drift, then retrain the Prediction model on a more recent, representative dataset; also establish an ongoing monitoring/retraining cadence (e.g., quarterly) rather than treating the initial training as permanent, since most production ML models — including no-code ones — degrade over time as the real-world context evolves.

---

## ⚡ Quick-Fire Round

- **Q: What AI Builder model type extracts raw text from an image?** → Text Recognition (OCR).
- **Q: What model type extracts structured key-value fields from a consistent document layout?** → Form Processing.
- **Q: What model type gives a yes/no outcome prediction based on historical Dataverse data?** → Prediction.
- **Q: What's consumed each time an AI Builder model runs an operation?** → AI Builder credits.
- **Q: What prebuilt model classifies text as positive/negative/neutral?** → Sentiment Analysis.
- **Q: What must you do to a custom model before it can be used in an app/flow?** → Train and publish it.
- **Q: What Canvas App control lets you bind directly to a trained Object Detection model?** → The AI Builder Object Detector component.
- **Q: Where do you monitor AI Builder credit consumption at the tenant/environment level?** → The Power Platform admin center's Capacity view.
- **Q: True/False: A single Form Processing model works well across many different, unrelated document layouts.** → False — it's trained per specific layout.
- **Q: What's a common mitigation for low-confidence AI Builder extractions in a flow?** → Route them to a human-review step instead of auto-processing.

---

## 🧩 Scenario-Based

**Scenario 1. A Power Automate flow uses AI Builder Form Processing to extract invoice totals and automatically pay vendors, but a misread total recently caused an overpayment. How do you redesign the process to prevent recurrence?**
Add a confidence-threshold check on the extracted total field, routing anything below the threshold (or any amount exceeding a defined dollar limit, regardless of confidence) to a mandatory human-approval step before payment is triggered, rather than fully automating high-value financial actions purely on AI extraction confidence; additionally, add a sanity-check validation (e.g., cross-referencing the extracted total against the sum of extracted line items) as a secondary automated guardrail before allowing full straight-through processing.

**Scenario 2. HR wants to use AI Builder to screen resumes and predict which candidates are likely to be strong hires, based on historical hiring data. What concerns should you raise before building this?**
Raise concerns about potential bias — if historical hiring data reflects past biased decisions (e.g., underrepresentation of certain groups due to prior hiring patterns), a Prediction model trained on that data risks learning and perpetuating those same biases, which carries real legal/ethical/compliance risk in a hiring context; recommend involving HR/legal/compliance stakeholders early to review the training data for fairness, consider whether protected-class-correlated features are inadvertently included, and treat any such model's output as one advisory input for human recruiters rather than an automated pass/fail gate, given the high-stakes and regulated nature of employment decisions.

**Scenario 3. A Canvas App uses AI Builder's Object Detection model for a quality-inspection use case, but accuracy in the field is noticeably worse than in testing. What would you investigate?**
Compare the lighting conditions, camera angles, and image quality of the field-captured photos against the training images — Object Detection models are sensitive to conditions not represented in training data (e.g., trained on well-lit indoor photos but deployed for outdoor/low-light inspections); expand the training dataset with more representative real-world sample images covering the actual range of conditions inspectors encounter, and retrain/republish the model rather than assuming the original training set was sufficient for all deployment conditions.

