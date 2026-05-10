# 🤖 Job Hunt Agent

An autonomous AI agent that automatically searches for jobs, tailors resumes, and notifies you through Google Sheets and Email.

---

## 📌 Overview

This agent automatically every evening:
1. Fetches 200–300 jobs from multiple job portals
2. Filters relevant jobs based on the candidate’s skills
3. Creates a tailored resume for every relevant job
4. Organizes everything inside Google Sheets
5. Sends priority-wise summary emails

---

## 🏗️ Project Structure

```text
job-hunt-agent/
│
├── agents/                          # LangChain Agents
│   ├── orchestrator.py              # Main brain - coordinates all agents
│   ├── job_search_agent.py          # Fetches jobs from all portals
│   ├── filter_agent.py              # Filters relevant jobs
│   ├── resume_agent.py              # Tailors resumes per job
│   ├── email_agent.py               # Sends emails
│   └── referral_agent.py            # Finds startup employees
│
├── tools/                           # LangChain Tools (for each agent)
│   ├── job_search/
│   │   ├── jsearch_tool.py          # JSearch API (Indeed + Glassdoor)
│   │   ├── adzuna_tool.py           # Adzuna API (Global + India)
│   │   ├── remotive_tool.py         # Remotive API (Remote jobs)
│   │   ├── arbeitnow_tool.py        # Arbeitnow API (Tech jobs)
│   │   ├── apify_linkedin_tool.py   # Apify LinkedIn scraper
│   │   └── apify_naukri_tool.py     # Apify Naukri scraper
│   │
│   ├── filter/
│   │   ├── keyword_filter_tool.py   # Stage 1: Fast keyword matching
│   │   └── llm_relevance_tool.py    # Stage 2: LLM deep analysis
│   │
│   ├── resume/
│   │   ├── read_resume_tool.py      # Reads the resume
│   │   ├── tailor_resume_tool.py    # Creates job-specific resumes using LLM
│   │   └── save_resume_tool.py      # Saves PDF/DOCX
│   │
│   ├── email/
│   │   └── gmail_tool.py            # Sends emails using Gmail SMTP
│   │
│   └── sheets/
│       └── gsheets_tool.py          # Inserts data into Google Sheets
│
├── database/
│   └── jobs.db                      # SQLite - duplicate tracking
│
├── resumes/
│   ├── templates/
│   │   └── base_resume.docx         # Base resume template
│   └── generated/                   # Tailored resumes will be saved here
│       └── [company]_[role]_[date].pdf
│
├── config/
│   ├── settings.py                  # All configuration in one place
│   ├── candidate_profile.py         # Candidate information (skills, preferences)
│   └── .env                         # API keys (not pushed to git)
│
├── email/
│   └── templates/
│       ├── top_matches.html         # Email 1 template (Top 10 jobs)
│       ├── good_matches.html        # Email 2 template (20–30 jobs)
│       └── all_jobs.html            # Email 3 template (remaining jobs)
│
├── sheets/
│   └── sheet_manager.py             # Google Sheets management
│
├── logs/
│   └── agent.log                    # Daily logs
│
├── scheduler.py                     # APScheduler - triggers every evening
├── main.py                          # Entry point
├── requirements.txt                 # Dependencies
└── README.md                        # This file
````

---

## 🛠️ Tech Stack

| Component          | Technology                  | Cost                   |
| ------------------ | --------------------------- | ---------------------- |
| LLM                | Qwen2.5 3B (llama.cpp)      | Free (Local)           |
| Agent Framework    | LangChain                   | Free                   |
| LinkedIn + Naukri  | Apify                       | $5/month               |
| Indeed + Glassdoor | JSearch API (RapidAPI)      | Free (200 calls/month) |
| Global Jobs        | Adzuna API                  | Free (250 calls/month) |
| Remote Jobs        | Remotive + Arbeitnow        | Free (Unlimited)       |
| Resume             | python-docx + reportlab     | Free                   |
| Email              | Gmail SMTP                  | Free                   |
| Database           | SQLite                      | Free                   |
| Tracking           | Google Sheets API (gspread) | Free                   |
| Scheduler          | APScheduler                 | Free                   |

**Total Cost: ~$5/month** (only for Apify)

---

## 📊 Google Sheet Structure

### Sheet 1 — Daily Jobs

| Column         | Description                               |
| -------------- | ----------------------------------------- |
| Date           | Date when the job was found               |
| Job Title      | Name of the role                          |
| Company        | Company name                              |
| Location       | City / Remote                             |
| Job URL        | Direct link                               |
| Match Score    | Score given by the LLM (1–10)             |
| Skills Match   | Matching skills                           |
| Missing Skills | Skills not available                      |
| Resume Path    | Path of tailored resume                   |
| Source Portal  | LinkedIn / Naukri / Indeed                |
| Posted Time    | When the job was posted                   |
| Status         | Pending / Applied / Rejected / Interested |

### Sheet 2 — Resume Tracker

| Column         | Description  |
| -------------- | ------------ |
| Job Title      | Role         |
| Company        | Company      |
| Resume Version | File name    |
| Created Date   | Date created |

### Sheet 3 — Daily Stats

| Column        | Description                   |
| ------------- | ----------------------------- |
| Date          | Day                           |
| Total Fetched | Total jobs fetched            |
| After Filter  | Relevant jobs after filtering |
| Emails Sent   | Total emails sent             |
| Sheet Updated | Yes/No                        |

---

## 📧 Email Format

**3 emails daily:**

```text
Email 1 — 🔥 Top Matches (Match Score 8+)
├── ~10 jobs
└── Best tailored resumes ready

