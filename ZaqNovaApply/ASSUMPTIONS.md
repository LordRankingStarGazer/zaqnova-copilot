# Assumptions

Documented per the autonomous operating rules: where an input was absent, a safe, practical assumption was made and recorded here.

## Architecture target

**Assumption:** ZaqNova Apply should match the existing ZaqNova product line (Interview CoPilot) — a self-contained, client-side, single-file web app — rather than the full multi-service production SaaS described in the requirements baseline.

**Why:** The runtime environment is a Windows laptop with no backend runtime, no database, no cloud credentials, and no object storage or queue infrastructure. The existing ZaqNova apps (`index.html`, `interview-assistant.html`) are single-file React apps using CDN libraries, browser-side document parsing, browser-side multi-provider AI, and `localStorage`. The user explicitly selected the "single-file client app" target. Honoring the honesty rule (never fabricate deployments, tests, or infrastructure), a client-side app is something that can actually be built, run, and verified here.

## Consequent scope decisions

| Requirement area | Decision | Rationale |
|---|---|---|
| Persistence | Browser `localStorage` instead of Postgres | No DB available; data is per-device and user-owned. |
| Object storage | Generated files are produced on demand and downloaded; not persisted to a bucket | No object storage available; avoids storing PII server-side. |
| Background queue | In-page async worker pool with bounded concurrency + retries | No external queue; provides observable per-row status, retry, and isolation within the browser. |
| AuthN/Z | Single-user, device-local; no login | No identity provider; data isolation is achieved by the data never leaving the device. |
| Malware scanning | Documented adapter point only (magic-byte + MIME + size validation implemented) | No scanning service; validation mitigates the most common spoofing/oversize vectors. |
| CI/CD, staging/prod | Not applicable to a static single file | No deployment platform or credentials supplied. The file is deployment-ready to any static host. |
| Billing/usage ledger | `audit()` event log kept locally; no payment provider | Monetization was not requested. |

## Defaults chosen

- **Max upload size:** 5 MB (spec default), `MAX_UPLOAD_MB` constant.
- **Bulk row limit:** 50 (`BULK_ROW_LIMIT`), surfaced in the UI.
- **Bulk concurrency:** 2 (`BULK_CONCURRENCY`); **max retries:** 2 with exponential-ish backoff.
- **Default language:** English. **Default model:** Claude Sonnet 4 (matches the existing apps' default provider ordering).
- **Default tailoring mode:** Balanced.

## Deliberately out of scope (per baseline)

- Job scraping and automatic job submission — explicitly excluded by the requirements.
- OCR for scanned/image PDFs — flagged as low-confidence extraction instead.

## What "deployment" means here

No deployment credentials or platform were provided, so **no production deployment was performed and no deployment URL is claimed.** The application is deployment-ready: it is a static bundle that runs on any static host (e.g. GitHub Pages, Netlify, S3+CloudFront, or `python -m http.server`). It was verified to load and run locally via a static server and an automated browser preview.
