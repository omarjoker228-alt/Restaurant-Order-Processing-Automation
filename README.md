# 🍽️ Restaurant Order Processing Automation System

An intelligent, end-to-end system for automatically receiving and confirming restaurant orders based on a **Risk-Based Decision Engine** using **n8n** and **AI (LLM) pipelines**[cite: 1].

---

## 📌 Project Overview

The system receives direct customer orders via **WhatsApp Business API** or **Web Forms**, and analyzes the order text using AI to check item availability in stock, pricing, and delivery coverage area[cite: 1, 2]. The system automatically determines whether human review is needed based on a Risk Evaluation algorithm before sending the ticket to the kitchen[cite: 1].

### 💡 Key Differentiators from Traditional Systems:
- **Not Just a Chatbot:** It does not merely respond with static questions; it executes a complete 8-stage operational pipeline[cite: 1].
- **Not a Manual POS System:** It avoids manual entry errors during peak hours and handles special ingredient instructions smoothly[cite: 1, 2].
- **Saves Marketplace Commissions:** Targeted at direct orders owned by the restaurant to save delivery app commissions ranging from 15% to 30%[cite: 2].

---

## 🏗️ Workflow Architecture

The operational pipeline consists of 8 core steps inside the **n8n** automation engine[cite: 1]:

1. **Trigger:** Incoming message (text, voice, or image) received via Webhook[cite: 1, 6].
2. **Preprocessing (Media Conversion):**
   - Converts voice notes to text using **Whisper API**[cite: 2, 6].
   - Analyzes menu images and order photos using **Vision API**[cite: 2, 6].
3. **Dedup Check:** Verifies no duplicate orders from the same user within a 2-minute time window to prevent double charging[cite: 5].
4. **AI Extraction:** Extracts meal details, quantities, and critical notes with `temperature = 0` to ensure deterministic outputs[cite: 5, 6].
5. **Menu & Stock Validation:** Matches extracted data against the database using **Fuzzy Matching** to verify live stock and menu items[cite: 5].
6. **Risk Evaluation:** Categorizes the order into decision paths (Low, Medium, High, Direct Reject)[cite: 1, 4].
7. **Execution & Routing:** Dispatches kitchen tickets, sends instant alerts via Telegram/Slack to supervisors, or returns polite rejection/alternative messages to the customer[cite: 4, 6].
8. **Storage & Daily Reporting:** Stores encrypted data and generates automated daily reports[cite: 1, 5].

---

## 🎯 Risk Matrix & Decision Logic

The decision tree is constructed based on the financial or health impact of potential decision errors[cite: 4]:

| Risk Level | Decision Criteria & Thresholds | Automated Action Taken |
| :--- | :--- | :--- |
| **Low Risk** | Standard order, all items in stock, normal order value, and within delivery range[cite: 4]. | Instant automated confirmation + kitchen ticket dispatched + customer notified of prep time[cite: 4]. |
| **Medium Risk** | Order value exceeds medium threshold, or ambiguous special request[cite: 4]. | Instant employee alert via Telegram/Slack with a 5-minute timeout before auto-escalating to the branch manager[cite: 4, 6]. |
| **High Risk** | Large bulk orders (Catering) or explicit food allergy alerts[cite: 4]. | Immediately halts automation and routes the order to the branch manager for full manual review[cite: 4]. |
| **Direct Reject** | Item out of stock or delivery address explicitly outside coverage area[cite: 4]. | Polite automated apology message with alternative item suggestions or pickup option[cite: 4]. |

> **Note:** All monetary thresholds, distance ranges, and timeout durations are fully configurable via the `thresholds` table in the database without modifying source code (Non-Hardcoded)[cite: 4].

---

## 🛠️ Tech Stack

- **Automation Engine:** n8n (Self-Hosted on Docker)[cite: 6]
- **Database:** PostgreSQL (SSL Encrypted)[cite: 5, 6]
- **AI Pipelines:** OpenAI GPT-4o-mini / Gemini Flash (`temperature = 0`)[cite: 5, 6]
- **Media Processing:** Whisper API (Voice-to-Text) & Vision API (Image Processing)[cite: 2, 6]
- **Communication APIs:** WhatsApp Business API, Telegram / Slack Webhooks[cite: 6]

---

## 🔒 Reliability, Security & Cost Efficiency

- **Deterministic AI Outputs:** Fixed `temperature = 0` setting and saving `raw AI response` for auditability[cite: 5].
- **Error Handling:** Uses `throw Error` and `Error Trigger Nodes` to transition ambiguous orders to `needs_clarification` status and alert staff immediately[cite: 5, 6].
- **Data Privacy & Security:** Full SSL encryption for database connections, Role-Based Access Control (RBAC), and customer PII anonymization policies[cite: 5].
- **Cost Efficiency (Cost Metrics):**
  - Average usage: 300 - 600 Tokens per order[cite: 5].
  - Estimated cost per order: Less than `$0.00055`[cite: 5].
  - Processing 50 orders daily costs less than `$1.00` per month in AI API usage[cite: 5].

---

## 🗄️ Database Schema Preview

The `sql/` directory contains DDL scripts to set up the following tables[cite: 6]:

- `menu_items`: Item details, pricing, and base ingredients[cite: 2].
- `stock`: Real-time stock status[cite: 6].
- `orders`: Order history, evaluation outputs, and current fulfillment status[cite: 2, 5, 6].
- `thresholds`: Flexible risk limits, distance ranges, and timeouts[cite: 4].

---

## 🚀 Quick Start

1. **Database Setup:** Execute `sql/01_schema.sql` on your PostgreSQL database instance[cite: 6].
2. **Import Workflow:** Import `workflows/n8n-main-pipeline.json` into your n8n instance[cite: 6].
3. **Environment Configuration:** Create a `.env` file based on `.env.example` and add your API credentials for OpenAI/Gemini and WhatsApp Business[cite: 6].
