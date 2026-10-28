# EthiCanvas AI

Applicazione React + TypeScript con archivio locale e contenitore Tauri 2. L'assistente Gemini è facoltativo: senza configurarlo l'applicazione funziona interamente in locale.

## Avvio web

Richiede Node.js 22.12+ (o 20.19+).

```bash
npm ci
npm run dev
```

Aprire http://localhost:1420. Per una distribuzione web: `npm run build` e pubblicare il contenuto di `dist/` su un hosting statico. Nessun backend e nessun account richiesti. Il database è IndexedDB, legato all'origine e al profilo del browser. Nessun font remoto; l'unica connessione esterna possibile è quella verso la Gemini API, solo se l'utente attiva l'assistente.

## Desktop Tauri

Installare Rust e i prerequisiti di Tauri per il proprio sistema: https://v2.tauri.app/start/prerequisites/

```bash
npm ci
npm run tauri dev
npm run tauri build
```

Su Linux servono anche `libwebkit2gtk-4.1-dev`, `librsvg2-dev` e `libdbus-1-dev` (per il portachiavi). Firma e notarizzazione non sono configurate. Il backend usa `tauri-plugin-sql` con SQLite.

### Build automatiche su GitHub

- [.github/workflows/ci.yml](.github/workflows/ci.yml): a ogni push su `main` e a ogni pull request esegue test, build, test browser e test Rust.
- [.github/workflows/release.yml](.github/workflows/release.yml): crea i pacchetti **Linux** (deb, rpm, AppImage) e **Windows** (msi, exe) con `tauri-action`. Parte con un tag `v*` (ad esempio `git tag v0.2.0 && git push --tags`) oppure a mano dalla scheda Actions; i pacchetti vengono allegati a una release in bozza, da controllare prima della pubblicazione. Non sono firmati: Windows SmartScreen mostrerà un avviso.
- [.github/workflows/pages.yml](.github/workflows/pages.yml): a ogni push su `main` pubblica la versione web su GitHub Pages, all'indirizzo <https://dclfbk.github.io/ethicanvasaitool/>. I dati restano nel browser di chi la usa (IndexedDB): il sito non ha un server né un archivio condiviso.

### Aggiornamenti dell'app desktop

All'avvio l'app desktop controlla `https://github.com/dclfbk/ethicanvasaitool/releases/latest/download/latest.json`. Se trova una versione più recente chiede all'utente se installarla: scarica il pacchetto, ne verifica la firma, lo installa e riavvia. Il controllo si può disattivare in *Metodo e impostazioni*. La versione web non fa questo controllo: la aggiorna chi gestisce il server.

Il numero di versione va aggiornato in [package.json](package.json), [src-tauri/Cargo.toml](src-tauri/Cargo.toml) e [src-tauri/tauri.conf.json](src-tauri/tauri.conf.json) e deve corrispondere al tag (`v0.2.0` → `0.2.0`). Solo le release pubblicate, non le bozze né le pre-release, vengono proposte agli utenti.

Configurazione da fare una volta, prima della prima release:

1. Generare la coppia di chiavi di firma: `npm run tauri signer generate -- -w ~/.tauri/ethicanvasaitool.key`. Conservare la chiave privata e la password fuori dal repository: se si perdono, le installazioni esistenti non potranno più aggiornarsi.
2. Copiare il contenuto di `~/.tauri/ethicanvasaitool.key.pub` nel campo `plugins.updater.pubkey` di [src-tauri/tauri.conf.json](src-tauri/tauri.conf.json) (già fatto per la chiave attuale).
3. Nei secret del repository GitHub aggiungere `TAURI_SIGNING_PRIVATE_KEY` (il contenuto della chiave privata) e `TAURI_SIGNING_PRIVATE_KEY_PASSWORD`.

Il workflow di release usa [src-tauri/tauri.release.conf.json](src-tauri/tauri.release.conf.json) per creare i pacchetti di aggiornamento firmati e allegare `latest.json` alla release. Le build locali non ne hanno bisogno.

