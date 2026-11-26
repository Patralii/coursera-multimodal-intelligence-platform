# Coursera Multimodal Intelligence Platform

App Link- https://coursera-multimodal-intelligence-platform-kdjxjscmnmq3zxzadfjs.streamlit.app/

Online courses produce a lot of data: quiz answers, student comments, videos, PDFs and slides. The problem is that all of it sits in different places, so a teacher has no easy way to see where students are actually struggling.

Our project brings everything into one app. An educator can ask a question like "Why are students struggling with while loops?" and get an AI-written report backed by real evidence from the course. The educator can then approve, reject or ask for changes.

This is a college/learning project, so it is a prototype and not a production system.

---

## What the app can do

- Show which topics have the most wrong quiz answers
- Search quiz results, student comments, video transcripts, PDF notes and video frames together
- Generate a report with key findings, evidence and teaching suggestions (using Gemini)
- Check that every citation in the report points to real evidence
- Let the educator approve, reject or request changes, and save that decision

---

## Tech stack

| What | Tool |
| --- | --- |
| Language | Python |
| Data processing | Pandas, Pathlib, CSV files |
| App / frontend | Streamlit |
| Video frames | OpenCV |
| Speech to text | Faster-Whisper |
| AI model (text + vision) | Google Gemini |
| Vector database | ChromaDB |
| Embeddings | Sentence Transformers |
| PDF reading | PyMuPDF |
| PowerPoint reading | python-pptx |
| Saving educator reviews | SQLite |
| Editor | VS Code |

The app does not need a separate API server. Streamlit calls the Python functions directly.

---

## How it works

```text
Raw data (quizzes, comments, videos, PDFs, slides)
        |
        v
Processing
  - Quiz analytics
  - Student comments
  - Video transcription
  - Frame extraction + Gemini image analysis
  - PDF / PPTX text extraction
        |
        v
Embeddings stored in ChromaDB
        |
        v
Topic-aware evidence search
        |
        v
Gemini writes the report
        |
        v
Citation ID check
        |
        v
Streamlit dashboard
        |
        v
Educator review (saved in SQLite)
```

---

## Project structure

```text
coursera-intelligence-platform/
|
|-- ai/                  RAG pipeline, indexing, citation validator
|-- backend/
|   |-- review_store.py  saves educator reviews in SQLite
|
|-- frontend/
|   |-- app.py           Streamlit dashboard
|   |-- pages/
|       |-- 2_Unified_Educator_Assistant.py
|
|-- pipelines/
|   |-- inspect_data.py          look at the raw datasets
|   |-- check_relationships.py   check datasets link correctly
|   |-- analyze_learning.py      main quiz analytics
|   |-- transcribe_video.py      video to text
|   |-- extract_video_frames.py  pull frames from video
|   |-- ingest_documents.py      read PDFs and slides
|   |-- (other mapping and frame analysis scripts)
|
|-- tests/
|   |-- validate_quiz_analysis.py   checks the quiz numbers
|   |-- evaluate_rag.py             checks the RAG output
|   |-- (other test and audit scripts)
|
|-- data/
|   |-- raw/             input files
|   |-- processed/       generated output
|
|-- vector_database/     ChromaDB storage
|-- requirements.txt
|-- .env.example
|-- README.md
```

---

## Team and contributions

| Member | What they worked on |
| --- | --- |
| Patrali | Data Processing and Quiz Analytics: `inspect_data.py`, `check_relationships.py`, `analyze_learning.py`, `validate_quiz_analysis.py`. Also `backend/review_store.py` (SQLite review storage). |
| [Teammate name] | [their part] |
| [Teammate name] | [their part] |

### Quiz analytics module

This module turns the raw quiz and comment files into one table that shows which topics students find hardest.

