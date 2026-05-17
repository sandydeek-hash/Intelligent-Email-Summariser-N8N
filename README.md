# 📧 AI Email Summarizer — n8n Workflow

> Automated daily email digest using n8n, Gmail, and OpenAI GPT-4o Mini

---

## 🧩 Problem

Managing a high volume of daily emails is time-consuming and cognitively expensive. Key information and action items get buried. The goal was to build a lightweight, automated workflow that reads the latest emails once a day and delivers a clean, structured summary — without manual intervention.

---

## 💡 Solution

An n8n workflow that:
1. Fetches the latest emails from Gmail (last 24 hours)
2. Aggregates and structures the email data
3. Passes the content to OpenAI GPT-4o Mini with a structured prompt
4. Sends a formatted summary email with bullet points and action items back to the inbox — automatically

---

## 🏗️ Workflow Architecture

```
Manual / Schedule Trigger
        ↓
  Gmail Node (Get Emails)
        ↓
  Aggregate Node (Extract: id, snippet, Subject, From, To)
        ↓
  Wait Node (2 seconds — rate limit buffer)
        ↓
  OpenAI GPT-4o Mini (Summarize + Action Items)
        ↓
  Gmail Node (Send Summary Email)
```

---

## 🔧 Tech Stack

| Component | Tool |
|-----------|------|
| Workflow Automation | [n8n](https://n8n.io) |
| Email Integration | Gmail (OAuth2) |
| AI Model | OpenAI GPT-4o Mini |
| Trigger | Manual (extensible to daily schedule) |
| Output | Formatted summary email |

---

## 🤖 AI Prompt Design

The OpenAI node uses a structured system prompt to enforce consistent output format:

```
You are an expert email assistant. Summarize all emails into quick summaries 
in bullet points along with action items if any. Use a professional tone. 
Include relevant links.

Structure:
- Subject line
- Short intro
- Bullet summaries
- Links section
```

**Design decision:** Emails are concatenated into a single prompt (not one API call per email) to stay within rate limits and reduce latency.

---

## ⚙️ Setup Instructions

### Prerequisites
- n8n account (cloud or self-hosted)
- Gmail account with OAuth2 credentials configured in n8n
- OpenAI API key

### Steps

1. **Import the workflow**
   - In n8n, go to **Workflows → Import from File**
   - Upload `workflow/email-summarizer-sanitized.json`

2. **Configure Gmail credentials**
   - In the `Get many messages` node → set your Gmail OAuth2 credential
   - In the `Send a message` node → set the same Gmail credential
   - Update the `sendTo` field with your email address

3. **Configure OpenAI credentials**
   - In the `Message a model` node → add your OpenAI API key

4. **Set the date filter** *(optional)*
   - In the `Get many messages` node, update `receivedAfter` to a dynamic expression for rolling 24-hour window:
     ```
     {{ $now.minus(1, 'days').toISO() }}
     ```

5. **Switch to a scheduled trigger** *(for daily automation)*
   - Replace the manual trigger with a **Schedule Trigger** node
   - Set to run once daily (e.g. 7:00 AM)

6. **Test and activate**
   - Run manually first to validate output
   - Activate the workflow for automated daily runs

---

## 📌 Key Design Decisions

| Decision | Rationale |
|----------|-----------|
| GPT-4o Mini over GPT-4o | Higher rate limits, lower cost, sufficient quality for summarization |
| Aggregate node before AI | Consolidates email fields cleanly before passing to the model |
| Wait node (2 sec) | Prevents rate limit errors on OpenAI API |
| Single prompt for all emails | One API call per run — avoids burst rate limit issues |
| `slice(1).join()` in send node | Strips the subject line from the body to use separately in email subject field |

---

## 🚧 Limitations & Future Improvements

- Currently hardcoded to fetch last 2 emails — extend to dynamic count
- Date filter is static — replace with `$now` expression for true daily automation
- No error handling node — add a fallback email if the AI call fails
- Could add Gmail label filtering to summarize only specific categories (e.g. newsletters, client emails)

---

## 📁 Repo Structure

```
email-summarizer-n8n/
├── README.md                          ← This file (case study)
├── workflow/
│   └── email-summarizer-sanitized.json  ← Importable n8n workflow (credentials removed)
└── screenshots/
    └── workflow-canvas.png            ← Add your own screenshot here
```

---

## 👩‍💼 About

Built by **Sandhya** — Senior Technical Product Manager with expertise in AI-powered platforms and workflow automation.

This project is part of a broader AI TPM portfolio demonstrating hands-on ability to design, build, and ship AI-integrated workflows.

🔗 [LinkedIn](#) | [Portfolio](#)

---

*Note: All credentials have been removed from the exported workflow. You will need to configure your own Gmail OAuth2 and OpenAI API credentials in n8n before running.*
