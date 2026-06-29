# AI MCQ & Study Hub Backend

This repository contains the Python backend for the AI-powered MCQ Generator mobile app. It handles file parsing (PDF/Images), text extraction via OCR, and calls the AI model to generate structured multiple-choice questions.

## Features
- **PDF Processor:** Extracts text from textbooks and documents.
- **Image Scanner:** Utilizes Tesseract OCR to read text from uploaded photos.
- **AI MCQ Engine:** Formats text into strict JSON-structured multiple-choice quizzes.
- **Study Mode Ready:** Supports countdown configs and export functionalities.

## Setup Instructions

1. **Install System Dependencies** (Required for OCR):
   - **Ubuntu/Debian:** `sudo apt-get install tesseract-ocr`
   - **macOS:** `brew install tesseract`
   - **Windows:** Download the installer from GitHub Tesseract documentation.

2. **Install Python Packages:**
   ```bash
GEMINI_API_KEY=your_actual_api_key_here   pip install -r requirements.txt
uvicorn app:app --reload
---

## 🐍 `app.py`
This is the core application script. It uses **FastAPI** to handle requests from your Flutter mobile app and coordinates the text extraction and AI quiz generation.

```python
import os
import json
from fastapi import FastAPI, UploadFile, File, Form, HTTPException
from fastapi.responses import JSONResponse
import pdfplumber
from PIL import Image
import pytesseract
import google.generativeai as genai
from dotenv import load_dotenv

# Load API credentials
load_dotenv()
API_KEY = os.getenv("GEMINI_API_KEY")

if not API_KEY:
    raise ValueError("GEMINI_API_KEY is missing from environment variables.")

genai.configure(api_key=API_KEY)

app = FastAPI(title="AI MCQ Generator API")

def clean_and_parse_json(text: str):
    """Safely extracts and parses JSON content from the AI response."""
    try:
        # Strip potential markdown formatting if returned by the model
        if text.startswith("```json"):
            text = text.split("```json")[1].split("```")[0]
        elif text.startswith("```"):
            text = text.split("```")[1].split("```")[0]
        return json.loads(text.strip())
    except Exception as e:
        raise HTTPException(status_code=500, detail=f"Failed to parse AI response into JSON: {str(e)}")

@app.post("/generate-mcq")
async def generate_mcq(
    file: UploadFile = File(...),
    num_questions: int = Form(10)
):
    extracted_text = ""
    filename = file.filename.lower()

    try:
        # 1. Handle PDF Processing
        if filename.endswith(".pdf"):
            with pdfplumber.open(file.file) as pdf:
                # Extract text from up to the first 5 pages to avoid token overloads
                pages = pdf.pages[:5]
                extracted_text = "".join([page.extract_text() or "" for page in pages])
        
        # 2. Handle Image Processing (Scanner Feature)
        elif filename.endswith((".png", ".jpg", ".jpeg")):
            image = Image.open(file.file)
            extracted_text = pytesseract.image_to_string(image)
        
        else:
            raise HTTPException(status_code=400, detail="Unsupported file format. Please upload a PDF or an Image.")

        if not extracted_text.strip():
            raise HTTPException(status_code=400, detail="Could not extract any clear text from the uploaded file.")

        # 3. AI Engineering System Prompt
        model = genai.GenerativeModel('gemini-1.5-flash')
        prompt = f"""
        You are an expert academic evaluator. Based strictly on the following text context, generate exactly {num_questions} high-quality Multiple Choice Questions (MCQs).
        
        Context text:
        {extracted_text[:4000]}

        You must output the result strictly as a valid JSON array matching this schema:
        [
          {{
            "question": "The question text here?",
            "options": ["Option A", "Option B", "Option C", "Option D"],
            "answer": "The exact string match of the correct option",
            "explanation": "Brief context on why this answer is correct."
          }}
        ]
        Do not include any intro text, conversational filler, or extra formatting characters outside the valid JSON array.
        """

        response = model.generate_content(prompt)
        quiz_data = clean_and_parse_json(response.text)
        
        return JSONResponse(content={"status": "success", "quiz": quiz_data})

    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)