1. `inspect_data.py` loads every raw CSV and prints its size, columns, missing values and a few sample rows.
2. `check_relationships.py` checks that the datasets connect properly, for example that every topic in the comments exists in the topic file.
3. `analyze_learning.py` groups quiz answers by course, video and topic, counts attempts and correct answers, and works out the incorrect-answer rate. It then adds comment counts and topic names and sorts everything from hardest to easiest. It stops with an error if there are duplicate topic IDs or a quiz topic with no match.
4. `validate_quiz_analysis.py` recalculates the quiz numbers on its own and compares them with the expected results, so we are not just assuming the output is right.

The final output is `data/processed/learning_difficulty_summary.csv`, and the rest of the system uses it as evidence.

---

## Example

Question: "Why are students struggling with Python while loops?"

The app will:

1. Get the quiz performance for while loops
2. Get student comments about while loops
3. Search the video transcript
4. Search the lesson PDF and slides
5. Get descriptions of the related video frames
6. Ask Gemini to write a report from that evidence
7. Check the citation IDs
8. Show the report to the educator for review

In our demo data, while loops had 120 attempts, 80 of them wrong, so the incorrect-answer rate is 66.67%. Type casting was next at 64.17%, then for loops at 55%.

These numbers come from demo data and are not real Coursera results.

---

## Setup

You need Python, pip and a Google Gemini API key.

Create a virtual environment (Windows PowerShell):

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
```

Install the packages:

```powershell
python -m pip install -r requirements.txt
```

If installation fails, try a different Python version, because some packages only have wheels for certain versions.

Now copy `.env.example` to a new file called `.env` and add your key:

```env
GEMINI_API_KEY=your_own_gemini_api_key
```

Never upload your real key to GitHub.

---

## Running the project

The data has to be processed before the app has anything to show. This is the order we use:

```powershell
# 1. Quiz analytics
python pipelines/inspect_data.py
python pipelines/check_relationships.py
python pipelines/analyze_learning.py
python tests/validate_quiz_analysis.py

# 2. Video transcript
python pipelines/transcribe_video.py
python pipelines/map_transcript_topics.py

# 3. Video frames
python pipelines/extract_video_frames.py
python pipelines/map_video_frames.py
python pipelines/analyze_video_frames.py

# 4. PDFs and slides
python pipelines/ingest_documents.py

# 5. Build the ChromaDB index
python ai/build_evidence_index.py
python ai/index_transcripts.py
python ai/index_documents.py
python ai/index_visual_analysis.py
```

Then start the app:

```powershell
python -m streamlit run frontend/app.py
```

Open http://localhost:8501 in your browser. The dashboard shows the course analytics, and the Unified Educator Assistant page generates AI reports and handles educator reviews.

The demo uses one sample Python tutorial video and made-up datasets. The video and the generated files are not included in the repository, so you need to add your own source files and run the scripts above to recreate the data.

---

## Testing

```powershell
python tests/evaluate_rag.py
```

This is a structural check on 5 topics (while loops, for loops, functions, conditionals and error handling). In our last run, all 5 passed and invalid citation detection also passed.

It checks things like topic consistency, duplicate evidence, missing sources and valid citation IDs. Passing does not prove that the AI's answers are factually correct.

---

## Limitations

- **Demo data.** Several datasets are made up for the demo, so results are not real student outcomes.
- **Approximate topics.** Video topic ranges are rough and can overlap. Frame descriptions may carry a wrong topic label.
- **Citation check is basic.** It checks that the IDs exist. It does not check that each claim is really supported by its source.
- **AI can be wrong.** Gemini can make mistakes or guess, which is why an educator reviews every report.
- **Not production ready.** There is no login, access control, monitoring or privacy setup yet. It only runs locally.

---

## Future improvements

- Check each claim against its source, not just the citation ID
- More accurate video topic splitting
- Real course data, with permission
- Support for more than one course
- Login and user roles
- Deployment and monitoring
- Automatic data ingestion

---

## Who is it for?

Teachers, instructors and course admins. The app helps them make decisions faster. It does not replace their judgment.
