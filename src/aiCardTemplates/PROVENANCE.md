# AI Product Card templates

The files in `cards/` and `metadata/` are the AI Product Card templates of Fondazione Bruno Kessler (Smart Community
Lab), copied unchanged from https://github.com/smartcommunitylab/ai_product_card_templates (folder `aipc_template`,
commit c995940718a7) and used with the permission of FBK.

EthiCanvasAI fills their `{{ name | default("…", true) }}` placeholders and their empty YAML values with the answers
of the canvas (see `src/aiCards.ts`). To follow a new version of the templates, replace these files and check the
placeholder names used in `src/aiCards.ts`.

Framework: A. Celepija, A. Palmero Aprosio, B. Lepri, R. Kazhamiakin, "AI product cards: a framework for code-bound
formal documentation cards in the public administration", Data & Policy 7:e1, 2025, doi:10.1017/dap.2024.55.
