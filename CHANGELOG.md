# Changelog

All notable changes to EthiCanvasAI are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

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