Email 2 — ✅ Good Matches (Match Score 6–8)
├── ~20–30 jobs
└── Resumes ready

Email 3 — 📋 Full Sheet Link
└── Remaining jobs
    + Google Sheet link
```

---

## ⚙️ Agent Flow

```text
6 PM → Scheduler triggers

Orchestrator Agent
│
├─► Job Search Agent
│     ├── Apify LinkedIn  (50 jobs, last 24hr)
│     ├── Apify Naukri    (50 jobs, last 24hr)
│     ├── JSearch API     (50 jobs, Indeed/Glassdoor)
│     ├── Adzuna API      (50 jobs, Global+India)
│     └── Remotive        (50+ jobs, Remote)
│         Total: 200–300 jobs fetched
│
├─► Filter Agent
│     ├── Stage 1: Keyword filter   → 200 to 100–150
│     └── Stage 2: LLM relevance    → 150 to 80–100
│
├─► Resume Agent
│     └── Create tailored resume for every job
│         Save to: resumes/generated/
│
├─► Sheets Agent
│     └── Update Google Sheets
│         (all 80–100 jobs data)
│
└─► Email Agent
      ├── Email 1: Top 10 (score 8+)
      ├── Email 2: Next 20–30 (score 6–8)
      └── Email 3: Sheet link + remaining jobs
```

---

## 🚀 Setup Guide

### Step 1 — Install Dependencies

```bash
pip install langchain langchain-community
pip install llama-cpp-python --extra-index-url https://abetlen.github.io/llama-cpp-python/whl/cu121
pip install apify-client gspread python-docx
pip install requests APScheduler python-dotenv
```

### Step 2 — Download the Model

```bash
# Download Qwen2.5 3B Q4_K_M from HuggingFace
# TheBloke/Qwen2.5-3B-Instruct-GGUF
```

### Step 3 — Setup API Keys

```bash
# Add these inside config/.env file:
APIFY_API_KEY=your_key
JSEARCH_API_KEY=your_key
ADZUNA_APP_ID=your_id
ADZUNA_APP_KEY=your_key
GMAIL_EMAIL=your@gmail.com
GMAIL_APP_PASSWORD=your_app_password
GOOGLE_SHEET_ID=your_sheet_id
```

### Step 4 — Setup Candidate Profile

```python
# config/candidate_profile.py
profile = {
    "name": "Your Name",
    "skills": ["Python", "Django", "REST API"],
    "experience": "2 years",
    "preferred_roles": ["Backend Developer", "Python Developer"],
    "location": "India",
    "remote_ok": True,
    "job_type": "Full-time"
}
```

### Step 5 — Run the Project

```bash
python main.py
```

---

## 📅 Development Roadmap

```text
Week 1:
  ✅ Project Structure
  ⬜ LangChain + Qwen2.5 Setup
  ⬜ Job Search Tools (Free APIs)
  ⬜ Apify Integration
  ⬜ Duplicate Filter (SQLite)

Week 2:
  ⬜ LLM Relevance Scoring
  ⬜ Resume Tailor Agent
  ⬜ Google Sheets Integration
  ⬜ Email Agent
  ⬜ Orchestrator + Scheduler

Week 3:
  ⬜ Referral Agent (Startup employees)
  ⬜ Testing + Bug fixes
  ⬜ Optimization (Speed + Cost)
```

---

## ⚠️ Important Notes

* Use Apify’s $5 free credits wisely — run only once daily
* Use Gmail App Password (enable 2FA first)
* Create a Google Sheets Service Account for access
* Qwen2.5 model will load fully on GPU (4GB VRAM required)
* SQLite duplicate tracking ensures the same job does not appear repeatedly

---

## 📝 License

Personal use only. Respect the Terms of Service of job portals.