## Funzioni implementate

- Archivio ricercabile anche nel testo delle risposte; filtri per progetti in pausa e compilati.
- Creazione di progetti e caso dimostrativo esplicitamente identificato.
- App desktop: controllo di nuove versioni all'avvio (disattivabile) e aggiornamento firmato, sempre su conferma dell'utente.
- 20 domande in 6 tappe: contesto e cinque sezioni del canvas.
- Suggerimenti predefiniti, sempre da adattare e confermare.
- Salvataggio automatico serializzato, sospensione e ripresa sullo stesso passo.
- Bozza, risposta confermata, attesa di contributo e non applicabile motivato.
- Figura da coinvolgere e quesito da sottoporle, senza invio automatico.
- Riferimenti per domanda: documenti, leggi, dataset, esperti e verifiche.
- Allegati locali fino a 10 MB ciascuno, inclusi nel database in formato data URL.
- Riepilogo parziale o completo con lacune, figure da coinvolgere e profilo descrittivo dichiarato.
- Rapporto Markdown e backup JSON completo di allegati. L'importazione crea copie senza sovrascrivere progetti esistenti (massimo 100 MB per pacchetto). Sono accettati i backup della versione 0.1 (formato 1) e quelli attuali (formato 2).
- Estratti testuali per ogni riferimento, incollati a mano o estratti in locale dai file TXT, MD, CSV e JSON.
- Assistente Gemini facoltativo: «Spiegami la domanda», «Aiutami a formulare una risposta», «Che cosa manca per rispondere?».

## Assistente Gemini (facoltativo)

### Che cosa serve

