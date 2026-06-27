# 🏛️ Confident Group — AI Lead Qualification System

> **Stage 2 Technical Assessment · Task 1**
> AI Automation & Integration Executive · Bangalore Technology Team

![Workflow](https://img.shields.io/badge/Built%20with-n8n-orange?style=flat-square)
![AI](https://img.shields.io/badge/AI-Google%20Gemini-blue?style=flat-square)
![CRM](https://img.shields.io/badge/CRM-HubSpot-red?style=flat-square)
![Status](https://img.shields.io/badge/Status-Live-brightgreen?style=flat-square)

---

## 📌 Overview

An end-to-end **AI-powered lead qualification workflow** built for Confident Group, a premium real estate developer in Kerala, India.

When a potential buyer submits a property enquiry via Google Forms, the system automatically:
- Scores the lead using **Google Gemini AI** (1–10)
- Classifies them as **HOT / WARM / COLD**
- Logs them to a **Google Sheets audit trail**
- Creates or updates a **HubSpot CRM contact** with AI scores
- Sends a **branded HTML email** to the sales team
- Posts a **Slack alert** with full lead details and next action

All of this happens in **under 3 seconds** — with zero manual intervention.

---

## 🏗️ System Architecture

```
Customer fills Google Form
         │
         ▼
┌─────────────────────┐
│  Google Sheets      │  ← Form responses auto-saved here
│  (Trigger — new row)│
└────────┬────────────┘
         │
         ▼
┌─────────────────────┐
│  Google Gemini AI   │  ← Scores lead 1–10, classifies HOT/WARM/COLD
│  (HTTP Request)     │     Returns: score, classification, reason,
└────────┬────────────┘     recommended_action, priority_flag
         │
         ▼
┌─────────────────────┐
│  Parse AI Response  │  ← Code node: safely parses JSON, merges
│  (Code Node)        │     lead data + AI result, adds emoji/color
└────────┬────────────┘
         │
    ┌────┴────────────────────────┐
    │                             │
    ▼                             ▼
┌──────────┐              ┌────────────────────┐
│ Log Lead │              │   IF: HOT Lead?    │
│ (Sheets) │              └──────┬─────────────┘
└──────────┘                     │
                      ┌──────────┴──────────┐
                      │ YES (HOT)           │ NO (WARM/COLD)
                      ▼                     ▼
              ┌───────────────┐    ┌───────────────┐
              │ HubSpot (HOT) │    │ HubSpot (W/C) │
              │ Gmail (HOT)   │    │ Gmail (W/C)   │
              │ Slack (HOT)   │    │ Slack (W/C)   │
              └───────────────┘    └───────────────┘
```

---

## 🔧 Tech Stack

| Component | Tool |
|---|---|
| Automation engine | [n8n](https://n8n.io) |
| Lead capture | Google Forms + Google Sheets |
| AI scoring | Google Gemini AI (via REST API) |
| CRM | HubSpot (Free tier) |
| Email notification | Gmail (OAuth2) |
| Team alerts | Slack |
| Audit trail | Google Sheets (Lead Log tab) |

---

## 📋 Google Form Fields

The form captures the following lead details:

| Field | Type |
|---|---|
| Full Name | Short text |
| Email | Short text |
| Phone Number | Short text |
| Which project are you interested in? | Dropdown |
| Location Preference | Dropdown |
| Budget Range | Dropdown |
| Purpose of Purchase | Dropdown |
| Timeline to Buy | Dropdown |
| How did you hear about us? | Dropdown |
| Any specific requirements? | Paragraph |

---

## 🤖 AI Scoring Logic

The Google Gemini AI node evaluates each lead on four dimensions:

| Dimension | Weight |
|---|---|
| Budget fit (vs. project pricing) | High |
| Purchase intent (self-use vs. investment) | High |
| Timeline urgency (within 3 months vs. exploring) | Medium |
| NRI / high-value investor potential | High |

**Output JSON from AI:**
```json
{
  "score": 8,
  "classification": "HOT",
  "reason": "High budget, NRI investor, immediate timeline.",
  "recommended_action": "Call within 1 hour. Offer site visit this weekend.",
  "priority_flag": true
}
```

**Classification thresholds (AI-determined):**

| Label | Meaning | Action |
|---|---|---|
| 🔥 HOT | Score 7–10, high intent | Immediate call + priority email + Slack ping |
| 🌤️ WARM | Score 4–6, moderate interest | Follow-up within 24 hrs |
| ❄️ COLD | Score 1–3, just exploring | Add to nurture sequence |

---

## 📂 Repository Structure

```
confident-group-lead-qualification/
│
├── workflow/
│   └── confident_group_lead_qualification_FINAL.json   ← n8n workflow (import this)
│
└── README.md                                            ← You are here
```

---

## 🚀 Setup Instructions

### Prerequisites
- [n8n account](https://app.n8n.cloud) (free trial)
- [Google account](https://google.com) (Forms + Sheets + Gmail)
- [Google AI Studio API key](https://aistudio.google.com/app/apikey) (free)
- [HubSpot free account](https://hubspot.com)
- [Slack workspace](https://slack.com)

---

### Step 1 — Import the Workflow

1. Log in to [app.n8n.cloud](https://app.n8n.cloud)
2. Click **New Workflow**
3. Click ⋮ menu → **Import from File**
4. Upload `confident_group_lead_qualification_FINAL.json`

---

### Step 2 — Set Up Google Form & Sheet

1. Create a Google Form with the fields listed above
2. In the Form → **Responses** tab → click the green Sheets icon
3. This links responses to a Google Sheet automatically
4. In n8n, connect **Node 1 (Google Sheets Trigger)** to this sheet

---

### Step 3 — Create the Lead Log Sheet

1. Open the linked Google Sheet
2. Add a second tab named **`Lead Log`**
3. Add these column headers in Row 1:

```
Timestamp | Full Name | Email | Phone | Property Interest | Location |
Budget | Purpose | Timeline | Source | Requirements |
AI Score | Classification | Reason | Recommended Action | Priority Flag
```

---

### Step 4 — Add Your API Key (Gemini AI)

1. Go to [aistudio.google.com/app/apikey](https://aistudio.google.com/app/apikey)
2. Click **Create API Key** → copy it
3. In n8n → open **Node 2 (Claude AI — Score Lead)**
4. In the Header Parameters, replace `YOUR_ANTHROPIC_API_KEY` with your Gemini key
5. Update the URL to:
```
https://generativelanguage.googleapis.com/v1beta/models/gemini-1.5-flash:generateContent?key=YOUR_KEY
```

---

### Step 5 — Connect Credentials

In n8n → **Credentials** tab, connect:

| Service | Credential Type |
|---|---|
| Google Sheets | Google Sheets OAuth2 |
| Gmail | Gmail OAuth2 |
| HubSpot | HubSpot App Token |
| Slack | Slack API |

---

### Step 6 — Create HubSpot Custom Properties

In HubSpot → **Settings → Properties → Create Property**:

| Property Name | Type |
|---|---|
| `lead_score` | Number |
| `lead_classification` | Single-line text |
| `ai_recommended_action` | Single-line text |
| `priority_flag` | Checkbox |
| `property_interest` | Single-line text |
| `budget_range` | Single-line text |
| `purchase_purpose` | Single-line text |
| `buy_timeline` | Single-line text |
| `lead_source` | Single-line text |

---

### Step 7 — Update Placeholders

Open each node and replace these placeholders:

| Placeholder | Replace With |
|---|---|
| `YOUR_GOOGLE_SHEET_ID` | ID from your Form-linked Sheet URL |
| `YOUR_LEAD_LOG_SHEET_ID` | ID of the sheet with Lead Log tab |
| `YOUR_ANTHROPIC_API_KEY` | Your Google Gemini API key |
| `YOUR_SALES_TEAM_EMAIL@gmail.com` | Your sales email address |
| `YOUR_SLACK_CHANNEL_ID` | Slack → right-click channel → Copy link |

---

### Step 8 — Test & Activate

1. Click **Test Workflow** in n8n
2. Submit a test entry in your Google Form
3. Watch data flow through all 11 nodes live
4. Fix any field mapping issues (n8n shows exactly where it breaks)
5. Click **Activate** (toggle top right) → workflow is now live 24/7

---

## 📧 Email Notification Preview

The sales team receives a branded HTML email for every lead:

- **Subject:** `🔥 HOT Lead: John Smith — Villa (2Cr+)`
- **Body:** Confident Group branded header, AI score card with colour-coded classification, full lead details table, recommended action highlighted in green, timestamp

---

## 📊 What Gets Stored

Every lead is saved in **two places**:

**Google Sheets (Lead Log)** — your audit trail:
- All 10 form fields
- AI score, classification, reason
- Recommended action, priority flag
- Timestamp

**HubSpot CRM** — your sales pipeline:
- Contact created/updated with email as unique identifier
- All AI scores stored as custom properties
- Ready for deal creation and follow-up sequences

---

## 🔔 Slack Alert Format

```
🔥 HOT LEAD — Confident Group
⭐ PRIORITY — NRI or High-Value Lead

Name: John Smith
Phone: +91-98765-43210  |  Email: john@example.com
Interest: Villa in Kochi
Budget: 2Cr+  |  Purpose: NRI Investment
Timeline: Within 3 months  |  Source: Referral

━━━━━━━━━━━━━━━━━━━━
AI Score: 9/10  →  HOT
Reason: NRI buyer with 2Cr+ budget and immediate timeline.
Next Action: Call within 1 hour. Offer Heritage Homes site visit.
━━━━━━━━━━━━━━━━━━━━
Requirements: Looking for Vastu-compliant 4BHK
Received: 27/06/2026, 14:32
```

---

## ⚠️ n8n Free Trial Limits

| Limit | Value |
|---|---|
| Active workflows | 5 |
| Executions / month | 2,500 |
| Google Sheets poll interval | Every 60 seconds |
| AI API cost | Gemini free tier — 1,500 requests/day |

---

## 🧠 Built By

**Sanooj Babu Kakkoth**
AI Automation & Integration Executive Candidate
Confident Group — Bangalore Technology Team · 2026

---

## 📄 Licence

This project was built as part of a technical assessment for Confident Group and is intended for demonstration purposes.
