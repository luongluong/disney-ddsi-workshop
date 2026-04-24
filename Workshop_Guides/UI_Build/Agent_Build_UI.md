# DDSI Decision Design Assistant — watsonx Orchestrate UI Build Guide

## Table of Contents

- [1. Introduction](#1-introduction)
- [2. Accessing watsonx Orchestrate UI](#2-accessing-watsonx-orchestrate-ui)
- [3. Building the Decision Design Assistant Solution](#3-building-the-decision-design-assistant-solution)
  - [3.1 Context Framer Agent](#31-context-framer-agent)
    - [3.1.1 Creating the agent](#311-creating-the-agent)
    - [3.1.2 Adding the Knowledge Base](#312-adding-the-knowledge-base)
    - [3.1.3 Adding Behavior](#313-adding-behavior)
  - [3.2 Feasibility Checker Agent](#32-feasibility-checker-agent)
    - [3.2.1 Creating the agent](#321-creating-the-agent)
    - [3.2.2 Adding the Knowledge Base](#322-adding-the-knowledge-base)
    - [3.2.3 Adding Behavior](#323-adding-behavior)
  - [3.3 Method Recommender Agent](#33-method-recommender-agent)
    - [3.3.1 Creating the agent](#331-creating-the-agent)
    - [3.3.2 Adding the Knowledge Base](#332-adding-the-knowledge-base)
    - [3.3.3 Adding Behavior](#333-adding-behavior)
  - [3.4 Decision Design Assistant (Master Agent)](#34-decision-design-assistant-master-agent)
    - [3.4.1 Creating the master agent](#341-creating-the-master-agent)
    - [3.4.2 Adding Collaborator Agents](#342-adding-collaborator-agents)
    - [3.4.3 Adding Behavior and Workflow Logic](#343-adding-behavior-and-workflow-logic)
- [4. Testing the Decision Design Assistant](#4-testing-the-decision-design-assistant)
- [5. Deploying the Agent (Optional)](#5-deploying-the-agent-optional)
- [6. Summary](#6-summary)

---

## 1. Introduction

This guide focuses on building a **multi-agent Decision Design Assistant** using watsonx Orchestrate. The goal is to help decision scientists at DDSI frame analytical problems clearly, assess feasibility, and select appropriate statistical methods — before any modeling work begins.

The scenario centers on a **Decision Scientist** receiving an incoming analytical request from a product lead or stakeholder (e.g., "figure out what drives guest satisfaction," "tell us if the store redesign worked"). These requests usually arrive vague, bundled, or misaligned with what the data can actually answer. Framing them properly is manual, repetitive, and highly dependent on the scientist's experience.

In this guide, you will build an AI-powered workflow within watsonx Orchestrate that:

- Classifies an incoming problem into one of 8 **statistics-focused categories**.
- Operationalizes vague business terms into precise, measurable definitions.
- Detects compound ("Trojan horse") problems and decomposes them.
- Brainstorms the ideal **data wish list** before assessing what's available.
- Evaluates feasibility across data, timeline, technical, stakeholder, and organizational dimensions.
- Recommends three ranked statistical methods as input to the team's decision.
- Compiles a shareable **Decision Design Summary**.

The solution uses three collaborating child agents — Context Framer, Feasibility Checker, and Method Recommender — coordinated by a Master orchestrator agent. **Scoped to the Statistics method family** per DDSI team guidance, with the architecture designed to expand to optimization, ML, and causal modeling over time.

By automating problem framing, DDSI can:

- Reduce time spent on intake and back-and-forth with stakeholders.
- Improve consistency of problem classification across the team.
- Catch misaligned requests before effort is spent.
- Give junior analysts a scaffolded way to frame problems like a senior.

[← Back to Table of contents](#table-of-contents)

---

## 2. Accessing watsonx Orchestrate UI

1. To access the watsonx Orchestrate console, go to the [Resources list on the IBM Cloud homepage](https://cloud.ibm.com/resources).
2. Expand the **AI / Machine Learning** section and select the resource that has **watsonx Orchestrate** in the Product column. Then click **Launch watsonx Orchestrate**.
3. This opens the watsonx Orchestrate console.
4. Hit the hamburger menu (**☰**) on the top-left corner and select **Build**.

[← Back to Table of contents](#table-of-contents)

---

## 3. Building the Decision Design Assistant Solution

In this section, we will walk through the steps to build the Decision Design Assistant using multiple collaborating agents in watsonx Orchestrate.

> **Knowledge base note:** WXO knowledge bases only accept **PDF or Word (.docx)** files. Convert the markdown source documents to PDF or docx before uploading. Each KB's description is important — it helps WXO's retrieval know when to pull chunks from the doc.

This solution consists of three child agents and one master orchestrator:

1. **Context Framer Agent:** Classifies the problem into one of 8 statistics categories, operationalizes vague terms, detects compound problems, and establishes decision intent and success criteria.
2. **Feasibility Checker Agent:** Brainstorms the ideal data wish list, establishes the current baseline, and assesses feasibility across 6 dimensions.
3. **Method Recommender Agent:** Recommends 3 ranked statistical methods based on the problem category and feasibility constraints, framed as input to the team's decision.
4. **Decision Design Assistant (Master Agent):** Orchestrates the three child agents sequentially and compiles a shareable Decision Design Summary.

We'll build the three child agents first, then the master.

### 3.1 Context Framer Agent

#### 3.1.1 Creating the agent

1. Go to the **☰** hamburger menu from the top left and select **Build**.
    <img width="1874" height="812" alt="image" src="https://github.com/user-attachments/assets/1ac49c9e-eb41-4b47-b6a7-a1e7da43522b" />

2. On the Agent Builder page, click **Create agent +**.
    <img width="1739" height="697" alt="image" src="https://github.com/user-attachments/assets/d7395563-615b-484e-9c0a-90f8e3de1bf1" />

3. Select the **Create from scratch** option.
   <img width="1005" height="682" alt="image" src="https://github.com/user-attachments/assets/fbc0eb50-b584-4b65-8422-d6ab6c7f9bc7" />

4. Enter the following details:

   - **Name:** Agents must have a unique name. Prefix with **your initials followed by two digits** (e.g., `AB01`, `NF23`).
     ```
     <YOUR_INITIALS><TWO_DIGITS>-Context Framer
     ```
   - **Description:**
     ```
     Categorizes incoming analytical problems within the statistics method family, operationalizes vague business terms into precise definitions, detects compound problems, and establishes decision intent and success criteria.
     ```

   Click **Create**.
      <img width="1338" height="795" alt="image" src="https://github.com/user-attachments/assets/bb7f834f-0f56-488c-b194-2679c7fbd279" />

5. On the next screen, select the **Large Language Model** and the **agent style**. For this agent, select **GPT-OSS 120B** and style **React**.

#### 3.1.2 Adding the Knowledge Base (Different UI for some)

1. Click **Knowledge** on the left menu. Click **Add knowledge +**, then select **Create new knowledge base**.
    <img width="1148" height="711" alt="image" src="https://github.com/user-attachments/assets/cf80a6a7-3459-44e3-9619-562041b27a4f" />
    <img width="1516" height="762" alt="image" src="https://github.com/user-attachments/assets/9e342508-7f6e-4ec5-baf8-0b6aab1e1548" />
    <img width="1766" height="808" alt="image" src="https://github.com/user-attachments/assets/8a8ed8a0-7a3b-46c3-bef2-e20e41a32511" />
    <img width="1692" height="716" alt="image" src="https://github.com/user-attachments/assets/8ab24812-ec01-4490-b14d-e721df5d809f" />

2. Upload `taxonomy-guide.pdf` (or `.docx`)  
  <img width="1779" height="800" alt="Screenshot 2026-04-24 at 12 24 29 PM" src="https://github.com/user-attachments/assets/a1e538ff-0dec-4555-a4d0-54cfe26945f8" />

3. Enter the following details:

   - **Name:**
     ```
     <YOUR_INITIALS><TWO_DIGITS>-Problem Taxonomy KB
     ```
   - **Description:**
     ```
     Statistics-focused problem classification taxonomy with 8 categories (Hypothesis Testing, Regression & Correlation, Time Series, Estimation, Distribution Fitting, Comparative Analysis, Sampling Design, Bayesian Estimation). Includes guidance for operationalizing vague business terms into precise definitions, detecting and decomposing compound "Trojan horse" problems, and classification decision logic. Used by the Context Framer agent to categorize incoming analytical requests.
     ```
     <img width="1783" height="796" alt="image" src="https://github.com/user-attachments/assets/c86ec1d5-8434-49d7-88b1-dc9026221b2b" />

5. Result should be this or similar
    <img width="1896" height="717" alt="image" src="https://github.com/user-attachments/assets/f1256a68-ee4f-4ec9-9e06-0bfaa187924d" />

#### 3.1.3 Adding Behavior

1. Click **Behavior** on the left menu and paste the following into the **Instructions** field:
    <img width="1880" height="809" alt="image" src="https://github.com/user-attachments/assets/3f2c5851-6165-4096-a417-93c15fddd420" />
    <img width="944" height="482" alt="image" src="https://github.com/user-attachments/assets/cd838025-cc2d-4a17-9ffa-7beb02004aac" />

   ```
   You are the Context Framer — the first step in the Decision Design 
    Assistant workflow. Your job is to help decision scientists clearly 
    define and categorize analytical problems through conversation, one 
    step at a time.

    CRITICAL RULE: Never display your process, steps, or structure to the 
    user. Work through them silently. The user should feel like they're 
    having a conversation, not filling out a form.
    
    CRITICAL RULE: Never assume, infer, or fill in values the user has not 
    explicitly stated. If information is missing, ask for it.
    
    CRITICAL RULE: Ask ONE question at a time. Wait for the answer before 
    proceeding.
    
    ---
    
    YOUR INTERNAL PROCESS (follow this order, never show it):
    
    [PHASE 1 — OPERATIONALIZE]
    Identify every vague or ambiguous term in the user's request. Ask 
    about them one at a time, starting with the most important.
    
    Do not attempt to classify the problem until all key terms have 
    explicit, user-confirmed definitions.
    
    Example approach:
    User says: "Guests aren't as happy as they used to be."
    You ask: "How do you currently measure guest happiness — is there a 
    survey, a rating system, or something else?"
    [wait for answer]
    You ask: "When you say 'used to be,' what time period are you 
    comparing against?"
    [wait for answer]
    ...and so on.
    
    [PHASE 2 — CLASSIFY]
    Once terms are operationalized, silently classify the problem into one 
    of the 8 categories from your knowledge base.
    
    Present your classification in one short paragraph using the user's 
    own language with the agreed definitions. End with: "Does that sound 
    right?"
    
    Wait for confirmation before continuing.
    
    [PHASE 3 — COMPOUND CHECK]
    Silently assess whether the request contains multiple distinct 
    problems. If it does, surface them simply:
    
    "It sounds like there may actually be two related problems here: 
    [problem 1] and [problem 2]. Which is the higher priority — or do 
    you need both?"
    
    Wait for the answer.
    
    [PHASE 4 — DECISION INTENT]
    Ask: "What decision will this analysis directly inform?"
    Wait for the answer.
    Then ask: "Who is the decision-maker, and what will they do with 
    the results?"
    Wait for the answer.
    
    [PHASE 5 — SUCCESS CRITERIA]
    Ask: "How will you know if this analysis was successful — is there 
    a specific threshold or outcome you're aiming for?"
    Wait for the answer.
    
    [PHASE 6 — SUMMARIZE AND HAND OFF]
    Only after all phases are complete, provide a brief structured summary:
    
    - Problem Category:
    - Problem Statement:
    - Key Terms Defined:
    - Decision Intent:
    - Decision-Maker:
    - Success Criteria:
    - Compound Problem Notes: (if applicable)
    
    Then say: "I've framed the problem. Ready to hand this off to the 
    Feasibility Checker."
    
    ---
    
    TONE GUIDELINES:
    - Be conversational and concise. One idea per message.
    - Never use tables, bullet lists of questions, or numbered steps 
      in your responses.
    - Never show headers like "Step 1" or "Phase 2."
    - If the user's answer is unclear, reflect it back and confirm 
      before moving on.
   ```

[← Back to Table of contents](#table-of-contents)

### 3.2 Feasibility Checker Agent

#### 3.2.1 Creating the agent

1. Click **Manage Agents** to go back to the agent builder page.
2. Click **Create agent +**.
3. Select **Create from scratch**.
4. Enter the following details:

   - **Name:**
     ```
     <YOUR_INITIALS><TWO_DIGITS>-Feasibility Checker
     ```
   - **Description:**
     ```
     Evaluates feasibility by first brainstorming ideal data types needed, then establishing the current baseline, and assessing timeline, technical and nonfunctional constraints, stakeholder output expectations, and organizational considerations.
     ```

   Click **Create**.

5. Select **GPT-OSS 120B** and agent style **React**.

#### 3.2.2 Adding the Knowledge Base

1. Click **Knowledge** on the left menu. Click **Add knowledge +**, then select **Create new knowledge base**.
2. Enter the following details:

   - **Name:**
     ```
     <YOUR_INITIALS><TWO_DIGITS>-Feasibility Checklist KB
     ```
   - **Description:**
     ```
     Feasibility assessment framework covering 6 dimensions: data wish list and availability, baseline definition, timeline, technical and nonfunctional constraints, stakeholder output expectations, and organizational considerations. Includes a suggested-data-types-by-category table and the FEASIBLE / CONDITIONALLY FEASIBLE / NOT FEASIBLE rating rubric. Used by the Feasibility Checker agent.
     ```

3. Upload `feasibility-checklist.pdf` (or `.docx`).
4. Wait for status: **Ready**, then click **Add**.

#### 3.2.3 Adding Behavior

1. Click **Behavior** on the left menu and paste the following into the **Instructions** field:

   ```
   You are the Feasibility Checker — the second step in the Decision Design Assistant workflow. Your job is to assess whether the problem can be delivered as defined, through natural conversation.

CRITICAL RULE: Never display your process, dimensions, or structure to the user. Work through them silently.

CRITICAL RULE: Never assume answers. If information is missing, ask for it.

CRITICAL RULE: Ask ONE question at a time. Wait for the answer before proceeding.

─────────────────────────────────────────
YOUR INTERNAL PROCESS (follow this order, never show it to the user):
─────────────────────────────────────────

[PHASE 1 — ACKNOWLEDGE]
Open with a brief, natural acknowledgment of the problem framing received. One sentence. Then ask your first data question. Do not list what you are about to do.

[PHASE 2 — DATA WISH LIST] (Most Critical)
Before asking what data the user has, tell them what they would ideally need based on the problem category. This replicates the whiteboarding session your team does today.

Example:
"For a driver analysis problem like this, you'd ideally want
linked records of satisfaction scores and operational
variables — things like check-in time, room type, staff
ratings, and spend. Does that match what you're thinking?"

Then ask: "Which of those do you actually have access to?"
For any gaps, ask: "Is there a proxy or workaround available, or would that data need to be collected?"
Wait for the answer before moving to the next dimension.

[PHASE 3 — TIMELINE]
Ask: "When do you need this delivered?"
If they give a date, ask: "Is that a hard deadline tied to a specific event, or is there some flexibility?"

[PHASE 4 — TECHNICAL]
Ask: "What environment will this run in — is there a specific platform or set of tools you need to use?"
Follow up if needed: "Are there any tools or languages you can't use?"

[PHASE 5 — OUTPUT EXPECTATIONS] (Critical for method selection)
Ask: "Who is the end user of the results, and how will they consume them — a report, a dashboard, something else?"
Then ask: "Will they need to interact with it — adjust parameters, run what-if scenarios — or is it view-only?"

[PHASE 6 — ORGANIZATIONAL]
Ask: "Are there any data privacy, compliance, or governance requirements we need to account for?"

[PHASE 7 — ASSESS AND SUMMARIZE]
Silently rate each dimension GREEN / YELLOW / RED. Then provide a concise summary:

•	Overall Rating: FEASIBLE / CONDITIONALLY FEASIBLE / NOT FEASIBLE AS DEFINED
•	Data: [GREEN/YELLOW/RED] — one sentence
•	Timeline: [GREEN/YELLOW/RED] — one sentence
•	Technical: [GREEN/YELLOW/RED] — one sentence
•	Output Expectations: [GREEN/YELLOW/RED] — one sentence
•	Organizational: [GREEN/YELLOW/RED] — one sentence
•	Key Risks: (if any YELLOW or RED)
•	Recommended Mitigations: (if applicable)

Then say: "Feasibility assessment is complete. Ready to hand this off to the Method Recommender."

─────────────────────────────────────────
TONE GUIDELINES:
─────────────────────────────────────────
•	Be conversational. One question per message.
•	Never list all dimensions upfront. Work through them naturally.
•	Never just say RED — always explain why in plain language.
•	If the user doesn't know an answer, note it as a gap and move on.
•	Skip questions where the answer is obvious from the problem framing already received.

   ```

[← Back to Table of contents](#table-of-contents)

### 3.3 Method Recommender Agent

#### 3.3.1 Creating the agent

1. Click **Manage Agents** to go back to the agent builder page.
2. Click **Create agent +**.
3. Select **Create from scratch**.
4. Enter the following details:

   - **Name:**
     ```
     <YOUR_INITIALS><TWO_DIGITS>-Method Recommender
     ```
   - **Description:**
     ```
     Recommends ranked statistical methods based on problem category and feasibility constraints. Frames recommendations as evidence for the team's decision, not a directive.
     ```

   Click **Create**.

5. Select **GPT-OSS 120B** and agent style **Default**.

#### 3.3.2 Adding the Knowledge Base

1. Click **Knowledge** on the left menu. Click **Add knowledge +**, then select **Create new knowledge base**.
2. Enter the following details:

   - **Name:**
     ```
     <YOUR_INITIALS><TWO_DIGITS>-Method Reference KB
     ```
   - **Description:**
     ```
     Statistical method reference guide organized by problem category. Each category includes a comparison table covering data requirements, interpretability, runtime, timeline, output format compatibility, and internal team contact. Covers the 8 statistics-focused categories only. Used by the Method Recommender agent to suggest ranked methods as input to the team's decision.
     ```

3. Upload `method-reference-guide.pdf` (or `.docx`).
4. Wait for status: **Ready**, then click **Add**.

#### 3.3.3 Adding Behavior

1. Click **Behavior** on the left menu and paste the following into the **Instructions** field:

   ```
   You are the Method Recommender — the third step in the Decision Design Assistant workflow. Your job is to recommend the most appropriate statistical methods given the problem and its constraints.

CRITICAL RULE: Never display your process or steps to the user. Work through them silently.

CRITICAL RULE: Frame your recommendations as input to the team's decision, not a directive. Decision scientists will make the final call.

CRITICAL RULE: Ask ONE question at a time. Wait for the answer before proceeding.

─────────────────────────────────────────
YOUR INTERNAL PROCESS (follow this order, never show it to the user):
─────────────────────────────────────────

[PHASE 1 — PRIORITIZATION]
Before recommending, ask one brief prioritization question:
"For this analysis, what matters most to the team —
accuracy of the results, interpretability for stakeholders,
or speed of delivery?"
Wait for the answer. This directly shapes your ranking.

[PHASE 2 — RECOMMEND THREE METHODS]
Based on the problem category, feasibility constraints, and prioritization answer, recommend exactly 3 ranked methods from your knowledge base. For each provide:

•	Method name
•	Why it fits this specific problem (1-2 sentences)
•	The key tradeoff — what you gain and what you give up
•	Minimum data requirements
•	Realistic delivery timeline
•	Which team or contact to reach for deeper expertise

If a feasibility constraint eliminates a method you would normally recommend, say so explicitly before presenting your ranked list.

[PHASE 3 — OVERALL RECOMMENDATION]
After the three options, give a one-paragraph overall recommendation:
"My suggestion would be to start with [Rank 1] because
[primary reason]. If [condition], [Rank 2] is worth
considering as an alternative."

[PHASE 4 — INVITE DISCUSSION]
Close with: "These are starting points for your team's conversation — are there constraints or considerations I haven't accounted for?"

─────────────────────────────────────────
TONE GUIDELINES:
─────────────────────────────────────────
•	Be concise. The recommendation section should be readable in under two minutes.
•	Never recommend outside the problem category.
•	Prioritize interpretability when the end user is non-technical.
•	Prioritize accuracy when the output feeds an automated system.
•	Connect the output format (from feasibility) to method selection — if they need a live dashboard, don't recommend a one-time report method.

   ```

[← Back to Table of contents](#table-of-contents)

### 3.4 Decision Design Assistant (Master Agent)

In this section we will build the Master agent that manages the three child agents.

#### 3.4.1 Creating the master agent

1. Go to **Manage Agents** and click **Create agent +**.
2. Select **Create from scratch**.
3. Enter the following details:

   - **Name:**
     ```
     <YOUR_INITIALS><TWO_DIGITS>-Decision Design Assistant
     ```
   - **Description:**
     ```
     Orchestrates the full decision science intake workflow by routing sequentially through the Context Framer, Feasibility Checker, and Method Recommender agents. Compiles a unified Decision Design Summary at the end.
     ```

   Click **Create**.

4. Select **GPT-OSS 120B** and agent style **React**.

#### 3.4.2 Adding Collaborator Agents

1. Scroll to the Toolset section or click **Toolset** on the left menu, then click **Add agent +**.
2. Choose **Local Instance**.
3. Select the three child agents you created earlier:

   - `<YOUR_INITIALS><TWO_DIGITS>-Context Framer`
   - `<YOUR_INITIALS><TWO_DIGITS>-Feasibility Checker`
   - `<YOUR_INITIALS><TWO_DIGITS>-Method Recommender`

   Click **Add to agent**. All three should now appear on the main agent page.

#### 3.4.3 Adding Behavior and Workflow Logic

1. Click **Behavior** on the left menu and enter the following in the **Instructions** field:

   ```
   You are the Decision Design Assistant — the primary interface for decision scientists. You coordinate a three-step workflow. Keep your own messages brief. Let the collaborator agents do the detailed work.

CRITICAL RULE: Follow the sequential order strictly. Do not route to the next agent until the current agent has fully completed its work.

CRITICAL RULE: Pass all accumulated context forward at each handoff — each agent needs everything the prior agents established.

─────────────────────────────────────────
YOUR WORKFLOW:
─────────────────────────────────────────

STEP 1 — CONTEXT FRAMER
Route the user's problem to the Context Framer.

Do NOT advance to Step 2 until ALL of the following are confirmed:
•	Problem category assigned (one of the 8 statistics categories)
•	All vague business terms operationalized with user-confirmed definitions
•	Compound problem check complete
•	Classification confirmed by the user
•	Decision intent captured
•	Success criteria defined

If the Context Framer's output contains any assumed, inferred, or placeholder values — phrases like 'e.g.', 'such as', 'TBD', or 'to be confirmed' — route back to the Context Framer with the outstanding items explicitly listed.

STEP 2 — FEASIBILITY CHECKER
Pass the complete problem framing to the Feasibility Checker.

Do NOT advance to Step 3 until ALL of the following are confirmed:
•	Data wish list brainstormed and availability assessed
•	Timeline assessed
•	Technical constraints assessed
•	Output expectations captured
•	Organizational constraints assessed
•	Overall feasibility rating issued (FEASIBLE / CONDITIONALLY FEASIBLE / NOT FEASIBLE)

If NOT FEASIBLE AS DEFINED: present the blockers to the user and ask whether they want to reframe the problem (route back to Context Framer) or proceed with documented risks.

STEP 3 — METHOD RECOMMENDER
Pass the complete problem framing AND feasibility assessment to the Method Recommender.

STEP 4 — FINAL SUMMARY
After all three agents complete, compile a Decision Design Summary:

Problem Framing
•	Category:
•	Problem Statement:
•	Decision Intent:
•	Decision-Maker:
•	Success Criteria:

Feasibility Assessment
•	Overall Rating:
•	Key Constraints:
•	Risks and Mitigations:

Recommended Approach
•	Primary Method:
•	Alternatives:
•	Recommended Contact:

Suggested Next Steps

─────────────────────────────────────────
OTHER GUIDELINES:
─────────────────────────────────────────
•	If the user asks to skip a step, explain its value briefly but respect their choice.
•	If the user asks a simple question (not an analytical problem), answer directly 
   ```

[← Back to Table of contents](#table-of-contents)

---

## 4. Testing the Decision Design Assistant

Now we're all set to test the full workflow. Start chatting in the **Preview** tab shown on the right side of the Agent Builder screen for the Master agent.

### Test 1: Full pipeline (Scenario 1 — Guest Satisfaction Drivers)

Enter the following:

```
Our resort hotel guests aren't as happy as they used to be. We need to figure out what's going on.
```

Verify the workflow:

- [ ] **Context Framer invoked** → asks you to define "not as happy" (which metric?) and "resort hotels" (which properties?). Classifies as **Regression & Correlation Analysis**. Confirms before moving on.
- [ ] **Feasibility Checker invoked** → leads with the data wish list BEFORE asking what you have. Asks about current baseline. Asks about output interaction model.
- [ ] **Method Recommender invoked** → returns exactly 3 methods from the Regression & Correlation section of the KB. Each includes runtime, tradeoff, and team contact.
- [ ] **Final Decision Design Summary produced** — shareable format with all sections populated.

### Test 2: Trojan horse detection (Scenario 4 — Special Event Strategy)

Enter:

```
I need to figure out the best pricing to attract young families to special events at the parks.
```

Verify:

- [ ] Context Framer catches that this is a **compound problem** (pricing optimization + audience segmentation + demand forecasting + event attribution).
- [ ] It operationalizes "young families" and "special events."
- [ ] It flags any sub-problem that falls outside the statistics scope.

### Test 3: NOT FEASIBLE path

Enter:

```
We want to predict next year's attendance by park and by day, but we can't share any historical attendance data — it's locked behind legal review for another 6 months. Deadline is Friday.
```

Verify:

- [ ] Feasibility Checker rates this **NOT FEASIBLE**.
- [ ] Orchestrator offers to reframe or proceed with documented risks.

You have now successfully tested the end-to-end multi-agent workflow, including the happy path, compound problem detection, and the infeasibility off-ramp.

[← Back to Table of contents](#table-of-contents)

---

## 5. Deploying the Agent (Optional)

In this section, we will deploy the Decision Design Assistant along with its child agents so they are accessible through the watsonx Orchestrate chat.

1. Turn on the toggle for **Home page** so your agent shows up in the watsonx Orchestrate Chat home page once deployed.
2. Click the **Deploy** button in the top-right corner.
3. Click **Deploy** again in the bottom-right of the Pre-deployment summary page. It may take a few seconds.
4. To test from the AI Chat window, click the **☰** hamburger menu and select **Chat**.
5. Make sure your **Decision Design Assistant** is selected in the dropdown. You can now test your agent from the chat interface.

[← Back to Table of contents](#table-of-contents)

---

## 6. Summary

In this guide, we successfully built a multi-agent Decision Design Assistant in watsonx Orchestrate designed to automate the intake and framing of analytical requests. The workflow retrieves an incoming problem, classifies and operationalizes it, assesses feasibility, and produces a ranked set of method recommendations.

Throughout the guide, we accomplished the following key learning objectives:

- Built 3 specialized child agents, each with its own knowledge base attached during the build:
  - **Context Framer** (+ Problem Taxonomy KB) — classifies and operationalizes incoming problems.
  - **Feasibility Checker** (+ Feasibility Checklist KB) — brainstorms the data wish list and assesses constraints.
  - **Method Recommender** (+ Method Reference KB) — suggests 3 ranked statistical methods.
- Built a **master orchestrator** — the Decision Design Assistant — to coordinate the three child agents sequentially and compile a shareable summary.
- Added behavior instructions to enable intelligent coordination and context-passing between agents.
- Tested the solution across the happy path, compound problem detection, and infeasibility scenarios.
- Optionally deployed the agent for real-time interaction through the watsonx Orchestrate chat.

By completing this guide, you have gained hands-on experience in building multi-agent solutions that embed decision-science intuition into the intake process, reduce misaligned projects early, and give junior analysts a scaffolded way to frame problems like a senior.

[← Back to Table of contents](#table-of-contents)
