# A2A-support-ticket-router
Multi-agent A2A customer support pipeline —  6 AI agents | Google Gemini | n8n | Sentiment |  Quality Judge | Escalation Routing


# 🤖 A2A Customer Support Ticket Router (Enhanced)

A **6-agent AI pipeline** built in [n8n](https://n8n.io) that automates end-to-end customer support ticket handling using the **Agent-to-Agent (A2A)** architectural pattern. Each specialized agent performs a single well-defined job and passes its structured output to the next — no monolithic prompts, no guesswork.

> **Processes a ticket in under 25 seconds** — from raw customer message to quality-evaluated reply, escalation response, and structured audit log.

---

## 🏗️ Pipeline Architecture

```
Chat Trigger
    │
    ▼
Sentiment Agent       ← tone + urgency detection
    │
    ▼
Classifier Agent      ← billing / technical / general
    │
    ▼
Response Agent        ← empathy-adjusted email draft
    │
    ▼
Quality Judge Agent   ← LLM-as-Judge scoring (1–10)
    │
    ├──────────────────────────────────┐
    ▼                                  ▼
Escalation Agent              Ticket Summary Agent
(senior priority reply)       (structured JSON audit log)
```

The pipeline is **linear-then-parallel**: the first four agents run in sequence, then the Quality Judge fans out to both the Escalation Agent and Ticket Summary Agent simultaneously.

---

## ⚙️ Tech Stack

| Component | Technology | Purpose |
|-----------|-----------|---------|
| Workflow Engine | n8n v2.2.4 | Visual AI pipeline orchestration |
| AI Model | Google Gemini 2.5-flash | LLM powering all 6 agents |
| Protocol | Agent-to-Agent (A2A) | Inter-agent communication pattern |
| Trigger | n8n Chat Trigger | Built-in chat interface for testing |
| Output | JSON + Chat Response | Structured ticket log + draft reply |

---

## 🧩 Agent Breakdown

### 1. Sentiment Agent
Analyzes the customer message for emotional tone and urgency.

**Output:**
```json
{
  "sentiment": "angry",
  "urgency": "high",
  "reason": "Customer reports triple billing and demands immediate refund"
}
```

### 2. Classifier Agent
Labels the ticket category using the message and sentiment context.

**Output:** Single word — `billing`, `technical`, or `general`

### 3. Response Agent
Drafts a customer reply with tone adapted to the sentiment and category. Angry or high-urgency tickets automatically receive more empathetic, apologetic language.

### 4. Quality Judge Agent
Evaluates the draft reply using the **LLM-as-Judge** pattern — one AI scores another AI's output.

**Output:**
```json
{
  "score": 9,
  "tone": "professional",
  "empathy": "high",
  "clarity": "clear",
  "verdict": "approved",
  "feedback": "Response addresses urgency well and maintains a calming tone."
}
```

### 5. Escalation Agent
Generates a senior priority escalation response with a Case ID, callback offer, and full context from all prior agents. Runs in parallel with the Ticket Summary Agent.

### 6. Ticket Summary Agent
Creates a structured JSON audit log for every ticket with an auto-generated ticket ID and ISO timestamp. Runs in parallel with the Escalation Agent.

**Output:**
```json
{
  "ticket_id": "TKT-20260613045942",
  "category": "billing",
  "urgency": "high",
  "sentiment": "angry",
  "summary": "Customer charged $149 three times, demands immediate refund",
  "resolution": "Agent apologized, initiated full refund for all three charges",
  "quality_score": 9,
  "escalated": true,
  "timestamp": "2026-06-13T04:59:42.979+05:30"
}
```

---

## ✅ Key Features

- **Sentiment & urgency detection** — four-level sentiment scale (positive / neutral / negative / angry)
- **Automatic ticket classification** — maps to real contact center routing queues
- **Empathy-adjusted response drafting** — tone adapts based on detected sentiment
- **LLM-as-Judge quality evaluation** — multi-dimensional scoring with no human bottleneck
- **Senior escalation routing** — context-aware priority response with Case ID and callback
- **Structured JSON audit logging** — auto-generated ticket IDs and timestamps for every interaction
- **Parallel execution** — Escalation and Summary agents run simultaneously for speed

---

## 📊 Test Results

### Test 1 — Simple Greeting (`"hi"`)

| Field | Value |
|-------|-------|
| Sentiment | neutral |
| Urgency | low |
| Category | general |
| Quality Score | 9/10 |
| Escalated | false |
| Execution Time | 16.3 seconds |

### Test 2 — Angry Billing Complaint

> *"This is absolutely ridiculous! I've been charged $149 THREE times this month and nobody is responding. I want a full refund NOW!"*

| Field | Value |
|-------|-------|
| Sentiment | angry |
| Urgency | high |
| Category | billing |
| Quality Score | 9/10 |
| Escalated | true |
| Escalation | Senior specialist, Case ID, callback offered, $447 refund confirmed |
| Execution Time | 23.3 seconds |

---

## 🚀 Getting Started

### Prerequisites

- [n8n](https://n8n.io) v2.2.4+
- Google Gemini API key (via Google AI Studio or Google Cloud)

### Import & Run

1. Clone or download this repository
2. Open your n8n instance
3. Go to **Workflows → Import from file**
4. Select `A2A_Customer_Support_Ticket_Router_Enhanced.json`
5. Open the workflow and configure the **Google Gemini (PaLM) API** credential on each Gemini model node
6. Activate the workflow and open the **Chat** panel to test

---

## 🔮 Planned Enhancements

| Enhancement | Description | Complexity |
|-------------|-------------|------------|
| Conditional Escalation | IF node to only escalate angry + high urgency tickets | Low |
| Memory Node | Window Buffer Memory for multi-turn conversations | Low |
| Email Delivery | Gmail/SendGrid node to send replies to customers | Low |
| Language Detection | Detect non-English tickets and reply in same language | Low |
| RAG Knowledge Base | Give Response Agent a vector-search tool for product FAQs | Medium |
| Database Logging | Persist Ticket Summary JSON to Postgres/BigQuery | Medium |
| Webhook Integration | Replace Chat Trigger with Avaya Infinity/Salesforce webhook | Medium |
| Guardrails Agent | Prompt injection detection before Sentiment Agent | Medium |
| Analytics Dashboard | Power BI/Looker dashboard from ticket JSON logs | High |
| Human-in-the-Loop | Approval step for low-confidence Quality Judge verdicts | High |

---

## 🧠 Design Principles

**Why 6 separate agents instead of one big prompt?**
Single responsibility. Each agent is optimized for its specific task with a focused prompt, producing more reliable and debuggable outputs. When something goes wrong, you know exactly which agent failed and why. The pipeline is also modular — swap out any agent independently without touching the others.

**Why LLM-as-Judge?**
It scales infinitely, requires no human QA bandwidth, and produces consistent, structured evaluation scores. The tradeoff is that the judge LLM can carry its own biases — in production, calibrate it against human-rated examples.

**What is the A2A protocol?**
Agent-to-Agent (A2A) is an open protocol announced by Google in April 2025 that standardizes how AI agents discover each other, delegate tasks, and pass structured outputs. This workflow implements the A2A *pattern* using n8n as the orchestrator, with each AI Agent node acting as a discrete A2A participant.

---

## 👤 Author

**Amit Garg**  
IT Architect II, Avaya | PG Program, UT Austin  
June 2026

---

## 📁 Related Projects

| Project | Pattern | Differentiator |
|---------|---------|----------------|
| GlobalEdge RAG Pipeline | RAG + Vector Search | Pinecone, OpenAI embeddings, document retrieval |
| ShopNest Ticket Intelligence | LLM Classification + Dual Judge | 10-node pipeline, GPT-4, evaluation scoring |
| **A2A Support Router (Enhanced)** | **Multi-Agent A2A Orchestration** | **6 specialized agents, sentiment + escalation routing** |

---

*Tags: `#GenAI` `#AIAgents` `#A2A` `#n8n` `#ContactCenter` `#CCaaS` `#LLM` `#GoogleGemini`*
