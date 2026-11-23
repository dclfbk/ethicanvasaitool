# Changelog

All notable changes to EthiCanvasAI are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.5.1] - 2026-11-23

### Added
- The application version is always visible: at the bottom of the sidebar, linked to its release notes, and in the page footer.

### Changed
- Vector logos with transparent background (`src/assets/logos/*.svg`): EU NextGenerationEU converted from the official European Commission EPS, emblem of the Italian Republic and PAT coat of arms from Wikimedia Commons (public domain), Department for digital transformation mark from innovazione.gov.it with its name in Titillium Web, FBK from its brand book. `npm run logos` renders the PNG files used by the PDF exports. The EU emblem is never smaller than the other logos.
- The AI Product Card export fills the official FBK templates (`src/aiCardTemplates`, from smartcommunitylab/ai_product_card_templates) instead of cards written from scratch.

### Fixed
- The dashboard smoke test no longer depends on the time of day.

## [0.5.0] - 2026-11-23

### Added
- Questionnaire `canvas-pa-0.2`, aligned cell by cell with the AI Ethics Canvas: 39 questions instead of 20, following the schema reviewed with the canvas authors. Questions now have structured fields: sub-questions, yes/no rows with advice on "no", single or multiple options, "done" ticks, a priority order and "Altro" rows that can be added (`src/AnswerFields.tsx`, `src/answers.ts`). The free text becomes "Ulteriori dettagli".
- One request per person involved: each person added with "Serve un’altra figura" has their own "Che cosa deve chiarire" field, and contribution files carry only the recipient's request.
- AI Product Card export (`src/aiCards.ts`): a ZIP with the descriptive cards (AI product, domain problem, data, model, adaptation, optimization) in Markdown and the declarative specifications in YAML, following Celepija et al., Data & Policy 7:e1 (2025). The cards are filled from the canvas; technical parts are left for AI scientists and engineers. Names of people involved and internal notes are left out.
- ESLint and Prettier (`npm run lint`, `npm run format`), and a test that keeps the cloud endpoint allowlists in TypeScript and Rust identical.

### Changed
- The canvas PDF in the original style writes each field in its own cell and ticks the printed check boxes; answers without a place on the canvas continue in the final pages. The poster lays out sections with many questions in several columns.
- Reports, comparisons between versions, the assistant context, the case knowledge base and contribution files use the whole structured answer as text.
- Backup format 7. Projects, backups and contribution files of `canvas-pa-0.1` are migrated when read: answers keep their place, the answers to the two removed questions ("Come capirete se funziona davvero?" and "Chi rischia di rimanere fuori?") move to the notes of related questions, and a version "Prima dell’aggiornamento del questionario" keeps the original.
- User guide and screenshots updated.

### Fixed
- Semicolons in the exported iCalendar review event are escaped as RFC 5545 requires.
- The report sent to the assistant never includes the names of people involved.

## [0.4.1] - 2026-11-18

### Added
- Merged `origin/main` (`0.3.0` features): regulatory AI Act checklist (`RegulatoryPanel`), offline PWA support (`serviceWorker.js`, `manifest.webmanifest`), local PDF/DOCX/ODT text extraction (`pdfjs-dist`), Gemini streaming with reasoning display, and original-template Canvas PDF export.
- Lazy loading (`React.lazy` and `Suspense`) for all secondary panels and modals in `src/App.tsx`, reducing the initial main JS bundle from 576 kB to 372 kB.
- Verified that all CSS, fonts, icons, JS libraries, and assets are served from the same origin with no implicit external requests.

### Changed
- Bumped application version to `0.4.1` across `package.json`, `src-tauri/tauri.conf.json`, `src-tauri/Cargo.toml`, and `src/updates.ts`.

## [0.4.0] - 2026-11-18

### Added
- Topbar **Accessibility** section and configuration modal (`src/AccessibilityPanel.tsx`, `src/accessibility.ts`, `src/accessibility.css`):
  - High contrast and dark themes.
  - Color-vision adaptations (protanopia, deuteranopia, tritanopia, achromatopsia) and non-color visual cues (underlined links, bordered badges, patterned progress bars).
  - Text scaling (100%, 115%, 130%), wide spacing, and uniform readable font option.
  - Strong keyboard focus ring, enlarged interactive targets (44 px minimum), and reduced motion toggle, persisted locally.
- Multi-person support and cancel/toggle actions for *"Serve un’altra figura"* (`waiting` state), with `parsePeople` shared across `src/contributions.ts`, `src/dashboard.ts`, and `src/App.tsx`.
- Selectable review date picker directly in question P20 (`review`), synchronized with *Versioni e revisioni* and *Quadro dell’ente*.
- Direct link from the *Inquadramento normativo (AI Act)* notice in the report view to question P11 (`risk`).
- Visible application version in *Metodo e impostazioni* and on-demand GitHub release/tag version check (`checkWebVersion`, `compareVersions`) in `src/updates.ts` and `src/UpdatePanel.tsx`, plus automated version-alignment tests in `src/updates.test.ts`.

### Changed
- Clarified questionnaire prompts and explanations in `src/questionnaire.ts` (P1 AI necessity, P5 data collection goals, P6 anonymization/access protection, P9 training/reuse, P10 human oversight, P11 legal spreadsheet reference, P14 roles/explainability/feedback loops, P15 plain-language GDPR notice, P17 precautions/aggregated open data, P18 technical vs. social/cultural principles).
- Updated Italian user guide chapters (`docs/guida/01-primi-passi.md`, `docs/guida/02-compilare.md`) and regenerated guide screenshots.

## [0.3.0] - 2026-10-29

### Added
- Regulatory AI Act support checklist inside each project.
- Installable offline PWA web build and local text extraction from PDF, DOCX, and ODT attachments.
- Streaming responses and reasoning display for Gemini, plus original Canva-template PDF export.

## [0.2.0] - 2026-10-20

### Added
- Synchronised folder storage and file-based contributions without a server.
- Organisation dashboard and local model server support (Ollama / OpenAI-compatible).

## [0.1.0] - 2026-10-01

### Added
- Initial release of EthiCanvasAI (local archive, 20-question AI Ethics Canvas journey, optional Gemini assistant, PDF/DOCX/ODT/Markdown exports, and Italian user guide).
