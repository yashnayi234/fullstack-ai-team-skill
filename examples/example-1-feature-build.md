# Example 1 — Feature build: "Build a tool that summarizes research papers"

A typical end-to-end product request. The skill triages, scopes, picks a stack, and ships starter code.

---

## Prompt

> I want to build a small web tool where I can paste a PDF of a research paper and get back a structured summary — TL;DR, methods, findings, and limitations. Help me build it.

## Response

**[PM] — Scope & requirements**

- **Core flow**: Upload PDF or paste URL → structured summary in <30s.
- **Output schema**: `tldr`, `methods`, `key_findings[]`, `limitations[]`.
- **Success metric for MVP**: a domain expert can skip the abstract after reading our summary 80% of the time on a 10-paper test set.
- **Out of scope for v1**: accounts, history, multi-paper synthesis, citation graphs.

**[SD] — Architecture**

Stack: Next.js (App Router) frontend, FastAPI backend, Anthropic API for summarization. Stateless for MVP — add Postgres in v2 once we know what to persist.

```
[Browser] ──upload PDF──▶ [Next.js API route] ──▶ [FastAPI /summarize]
                                                        │
                                                        ▼
                                            [pypdf extract text]
                                                        │
                                                        ▼
                                            [Claude Sonnet, JSON mode]
                                                        │
                                                        ▼
                                            [Structured summary]
```

Why this stack: Next.js for fast deploy on Vercel, FastAPI because the heavy lifting (PDF parse + LLM) is Python-native, Claude Sonnet for the cost/quality balance on long-context summarization.

**[AIE] — Prompt + orchestration**

```python
# summarizer.py
import anthropic
import json

client = anthropic.Anthropic()

SUMMARIZE_PROMPT = """You are summarizing an academic paper for a researcher \
who wants to decide whether to read it in full.

Return ONLY valid JSON matching this schema:
{
  "tldr": "1-2 sentences, plain English",
  "methods": "2-3 sentences on what they actually did",
  "key_findings": ["finding 1", "finding 2", "finding 3"],
  "limitations": ["limitation 1", "limitation 2"]
}

Paper text:
<paper>
{paper_text}
</paper>

Return only the JSON object. No preamble."""

def summarize(paper_text: str) -> dict:
    msg = client.messages.create(
        model="claude-sonnet-4-5",
        max_tokens=1024,
        messages=[{
            "role": "user",
            "content": SUMMARIZE_PROMPT.format(paper_text=paper_text[:80_000])
        }]
    )
    return json.loads(msg.content[0].text)
```

**[BE] — API**

```python
# main.py
from fastapi import FastAPI, UploadFile, HTTPException
from fastapi.middleware.cors import CORSMiddleware
import pypdf
import io
from summarizer import summarize

app = FastAPI()
app.add_middleware(CORSMiddleware, allow_origins=["*"], allow_methods=["*"])

@app.post("/summarize")
async def summarize_paper(file: UploadFile):
    if file.content_type != "application/pdf":
        raise HTTPException(400, "PDF required")
    reader = pypdf.PdfReader(io.BytesIO(await file.read()))
    text = "\n".join(p.extract_text() or "" for p in reader.pages)
    if len(text) < 500:
        raise HTTPException(422, "Could not extract text from PDF")
    return summarize(text)
```

**[FE] — UI**

```tsx
// app/page.tsx
"use client";
import { useState } from "react";

export default function Home() {
  const [summary, setSummary] = useState<any>(null);
  const [loading, setLoading] = useState(false);

  async function onUpload(e: React.ChangeEvent<HTMLInputElement>) {
    const file = e.target.files?.[0];
    if (!file) return;
    setLoading(true);
    const fd = new FormData();
    fd.append("file", file);
    const res = await fetch("/api/summarize", { method: "POST", body: fd });
    setSummary(await res.json());
    setLoading(false);
  }

  return (
    <main className="max-w-2xl mx-auto p-8">
      <h1 className="text-2xl font-semibold mb-6">Paper Summarizer</h1>
      <input type="file" accept="application/pdf" onChange={onUpload} />
      {loading && <p className="mt-4">Summarizing…</p>}
      {summary && (
        <div className="mt-6 space-y-4">
          <section><h2 className="font-semibold">TL;DR</h2><p>{summary.tldr}</p></section>
          <section><h2 className="font-semibold">Methods</h2><p>{summary.methods}</p></section>
          <section>
            <h2 className="font-semibold">Key findings</h2>
            <ul className="list-disc pl-5">{summary.key_findings.map((f: string, i: number) => <li key={i}>{f}</li>)}</ul>
          </section>
          <section>
            <h2 className="font-semibold">Limitations</h2>
            <ul className="list-disc pl-5">{summary.limitations.map((l: string, i: number) => <li key={i}>{l}</li>)}</ul>
          </section>
        </div>
      )}
    </main>
  );
}
```

**[SD] — Risks**

- **PDF parse quality** is the silent killer. `pypdf` works for most papers but mangles two-column layouts. If users complain, swap in a layout-aware parser (`unstructured` or `pdfplumber`).
- **80k char truncation** loses content for 30+ page papers. v2 should chunk + summarize hierarchically.
- **No eval harness yet.** Before scaling, build a 20-paper test set with expert-written reference summaries and measure agreement.

---

## What the skill did well here

- PM scoped before anyone coded — caught "no accounts in v1" before the BE wasted time on auth.
- SD justified the stack in one line each, didn't dither.
- Risks were called out separately, not buried in the code comments.
- FE built directly to BE's response shape — no contract drift.
