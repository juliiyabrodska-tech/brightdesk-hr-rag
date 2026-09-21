# Brightdesk HR Knowledge Assistant (RAG)

A retrieval-augmented chatbot that lets employees ask HR questions in plain language and get an accurate answer pulled straight from the company's own policy documents, with the source cited — instead of digging through PDFs.

> **Note:** "Brightdesk" is a fictional company built for this demo. The HR policy documents in `docs/` are synthetic, written specifically to showcase the retrieval pipeline — not real client data.

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/juliiyabrodska-tech/brightdesk-hr-rag/blob/main/Brightdesk_HR_RAG_demo.ipynb)

Click the badge above to open and run the notebook yourself (you'll need your own free Gemini API key — see **Running it yourself** below). The notebook is committed with real output from an actual run, so you can also just read it here on GitHub without running anything.

## How it works

```
HR policy docs (docs/*.md)
        |
        v
sentence-aware chunking, with overlap (Cyrillic + Latin safe)
        |
        v
each chunk embedded with Gemini (gemini-embedding-001)
        |
        v
employee question -> expanded into 2-3 alternate phrasings (query expansion)
        |
        v
most relevant chunks retrieved by cosine similarity
        |
        v
Gemini generates an answer restricted to the retrieved context only
        |
        v
answer + source document name returned, through a Gradio chat UI
```

## Features

- **Multilingual retrieval** — tested in English and Ukrainian
- **Query expansion** — differently-worded questions ("How many vacation days do I get?" vs "What's the PTO policy?") still find the right document
- **Grounded answers only** — every answer is generated from retrieved text, with the source document cited, not a free-floating model guess
- **Chunking with overlap** — answers aren't cut off mid-context
- **Retry on transient errors** — Gemini API calls retry automatically on rate limits / temporary overload (503/429) instead of failing silently
- **Optional Slack demo** — a one-off Socket Mode bot (see the last section of the notebook) that answers questions in a single designated Slack channel, for demonstration purposes

## Status

The pipeline logic (chunking, embedding, retrieval, query expansion, grounded generation) has been exercised during development against all four policy docs, in English and Ukrainian, including a negative case with no matching policy. This repo's notebook does not yet have a saved, clean end-to-end run committed with it — run it yourself via the Colab badge above, or check back here once a full run has been captured and committed.

## Running it yourself

1. Open the notebook (badge above, or `Brightdesk_HR_RAG_demo.ipynb` directly)
2. Get a free Gemini API key from [Google AI Studio](https://aistudio.google.com/app/apikey)
3. In Colab: Secrets (key icon in the sidebar) -> add `GOOGLE_API_KEY` with that value, enable notebook access
4. Runtime -> Run all

## Tech

Google Gemini (embeddings + generation) - Python - pandas / numpy - Gradio - (optional) Slack Bolt for the Slack demo

## Project structure

```
.
|-- Brightdesk_HR_RAG_demo.ipynb   # main notebook, runs end-to-end
|-- docs/                          # synthetic HR policy documents (source data)
|-- requirements.txt
|-- LICENSE
```

## Limitations (by design, for a demo of this scope)

- Retrieval is a custom in-memory cosine-similarity search (numpy/pandas), not a dedicated vector database — fine at this scale, would swap in something like Chroma or pgvector for a larger document set
- No fine-tuning — this is prompt-grounded RAG
- Runs in Colab, not deployed as a persistent service
