# EthiCanvasAI

Applicazione React + TypeScript con archivio locale e contenitore Tauri 2. L'assistente Gemini è facoltativo: senza configurarlo l'applicazione funziona interamente in locale.

> **Versione alpha — instabile.** Questa versione è in prova, può contenere errori, perdere dati o cambiare senza preavviso. Non usarla per attività operative né con dati reali, personali o riservati. Esporta copie di sicurezza dei progetti che vuoi conservare.

## Guida

La guida per chi compila, chi coordina e chi installa è in [docs/guida](docs/guida/00-indice.md) ed è consultabile anche nell'app, dalla voce **Guida** del menu, senza connessione. Il testo è scritto in Markdown, che l'app legge al momento della build con un piccolo lettore interno ([src/guideMarkdown.ts](src/guideMarkdown.ts)), senza HTML dal testo. I link `#app-…` aprono le funzioni dell'app.

- Le etichette dell'interfaccia citate tra «» devono esistere nel codice: [src/guide.test.ts](src/guide.test.ts) lo verifica insieme a link, ancore e immagini. Se si rinomina un pulsante, il test segnala il capitolo da aggiornare.
- Gli screenshot in `docs/guida/img` si rigenerano dall'app con dati di esempio: `npm run guide:screenshots`.
- Le icone dell'app per tutte le piattaforme (desktop, mobile, PWA) si generano da `src-tauri/icons/app-icon.svg`: `npm run icons`.

## Avvio web

Richiede Node.js 22.12+ (o 20.19+).

```bash
npm ci
npm run dev
```

Aprire http://localhost:1420. Per una distribuzione web: `npm run build` e pubblicare il contenuto di `dist/` su un hosting statico. Nessun backend e nessun account richiesti. Il database è IndexedDB, legato all'origine e al profilo del browser. Nessun font remoto. La build produce anche il manifest e un service worker: la versione web si installa come app e si apre senza connessione, mentre le richieste verso altri indirizzi non passano mai dalla cache (vedi [src/serviceWorker.js](src/serviceWorker.js) e il plugin in [vite.config.ts](vite.config.ts)). Le sole connessioni esterne possibili partono da un'azione dell'utente: verso la Gemini API se attiva l'assistente, verso Google Drive o Microsoft Graph se collega un drive.

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
- [.github/workflows/release.yml](.github/workflows/release.yml): crea i pacchetti con `tauri-action`. Le release stabili includono **Linux** (deb, rpm, AppImage), **Windows** (msi, exe) e **macOS** (dmg e app *universal*, per Apple silicon e Intel); le pre-release alpha includono **Linux** (deb, AppImage), **Windows** (exe) e **macOS** (dmg), perché RPM e MSI non accettano questi identificativi SemVer. Parte con un tag `v*` oppure a mano dalla scheda Actions; i pacchetti vengono allegati a una release in bozza, da controllare prima della pubblicazione. I tag con suffisso, come `v0.3.0-alpha.2`, producono una pre-release. I pacchetti Windows non sono firmati: SmartScreen mostrerà un avviso. Quelli macOS vengono firmati e notarizzati se nel repository ci sono i secret `APPLE_CERTIFICATE` (certificato Developer ID in base64), `APPLE_CERTIFICATE_PASSWORD`, `APPLE_SIGNING_IDENTITY`, `APPLE_ID`, `APPLE_PASSWORD` (password per app) e `APPLE_TEAM_ID`; senza, sono costruiti non firmati e macOS li blocca al primo avvio.
- [.github/workflows/pages.yml](.github/workflows/pages.yml): a ogni push su `main` pubblica la versione web su GitHub Pages, all'indirizzo <https://dclfbk.github.io/ethicanvasaitool/>. I dati restano nel browser di chi la usa (IndexedDB): il sito non ha un server né un archivio condiviso.

### Aggiornamenti dell'app desktop