Una **chiave API Gemini**, creata in [Google AI Studio](https://aistudio.google.com/apikey). Un abbonamento all'app Gemini (ad esempio Google AI Pro) non dà accesso alle API. I termini Google consentono di rendere disponibile un client delle API a utenti nello Spazio economico europeo, in Svizzera e nel Regno Unito solo con servizi a pagamento, cioè con un progetto Google con fatturazione attiva.

### Configurazione

In **Metodo e impostazioni → Assistente Gemini**: attivare l'assistente, scegliere il modello, inserire la chiave e usare «Verifica connessione». «Elimina chiave e configurazione» cancella credenziale e preferenze.

- **Browser**: la chiave resta solo in memoria per la sessione e si perde chiudendo o ricaricando la pagina. Non viene scritta in localStorage, IndexedDB, URL, log o backup. La Gemini API accetta chiamate dirette dal browser (CORS), quindi non serve un servizio ponte. Google sconsiglia però di esporre chiavi lato client in produzione: la chiave è leggibile dagli script della pagina e dalle estensioni del browser. Conviene usare una chiave dedicata, con restrizioni e quota limitata. Per una distribuzione istituzionale è preferibile un proxy gestito dall'ente, che questa versione non include.
- **Desktop Tauri**: la chiave è salvata nel portachiavi del sistema (Keychain su macOS, Gestione credenziali su Windows, Secret Service su Linux, ad esempio GNOME Keyring o KWallet). Le chiamate HTTP partono dal processo Rust ([src-tauri/src/gemini.rs](src-tauri/src/gemini.rs)), che aggiunge la chiave e accetta solo due endpoint fissi: la chiave non torna mai all'interfaccia.

Modelli proposti, verificati sulla [documentazione Google](https://ai.google.dev/gemini-api/docs/models) a ottobre 2026: `gemini-3.8-flash` (predefinito), `gemini-3.6-flash`, `gemini-3.5-flash-lite`, `gemini-4-argon`. I modelli 2.x e quelli in anteprima sono esclusi. L'integrazione usa l'[Interactions API](https://ai.google.dev/gemini-api/docs/interactions) via REST, senza SDK. La lista è in [src/llm/gemini.ts](src/llm/gemini.ts) e va aggiornata quando Google ritira un modello.

### Quali dati vengono inviati

Prima di ogni richiesta l'utente vede l'elenco del contesto e l'anteprima esatta del testo. Si inviano solo la domanda corrente con spiegazione e origine, e inoltre, se selezionati: nome, ente e descrizione del progetto, la risposta già scritta, le risposte precedenti scelte una per una, i riferimenti autorizzati uno per uno (titolo, note, collegamento ed estratto testuale).

- Gli allegati non vengono mai inviati come file. Un collegamento viene indicato come «non consultato» e non viene aperto.
- L'autorizzazione di un riferimento non si estende agli altri né al resto dell'archivio.
- Le richieste usano `store: false`, quindi l'interazione non viene conservata sui server Google. Restano validi i [termini del servizio](https://ai.google.dev/gemini-api/terms): sul piano gratuito, fuori dallo Spazio economico europeo, Google può usare i contenuti per migliorare i prodotti. Non inviare dati personali o riservati.

### Come vengono trattate le risposte

- Il modello riceve istruzioni di usare solo il contesto fornito, di trattare il testo dei documenti come materiale da analizzare e non come istruzioni, di distinguere fatti dichiarati, informazioni documentate e ipotesi, di segnalare le informazioni mancanti e le figure da coinvolgere, e di non presentare misure pianificate come attuate.
- Le citazioni vengono controllate in locale. Un'affermazione «documentata» deve rimandare a un riferimento con estratto effettivamente inviato; altrimenti diventa «ipotesi da confermare» con l'avviso «citazione non corrispondente al contesto inviato».
- La proposta è modificabile e non sostituisce mai da sola la risposta. Se la risposta è vuota si può inserire come bozza; se esiste già si può aggiungere in coda oppure sostituire, con una conferma esplicita. In ogni caso la risposta torna allo stato «bozza».
- Per ogni inserimento vengono conservati fornitore, modello, azione, modalità e data, senza credenziali. La provenienza compare nella domanda e nel rapporto esportato.
- Le regole di completezza e gli stati della valutazione non dipendono da Gemini. Sulla domanda sull'inquadramento normativo il modello organizza informazioni e verifiche, ma non assegna categorie AI Act né conformità.
- La richiesta mostra lo stato di caricamento e si può annullare; gli invii duplicati sono bloccati. Gli errori (chiave non valida, permessi, quota, limite di frequenza, credito esaurito, modello non disponibile, timeout, rete assente, servizio non disponibile, risposta bloccata) hanno un messaggio dedicato e non toccano la risposta. Cambiando progetto o domanda la richiesta in corso viene annullata e il suo risultato scartato.

## Limiti espliciti

È un MVP, non una implementazione integrale di ogni quesito del PDF. Le domande sono adattamenti tematici con origine indicata. Sono incluse estensioni su chatbot, inclusione e riesame. I ruoli sono suggerimenti, non la riproduzione delle attribuzioni grafiche originali.

Non sono implementati classificazione normativa AI Act, revisione formale, cronologia completa delle valutazioni, estrazione del testo da PDF, Word o immagini, sincronizzazione, PWA offline, crittografia dell'archivio, integrazioni Ollama e Microsoft Copilot o un proxy per la chiave Gemini. Il rapporto non costituisce una certificazione. La presenza di evidenze non equivale a verifica della loro validità.

Il salvataggio locale non è un backup. Esportare copie, soprattutto prima di cancellare dati del browser o cambiare dispositivo. Gli allegati sono incorporati per portabilità; per archivi estesi sarà opportuno separarli in un file store con transazioni e controlli di integrità. La versione desktop e quella web hanno archivi separati.

## Struttura

- `src/questionnaire.ts`: sezioni e domande del questionario (CC BY-SA 4.0).
- `src/model.ts`: modello dati, regole di completezza, validazione importazione.
- `src/assets/`: font (OFL) e loghi del canvas, ripresi dal PDF dell'AI Ethics Canvas.
- `src/storage.ts`: adattatori IndexedDB / SQLite e download.
- `src/App.tsx`: archivio, percorso, riferimenti, riepilogo e impostazioni.
- `src/llm/provider.ts`: interfaccia comune dei fornitori LLM e degli archivi delle credenziali.
- `src/llm/gemini.ts`: adattatore Gemini (Interactions API, gestione errori, modelli).
- `src/llm/geminiBrowser.ts`, `src/llm/geminiTauri.ts`: trasporto e credenziali per browser (chiave in memoria) e desktop (portachiavi).
- `src/assistant.ts`: contesto autorizzato, istruzioni, schema della risposta, verifica delle citazioni, inserimento come bozza.
- `src/AssistantPanel.tsx`, `src/AssistantSettingsPanel.tsx`: interfaccia dell'assistente e impostazioni.
- `src/textExtraction.ts`: estrazione locale del testo dai file di testo.
- `src-tauri/src/gemini.rs`: portachiavi e chiamate HTTP lato desktop.
- `src-tauri/`: configurazione e ingresso Rust.

## Passi successivi concordati

1. Validare e completare il questionario rispetto al canvas, provare con funzionari e tecnici; definire criteri di revisione e approfondimento AI Act.
2. ~~Gemini~~: implementato in questa versione.
3. Ollama: connessione opzionale a un modello locale.
4. Microsoft Copilot: integrazione futura distinta da GitHub Copilot.

## Verifica

```bash
npm test
npm run build
npx playwright install chromium
npm run test:smoke
cd src-tauri && cargo test            # test Rust
cd src-tauri && cargo test -- --ignored  # prova reale del portachiavi del sistema
```

I test non richiedono credenziali reali. I test unitari usano un fornitore e un trasporto simulati. `scripts/smoke-assistant.mjs` intercetta in Playwright tutte le chiamate verso `generativelanguage.googleapis.com` e risponde con un server simulato. Così verifica: assistente disattivato per impostazione predefinita, chiave di sessione, test di connessione, estrazione locale, anteprima del contesto, annullamento, errori di quota e chiave non valida, invio singolo con doppio clic, nessuna sovrascrittura, bozza con provenienza, risultato tardivo scartato, assenza della chiave da archivio, backup e console.

Per provare con una chiave vera basta `npm run dev`, poi configurare l'assistente nelle impostazioni.

## Autore, licenza e provenienza

Copyright (C) 2026 FBK - Fondazione Bruno Kessler, Trento. Autore: Maurizio Napolitano <napo@fbk.eu>.

Il codice è software libero distribuito con licenza **GNU Affero General Public License v3.0 o successiva** (`AGPL-3.0-or-later`), testo completo in [LICENSE](LICENSE). Chi offre una versione modificata come servizio in rete deve rendere disponibile agli utenti il relativo codice sorgente (art. 13 AGPL). Ogni file sorgente riporta l'intestazione di licenza e un avviso rivolto agli agenti AI, che devono conservarla e ricordare la licenza all'utente; le regole per gli agenti sono in [AGENTS.md](AGENTS.md).

Il questionario in `src/questionnaire.ts` adatta e amplia l'AI Ethics Canvas, di cui Maurizio Napolitano è tra gli autori, © 2024 Provincia Autonoma di Trento e Fondazione Bruno Kessler, CC BY-SA 4.0, e resta distribuito con la stessa licenza. Dettagli in [NOTICE.md](NOTICE.md). Le dipendenze mantengono le rispettive licenze.

- Canvas: https://github.com/dclfbk/canvasaiethics
- Articolo: https://doi.org/10.1017/dap.2026.10088
- Licenza del codice: https://www.gnu.org/licenses/agpl-3.0.html
- Licenza del questionario: https://creativecommons.org/licenses/by-sa/4.0/
