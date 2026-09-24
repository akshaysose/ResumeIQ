# ResumeIQ

ResumeIQ is an AI-powered ATS resume analyzer that helps job seekers evaluate how well their resume matches a target job description. It combines resume parsing, NLP-based analysis, semantic match scoring, and actionable recommendations in a simple web app.

## Features

- Upload a resume in PDF, DOC, or DOCX format
- Paste a job description and compare it against the resume
- Generate an ATS compatibility score
- Detect missing keywords and skills
- Measure semantic similarity using embeddings
- Show category-wise feedback such as formatting, keywords, and content quality
- Save analysis history for logged-in users
- Export the final result as a PDF report

## Tech Stack

- Frontend: Streamlit
- Backend: FastAPI
- NLP: spaCy, Sentence Transformers
- AI suggestions: Groq API
- Authentication/DB: Supabase
- PDF generation: WeasyPrint, Jinja2

## Project Structure

```text
ResumeIQ/
├── backend/
│   ├── api/
│   ├── core/
│   ├── database/
│   ├── models/
│   ├── services/
│   └── utils/
├── frontend/
│   ├── assets/
│   ├── components/
│   ├── services/
│   ├── views/
│   └── streamlit_app.py
├── jupyter notebooks/
├── README.md
├── PROJECT.md
├── requirements.txt
├── .gitignore
└── .env
```

## How It Works

1. The user uploads a resume and enters a job description.
2. The backend parses the uploaded resume into raw text.
3. NLP models extract key skills, phrases, and content.
4. The resume and job description are compared using keyword and semantic similarity logic.
5. The ATS score is calculated using weighted categories such as formatting, keywords, content, skill validation, and ATS compatibility.
6. Feedback and recommendations are generated.
7. The result is displayed in the Streamlit UI and can be saved to history or exported as PDF.

## Setup

### 1. Clone the project

```bash
git clone https://github.com/akshaysose/ResumeIQ.git
cd ResumeIQ
```

### 2. Create a virtual environment

```bash
python -m venv venv
```

On Windows:

```bash
venv\Scripts\activate
```

On macOS/Linux:

```bash
source venv/bin/activate
```

### 3. Install dependencies

```bash
pip install -r requirements.txt
python -m spacy download en_core_web_md
```

### 4. Configure environment variables

Create a `.env` file in the project root and add values for:

```env
SUPABASE_URL=your_supabase_url
SUPABASE_KEY=your_service_role_key
SUPABASE_ANON_KEY=your_anon_key
SUPABASE_JWT_SECRET=your_jwt_secret
GROQ_API_KEY=your_groq_key
SENTENCE_TRANSFORMER_MODEL=all-MiniLM-L6-v2
```

### 5. Run the backend

```bash
uvicorn backend.main:app --reload --host 0.0.0.0 --port 8000
```

### 6. Run the frontend

```bash
streamlit run frontend/streamlit_app.py
```

## Notes

- Do not commit `.env` files or secret keys.
- The app may download the sentence-transformer model on first run.
- Groq is optional for basic scoring, but LLM-generated recommendations depend on it.

## License

This project is for educational and portfolio use.

Developed by: Akshay Sose
