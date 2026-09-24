# AI Resume ATS Analyzer — Project Documentation

## 1. Overview

This project is an ATS (Applicant Tracking System) resume scoring application that helps users evaluate how well a resume matches a given job description. It combines:

- A FastAPI backend for analysis and API endpoints
- A Streamlit frontend for the user interface
- NLP and semantic matching using spaCy and sentence-transformers
- Resume parsing for PDF and DOCX files
- Optional LLM-based suggestions via Groq
- Supabase authentication and history storage
- PDF report generation

The main goal is to analyze a resume, compare it against a job description, and return a score along with actionable feedback such as missing skills, weak keyword coverage, ATS compatibility issues, and content improvements.

---

## 2. What the project does

### Core functionality

1. User uploads a resume file
   - Supported formats: PDF, DOC, DOCX
   - Files are validated and parsed into text

2. User enters or pastes a job description
   - The system extracts candidate skills and requirements from the JD

3. Resume is analyzed using NLP
   - Extracts structured information such as skills, experiences, and keywords
   - Compares job requirements with resume content

4. ATS score is calculated
   - The app computes a weighted score based on:
     - formatting
     - keywords
     - content
     - skill validation
     - ATS compatibility

5. Detailed feedback is generated
   - Missing keywords
   - Weak sections or content gaps
   - Skills that are not validated
   - Suggestions for improvement

6. Results can be saved to history
   - Authenticated users can view past analyses

7. Reports can be exported as PDF
   - HTML reports are generated and converted into a PDF document

---

## 3. High-level architecture

The project is split into two main applications:

### Backend
- Handles parsing, analysis, scoring, auth, database, and report generation
- Exposed via FastAPI
- Runs on localhost:8000

### Frontend
- User-facing dashboard and views
- Built using Streamlit
- Runs on localhost:8501

Communication flow:

User -> Streamlit UI -> API request -> FastAPI backend -> resume analysis logic -> score + feedback -> response -> UI display

---

## 4. Folder-by-folder explanation

## backend/

This folder contains the server-side logic.

### backend/main.py
Main FastAPI application entrypoint.

Responsibilities:
- Creates the FastAPI app
- Loads NLP model and embedding model at startup
- Configures CORS
- Includes API routes
- Defines the root API route and health route

Important logic:
- On app startup, `lifespan` runs
- It loads `en_core_web_md` via spaCy
- If that fails, it falls back to a secondary model
- It loads the sentence-transformer embedding model (`all-MiniLM-L6-v2`)
- These models are attached to `app.state.nlp` and `app.state.embedder`

This means the backend keeps expensive NLP models loaded once instead of reloading for each request.

### backend/api/

#### backend/api/auth.py
Responsible for authentication and authorization.

This file usually verifies incoming Supabase JWTs or access tokens and extracts the current authenticated user ID.

It is used by API routes to restrict access to authenticated users and enforce user-specific history endpoints.

#### backend/api/routes.py
Main API endpoints of the project.

Key endpoints:
- `POST /api/v1/analyze-resume`
  - Accepts uploaded resume file and job description
  - Parses the resume
  - Runs full analysis pipeline
  - Builds response payload
  - Saves analysis to history
- `GET /api/v1/health`
  - Returns app health and model status
- `GET /api/v1/history`
  - Fetches authenticated user’s saved analyses
- `DELETE /api/v1/history/{analysis_id}`
  - Deletes a saved analysis from the user history
- `POST /api/v1/generate-pdf`
  - Generates a PDF report from analysis result data

Core flow in `analyze_resume`:
1. Reads uploaded file bytes
2. Calls `parse_resume_file()`
3. Runs `analyze_full_resume()`
4. Converts result into schema-driven response
5. Saves analysis to Supabase database if possible
6. Returns final JSON object to frontend

### backend/core/

#### backend/core/config.py
This is the central configuration file.

Responsibilities:
- Loads environment variables from `.env`
- Defines app metadata
- Declares supported file types and extensions
- Stores model names
- Defines score weights
- Stores Supabase and Groq configuration values

Example constants:
- `APP_TITLE`, `APP_VERSION`, `APP_DESCRIPTION`
- `ALLOWED_ORIGINS`
- `MAX_FILE_SIZE_BYTES`
- `SUPPORTED_MIME_TYPES`, `SUPPORTED_EXTENSIONS`
- `SPACY_MODEL_PRIMARY`, `SPACY_MODEL_SECONDARY`
- `SENTENCE_TRANSFORMER_MODEL`
- `SCORE_WEIGHTS`
- `SUPABASE_URL`, `SUPABASE_ANON_KEY`, `GROQ_API_KEY`

This file acts like the project’s configuration hub.

### backend/database/

#### backend/database/supabase_db.py
Handles Supabase database interactions.

Responsibilities:
- Save analysis to user history
- Fetch user history
- Delete records
- Read and write structured analytics data to the DB

