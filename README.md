
# Document RAG Chatbot (n8n + Gemini)

A chatbot that answers questions **only from documents you upload** (PDF, DOCX, TXT).
It is built with **n8n's built-in AI nodes**, with no custom code and no separate frontend.

- 🎥 Demo video: ADD LINK HERE
- 💼 LinkedIn post: ADD LINK HERE

## Features
- Upload PDF, DOCX or TXT files through a web form
- Chat window to ask questions about the documents
- Answers come only from the uploaded documents
- Shows the source (file name and page) with each answer
- Says "I couldn't find this information in the uploaded documents" when the answer is missing

## Tech Stack
- **n8n** (workflow automation, run with npx)
- **Google Gemini** (free API key): embeddings and answers
- **n8n Simple Vector Store** (in-memory vector database)

## How It Works

**Part 1: Upload**
```
Upload Form → Read file → Cut into chunks → Gemini embeddings → Vector Store
```

**Part 2: Chat**
```
Question → AI Agent → Search the Vector Store → Gemini answers from the found chunks → Answer + Sources
```

1. **Upload Document Form** takes a PDF, DOCX or TXT file.
2. **Read and Split File** reads the text and keeps the file name and page number.
3. **Cut Into Chunks** splits the text into pieces of 800 characters (150 overlap).
4. **Embeddings** turn each chunk into numbers that represent its meaning.
5. **Simple Vector Store** saves the chunks and numbers.
6. When a user asks a question, the **AI Agent** searches the store with the `documents` tool.
7. **Gemini** writes the answer using only the chunks it found, then adds a Sources line.

## Hallucination Guardrails
- The agent must search the documents first for every question.
- It can answer only from the search results and must never guess prices, dates, names or numbers.
- If the answer is not found, it replies: *"I couldn't find this information in the uploaded documents."*
- Every real answer ends with a Sources line (file name and page).

## Installation

1. Start n8n:
```bash
   npx n8n
```
   Then open http://localhost:5678
2. Get a free API key from https://aistudio.google.com/apikey
3. In n8n, go to **Workflows → Import from file** and choose `simple_rag_workflow.json`.
4. Go to **Credentials → Create → Google Gemini(PaLM) Api**, paste your key and save.
5. Select this credential in 3 nodes: **Embeddings (Save)**, **Embeddings (Search)** and **Gemini Chat Model**.
6. Click **Publish**.
7. Open the Production URL of **Upload Document Form**, upload a file, then open the Production URL of **Chat** and ask questions.

> Keep the terminal running. If n8n restarts, the stored documents are lost and you need to upload them again.

## Environment Variables / Secrets
The Gemini API key is stored in **n8n Credentials**, not in the workflow file. No secrets are committed to this repo.

## Test Questions

| # | Type | Question | Result |
|---|------|----------|--------|
| 1 | Direct answer | What services does the company offer? | |
| 2 | Direct answer | What is the price of a listed property? | |
| 3 | Direct answer | Where is the head office? | |
| 4 | Same document | What are the commission fees and payment terms? | |
| 5 | Multi-document | Which properties are available and what is the fee to buy one? | |
| 6 | Multi-document | What is the refund policy for the listed services? | |
| 7 | No answer | Do you sell properties in Paris? | |
| 8 | No answer | Who is the CEO's spouse? | |
| 9 | Hallucination test | What discount do you give on a property? (no discount exists) | |
| 10 | Hallucination test | What is the current price of Bitcoin? | |

## Limitations
- No admin dashboard (no document list, delete or statistics)
- Documents are stored in memory and are lost when n8n restarts
- Upload one file at a time
- Page numbers work for PDF files only
- The Sources line is written by the AI, so check it during testing
- Scanned (image-only) PDFs are not supported

## Files in this repo
- `simple_rag_workflow.json`: the n8n workflow (import it into n8n)
- `README.md`: this file