All'avvio l'app desktop controlla `https://github.com/dclfbk/ethicanvasaitool/releases/latest/download/latest.json`. Se trova una versione più recente chiede all'utente se installarla: scarica il pacchetto, ne verifica la firma, lo installa e riavvia. Il controllo si può disattivare in *Metodo e impostazioni*. La versione web non fa questo controllo: la aggiorna chi gestisce il server.

Il numero di versione va aggiornato in [package.json](package.json), [src-tauri/Cargo.toml](src-tauri/Cargo.toml), [src-tauri/tauri.conf.json](src-tauri/tauri.conf.json) e nei relativi lockfile e deve corrispondere al tag (`v0.3.0-alpha.2` → `0.3.0-alpha.2`). Solo le release stabili pubblicate, non le bozze né le pre-release, vengono proposte agli utenti dall’aggiornamento automatico.

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
- Rapporto Markdown e backup JSON completo di allegati. L'importazione crea copie senza sovrascrivere progetti esistenti (massimo 100 MB per pacchetto). Sono accettati i backup della versione 0.1 (formato 1), quelli con l'assistente (formato 2), quelli con la provenienza delle risposte precompilate dai casi a confronto (formato 3), quelli con versioni, revisioni e piano di riesame (formato 4), quelli con l’inquadramento normativo (formato 5) e quelli correnti (formato 6, che ammette voci della checklist con una nota ma ancora da esaminare). Le risposte a voci tolte da una versione successiva della checklist restano leggibili e vengono mostrate a parte.
- Estratti testuali per ogni riferimento, incollati a mano o estratti in locale dai file TXT, MD, CSV, JSON, DOCX, ODT e PDF (testo selezionabile, senza OCR).
- Assistente facoltativo con Gemini oppure con un modello locale (Ollama o server compatibile OpenAI): «Spiegami la domanda», «Aiutami a formulare una risposta», «Che cosa manca per rispondere?».
- Versioni, revisioni e riesame: istantanee della valutazione, revisioni con ruolo ed esito, confronto parola per parola, ripristino, data del riesame con file per il calendario.
- Checklist normativa di supporto, conclusioni e fonti dell’esperto, incluse in versioni ed esportazioni. È una bozza da verificare: non assegna categorie di rischio e non determina la conformità.
- Esportazione del canvas in PDF, nello stile grafico dell'AI Ethics Canvas originale oppure come poster A3 con pagine di dettaglio, e rapporto in DOCX o ODT, con la sintesi delle conclusioni proposta da Gemini se l'assistente è attivo.
- Archivio in una cartella sincronizzata (Google Drive, OneDrive, Nextcloud… tramite il loro programma) o direttamente su Google Drive e OneDrive di Microsoft 365: salvataggio e ripristino con unione dei progetti e cifratura facoltativa.
- Quadro dell'ente: stato dei progetti, riesami scaduti e vicini, contributi in attesa per figura, completamento per sezione, lacune ricorrenti, esportazione CSV.
- Contributi delle figure coinvolte senza server: richiesta e risposta come file, importazione in bozza.
- Casi a confronto: ricerca locale dei progetti simili nell'archivio e, con Gemini, analisi comparativa e precompilazione del questionario.

## Assistente Gemini (facoltativo)

### Che cosa serve