This enables the project to persist prior resume analyses for authenticated users.

### backend/models/

#### backend/models/schemas.py
Defines Pydantic data models used for validation and API responses.

Examples:
- `AnalysisResponse`
- `ComponentScores`
- `JDComparison`
- `SkillValidationDetails`

This ensures the backend returns a consistent response format. It also helps FastAPI validate request/response data automatically.

### backend/services/

This is the core core of the project logic. Each service handles a specific part of the analysis pipeline.

#### backend/services/resume_parser.py
Responsible for reading resume files and extracting text.

Responsibilities:
- Validates file type and size
- Reads PDF, DOC, and DOCX files
- Extracts text from the document
- Normalizes content for downstream analysis

The parse step is essential because everything after it depends on clean, extractable text.

#### backend/services/resume_analyzer.py
This is the most important business logic file.

It likely orchestrates the full analysis pipeline, including:
- text preprocessing
- keyword extraction
- semantic similarity checks
- skill validation
- ATS score aggregation
- summary/feedback generation

It produces a dictionary of outputs like:
- `ats_score`
- `component_scores`
- `issues_summary`
- `detailed_feedback`
- `jd_comparison`
- `skill_validation_details`
- `missing_keywords`
- `matched_keywords`
- `skills`
- `interpretation`

This is effectively the “brain” of the application.

#### backend/services/jd_matcher.py
Compares resume content with job description content.

Likely logic:
- Extract keywords from the JD
- Compare them with resume text
- Determine matched vs missing keywords
- Compute similarity scores
- Identify skill gaps

#### backend/services/ats_scorer.py
Calculates the final ATS score and component-level evaluation.

This file likely combines weighted scores from different categories to compute a final percentage.

#### backend/services/feedback_engine.py
Generates human-readable or structured feedback.

This may convert raw matching results into actionable suggestions such as:
- “Add more Python experience”
- “Include SQL in skills section”
- “Improve keyword density for the target role”

#### backend/services/groq_parser.py
Interfaces with Groq LLM APIs.

This may produce natural-language recommendations using LLM-generated output so the feedback feels richer and less rule-based.

#### backend/services/recommendation_engine.py
Creates suggestions based on the analyzed gap between a resume and JD.

It probably prioritizes recommendations:
- must-have skills to add
- experience to emphasize
- keywords to include
- structure improvements

#### backend/services/report_generator.py
Prepares HTML content for report export.

This is used before PDF generation and likely formats the analysis results into visual sections.

#### backend/services/pdf_export.py
Converts the generated report HTML into a PDF file.

This creates downloadable PDF reports for users.

### backend/utils/

#### backend/utils/file_utils.py
Helper functions for file content processing and default fallback results.

This file may provide:
- default grammar or grammar check style outputs
- default skill validation placeholders
- location and validation fallback data

It is used as a utility layer for common output structures.

#### backend/utils/matching.py
May contain lower-level string matching and similarity helpers.

This likely supports:
- fuzzy keyword matching
- normalized token comparison
- similarity operations used by the ATS matching logic

---

## 5. Frontend folder

## frontend/
This contains the user interface built with Streamlit.

### frontend/streamlit_app.py
Main frontend entry file.

Responsibilities:
- Configure Streamlit app settings
- Manage session state
- Render sidebar navigation
- Handle user sign-in / sign-up / OAuth login
- Navigate between views
- Load page-specific rendering components

This file contains the top-level view routing logic:
- landing
- scorer
- history
- resources

### frontend/views/
Pages or modules rendered inside the Streamlit app.

#### frontend/views/landing.py
Landing page for the app.

Likely shows introduction, branding, and onboarding prompt.

#### frontend/views/scorer.py
Main scoring view.

This is likely where users:
- upload resume
- paste JD
- click “Analyze”
- view ATS score and feedback

#### frontend/views/history.py
Displays saved past analyses for the logged-in user.

#### frontend/views/resources.py
Displays learning or support resources related to job search and resume improvement.

### frontend/components/
Reusable visual UI building blocks, organized by feature.

Examples:
- `score_display.py` — displays ATS score cards
- `jd_comparison.py` — match percentage and keyword comparison UI
- `recommendations.py` — recommendations panel
- `detailed_feedback.py` — detailed list of feedback
- `skill_validation.py` — skill validation results
- `strengths_issues.py` — strengths and issues summary
- `action_items.py` — actionable tasks list
- `dashboard.py` — dashboard-style summary
- `_helpers.py` — shared helper logic for UI

These components keep the app modular and easier to maintain.

### frontend/services/
Service layer for frontend-side API and auth calls.

#### frontend/services/api_client.py
Used to call the backend FastAPI endpoints from the Streamlit app.

This likely handles:
- resume upload
- JD submission
- fetching history
- PDF generation
- request/response parsing

#### frontend/services/supabase_client.py
Handles frontend authentication with Supabase.

