# Verifica della versione con assistente Gemini

Data: 26 ottobre 2026.

- Test unitari (Vitest): 27 superati. Coprono modello e backup (formati 1 e 2, migrazione, nessuna credenziale nei backup e nelle preferenze), adattatore Gemini con trasporto simulato (richiesta `store: false`, test di connessione, chiave non valida, permessi, quota, limite di frequenza, modello non disponibile, timeout, annullamento, rete, risposta bloccata o incompleta, endpoint non ammessi), chiave di sessione del browser (solo intestazione, mai nell'URL), contesto autorizzato e separazione tra progetti e riferimenti, verifica delle citazioni, nessuna sovrascrittura, indipendenza della valutazione.
- Build di produzione React/TypeScript: superata. La chiave non compare nel bundle.
- Test browser Chromium `scripts/smoke.mjs`: superato, senza richieste esterne.
- Test browser `scripts/smoke-assistant.mjs` con Gemini simulato: superato.
- Rust: `cargo check` senza avvisi, 2 test superati. La prova reale del portachiavi (Secret Service, Linux) è superata.
- Pacchetto desktop Linux (.deb): vedi sotto.
- Non verificati: chiamate a Google con una chiave reale, interfaccia desktop in esecuzione, pacchetti Windows e macOS (previsti tramite GitHub Actions).
