# Verifica della versione alpha con checklist normativa

Data: 29 ottobre 2026. Versione: 0.3.0-alpha.2.

- Test unitari (Vitest): 114 superati in 12 file. Coprono:
  - modello e backup (formati da 1 a 6, migrazione, nessuna credenziale nei backup e nelle preferenze);
  - adattatori Gemini e modello locale con trasporto simulato (errori, annullamento, endpoint non ammessi);
  - contesto autorizzato, verifica delle citazioni, nessuna sovrascrittura, indipendenza della valutazione;
  - checklist normativa (anche voci tolte da una checklist successiva), versioni e revisioni, contributi, casi a confronto, quadro dell'ente, esportazioni, archivio sul drive;
  - etichette della guida presenti nell'interfaccia.
- Build di produzione React/TypeScript: superata. Nel bundle non compaiono chiavi API.
- Test browser Chromium: superati tutti gli 11 script di `npm run test:smoke` (base, assistente, modello locale, casi a confronto, archivio sul drive, cartella sincronizzata, esportazioni, guida, versioni, contributi, quadro dell'ente), senza richieste esterne.
- Rust: `cargo check` senza avvisi, 9 test superati; 1 test ignorato (prova reale del portachiavi, da lanciare con `cargo test -- --ignored`).
- Non verificati: chiamate a Google con una chiave reale, interfaccia desktop in esecuzione, pacchetti Windows e macOS (previsti tramite GitHub Actions).
