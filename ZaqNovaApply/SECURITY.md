# Security & Threat Model

ZaqNova Apply handles resumes, which contain PII. Because it is a client-side app with no backend, the primary security posture is: **data stays on the user's device and is transmitted only to the AI provider the user explicitly selects.**

## Controls implemented

| Control | Implementation |
|---|---|
| File upload validation | Extension + MIME + magic-byte signature + size (`validateUpload`) blocks spoofed/oversized files |
| Untrusted-content handling | Resumes/JDs treated as data; injection scan + system-instruction hardening; React auto-escaping (no `dangerouslySetInnerHTML`) |
| Secret handling | No secrets in source/repo; API keys only in `localStorage`, sent only to the chosen provider, never logged |
| Log redaction | `audit()` stores actions + coarse metadata only; never resume bodies, contacts, or full JDs |
| Data minimisation | Only the necessary resume JSON and job text are sent to the model |
| User-controlled deletion | Privacy view deletes tailored resumes or all data; destructive actions require confirmation |
| Data isolation | Single-device, single-user; data never leaves the browser except to the AI provider |
| Safe filenames | `sanitizeFilename` strips unsafe characters before download |
| Decompression-bomb mitigation | Bulk import capped at `BULK_ROW_LIMIT` rows; uploads capped at `MAX_UPLOAD_MB` |
| Transport security | All provider calls use HTTPS endpoints |

## Threat model

| Threat | Mitigation | Residual risk |
|---|---|---|
| Malicious file upload | Signature/MIME/size validation; parsing in sandboxed browser libs | No malware scanning (documented adapter point) — a hostile file could still exploit a parser bug |
| Prompt injection in resume/JD | `scanForInjection` + untrusted-data system instructions + deterministic factual check | Novel phrasings may evade the regex; the deterministic check still blocks fabricated additions |
| Cross-user data access | N/A — single-device, no shared backend | Anyone with physical access to the unlocked browser can read `localStorage` |
| Broken object-level authZ | N/A — no server objects | — |
| External URL abuse (SSRF) | The app does **not** fetch job/company URLs server-side; URLs are stored as text only | — |
| AI hallucination / fabrication | Deterministic `factualCheck` + guardrail prompt + gap reporting + user confirmation before export | Heuristic; cannot verify truth of preserved facts |
| Data leakage via logs | `audit()` redaction; no analytics of resume content | Provider receives resume text by design (necessary for tailoring) |
| Insecure exports | Exports built locally; no embedded prompts/debug content; validated file structure | — |
| Queue abuse / DoS | Bounded concurrency, bounded retries, row caps | Client-only; a user can only affect their own browser |
| Compromised dependencies | Pinned CDN versions for parsing/export/import libs | React/Tailwind/Babel/pdf.js loaded from CDN — supply-chain trust in those CDNs |
| Secret exposure | Keys never in source/logs; `localStorage` only | A shared/compromised machine exposes the stored key |

## Hardening recommendations for a hosted deployment

- Serve over HTTPS with a strict Content-Security-Policy and Subresource Integrity on CDN scripts (or vendor the libraries).
- Pin exact dependency versions (the Babel and Tailwind CDNs are dev builds and print console warnings).
- For multi-user/SaaS use, add the server-side components listed as out-of-scope in `ASSUMPTIONS.md` (auth, per-tenant isolation, malware scanning, signed object storage, durable queue).
