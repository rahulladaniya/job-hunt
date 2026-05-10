# 🤖 Job Hunt Agent

Ek autonomous AI agent jo automatically job dhundhta hai, resume tailor karta hai, aur Google Sheet + Email ke zariye tumhe notify karta hai.

---

## 📌 Overview

Yeh agent daily shaam ko automatically:
1. Multiple job portals se 200-300 jobs fetch karta hai
2. Candidate ke skills ke basis pe relevant jobs filter karta hai
3. Har relevant job ke liye tailored resume banata hai
4. Sab kuch Google Sheet mein organize karta hai
5. Priority-wise summary email bhejta hai

---

## 🏗️ Project Structure

```
job-hunt-agent/
│
├── agents/                          # LangChain Agents
│   ├── orchestrator.py              # Main brain - sab agents coordinate karta hai
│   ├── job_search_agent.py          # Jobs fetch karta hai sab portals se
│   ├── filter_agent.py              # Relevant jobs filter karta hai
│   ├── resume_agent.py              # Resume tailor karta hai per job
│   ├── email_agent.py               # Email bhejta hai
│   └── referral_agent.py            # Startup employees dhundhta hai
│
├── tools/                           # LangChain Tools (har agent ke liye)
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
│   │   ├── read_resume_tool.py      # Resume padhta hai
│   │   ├── tailor_resume_tool.py    # LLM se job-specific resume banata hai
│   │   └── save_resume_tool.py      # PDF/DOCX save karta hai
│   │
│   ├── email/
│   │   └── gmail_tool.py            # Gmail SMTP se email bhejta hai
│   │
│   └── sheets/
│       └── gsheets_tool.py          # Google Sheets mein data daalta hai
│
├── database/
│   └── jobs.db                      # SQLite - duplicate tracking
│
├── resumes/
│   ├── templates/
│   │   └── base_resume.docx         # Base resume template
│   └── generated/                   # Tailored resumes yahan save honge
│       └── [company]_[role]_[date].pdf
│
├── config/
│   ├── settings.py                  # Sab configuration ek jagah
│   ├── candidate_profile.py         # Candidate ki info (skills, preferences)
│   └── .env                         # API keys (git mein nahi jayega)
│
├── email/
│   └── templates/
│       ├── top_matches.html         # Email 1 template (Top 10 jobs)
│       ├── good_matches.html        # Email 2 template (20-30 jobs)
│       └── all_jobs.html            # Email 3 template (remaining)
│
├── sheets/
│   └── sheet_manager.py             # Google Sheets management
│
├── logs/
│   └── agent.log                    # Daily logs
│
├── scheduler.py                     # APScheduler - daily shaam ko trigger
├── main.py                          # Entry point
├── requirements.txt                 # Dependencies
└── README.md                        # Yeh file
```

---

## 🛠️ Tech Stack

| Component | Technology | Cost |
|-----------|-----------|------|
| LLM | Qwen2.5 3B (llama.cpp) | Free (Local) |
| Agent Framework | LangChain | Free |
| LinkedIn + Naukri | Apify | $5/month |
| Indeed + Glassdoor | JSearch API (RapidAPI) | Free (200 calls/month) |
| Global Jobs | Adzuna API | Free (250 calls/month) |
| Remote Jobs | Remotive + Arbeitnow | Free (Unlimited) |
| Resume | python-docx + reportlab | Free |
| Email | Gmail SMTP | Free |
| Database | SQLite | Free |
| Tracking | Google Sheets API (gspread) | Free |
| Scheduler | APScheduler | Free |

**Total Cost: ~$5/month** (sirf Apify ke liye)

---

## 📊 Google Sheet Structure

