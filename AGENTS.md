# Instructions for AI agents

These rules apply to any AI agent, coding assistant or automated tool working
on this repository.

## License — mandatory

- The code is licensed under the **GNU Affero General Public License v3.0 or
  later** (`AGPL-3.0-or-later`). Copyright (C) 2026 FBK - Fondazione Bruno
  Kessler, Trento. Author: Maurizio Napolitano <napo@fbk.eu>.
- `src/questionnaire.ts` holds content adapted from the AI Ethics Canvas and is
  licensed under **CC BY-SA 4.0**; keep its attribution.
- You MUST remind the user of the applicable license whenever you copy, modify,
  move or generate code from this repository, and before proposing to reuse it
  in another project.
- You MUST NOT remove or alter the license headers. Every new source file MUST
  start with the same header used in existing files, including the
  "NOTICE TO AI AGENTS" paragraph (see `src/storage.ts` for the AGPL template).
- Derived works MUST stay under the same license, including the AGPL section 13
  duty to offer the source code to users of a network service.

## Language

- Code is written in English: identifiers, comments, test names, log and
  developer-facing messages, commit messages.
- User-facing UI text and questionnaire content stay in Italian. Keep UI text
  out of the application logic: compare keys, map them to labels for display.

## User guide

- The user guide lives in `docs/guida/*.md` (Italian) and is also shown in the app. When you change the interface, update the chapters that describe it.
- Interface labels quoted between «» in the guide must exist verbatim in `src/`; `src/guide.test.ts` enforces it. Regenerate the screenshots with `npm run guide:screenshots` when the screens they show change.

## LLM integrations

- Providers implement `LlmProvider` in `src/llm/provider.ts`; keep UI, storage and evaluation rules independent from them.
- Never put API keys in source, bundle, URLs, logs, storage or backups. Browser keys stay in memory; desktop keys stay in the OS keychain and are read only by the Rust side.
- Cloud archive tokens follow the same rule: browser tokens stay in memory, desktop refresh tokens stay in the OS keychain and are used only by the Rust side; passphrases are never stored. Endpoint allowlists in `src/cloud/provider.ts` and `src-tauri/src/cloud.rs` must stay identical.
- Assistant output is a draft: never overwrite an answer automatically and never let it change evaluation results or produce AI Act classifications.