Responsibilities:
- sign in with email/password
- sign up
- Google OAuth URL generation
- OAuth code exchange
- sign out

This file provides login flows for the app.

---

## 6. Core logic flow of the project

The project follows this logic:

### Step 1: User interacts with frontend
- User logs in (optional, if enabled)
- User chooses the ATS scoring page
- User uploads resume and enters job description

### Step 2: Frontend sends API request
The Streamlit app calls the backend endpoint, usually `POST /api/v1/analyze-resume`.

### Step 3: Backend validates file
The API route reads the uploaded file bytes and validates:
- file is present
- size is acceptable
- extension or MIME type is supported

### Step 4: Resume parsing
The uploaded file is sent to `parse_resume_file()`.

This function:
- reads the file contents
- extracts plain text
- removes noise/formatting artifacts
- produces a normalized text corpus for analysis

### Step 5: Analysis pipeline
`analyze_full_resume()` is called with:
- resume text
- spaCy NLP model
- sentence-transformers embedder
- job description

This step usually does several things:
- extracts skills
- identifies nouns/keywords
- compares the job description and resume content
- evaluates semantic similarity using embeddings
- measures keyword overlap
- checks missing skills and keywords
- computes component-level scores

### Step 6: Score aggregation
The project calculates a final ATS score using predefined weights.

In `backend/core/config.py`:

```python
SCORE_WEIGHTS = {
    "formatting": 20,
    "keywords": 25,
    "content": 25,
    "skill_validation": 15,
    "ats_compatibility": 15,
}
```

This means keyword and content matching have the largest impact, followed by formatting and compatibility.

### Step 7: Feedback generation
The system turns raw analysis outputs into actionable suggestions.

Examples:
- missing keywords
- weak skill coverage
- low semantic alignment with JD
- content not specific enough
- ATS compatibility concerns

### Step 8: Result formatting
The backend serializes the final data into `AnalysisResponse` using Pydantic models, which standardizes output to the frontend.

### Step 9: History saving
If the user is authenticated, the analysis is saved to Supabase so it can be shown later in the History screen.

### Step 10: PDF export (optional)
The report can be turned into a PDF using HTML templates and a PDF engine.

---

## 7. How matching works

The project combines multiple methods to judge resume-to-JD fit:

### Keyword match
- Job description keywords are extracted
- Resume text is checked for overlap
- Matched and missing keywords are identified

### Semantic similarity
- Both the resume and job description are converted to embeddings
- Similarity is measured between the two vector representations
- This captures meaning beyond exact word matches

### Skill validation
- Resume skills are compared with required skills
- Missing skills are flagged
- Skills may be validated by detecting them in text

### ATS compatibility
- The app may check for formatting issues or things ATS systems struggle with
- Such as dense text blocks, missing sections, poor structure, or weak keyword placement

This combination makes the scoring more realistic than just counting simple keyword hits.

---

## 8. Why the project matters

This project is useful because it helps candidates:
- understand how ATS systems evaluate resumes
- identify missing job-specific keywords
- improve relevance to the target position
- get structured recommendations before applying

It is essentially a smart resume optimizer and recruiter-facing ATS fit tool.

---

## 9. Environment and runtime setup

The project expects:
- Python environment
- dependencies from `requirements.txt`
- spaCy model: `en_core_web_md`
- sentence-transformer model: `all-MiniLM-L6-v2`
- environment variables for Supabase/Groq

### Typical run commands

Backend:

```bash
uvicorn backend.main:app --reload --host 0.0.0.0 --port 8000
```

Frontend:

```bash
streamlit run frontend/streamlit_app.py
```

---

## 10. Important files to remember

- [backend/main.py](backend/main.py)
- [backend/api/routes.py](backend/api/routes.py)
- [backend/core/config.py](backend/core/config.py)
- [backend/services/resume_analyzer.py](backend/services/resume_analyzer.py)
- [backend/services/resume_parser.py](backend/services/resume_parser.py)
- [backend/services/ats_scorer.py](backend/services/ats_scorer.py)
- [backend/services/jd_matcher.py](backend/services/jd_matcher.py)
- [backend/services/feedback_engine.py](backend/services/feedback_engine.py)
- [backend/database/supabase_db.py](backend/database/supabase_db.py)
- [frontend/streamlit_app.py](frontend/streamlit_app.py)
- [frontend/services/api_client.py](frontend/services/api_client.py)
- [frontend/services/supabase_client.py](frontend/services/supabase_client.py)
- [requirements.txt](requirements.txt)

---

## 11. Summary

This project is a full-stack AI-powered ATS resume evaluation system. It combines parsing, semantic matching, keyword comparison, recommendation generation, and report export in one app. The backend does the heavy NLP work while the frontend provides a simple interface for end users.

In short:
- upload a resume
- paste a job description
- compare the two using NLP and scoring logic
- get a job-fit score and actionable feedback
- save the history and optionally export a PDF report

That is the complete working model of the project.