Una **chiave API Gemini**, creata in [Google AI Studio](https://aistudio.google.com/apikey). Un abbonamento all'app Gemini (ad esempio Google AI Pro) non dà accesso alle API. I termini Google consentono di rendere disponibile un client delle API a utenti nello Spazio economico europeo, in Svizzera e nel Regno Unito solo con servizi a pagamento, cioè con un progetto Google con fatturazione attiva.

### Configurazione

In **Metodo e impostazioni → Assistente Gemini**: attivare l'assistente, scegliere il modello, inserire la chiave e usare «Verifica connessione». «Elimina chiave e configurazione» cancella credenziale e preferenze.

- **Browser**: la chiave resta solo in memoria per la sessione e si perde chiudendo o ricaricando la pagina. Non viene scritta in localStorage, IndexedDB, URL, log o backup. La Gemini API accetta chiamate dirette dal browser (CORS), quindi non serve un servizio ponte. Google sconsiglia però di esporre chiavi lato client in produzione: la chiave è leggibile dagli script della pagina e dalle estensioni del browser. Conviene usare una chiave dedicata, con restrizioni e quota limitata. Per una distribuzione istituzionale è preferibile un proxy gestito dall'ente, che questa versione non include. Nella versione web il pannello dell'assistente mostra queste avvertenze prima dell'inserimento della chiave, insieme all'invito a non inviare dati personali o riservati. Non limitare la chiave per sito (referrer HTTP): le richieste partono senza referrer e verrebbero rifiutate; limitarla invece alla sola Gemini API e con una quota di spesa.
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

## Versioni, revisioni e riesame

Dentro un progetto, **Versioni e revisioni** apre la cronologia della valutazione.

- **Versioni**: istantanee immutabili di nome, descrizione, risposte, note e riferimenti. Gli allegati non vengono copiati: restano titolo, collegamento ed estratto. L'app salva una versione da sola prima di un ripristino e prima della precompilazione dai casi a confronto.
- **Revisioni**: ruolo (obbligatorio) e nome (facoltativo) di chi rivede, esito («Valutazione condivisa», «Condivisa con condizioni», «Da rivedere»), condizioni e osservazioni. Ogni revisione salva la versione a cui si riferisce. Non cambia completezza né stati: non è una validazione. Le risposte modificate dopo l'ultima revisione sono segnalate sotto la domanda.
- **Confronto** tra due versioni, o con lo stato attuale: per ogni domanda, parole aggiunte e tolte, cambi di stato, riferimenti aggiunti e tolti; per l’inquadramento normativo, risposte e note della checklist, conclusioni, fonti e registrazione dell’esperto.
- **Ripristino** di una versione, dopo aver salvato lo stato attuale.
- **Riesame**: data, responsabile, eventi che lo anticipano e note. Le schede dell'archivio segnalano i riesami vicini o scaduti; «Aggiungi al calendario» scarica un evento iCalendar (`.ics`).
- Il riepilogo mostra l'ultima revisione e il prossimo riesame. Il rapporto DOCX/ODT e il PDF poster riportano lo storico completo; il PDF in stile originale lo riassume in copertina.

## Esportazione del canvas e rapporto

Nella pagina **Vedi il risultato** di ogni progetto, il pannello **Esporta il progetto** genera i documenti sul dispositivo, senza servizi esterni.

- **Canvas in PDF · stile originale**: riproduce l'impaginazione dell'AI Ethics Canvas (A4 orizzontale), cioè versione, loghi, cerchio numerato, titolo in Barlow Condensed, filetto azzurro e piede con la licenza CC BY-SA. I post-it sono colorati secondo il profilo della domanda: giallo con triangolo per AI user, ciano con cerchio per AI engineer, verde con quadrato per AI scientist, sfumato quando la domanda riguarda più profili. Le pagine sono:
  - la copertina, con il riquadro «Caso d'uso», i dati del progetto, l'avanzamento per sezione e la legenda;
  - la pagina «Caso d'uso» e le cinque sezioni numerate come nell'originale, una domanda per riquadro, con la risposta nella casella a bordo scuro;
  - le pagine «Risposte complete» per le risposte che non entrano nei riquadri, rinviate con il numero di pagina;
  - una pagina «Note e riferimenti» con figure, contributi richiesti, note, riferimenti e testo proposto dall'AI.

  Colori e posizioni sono ricavati dal PDF originale. Le caselle sono le 20 domande del percorso, non i riquadri del canvas cartaceo, perché il questionario ne è un adattamento.
- **Canvas in PDF · poster e dettaglio**: la prima pagina è un poster A3 orizzontale con le sei sezioni del canvas a riquadri colorati e le risposte abbreviate. Lo spazio di ogni riquadro viene diviso tra le domande in base alla lunghezza delle risposte; le risposte troncate terminano con «(segue)». Le pagine A4 successive riportano ogni risposta per intero, con stato, origine nel canvas, figure da coinvolgere, contributi richiesti, note, riferimenti e provenienza AI, poi i punti da completare e l'inquadramento normativo. Entrambi i formati usano `pdf-lib` e incorporano Fira Sans e Barlow Condensed (OFL), ridotti ai caratteri latini. La libreria e i font si caricano solo al momento dell'esportazione.
- **Rapporto DOCX o ODT**: contiene descrizione del progetto, stato della compilazione calcolato dall'applicazione (completezza, punti essenziali mancanti, contributi in attesa) e inquadramento normativo con un testo fisso. In allegato, se richiesto, ci sono tutte le risposte. I file sono scritti direttamente in Office Open XML e OpenDocument, senza librerie, e si aprono in Word e LibreOffice.
- **Sintesi delle conclusioni con Gemini** (solo con l'assistente attivo): Gemini riceve descrizione, stato calcolato, risposte e contributi richiesti. Su scelta dell'utente riceve anche note ed estratti dei riferimenti, sempre con l'anteprima esatta del testo; i nomi delle persone indicate non vengono mai inviati. Restituisce sintesi, conclusioni per sezione, punti di forza, rischi, azioni con il ruolo responsabile, informazioni mancanti e figure da coinvolgere.
  - Le citazioni vengono controllate: ogni voce riporta le proprie fonti, e una fonte non inviata viene segnalata «da verificare».
  - Le frasi che sembrano una classificazione AI Act o una dichiarazione di conformità vengono rimosse e mostrate all'utente.
  - La sintesi è una bozza modificabile: entra nel rapporto solo all'esportazione, con l'indicazione di modello e data. Non viene salvata nel progetto e non cambia completezza o stati.

## Quadro dell'ente

**Quadro dell'ente** nel menu riassume i progetti dell'archivio, esclusi gli esempi dimostrativi salvo scelta contraria. Tutto è calcolato sul dispositivo ([src/dashboard.ts](src/dashboard.ts)) e non cambia i progetti.

- Contatori: progetti per stato, completezza media, contributi in attesa, riesami scaduti ed entro 30 giorni.
- Riesami scaduti, vicini e non pianificati, ed esito dell'ultima revisione di ogni progetto.
- Contributi in attesa in tutti i progetti, dal più vecchio, filtrabili per figura.
- Completamento medio per sezione e lacune ricorrenti per domanda, in barre orizzontali con il valore scritto accanto. Una lacuna apre il primo progetto in cui manca, su quella domanda.
- **Esporta i progetti in CSV**: separatore punto e virgola e UTF-8 con BOM, per Excel e LibreOffice in italiano. Le celle che un foglio di calcolo interpreterebbe come formula sono precedute da un apostrofo.

## Contributi delle figure coinvolte

Dentro un progetto, **Contributi** prepara una richiesta per una figura da coinvolgere (DPO, fornitore, esperto) e importa la sua risposta. Lo scambio avviene con due file JSON, eventualmente cifrati con una passphrase (AES-256-GCM, come l'archivio), da mandare per email: non serve un server.

- La richiesta contiene le domande scelte (di base quelle in attesa per quella figura), il contributo richiesto, la descrizione del progetto e le risposte già scritte, se selezionate. Note, allegati e figure indicate non vengono inclusi. L'app propone il testo dell'email.
- Chi riceve apre l'app web, preme **Rispondi a una richiesta** e risponde senza un archivio: nulla resta salvato sul suo dispositivo. La risposta usa la stessa passphrase.
- All'importazione l'app controlla che la risposta riguardi il progetto aperto e salva prima una versione. Per ogni domanda chi coordina sceglie se usare la proposta, aggiungendola in coda o sostituendo la risposta. Le risposte accettate tornano in bozza, e nelle note resta chi ha dato il contributo e quando.
- Le richieste valgono per la stessa versione del questionario.

## Archivio sul drive personale

In **Metodo e impostazioni → Archivio sul tuo drive** si salva una copia dell'archivio, allegati compresi, e la si ripristina su questo o su un altro dispositivo. Non c'è sincronizzazione automatica: il salvataggio e il ripristino partono solo da un clic.

- **Cartella sincronizzata** (predefinita, senza configurazione): una cartella che un programma di sincronizzazione già presente sul computer tiene aggiornata. L'app scrive solo il file `ethicanvas-archivio.json`.
  - Sul desktop la cartella si sceglie con la finestra nativa (`tauri-plugin-dialog`) ed è ricordata dal processo Rust ([src-tauri/src/folder.rs](src-tauri/src/folder.rs)). L'interfaccia non passa mai un percorso, e la scrittura usa un file temporaneo poi rinominato.
  - Nel browser usa la File System Access API (Chrome, Edge), con il riferimento alla cartella in IndexedDB.
- **Google Drive o OneDrive con l'account**: disponibili solo se l'installazione è configurata con gli identificativi OAuth descritti sotto.

Per entrambe le modalità valgono le regole seguenti.

- **Dove finiscono i file**: un file `ethicanvas-archivio.json` nella cartella «EthiCanvas AI» del drive dell'utente. Su Google l'app usa l'ambito `drive.file` e vede solo i file che ha creato; su Microsoft usa `Files.ReadWrite` e `User.Read`, con soli account di lavoro o della PA (tenant `organizations`, oppure il tenant dell'ente).
- **Ripristino**: unisce i progetti uno per uno e non elimina nulla. I progetti nuovi vengono aggiunti, quelli cambiati solo sul drive aggiornati. Se un progetto è cambiato in entrambi i posti dall'ultimo salvataggio da questo dispositivo, si conservano tutte e due le versioni, una con il nome «· copia … in conflitto». I progetti eliminati in locale ricompaiono se sono ancora nel backup.
- **Salvataggio**: sostituisce il file sul drive. Se nel frattempo un altro dispositivo lo ha aggiornato, l'app chiede conferma e suggerisce di ripristinare prima. Google Drive e OneDrive conservano le versioni precedenti del file.
- **Cifratura facoltativa**, attiva per impostazione predefinita: AES-256-GCM con chiave derivata dalla passphrase (PBKDF2-SHA-256, 600.000 iterazioni), sul dispositivo e prima del caricamento. La passphrase (almeno 12 caratteri) non viene salvata e non è recuperabile. Senza cifratura il fornitore del drive può leggere i contenuti: va verificato che le regole dell'ente lo consentano.
- **Browser**: accesso in una finestra popup che ritorna a `oauth-callback.html`, una pagina statica senza script. Il token resta solo in memoria e si perde chiudendo o ricaricando la pagina; non si conservano refresh token. Google usa il flusso a token, Microsoft il codice di autorizzazione con PKCE.
- **Desktop Tauri**: accesso nel browser di sistema con PKCE e redirect su `127.0.0.1`/`localhost` a porta casuale. Il refresh token è salvato nel portachiavi del sistema; le chiamate HTTP partono dal processo Rust ([src-tauri/src/cloud.rs](src-tauri/src/cloud.rs)), che aggiunge il token e accetta solo gli endpoint di Drive e Graph. Il token non torna mai all'interfaccia. «Scollega» elimina la credenziale e, per Google, la revoca.

### Configurazione per chi distribuisce l'app

Ogni installazione usa le proprie registrazioni OAuth. Gli identificativi sono pubblici e vengono inseriti al momento della build; un servizio senza identificativo appare come non disponibile.

| Variabile | Dove | Registrazione |
|---|---|---|
| `VITE_GOOGLE_CLIENT_ID` | build web | Google Cloud, client OAuth «Applicazione web». URI di reindirizzamento: `https://<sito>/oauth-callback.html`; origine JavaScript: `https://<sito>`. Abilitare la Google Drive API. |
| `VITE_MS_CLIENT_ID`, `VITE_MS_TENANT` | build web | Microsoft Entra, piattaforma «Applicazione a pagina singola» con URI `https://<sito>/oauth-callback.html`. Permessi delegati `User.Read` e `Files.ReadWrite`. Tenant facoltativo, predefinito `organizations`. |
| `CANVAS_GOOGLE_DESKTOP_CLIENT_ID`, `CANVAS_GOOGLE_DESKTOP_CLIENT_SECRET` | build desktop | Google Cloud, client OAuth «App desktop». Google considera il secret di un'app installata non riservato, ma resta fuori dal codice sorgente. |
| `CANVAS_MS_CLIENT_ID`, `CANVAS_MS_TENANT` | build desktop | La stessa app Entra, con in più la piattaforma «Applicazioni per dispositivi mobili e desktop» e l'URI `http://localhost`. |

Nei workflow GitHub i valori vengono dalle variabili del repository `GOOGLE_WEB_CLIENT_ID`, `GOOGLE_DESKTOP_CLIENT_ID`, `MS_CLIENT_ID`, `MS_TENANT` e dal secret `GOOGLE_DESKTOP_CLIENT_SECRET`. Finché l'app Google è in modalità «test», possono accedere solo gli utenti di prova indicati nella console. In alcuni tenant Microsoft serve l'approvazione dell'amministratore, e l'app lo segnala.

## Modello locale

In **Metodo e impostazioni → Assistente AI** si può scegliere **Modello locale** al posto di Gemini: un server Ollama oppure compatibile con l'API di OpenAI (llama.cpp, vLLM, LM Studio), sul computer o dell'ente. Tutte le funzioni dell'assistente lo usano allo stesso modo, e i testi indicano sempre dove vanno i dati.

- Fornitore in [src/llm/local.ts](src/llm/local.ts): Ollama con `/api/chat` e lo schema JSON in `format` (finestra di contesto `num_ctx` dimensionata sulla richiesta, al massimo 32.768); OpenAI con `/v1/chat/completions` e `response_format` di tipo `json_schema`. Sono raggiungibili solo l'elenco dei modelli e la chat; l'indirizzo è http o https, senza credenziali, parametri o frammenti.
- Desktop: [src-tauri/src/local_llm.rs](src-tauri/src/local_llm.rs) conserva indirizzo e protocollo nella configurazione dell'app e la chiave facoltativa nel portachiavi, che viene cancellata se l'indirizzo cambia. Le richieste sono annullabili e hanno un limite di 5 minuti.
- Browser: richieste dirette, chiave solo in memoria. Il server deve consentire l'origine del sito, per Ollama con `OLLAMA_ORIGINS`.

## Casi a confronto

La sezione **Casi a confronto** usa come knowledge base i progetti dell'archivio locale, compresi quelli importati da un archivio condiviso dall'ente. Gli esempi dimostrativi e i progetti senza risposte non ne fanno parte.

- Lo scenario può essere un nuovo progetto descritto a parole oppure un progetto già presente nell'archivio. Si può aggiungere una domanda specifica.
- I casi più simili vengono ordinati sul dispositivo con una similarità testuale (TF-IDF) su nome, descrizione e risposte, mostrando le parole in comune. Questa parte funziona senza assistente e non invia nulla.
- Con l'assistente attivo l'utente sceglie quali casi inviare, con l'anteprima esatta del testo. Per ogni caso vengono inviati nome, ente, descrizione e risposte con il loro stato. Non vengono inviati allegati, estratti dei riferimenti, note, persone indicate e richieste di contributo.
- Gemini restituisce una sintesi, somiglianze, differenze e soluzioni da valutare per ogni caso, raccomandazioni, informazioni mancanti e figure da coinvolgere. Le citazioni vengono controllate in locale: una fonte non inviata viene segnalata come da verificare.
- Le risposte proposte sono modificabili e si applicano solo dopo una conferma. Partendo da uno scenario nuovo viene creato un nuovo progetto; partendo da un progetto esistente vengono riempite solo le risposte vuote, anche se nel frattempo ne sono state scritte altre. Ogni risposta entra come «bozza», con la provenienza (azione `prefill`) e i nomi dei casi citati.
- La domanda sull'inquadramento normativo non viene mai precompilata, così l'assistente non può proporre categorie AI Act.

## Limiti espliciti

È un MVP, non una implementazione integrale di ogni quesito del PDF. Le domande sono adattamenti tematici con origine indicata. Sono incluse estensioni su chatbot, inclusione e riesame. I ruoli sono suggerimenti, non la riproduzione delle attribuzioni grafiche originali.

Non sono implementati classificazione normativa AI Act, revisione formale, riconoscimento del testo (OCR) da immagini e PDF scansionati, estrazione dai vecchi formati Word (.doc), sincronizzazione automatica tra dispositivi, crittografia dell'archivio locale, integrazione Microsoft Copilot o un proxy per la chiave Gemini. Il rapporto non costituisce una certificazione. La presenza di evidenze non equivale a verifica della loro validità.

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
- `src/history.ts`, `src/historyText.ts`, `src/HistoryPanel.tsx`: versioni, revisioni, confronto, ripristino e riesame.
- `docs/guida/`, `src/GuidePanel.tsx`, `src/guideMarkdown.ts`: guida all'uso in Markdown e sua visualizzazione nell'app.
- `src/canvasOriginalPdf.ts`: canvas in PDF nello stile dell'AI Ethics Canvas originale.
- `src/canvasPdf.ts`: canvas in PDF come poster A3 con pagine di dettaglio.
- `src/pdfText.ts`, `src/pdfAssets.ts`: testo, a capo e troncamento condivisi; font e loghi incorporati nei PDF.
- `src/report.ts`, `src/documents.ts`, `src/zip.ts`, `src/ReportExportPanel.tsx`: rapporto, sintesi con Gemini, scrittura di DOCX e ODT.
- `src/dashboard.ts`, `src/DashboardPanel.tsx`: quadro dell'ente ed esportazione CSV.
- `src/contributions.ts`, `src/contributionFiles.ts`, `src/ContributionsPanel.tsx`, `src/RespondPanel.tsx`: contributi delle figure coinvolte.
- `src/cloud/folder.ts`, `src-tauri/src/folder.rs`: cartella sincronizzata nel browser e sul desktop.
- `src/cloud/`: archivio sul drive (protocolli Google Drive e Microsoft Graph, accesso nel browser e su desktop, cifratura, unione al ripristino); `src/CloudSyncPanel.tsx` è l'interfaccia.
- `src-tauri/src/cloud.rs`: accesso OAuth con redirect locale, portachiavi e chiamate HTTP lato desktop.
- `src/knowledge.ts`, `src/KnowledgePanel.tsx`: ricerca locale dei casi simili, analisi comparativa e precompilazione.
- `src/textExtraction.ts`, `src/unzip.ts`, `src/pdfExtraction.ts`: estrazione locale del testo da file di testo, DOCX e ODT (lettore ZIP senza librerie) e PDF (PDF.js, caricato solo al momento dell'uso).
- `src-tauri/src/gemini.rs`: portachiavi e chiamate HTTP lato desktop.
- `src-tauri/`: configurazione e ingresso Rust.

## Passi successivi concordati

1. Validare e completare il questionario rispetto al canvas, provare con funzionari e tecnici; definire criteri di revisione e approfondimento AI Act.
2. ~~Gemini~~: implementato in questa versione.
3. ~~Ollama~~: implementato come modello locale (Ollama o server compatibile OpenAI).
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

I test non richiedono credenziali reali. I test unitari usano un fornitore e un trasporto simulati. `scripts/smoke-assistant.mjs` intercetta in Playwright tutte le chiamate verso `generativelanguage.googleapis.com` e risponde con un server simulato. Così verifica: assistente disattivato per impostazione predefinita, chiave di sessione, test di connessione, estrazione locale, anteprima del contesto, annullamento, errori di quota e chiave non valida, invio singolo con doppio clic, nessuna sovrascrittura, bozza con provenienza, risultato tardivo scartato, assenza della chiave da archivio, backup e console. `scripts/smoke-knowledge.mjs` fa lo stesso per i casi a confronto: ordinamento locale senza assistente, invio dei soli casi selezionati senza note e nomi di persone, domanda normativa mai precompilata, fonti non riconosciute segnalate, nuovo progetto con bozze e provenienza, risposte esistenti intatte. `scripts/smoke-cloud.mjs` simula accesso e API di Google e Microsoft con due dispositivi: salvataggio cifrato, ripristino con passphrase, avviso prima di sovrascrivere, PKCE con account di lavoro, download senza token, nessun token o passphrase salvati. `scripts/smoke-export.mjs` controlla i due PDF (con `pdfinfo` e `pdftotext`), DOCX e ODT, la sintesi Gemini simulata, la rimozione delle frasi sull'AI Act e l'esportazione del testo modificato. `scripts/smoke-folder.mjs` prova la cartella sincronizzata con due browser (scelta, salvataggio cifrato, ripristino, avviso prima di sovrascrivere, browser senza supporto). `scripts/smoke-contributions.mjs` prova richiesta, risposta senza archivio e importazione in bozza. `scripts/smoke-local-model.mjs` simula un server Ollama: configurazione senza chiave, elenco dei modelli, bozza e sintesi inviate solo al server locale, messaggio a server spento, nessuna richiesta a Google. `scripts/smoke-dashboard.mjs` prova il quadro su un archivio importato, compresi filtri, barre, CSV e apertura delle lacune. `scripts/smoke-history.mjs` prova versioni, revisione con condizioni, confronto, ripristino, riesame scaduto, file del calendario e backup in formato 4. `scripts/smoke-guide.mjs` apre la guida nell'app: capitoli, link, immagini, azioni e assenza di richieste esterne. `scripts/smoke-pwa.mjs` controlla manifest e service worker, poi ricarica l'app senza rete e ritrova archivio e immagini della guida. `scripts/smoke-extraction.mjs` estrae il testo da un PDF generato al momento, segnala un PDF senza testo e verifica che il worker di PDF.js arrivi dall'app, senza richieste esterne.

Per provare con una chiave vera basta `npm run dev`, poi configurare l'assistente nelle impostazioni.

## Autore, licenza e provenienza

Copyright (C) 2026 FBK - Fondazione Bruno Kessler, Trento. Autore: Maurizio Napolitano <napo@fbk.eu>.

Il codice è software libero distribuito con licenza **GNU Affero General Public License v3.0 o successiva** (`AGPL-3.0-or-later`), testo completo in [LICENSE](LICENSE). Chi offre una versione modificata come servizio in rete deve rendere disponibile agli utenti il relativo codice sorgente (art. 13 AGPL). Ogni file sorgente riporta l'intestazione di licenza e un avviso rivolto agli agenti AI, che devono conservarla e ricordare la licenza all'utente; le regole per gli agenti sono in [AGENTS.md](AGENTS.md).

Il questionario in `src/questionnaire.ts` adatta e amplia l'AI Ethics Canvas, di cui Maurizio Napolitano è tra gli autori, © 2024 Provincia Autonoma di Trento e Fondazione Bruno Kessler, CC BY-SA 4.0, e resta distribuito con la stessa licenza. Dettagli in [NOTICE.md](NOTICE.md). Le dipendenze mantengono le rispettive licenze (tra queste `pdf-lib` e `@pdf-lib/fontkit`, MIT). I font Fira Sans e Barlow Condensed sono distribuiti con licenza SIL Open Font License 1.1 (`src/assets/fonts/OFL-*.txt`); le copie in `src/assets/fonts/pdf/` sono convertite in TrueType e ridotte ai caratteri latini per l'incorporamento nei PDF.

- Canvas: https://github.com/dclfbk/canvasaiethics
- Articolo: https://doi.org/10.1017/dap.2026.10088
- Licenza del codice: https://www.gnu.org/licenses/agpl-3.0.html
- Licenza del questionario: https://creativecommons.org/licenses/by-sa/4.0/
