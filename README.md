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


