AI-Powered Medical Report Simplifier
An intelligent FastAPI backend service designed to process complex medical lab reports and generate patient-friendly explanations using a multi-stage data processing pipeline.

Overview
Medical reports are often filled with technical jargon, abbreviations, and data points that are difficult for patients to understand. This project provides a robust API that accepts either a typed text report or a scanned image, processes the unstructured data, and returns a structured, simplified summary powered by a Large Language Model (LLM).

The system is architected with a focus on accuracy and safety, featuring a fallback OCR mechanism and two distinct guardrails to prevent data errors and model hallucinations.

Features
Dual Input: Accepts both raw text and image uploads (.jpeg, .png).

Robust OCR Pipeline: Uses Tesseract as the primary OCR engine and EasyOCR as a fallback, with image preprocessing to enhance accuracy.

Intelligent Text Normalization: Employs a combination of regex, typo correction, and fuzzy string matching to extract and standardize medical test results from unstructured text.

LLM-Powered Summarization: Leverages a free, open-source LLM via the Hugging Face Inference API to generate simple, easy-to-understand explanations for abnormal results.

Safety First (Guardrails):

OCR Confidence Guardrail: Rejects images that cannot be read with sufficient confidence after multiple attempts.

Hallucination Guardrail: Ensures the LLM only discusses tests that were present in the original input, preventing the fabrication of data.

System Architecture: A Data Processing Pipeline
The application functions as a sequential data processing pipeline, where the output of one stage serves as the input for the next. This is analogous to an ETL (Extract, Transform, Load) process in data engineering.

+---------------------------+
|      INPUT (Image/Text)   |
+-------------+-------------+
              |
              | [If Image]
              v
+-------------+-------------+      +---------------------------+
|   1. OCR & Extraction     +----->|   OCR Guardrail (Check)   |
| (Tesseract -> EasyOCR)    |      +---------------------------+
+-------------+-------------+
              |
              | [Raw Text]
              v
+-------------+-------------+
| 2. Text Normalization     |
| (Regex, Fuzzy Match, etc.)|
+-------------+-------------+
              |
              | [Structured JSON]
              v
+-------------+-------------+      +---------------------------+
|    3. LLM Summarization   +----->| Hallucination Guardrail   |
|    (Hugging Face API)     |      |        (Check)            |
+-------------+-------------+      +---------------------------+
              |
              | [Final JSON with Summary]
              v
+-------------+-------------+
|       FINAL OUTPUT        |
+---------------------------+

Tech Stack
Backend: FastAPI, Uvicorn

Data Validation: Pydantic

OCR: Tesseract, EasyOCR, OpenCV

Text Processing: spaCy, scispaCy, RapidFuzz

LLM: Hugging Face Inference API (Mistral-7B Model)

Async & Concurrency: asyncio, ProcessPoolExecutor

Setup and Installation
Prerequisites
Python 3.11+

Tesseract OCR Engine.

Windows: Install from the UB Mannheim build. Ensure the installation path is added to your system's PATH or updated in the .env file.

macOS: brew install tesseract

Linux: sudo apt-get install tesseract-ocr

Installation Steps
Clone the repository:

git clone [https://github.com/your-username/medical-report-simplifier.git](https://github.com/your-username/medical-report-simplifier.git)
cd medical-report-simplifier

Create and activate a virtual environment:

python -m venv venv
# On Windows
.\venv\Scripts\activate
# On macOS/Linux
source venv/bin/activate

Install dependencies:

pip install -r requirements.txt

Configure Environment Variables:

Create a .env file by copying the .env.example file.

Add your Hugging Face API Token to the HUGGINGFACE_API_TOKEN field.

If Tesseract is not in your system's PATH, update the TESSERACT_PATH variable.

Run the application:

uvicorn app.main:app --port 5003

The API will be available at http://127.0.0.1:5003.

API Endpoints
The application exposes two main endpoints for processing medical reports.

1. Process from Text
Endpoint: POST /api/process-report/text

Description: Accepts a raw string of text from a medical report.

Request Body: multipart/form-data with a text field.

2. Process from Image
Endpoint: POST /api/process-report/image

Description: Accepts an image file (.jpeg, .png) of a medical report.

Request Body: multipart/form-data with a file field.

How to Use
You can interact with the API via the auto-generated documentation at http://127.0.0.1:5003/docs or by using a tool like curl.

Example: Processing Text with curl
curl -X POST "[http://127.0.0.1:5003/api/process-report/text](http://127.0.0.1:5003/api/process-report/text)" \
-H "Content-Type: multipart/form-data" \
-F "text=Patient results: hemglobin: 10.5 g/dl, WBC Count: 13000 /uL"

Example: Processing an Image with curl
curl -X POST "[http://127.0.0.1:5003/api/process-report/image](http://127.0.0.1:5003/api/process-report/image)" \
-H "Content-Type: multipart/form-data" \
-F "file=@/path/to/your/sample_report.jpg"
