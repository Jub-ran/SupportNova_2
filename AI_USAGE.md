# AI Usage & Compliance Documentation

## 1. Role of Artificial Intelligence in SupportNova
SupportNova utilizes Large Language Models (LLMs), specifically Google Gemini, strictly for **Natural Language Understanding (NLU)** and **Data Extraction**. 

The AI is responsible for:
- Reading unstructured text.
- Identifying the core intent (Issue Category).
- Gauging the sentiment of the customer.
- Drafting a polite, context-aware initial response.

## 2. Strict AI Governance (Zero Trust Architecture)
SupportNova operates on a **"Zero Trust AI"** model. We do not trust the AI to make final business decisions.
- **No Direct Database Writes:** The AI cannot directly modify database records. Its output is treated as a "Suggestion."
- **Ground-Truth Interception:** Every JSON output from the AI is intercepted by `src/python_validation/engine.py`. This Python script verifies the AI's suggestions against a strict SQLite database containing 100 Rules and 30 Escalation paths.
- **Human in the Loop (HITL):** No final resolution is sent to the customer without a human Agent clicking "Approve" on the Agent Dashboard.

## 3. AI Assisted Development
During the development of SupportNova, AI coding assistants were used to:
- Generate boilerplate FastAPI code.
- Write CSS for the Glassmorphism UI.
- Draft documentation templates.
All AI-generated code was manually reviewed, tested, and refactored by human developers to ensure security and performance.