### Sheet 1 — Daily Jobs
| Column | Description |
|--------|-------------|
| Date | Jis din job mili |
| Job Title | Role ka naam |
| Company | Company name |
| Location | City / Remote |
| Job URL | Direct link |
| Match Score | LLM ne diya (1-10) |
| Skills Match | Jo skills match hui |
| Missing Skills | Jo skills nahi hain |
| Resume Path | Tailored resume ka path |
| Source Portal | LinkedIn / Naukri / Indeed |
| Posted Time | Job kab post hui |
| Status | Pending / Applied / Rejected / Interested |

### Sheet 2 — Resume Tracker
| Column | Description |
|--------|-------------|
| Job Title | Role |
| Company | Company |
| Resume Version | File name |
| Created Date | Kab banaya |

### Sheet 3 — Daily Stats
| Column | Description |
|--------|-------------|
| Date | Din |
| Total Fetched | Kitni jobs fetch hui |
| After Filter | Relevant kitni |
| Emails Sent | Kitne emails gaye |
| Sheet Updated | Yes/No |

---

## 📧 Email Format

**3 emails daily:**

```
Email 1 — 🔥 Top Matches (Match Score 8+)
├── ~10 jobs
└── Best tailored resumes ready

Email 2 — ✅ Good Matches (Match Score 6-8)
├── ~20-30 jobs
└── Resumes ready

Email 3 — 📋 Full Sheet Link
└── Remaining sab jobs
    + Google Sheet link
```

---

## ⚙️ Agent Flow

```
Shaam 6 baje → Scheduler trigger kare

Orchestrator Agent
│
├─► Job Search Agent
│     ├── Apify LinkedIn  (50 jobs, last 24hr)
│     ├── Apify Naukri    (50 jobs, last 24hr)
│     ├── JSearch API     (50 jobs, Indeed/Glassdoor)
│     ├── Adzuna API      (50 jobs, Global+India)
│     └── Remotive        (50+ jobs, Remote)
│         Total: 200-300 jobs fetched
│
├─► Filter Agent
│     ├── Stage 1: Keyword filter   → 200 se 100-150
│     └── Stage 2: LLM relevance    → 150 se 80-100
│
├─► Resume Agent
│     └── Har job ke liye tailored resume banao
│         Save karo: resumes/generated/
│
├─► Sheets Agent
│     └── Google Sheet update karo
│         (sab 80-100 jobs ka data)
│
└─► Email Agent
      ├── Email 1: Top 10 (score 8+)
      ├── Email 2: Next 20-30 (score 6-8)
      └── Email 3: Sheet link + baaki sab
```

---

## 🚀 Setup Guide

### Step 1 — Dependencies Install karo
```bash
pip install langchain langchain-community
pip install llama-cpp-python --extra-index-url https://abetlen.github.io/llama-cpp-python/whl/cu121
pip install apify-client gspread python-docx
pip install requests APScheduler python-dotenv
```

### Step 2 — Model Download karo
```bash
# HuggingFace se Qwen2.5 3B Q4_K_M download karo
# TheBloke/Qwen2.5-3B-Instruct-GGUF
```

### Step 3 — API Keys Setup karo
```bash
# config/.env file mein daalo:
APIFY_API_KEY=your_key
JSEARCH_API_KEY=your_key
ADZUNA_APP_ID=your_id
ADZUNA_APP_KEY=your_key
GMAIL_EMAIL=your@gmail.com
GMAIL_APP_PASSWORD=your_app_password
GOOGLE_SHEET_ID=your_sheet_id
```

### Step 4 — Candidate Profile Setup karo
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

### Step 5 — Run karo
```bash
python main.py
```

---

## 📅 Development Roadmap

```
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

- Apify $5 free credits wisely use karo — din mein ek baar hi chalao
- Gmail ke liye App Password use karo (2FA enable karo pehle)
- Google Sheets Service Account banao access ke liye
- Qwen2.5 model poora GPU pe load hoga (4GB VRAM needed)
- SQLite duplicate tracking ensure karta hai same job baar baar na aaye

---

## 📝 License

Personal use only. Job portals ke Terms of Service respect karo.