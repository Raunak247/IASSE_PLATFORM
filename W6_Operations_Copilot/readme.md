# RIPCE — Researcher Intelligence & Personalized Conference Experience

> **IAASSE W6 Deliverable** — AI-powered operations platform built on n8n Cloud  
> **Instance:** `raunak420.app.n8n.cloud`

---

## 📌 Overview

RIPCE is a fully automated research conference intelligence system. It registers researchers, analyzes their profiles using AI, generates personalized conference recommendations, and produces detailed participation reports — all triggered by a single webhook call.

The W6 layer adds a complete operations platform: health monitoring, AI chat assistant, living documentation, and SOP generation.

---

## 🏗️ Architecture

```
POST /webhook/ripce-register
        ↓
      WF1 — Registration & Validation
        ↓
      WF2 — AI Profile Analyzer & Similarity Engine
        ↓
      WF3 — Recommendations Engine
        ↓
      WF4 — Report Generation
```

### W6 Operations Layer
```
W6-A — Health Monitor        (runs hourly)
W6-B — Knowledge Copilot     (AI chat for IAASSE staff)
W6-C — Living Documentation  (runs daily)
W6-D — SOP & Incident        (on-demand via webhook)
```

---

## 🔄 Workflows

| ID | Name | Webhook | Status |
|----|------|---------|--------|
| WF1 | Researcher Registration & Validation | `POST /webhook/ripce-register` | ✅ Active |
| WF2 | AI Research Analyzer & Similarity Engine | `POST /webhook/ripce-analyze` | ✅ Active |
| WF3 | Recommendations Engine | `POST /webhook/ripce-recommend` | ✅ Active |
| WF4 | Report Generation Engine | `POST /webhook/ripce-report` | ✅ Active |
| W6-A | Health Monitor | Scheduled (hourly) | ✅ Active |
| W6-B | IAASSE Knowledge Copilot | Chat UI | ✅ Active |
| W6-C | Living Documentation Engine | Scheduled (daily) | ✅ Active |
| W6-D | SOP & Incident Response | `POST /webhook/iaasse-sop` | ✅ Active |

---

## 🚀 Quick Start

### Register a Researcher

```bash
curl -X POST https://raunak420.app.n8n.cloud/webhook/ripce-register \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Dr. Raunak Sharma",
    "email": "raunak@ycce.in",
    "research_domain": "Artificial Intelligence",
    "abstract": "Exploring deep learning for NLP in low-resource languages.",
    "organization": "YCCE Nagpur",
    "country": "India",
    "keywords": "deep learning, NLP, transformers",
    "years_of_experience": 3
  }'
```

**Response:**
```json
{
  "success": true,
  "researcher_id": "RIPCE-20260625-MQTDXB3W",
  "message": "Registration successful! Your personalized AI report will be generated shortly."
}
```

### Get an SOP

```bash
curl -X POST https://raunak420.app.n8n.cloud/webhook/iaasse-sop \
  -H "Content-Type: application/json" \
  -d '{
    "request_type": "sop",
    "sop_name": "daily_operations"
  }'
```

### Troubleshoot an Error

```bash
curl -X POST https://raunak420.app.n8n.cloud/webhook/iaasse-sop \
  -H "Content-Type: application/json" \
  -d '{
    "request_type": "troubleshoot",
    "error_message": "MongoDB insert failed",
    "workflow_name": "WF1",
    "severity": "high"
  }'
```

---

## 📋 Required Fields

### WF1 Registration
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | ✅ | Full name of researcher |
| `email` | string | ✅ | Email address |
| `research_domain` | string | ✅ | Primary research domain |
| `abstract` | string | ✅ | Research abstract |
| `organization` | string | ❌ | Institution name |
| `country` | string | ❌ | Country |
| `keywords` | string | ❌ | Comma-separated keywords |
| `years_of_experience` | number | ❌ | Years of research experience |

---

## 🔑 Credentials

| Credential Name | Type | Used By | Status |
|----------------|------|---------|--------|
| `Anthropic account` | Anthropic API | WF2, WF3, W6-B, W6-C, W6-D | ✅ Configured |
| `Google Gemini(PaLM) Api account` | Google Gemini | WF2, WF3, WF4, W6-B, W6-C, W6-D | ✅ Configured |
| `OpenAI account` | OpenAI API | W6-B, W6-C, W6-D, WF4 | ✅ Configured |
| `Google Sheets account` | Google Sheets OAuth2 | WF1 | ✅ Configured |
| `MongoDB account` | MongoDB | WF1, WF2, WF3 | ✅ Configured |
| `n8n API Key` | Header Auth | W6-A, W6-C | ⚠️ Setup required |

