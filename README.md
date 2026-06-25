# 🏥 Clinical AI Hallucination Detection System

## 📖 Project Overview

AI models used in clinical settings (like radiology report generators) can **hallucinate** — they confidently output medical findings that are factually wrong or unsupported by the actual image/data. In healthcare, this is dangerous. 

This project builds a pipeline that **automatically catches those hallucinations** before they're trusted.

### What It Does
Given a medical image (chest X-ray) and patient record, the system:
1. **Retrieves** the most relevant clinical evidence from a vector database.
2. **Uses an LLM** to generate a diagnostic finding statement.
3. **Runs a fact-checking guardrail** to verify the statement is grounded in evidence.
4. **Evaluates** accuracy across 50+ records and exports a full audit trail.

---

## ⚙️ How It Works — The 5-Layer Pipeline

```text
Medical Image / Patient Record
         │
         ▼
┌─────────────────────────────────┐
│  LAYER 1: INGESTION             │
│  Load 50+ VQA-RAD records       │
│  ├─ Text  → MiniLM embeddings   │  ← "all-MiniLM-L6-v2"
│  └─ Image → CLIP  embeddings    │  ← "openai/clip-vit-base-patch32"
│     Both stored in ChromaDB     │
└────────────────┬────────────────┘
                 │
                 ▼
┌─────────────────────────────────┐
│  LAYER 2: RETRIEVAL             │
│  Find the most relevant record  │
│  via multimodal fusion score:   │
│  0.6 × text_sim + 0.4 × img_sim │
└────────────────┬────────────────┘
                 │
                 ▼
┌─────────────────────────────────┐
│  LAYER 3: GENERATION            │
│  Feed retrieved context into    │
│  Flan-T5-Base LLM               │
│  → Produces a clinical finding  │
│  grounded in real evidence      │
└────────────────┬────────────────┘
                 │
                 ▼
┌─────────────────────────────────┐
│  LAYER 4: NLI GUARDRAIL         │
│  (The Safety Check)             │
│  Compare Generation vs Evidence │
│  via Cross-Encoder NLI model    │
│  → Entailment Score (0 to 1)    │
└────────────────┬────────────────┘
                 │
                 ▼
┌─────────────────────────────────┐
│  LAYER 5: VERDICT               │
│  If NLI Score ≥ 0.80: "SAFE"    │
│  If NLI Score < 0.80: "FLAGGED" │
└─────────────────────────────────┘
