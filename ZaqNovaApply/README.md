# ZaqNova Apply

A secure, responsive, **single-file** web application for tailoring factually-accurate, ATS-ready resumes — one job at a time or in bulk. Part of the ZaqNova product line, built in the same client-side idiom as ZaqNova Interview CoPilot (React + Tailwind + Babel via CDN, `pdf.js`/`mammoth` parsing, multi-provider AI with your own keys).

> **Architecture note:** This is a client-side application. All data lives in your browser's `localStorage`; nothing is uploaded to any ZaqNova server. Resume text is sent **only** to the AI provider you select, only at generation time. See [ASSUMPTIONS.md](ASSUMPTIONS.md) for why this target was chosen over a full backend SaaS.

## What it does

- **Resume profiles** — upload PDF/DOCX/TXT (≤5 MB), validate by extension + MIME + magic-byte signature + size, parse into structured sections, review and correct extraction, manage multiple profiles (duplicate/archive/delete).
- **Job targets** — add one job manually (with optional AI helpers for title/description/skills) or import many from CSV/Excel with per-row validation, deduplication, and a downloadable template.
- **Tailoring** — Conservative/Balanced/Aggressive modes, language, years, seniority, tone, page length, keep-original and shorten toggles. A factually-constrained pipeline that never fabricates experience.
- **Workspace** — preview, structured editor, original-vs-tailored comparison, estimated match score with keyword coverage, qualification gaps, factual/ATS/injection warnings, section regeneration, version history + restore.
- **Bulk generator** — asynchronous queue with bounded concurrency, per-row status, bounded retries with backoff, ZIP of all DOCX, and a CSV run report.
- **Exports** — DOCX, PDF, TXT, ATS-plain-text. Filenames follow `FirstName_LastName_TargetRole_Company_TailoredResume.ext`.
- **Privacy** — user-controlled deletion of tailored resumes or all data.

## Run it

It's a static site. Any static file server works.

```bash
# from this folder
python -m http.server 4178
# open http://localhost:4178
```

Or simply open `index.html` in a modern browser (some browsers restrict `file://` for module/worker loads; the static server is recommended).

## Configure

1. Open **Settings**.
2. Choose an AI model (Anthropic / OpenAI / Google).
3. Paste the matching API key. Keys are stored only in this browser (`localStorage`) and sent only to that provider. Get keys:
   - Anthropic: https://console.anthropic.com/settings/keys
   - OpenAI: https://platform.openai.com/api-keys
   - Google: https://aistudio.google.com/app/apikey

There are **no secrets in this repository.** Keys are user-supplied at runtime.

## Tech

| Concern | Choice |
|---|---|
| UI | React 18 + Tailwind (CDN), Babel standalone (classic JSX runtime) |
| Parsing | `pdf.js` (PDF), `mammoth` (DOCX), native (TXT) |
| AI | Provider abstraction over Anthropic / OpenAI / Gemini REST APIs |
| Exports | `JSZip` (DOCX OOXML + ZIP), `jsPDF` (PDF) |
| Import | `SheetJS/xlsx` (CSV + Excel) |
| Storage | Browser `localStorage` |

## Documentation

- [ASSUMPTIONS.md](ASSUMPTIONS.md) — material assumptions and what was deliberately out of scope
- [REQUIREMENTS_TRACEABILITY.md](REQUIREMENTS_TRACEABILITY.md) — TR-001…TR-015 mapping
- [AI_GUARDRAILS.md](AI_GUARDRAILS.md) — how factual consistency and prompt-injection protection work
- [SECURITY.md](SECURITY.md) — controls and threat model
- [TESTING.md](TESTING.md) — what was verified, how, and the known gaps

## Known limitations

- AI generation requires an internet connection and a valid provider key; it cannot be exercised offline.
- Parsing quality for image-only/scanned PDFs is limited (no OCR); the app flags low extraction confidence.
- Being client-side, there is no server-side authentication, multi-user isolation, malware scanning, durable queue, or signed object storage. These are documented as out of scope for this target in [ASSUMPTIONS.md](ASSUMPTIONS.md).