---

## 🛠️ Admin Operations

### Rotate API Key
1. Go to n8n → **Settings → Credentials**
2. Click the credential to update
3. Replace the API key value
4. Click **Save** — no workflow restart needed

### Modify AI Prompts
- **WF2:** Open workflow → click `Build AI Profile` node → edit the profile logic
- **WF3:** Open workflow → click `Build Recommendations` node → edit recommendation fields
- **WF4:** Open workflow → click `Generate & Send Report` node → edit report template

### Add New Research Domain
1. Open WF2 → click `Build AI Profile` node
2. Add domain detection logic in the `sub_domains` expression
3. Save and test

### Create New Campaign
POST to `/webhook/ripce-register` with researcher data — the entire pipeline runs automatically.

### Check Failed Executions
1. Go to `raunak420.app.n8n.cloud/executions`
2. Filter by **Error** status
3. Click any execution to see which node failed and why

---

## 📊 W6 Operations Platform

### W6-A: Health Monitor
- Runs **every hour** automatically
- Computes a **0–100 health score** based on execution success rates
- Fires an alert to `/webhook/iaasse-alert` if score drops below 80
- Logs to `iaasse_health_log` Data Table

### W6-B: Knowledge Copilot
- AI chat interface for IAASSE staff
- Knows the full RIPCE system architecture
- Access via Chat URL in workflow Settings tab
- Ask anything: "How does WF2 work?", "Why did this fail?", "How do I rotate credentials?"

### W6-C: Living Documentation
- Runs **daily** to regenerate system documentation
- Triggered manually anytime via **Execute workflow** button
- Stores versioned Markdown docs in `iaasse_documentation` Data Table

### W6-D: SOP & Incident Response
- **Endpoint:** `POST /webhook/iaasse-sop`
- `request_type: "sop"` — generates any operational SOP
- `request_type: "troubleshoot"` — diagnoses errors with root cause analysis

**Available SOPs:**
- `daily_operations`
- `weekly_monitoring`
- `monthly_maintenance`
- `quarterly_review`
- `incident_response`
- `credential_rotation`
- `backup_recovery`
- `prompt_update`
- `workflow_activation`
- `new_campaign_launch`

---

## 🩺 Troubleshooting

| Error | Likely Cause | Fix |
|-------|-------------|-----|
| WF1 returns 400 | Missing required fields | Include `name`, `email`, `research_domain`, `abstract` |
| WF2 not triggering | WF1 failed before trigger step | Check WF1 execution log |
| WF3 not triggering | WF2 failed | Check WF2 execution log |
| WF4 not triggering | WF3 failed | Check WF3 execution log |
| MongoDB fails | Credential expired or IP not whitelisted | Update MongoDB credential, whitelist IP in Atlas |
| Google Sheets fails | OAuth token expired | Re-authorize Google Sheets credential |
| Gemini API fails | Invalid or expired API key | Replace key in Google Gemini credential |
| W6-A failing hourly | n8n API Key not configured | Create Header Auth credential with n8n API token |

---

## 📁 Data Tables

| Table | Contents |
|-------|----------|
| `iaasse_health_log` | Hourly health scores and execution stats |
| `iaasse_documentation` | Versioned system documentation |
| `iaasse_sop_log` | History of SOP and troubleshoot requests |
| `ripce_reports` | Generated conference participation reports |

---

## 🔗 Links

| Resource | URL |
|----------|-----|
| n8n Instance | https://raunak420.app.n8n.cloud |
| Executions Log | https://raunak420.app.n8n.cloud/executions |
| Credentials | https://raunak420.app.n8n.cloud/credentials |
| W6-B Copilot | Open W6-B workflow → Settings → Chat URL |

---

## 👥 Project Info

- **Project:** IAASSE W6 — Autonomous AI Operations Copilot
- **Platform:** n8n Cloud
- **AI Models:** Google Gemini 2.0 Flash, OpenAI GPT-4o
- **Built:** June 2026