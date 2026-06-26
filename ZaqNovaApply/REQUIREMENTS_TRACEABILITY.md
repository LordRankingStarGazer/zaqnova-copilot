# Requirements Traceability Matrix

Maps the baseline user stories (TR-001…TR-015) to interface location, implementation, and verification. All code references are to `index.html`.

Legend — **Status:** ✅ implemented & verified · 🟡 implemented, AI path needs a live key to exercise end-to-end · ⛔ out of scope (see ASSUMPTIONS.md).

| ID | Story | UI location | Implementation | Verification | Status |
|----|-------|-------------|----------------|--------------|--------|
| TR-001 | Upload PDF/DOCX/TXT base resumes | Resumes → Upload | `validateUpload` (ext+MIME+magic-byte+size), `extractFileText`, `parseResume`, `scoreExtraction` | Magic-byte validation logic reviewed; extraction-confidence + warnings deterministic; parse needs live key | 🟡 |
| TR-002 | Enter job title, company URL, description, skills manually | Jobs → Manual entry | `JobsView` manual form, `EMPTY_JOB`, `validateImportRow` | Form renders; required-field validation verified | ✅ |
| TR-003 | Import multiple jobs via Excel/CSV | Jobs → Bulk import | `parseImportFile` (SheetJS), `validateImportRow`, dedup, `ImportReview` | Validation + dedup logic verified; sample template header verified | ✅ |
| TR-004 | AI helpers for title/description/skills | Jobs → ✨ buttons | `aiHelper('title'|'desc'|'skills')` | UI present; needs live key | 🟡 |
| TR-005 | Choose to preserve original content | Tailor controls | `opts.keepOriginal` → prompt | Wired into `buildTailorUser`; needs live key | 🟡 |
| TR-006 | Choose to shorten the resume | Tailor controls | `opts.shorten` → prompt | Wired; needs live key | 🟡 |
| TR-007 | Configure language/experience/seniority/tone/role/mode | Tailor controls, Settings | `TailorControls`, `DEFAULT_OPTS`, model select | Controls render and bind | ✅ |
| TR-008 | Preview, edit, download, save every tailored resume | Workspace tabs | `WorkspaceView` preview/edit, `saveVersion`, `exportResume` | Render + exports verified; generation needs key | 🟡 |
| TR-009 | Compare original vs tailored | Workspace → Compare | `CompareView` (line diff) | Renders from source vs current version | ✅ |
| TR-010 | Match score + keyword coverage | Workspace → Match & warnings | `computeScore`, `ScorePanel` | Scoring verified (req/pref/title weighting, missing/matched) | ✅ |
| TR-011 | Regenerate selected sections independently | Workspace → Edit → ✨ | `regenerateSection`, `runTailoringPipeline({onlySection})` | Section splice logic reviewed; needs key | 🟡 |
| TR-012 | Maintain multiple base-resume profiles | Resumes | `ProfilesView` create/duplicate/archive/delete | Verified (archive filter, delete confirm) | ✅ |
| TR-013 | Export DOCX/PDF/TXT/ATS text | Workspace export buttons | `toDocxBlob`, `toPdfBlob`, `toPlainText` | DOCX OOXML parts + PDF magic verified valid | ✅ |
| TR-014 | History + restore previous versions | Workspace → Versions | `rec.versions`, `restore` | Version array + current pointer verified | ✅ |
| TR-015 | Monitor bulk progress, failures, retries | Bulk | `BulkView` worker pool, statuses, `retryRow`, ZIP, CSV | Status model + ZIP/CSV builders verified; runs need key | 🟡 |

## Non-functional / cross-cutting

| Requirement | Implementation | Status |
|---|---|---|
| Factual consistency (no fabrication) | `factualCheck` deterministic + `buildGuardrailSystem` prompt | ✅ caught fake employer/cert/metric in test |
| Prompt-injection defence | `scanForInjection`, untrusted-data system instructions | ✅ pattern scan verified |
| Filename sanitisation | `sanitizeFilename`, `buildFilename` | ✅ verified |
| Responsive mobile + desktop | Sidebar + mobile bottom nav, responsive grids | ✅ verified at 375px and desktop |
| Accessibility | Labels, focus rings, `role=status/dialog/progressbar`, aria-labels | 🟡 implemented; no formal audit run |
| Data deletion | `DataView` wipe controls | ✅ |
| Audit logging (redacted) | `audit()` ring buffer, no resume bodies | ✅ |
| Server-side auth, multi-tenant isolation, queue, malware scan, signed URLs | — | ⛔ out of scope (ASSUMPTIONS.md) |
