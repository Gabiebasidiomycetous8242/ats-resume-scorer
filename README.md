# 🎯 ATS Resume Scorer

An AI-powered resume analyzer that scores resumes for ATS (Applicant Tracking System) compatibility, validates claimed skills against real project/experience evidence, and — optionally — compares a resume against a specific job description for a tailored match score.

![Python](https://img.shields.io/badge/Python-3.11+-blue?logo=python&logoColor=white)
![FastAPI](https://img.shields.io/badge/FastAPI-Backend-009688?logo=fastapi&logoColor=white)
![Streamlit](https://img.shields.io/badge/Streamlit-Frontend-FF4B4B?logo=streamlit&logoColor=white)
![Supabase](https://img.shields.io/badge/Supabase-Auth%20%26%20DB-3ECF8E?logo=supabase&logoColor=white)

---

## Overview

You upload a resume (PDF or DOCX), and the backend:

1. Extracts the raw text (`pdfplumber` / `PyPDF2` for PDFs, `python-docx` for Word docs)
2. Sends it to a Groq-hosted LLM (`llama-3.3-70b-versatile`) to parse it into structured fields — contact info, skills, experience, projects, education, action verbs, keywords
3. Cross-references claimed **skills** against your **projects/experience** to see which ones are actually backed by evidence
4. Scores the resume across five weighted categories to produce an overall **ATS Score out of 100**
5. If you also provide a job description, runs a separate comparison pass: keyword overlap, semantic similarity (via sentence-transformer embeddings), and a skills-gap analysis
6. Generates detailed, actionable feedback — and can export the whole thing as a combined PDF report

## ✨ Features

- 📄 **Resume parsing** — PDF and DOCX, up to 5 MB
- 🎯 **ATS Score (0–100)** — broken down into Formatting, Keywords, Content Quality, Skill Validation, and ATS Compatibility
- ✅ **Skill validation** — flags which skills in your Skills section are actually demonstrated in your Projects/Experience, and which are unsupported
- 📋 **Job description matching** — keyword match %, semantic similarity, matched/missing keywords, and a skills gap list
- 🔍 **Detailed, prioritized feedback** — each issue includes severity (High / Moderate / Low), why it matters, how to fix it, and concrete action items
- 📑 **PDF report export** — a combined, multi-page report (score summary, skill validation, JD match, recommendations)
- 🔐 **Authentication** — email/password and Google OAuth, via Supabase
- 🕘 **Analysis history** — past analyses are saved per user and can be revisited or deleted

## 🧱 Tech Stack

| Layer | Technology |
|---|---|
| Backend API | FastAPI + Uvicorn |
| Frontend | Streamlit |
| NLP | spaCy (`en_core_web_md` / `en_core_web_sm`) |
| Embeddings | `sentence-transformers` (`all-MiniLM-L6-v2`) |
| LLM parsing | Groq (`llama-3.3-70b-versatile`) |
| Auth & Database | Supabase (Postgres + Auth, incl. Google OAuth) |
| Fuzzy matching | RapidFuzz |
| PDF generation | WeasyPrint + Jinja2 templates |
| File parsing | pdfplumber, PyPDF2, python-docx, python-magic |

## 📁 Project Structure

```
AI_ATS_SCORER/
├── backend/
│   ├── main.py                 # FastAPI app + model-loading lifespan
│   ├── api/
│   │   ├── auth.py             # Supabase JWT verification
│   │   └── routes.py           # /api/v1/* endpoints
│   ├── core/
│   │   └── config.py           # env vars, score weights, model names
│   ├── database/
│   │   └── supabase_db.py      # analysis history persistence
│   ├── models/
│   │   └── schemas.py          # Pydantic request/response models
│   ├── services/
│   │   ├── resume_parser.py    # PDF/DOCX → raw text
│   │   ├── groq_parser.py      # raw text → structured JSON (LLM)
│   │   ├── jd_matcher.py       # resume vs job description comparison
│   │   ├── ats_scorer.py       # scoring logic
│   │   ├── feedback_engine.py  # issue detection + action items
│   │   ├── resume_analyzer.py  # orchestrates the full pipeline
│   │   ├── report_generator.py # renders HTML report templates
│   │   └── pdf_export.py       # HTML → combined PDF
│   ├── templates/               # Jinja2 templates for the PDF report
│   └── utils/
│       ├── file_utils.py       # logging, error types, default stubs
│       └── matching.py         # fuzzy skill/keyword matching
└── frontend/
    ├── streamlit_app.py         # entry point, auth UI, navigation
    ├── views/                   # landing, scorer, history, resources
    ├── components/              # dashboard sections (score, feedback, etc.)
    └── services/
        ├── api_client.py        # calls to the backend
        └── supabase_client.py   # sign-in/up, Google OAuth
```

## 🚀 Getting Started

### Prerequisites

- Python 3.11+
- A [Supabase](https://supabase.com) project (free tier is fine)
- A [Groq](https://console.groq.com) API key

### 1. Clone the repo

```bash
git clone https://github.com/siddemmohankrishna/ats-resume-scorer.git
cd ats-resume-scorer
```

### 2. Install dependencies

```bash
pip install fastapi "uvicorn[standard]" python-dotenv "pyjwt[crypto]" httpx python-multipart pydantic
pip install spacy sentence-transformers rapidfuzz groq jinja2 weasyprint
pip install pdfplumber PyPDF2 python-docx streamlit requests supabase
```

> **Windows only:** the plain `python-magic` package doesn't work without `libmagic`. Use `pip install python-magic-bin` instead. On macOS/Linux, `python-magic` works as-is (macOS may also need `brew install libmagic`).

### 3. Download the spaCy model

```bash
python -m spacy download en_core_web_md
```

### 4. Configure environment variables

Create a `.env` file in the project root:

```env
SUPABASE_URL=https://your-project.supabase.co
SUPABASE_KEY=your-supabase-key
SUPABASE_JWT_SECRET=your-supabase-jwt-secret
GROQ_API_KEY=your-groq-api-key
```

Get the Supabase values from **Project Settings → API** in your Supabase dashboard.

For the Streamlit frontend, also add (typically in `.streamlit/secrets.toml`):

```toml
SUPABASE_URL = "https://your-project.supabase.co"
SUPABASE_ANON_KEY = "your-anon-key"
```

### 5. Run the backend

```bash
uvicorn backend.main:app --reload --port 8000
```

Visit `http://127.0.0.1:8000/docs` for interactive API docs, or `http://127.0.0.1:8000/api/v1/health` for a quick status check.

### 6. Run the frontend

```bash
streamlit run frontend/streamlit_app.py
```

## 🔌 API Reference

All endpoints except `/api/v1/health` require a Supabase-issued `Authorization: Bearer <token>` header.

| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/api/v1/analyze-resume` | Upload a resume (+ optional JD) for analysis |
| `GET` | `/api/v1/health` | Health check — no auth required |
| `GET` | `/api/v1/history` | Get the signed-in user's past analyses |
| `DELETE` | `/api/v1/history/{id}` | Delete one history entry |
| `POST` | `/api/v1/generate-pdf` | Generate a PDF report from an analysis result |
| `GET` | `/api/v1/history/{id}/pdf` | Generate a PDF report for a saved history entry |

## 🧪 Scoring Methodology

The overall ATS Score (0–100) is a weighted sum of five components:

| Component | Weight | What it measures |
|---|---|---|
| Formatting | 20 | Structure, section headers, bullet points |
| Keywords & Skills | 25 | Keyword density and relevance |
| Content Quality | 25 | Action verbs, quantified achievements |
| Skill Validation | 15 | Skills backed by project/experience evidence |
| ATS Compatibility | 15 | Clean formatting, no parsing blockers |

When a job description is provided, the match percentage is `keyword_overlap × 0.6 + semantic_similarity × 0.4`.

## ☁️ Deployment Notes

The backend loads spaCy + PyTorch + `sentence-transformers` into memory at startup, which typically needs **700MB–1.5GB+ RAM**. If deploying to a memory-constrained host (e.g. Render's free/Starter tiers, both capped at 512MB):

- Set `SPACY_MODEL_PRIMARY=en_core_web_sm` as an environment variable (lighter than the default `en_core_web_md`), and update your build command to download that model instead.
- If it still runs out of memory, you'll need a plan with more RAM (e.g. Render's Standard tier, 2GB).
- Ensure `requirements.txt` pins the CPU build of PyTorch (`--extra-index-url https://download.pytorch.org/whl/cpu`) rather than the much larger CUDA build, which isn't needed since there's no GPU in these environments.

## 🛠️ Troubleshooting

| Symptom | Likely cause |
|---|---|
| `ImportError: cannot import name 'X' from 'module'` | A function/class is imported under a name that doesn't match its actual definition — check spelling and casing in both places |
| `ModuleNotFoundError: No module named 'backend'` | You're running `uvicorn` from inside the `backend/` folder — run it one level up, from the project root |
| `OSError: Can't find model 'en_core_web_md'` | The spaCy model wasn't downloaded in this environment — run `python -m spacy download en_core_web_md` |
| `python` not found on Windows, but `pip` works | Windows' App Execution Alias is intercepting `python` — use the `py` launcher, or disable the alias in Settings → Apps → Advanced app settings |
| Backend returns 401 on every request | Missing/invalid `Authorization: Bearer <token>` header, or `SUPABASE_JWT_SECRET`/`SUPABASE_URL` not set |
| `Out of memory` during deploy | See **Deployment Notes** above |

## 🗺️ Known Limitations

- Grammar/spelling checking and location-privacy detection are currently **stubbed** (`get_default_grammar_results` / `get_default_location_results`) rather than fully implemented — they always report zero issues.
- `recommendation_engine.py` is built but not yet wired into the main analysis pipeline.
- Job description upload only supports pasted text or `.txt` files — PDF/DOCX job descriptions aren't parsed yet.

## 🤝 Contributing

Issues and pull requests are welcome. If you're fixing a bug, a short description of the failure (error message + which endpoint/view triggered it) makes it much faster to review.

## 📄 License

No license has been specified yet for this repository. Add a `LICENSE` file (e.g. MIT) if you want to formalize how others can use this code.

---

## 👨‍💻 Author

**Siddem Mohan Krishna** — B.Sc. Data Science Student

Interested in **AI, Machine Learning, Data Science, Backend Engineering, and NLP**.

### 🔗 Connect With Me

- 💻 **GitHub:** [siddemmohankrishna](https://github.com/siddemmohankrishna)
- 💼 **LinkedIn:** [Siddem Mohan Krishna](https://www.linkedin.com/in/siddem-mohan-krishna-247984378/)
