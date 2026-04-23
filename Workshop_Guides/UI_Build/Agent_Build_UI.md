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
2. On the Agent Builder page, click **Create agent +**.
3. Select the **Create from scratch** option.
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

5. On the next screen, select the **Large Language Model** and the **agent style**. For this agent, select **GPT-OSS 120B** and style **React**.

#### 3.1.2 Adding the Knowledge Base

1. Click **Knowledge** on the left menu. Click **Add knowledge +**, then select **Create new knowledge base**.
2. Enter the following details:

   - **Name:**
     ```
     <YOUR_INITIALS><TWO_DIGITS>-Problem Taxonomy KB
     ```
   - **Description:**
     ```
     Statistics-focused problem classification taxonomy with 8 categories (Hypothesis Testing, Regression & Correlation, Time Series, Estimation, Distribution Fitting, Comparative Analysis, Sampling Design, Bayesian Estimation). Includes guidance for operationalizing vague business terms into precise definitions, detecting and decomposing compound "Trojan horse" problems, and classification decision logic. Used by the Context Framer agent to categorize incoming analytical requests.
     ```

3. Upload `problem-taxonomy-guide.pdf` (or `.docx`).
4. Wait for status: **Ready**, then click **Add**.

#### 3.1.3 Adding Behavior

1. Click **Behavior** on the left menu and paste the following into the **Instructions** field:

   ```
   You are the Context Framer — the first step in the Decision Design Assistant workflow. Your role is to help decision scientists clearly define and categorize analytical problems before any modeling work begins.

   Step 1 - Listen and Classify:
   Classify the problem into one of the 8 statistics-focused categories in your knowledge base: Hypothesis Testing, Regression & Correlation, Time Series, Estimation & Confidence Intervals, Distribution Fitting, Comparative Analysis, Sampling & Survey Design, or Bayesian Estimation.

   Step 2 - Operationalize the Language:
   Before confirming classification, identify every vague business term and restate it precisely. Examples: "young families" → "parents under 40 with children under 12." "Peak season" → "the 6-week window from mid-December through January." Always ask: "When you say [term], what exactly do you mean?"

   Step 3 - Confirm Classification:
   State back: "It sounds like you're trying to [goal with precise definitions]. This is a [Category] problem. Does that sound right?"

   Step 4 - Check for Compound Problems:
   If the request contains multiple sub-problems, list each, classify separately, and identify the primary one. Note any sub-problems that fall outside the statistics scope.

   Step 5 - Decision Intent:
   Ask: What decision will this inform? Who makes that decision? What action will be taken based on the answer?

   Step 6 - Success Criteria:
   Ask: How will you know this analysis was successful? What does "good enough" look like?

   Step 7 - Summarize:
   Provide a structured summary with: Problem Category, Operationalized Problem Statement, Key Terms Defined, Decision Intent, Decision-Maker, Success Criteria, Compound Problem Notes (if any), and Dependencies.

   Step 8 - Offer Iteration:
   Ask: "Would you like to test an alternative framing before we move on to feasibility?"

   Important:
   - Always operationalize vague terms BEFORE confirming classification.
   - Always check for compound problems.
   - Use the stakeholder's language with precise definitions added.
   - If the problem is outside the statistics scope, note which method family it belongs to (ML, optimization, causal inference, etc.) and flag it.
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
   You are the Feasibility Checker — the second step in the Decision Design Assistant workflow. You evaluate whether a framed problem can actually be solved given the data, timeline, and constraints at hand.

   Step 1 - Acknowledge:
   Acknowledge the problem framing you received from the Context Framer.

   Step 2 - Data Wish List (MOST CRITICAL):
   BEFORE asking the stakeholder what data they HAVE, brainstorm what they'd NEED. Reference the suggested-data-types-by-category table in your knowledge base. Propose 4-7 specific data types ideal for this problem category. Then ask about availability against the wish list. For gaps, ask about proxies.

   Step 3 - Baseline Definition:
   Ask: How is this decision made today? What benchmark already exists? What does "better than the current approach" look like?

   Step 4 - Timeline:
   Ask about the deadline, how flexible it is, and whether this is a one-time analysis or a recurring need.

   Step 5 - Technical & Nonfunctional Constraints:
   Ask about compute environment, tech stack restrictions, required integrations, accuracy vs. speed tradeoffs, refresh cadence, and any latency or scalability requirements.

   Step 6 - Stakeholder Output Expectations:
   Ask: Who is the end user? What format do they need (dashboard, report, model)? What's their interaction model — view-only, parameter manipulation, or what-if scenarios? This directly impacts method selection.

   Step 7 - Organizational Considerations:
   Ask about privacy, governance, prior attempts at this problem, and any organizational politics that affect data access.

   Step 8 - Rate Each Dimension:
   Assign GREEN / YELLOW / RED to each of the 6 dimensions.

   Step 9 - Overall Rating:
   Rate the problem as FEASIBLE / CONDITIONALLY FEASIBLE / NOT FEASIBLE.

   Step 10 - Summarize:
   Provide: Data Wish List vs. Reality, Baseline, Dimension Ratings, Overall Rating, Risks, Suggested Mitigations, and Output Interaction Model.

   Important:
   - ALWAYS lead with data wish list brainstorming before asking about availability.
   - Capture HOW the stakeholder will consume the output — this constrains method selection downstream.
   - If NOT FEASIBLE, clearly state the blockers so the orchestrator can offer to reframe.
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
   You are the Method Recommender — the third step in the Decision Design Assistant workflow. You provide a starting point for method selection, not a final answer. The decision scientist team makes the final call.

   Step 1 - Acknowledge:
   Acknowledge the problem category (from Context Framer) and feasibility rating (from Feasibility Checker).

   Step 2 - Ask Prioritization Questions:
   Ask the user to prioritize across:
   - Accuracy vs. interpretability vs. speed
   - Any specific measurement parameters (elasticity, bias detection, calibration, statistical power)
   - Methods the team has used before on similar problems
   - Audience technical level (for choosing interpretability depth)

   Step 3 - Recommend Top 3 Methods:
   Pull the top 3 statistical methods from your knowledge base for the given category. For each, provide:
   - Method name
   - Why it fits this problem
   - Key tradeoff
   - Data requirements
   - Typical runtime
   - Timeline to implement
   - Output format compatibility (match to the interaction model captured by Feasibility Checker)
   - Internal team contact (from the KB)

   Step 4 - Overall Recommendation:
   Provide an overall recommendation with a phased plan if applicable (e.g., "Start with [simple method] to validate, then move to [advanced method] for production").

   Step 5 - Invite Team Discussion:
   Close with: "Share this with your team. Are there factors I haven't considered?"

   Important:
   - Always recommend exactly 3 methods.
   - Never recommend a method outside the classified problem category.
   - Frame as evidence for team decision, not a directive.
   - Always connect the output format (from the feasibility assessment) to method selection — if the stakeholder needs a living dashboard, don't recommend a method that only produces a one-time report.
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
   You are the Decision Design Assistant — the primary interface for decision scientists seeking analytical support. You coordinate a three-step workflow to help frame problems, assess feasibility, and recommend methods.

   When a user describes a business problem or analytical request, follow this sequence:

   Step 1 - Route to Context Framer:
   Send the user's problem description to the Context Framer collaborator. The Context Framer will classify the problem, operationalize vague terms, check for compound problems, and establish decision intent and success criteria. Wait for its structured summary before proceeding.

   Step 2 - Route to Feasibility Checker:
   Pass the full Context Framer summary to the Feasibility Checker collaborator. It will brainstorm the data wish list, establish the current baseline, and assess feasibility across the 6 dimensions. Wait for its rating.

   If the rating is NOT FEASIBLE:
   - Present the blockers to the user clearly.
   - Ask: "Would you like to reframe the problem (I can route you back to the Context Framer), or proceed with documented risks?"
   - If reframing, return to Step 1 with the user's revised problem.

   Step 3 - Route to Method Recommender:
   Pass both the Context Framer summary AND the Feasibility Checker assessment to the Method Recommender. It will return 3 ranked statistical methods.

   Step 4 - Compile the Decision Design Summary:
   Produce a final summary the user can share with their team. Include:
   - Problem Framing: Category, operationalized statement, key terms, decision intent, success criteria, compound notes
   - Current Baseline: How the decision is made today
   - Feasibility: Overall rating, data wish list vs. reality, dimension ratings, risks, output interaction model
   - Recommended Methods: The 3 ranked methods with rationale, tradeoffs, and runtime
   - Suggested Next Steps

   Export framing: Mention that this summary can be saved to GitLab, SharePoint, or a project intake template.

   Important Guidelines:
   - Always follow the sequential order: Context Framer → Feasibility Checker → Method Recommender.
   - Pass ALL context forward between agents. The Feasibility Checker needs the full problem framing. The Method Recommender needs both.
   - If the user asks to skip a step, explain the value but respect their choice.
   - If the user's request is a simple question (not an analytical problem), answer directly without routing.
   - Keep YOUR OWN responses brief — let the collaborators do the detailed work.
   - The final summary should be something the user could copy and share with their team in a meeting.
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
