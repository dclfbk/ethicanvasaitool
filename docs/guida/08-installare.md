# Installare e configurare

Questo capitolo è per chi installa l’applicazione per l’ente o la gestisce: servizi informatici, responsabili della sicurezza, chi prepara le build.

## Versione web o desktop

| | Web | Desktop |
|---|---|---|
| Dove sono i dati | nel browser (IndexedDB), legati al sito e al profilo | in un database SQLite locale |
| Chiave Gemini | solo in memoria, per la scheda aperta | nel portachiavi del sistema operativo |
| Accesso al drive | token in memoria, da ripetere a ogni sessione | refresh token nel portachiavi |
| Chiamate esterne | dal browser, direttamente ai servizi | dal processo Rust, verso endpoint fissi |
| Aggiornamenti | li pubblica chi gestisce il sito | automatici, con firma verificata |

Per dati riservati o per usare l’assistente, la versione desktop è più protetta: chiavi e token non passano dall’interfaccia.

## Versione web

La versione web è un sito statico, senza server applicativo né account.

```bash
npm ci
npm run build
```

Pubblica il contenuto di `dist/` su qualunque hosting statico. Il workflow `.github/workflows/pages.yml` lo fa su GitHub Pages a ogni push su `main`.

Serve Node.js 22.12 o successivo (oppure 20.19 o successivo). Se `npm install` fallisce con l’errore `Cannot read properties of null (reading 'edgesOut')`, usa un npm recente: `npx npm@11 install`.

> **Licenza AGPL.** Se l’ente pubblica una versione modificata come servizio in rete, deve rendere disponibile agli utenti il codice sorgente corrispondente (art. 13 AGPL).

## Versione desktop

Installa Rust e i prerequisiti di Tauri per il sistema (su Linux servono anche `libwebkit2gtk-4.1-dev`, `librsvg2-dev` e `libdbus-1-dev`), poi:

```bash
npm ci
npm run tauri build
```

Il workflow `.github/workflows/release.yml` produce i pacchetti Linux (deb, rpm, AppImage) e Windows (msi, exe) a ogni tag `v*`, in una release in bozza da controllare. I pacchetti non sono firmati con un certificato di codice: Windows SmartScreen mostra un avviso.

### Aggiornamenti

All’avvio l’app desktop controlla se su GitHub c’è una release più recente e chiede all’utente se installarla. Il pacchetto è verificato con una firma: se non corrisponde, l’installazione viene bloccata. L’utente può disattivare il controllo con «Controlla gli aggiornamenti all’avvio» o lanciarlo con «Controlla ora».

Per pubblicare aggiornamenti servono una coppia di chiavi di firma e i secret `TAURI_SIGNING_PRIVATE_KEY` e `TAURI_SIGNING_PRIVATE_KEY_PASSWORD` nel repository. Conserva la chiave privata fuori dal repository: se si perde, le installazioni esistenti non potranno più aggiornarsi. La procedura completa è nel README.

## L’assistente Gemini

Ogni utente inserisce una chiave API Gemini creata in Google AI Studio. Per l’ente:

- crea chiavi dedicate a questa applicazione, limitate alla Gemini API e con una quota di spesa bassa;
- per utenti nello Spazio economico europeo i termini Google richiedono servizi a pagamento, cioè un progetto con fatturazione attiva;
- non limitare la chiave per sito (referrer HTTP): le richieste partono senza referrer e verrebbero rifiutate;
- nella versione web la chiave è leggibile dagli script della pagina e dalle estensioni del browser. Per una distribuzione istituzionale è preferibile la versione desktop, o un proxy gestito dall’ente, che questa versione non include.

Le richieste usano l’opzione che disattiva la conservazione delle interazioni sui server Google. Restano validi i termini del servizio.

## L’archivio sul drive

Per abilitare Google Drive e OneDrive l’ente registra l’applicazione presso i due servizi e inserisce gli identificativi al momento della build. Sono identificativi pubblici, non segreti; un servizio senza identificativo appare come non configurato.

| Variabile | Build | Registrazione |
|---|---|---|
| `VITE_GOOGLE_CLIENT_ID` | web | Google Cloud, client OAuth **Applicazione web**, con reindirizzamento a `https://<sito>/oauth-callback.html` e la Google Drive API abilitata |
| `VITE_MS_CLIENT_ID`, `VITE_MS_TENANT` | web | Microsoft Entra, piattaforma **Applicazione a pagina singola** con lo stesso reindirizzamento; permessi `User.Read` e `Files.ReadWrite` |
| `CANVAS_GOOGLE_DESKTOP_CLIENT_ID`, `CANVAS_GOOGLE_DESKTOP_CLIENT_SECRET` | desktop | Google Cloud, client OAuth **App desktop** |
| `CANVAS_MS_CLIENT_ID`, `CANVAS_MS_TENANT` | desktop | la stessa app Entra, con la piattaforma per dispositivi mobili e desktop e l’indirizzo `http://localhost` |

Il tenant Microsoft predefinito è `organizations`, cioè solo account di lavoro o della PA; indica il tenant dell’ente per limitarlo ai suoi utenti. In alcuni tenant serve l’approvazione dell’amministratore. Su Google, finché l’app è in modalità di test, possono accedere solo gli utenti di prova indicati nella console.

Nei workflow GitHub i valori vengono dalle variabili del repository `GOOGLE_WEB_CLIENT_ID`, `GOOGLE_DESKTOP_CLIENT_ID`, `MS_CLIENT_ID`, `MS_TENANT` e dal secret `GOOGLE_DESKTOP_CLIENT_SECRET`.

## Sicurezza e dati in sintesi

- **Nessun server applicativo.** I dati restano sul dispositivo. Le uniche connessioni esterne sono verso la Gemini API, se l’utente attiva l’assistente; verso Google Drive o Microsoft Graph, se collega un drive; verso GitHub, per il controllo degli aggiornamenti dell’app desktop.
- **Endpoint fissi.** Le chiamate verso Gemini, Drive e Graph sono limitate a un elenco fisso di indirizzi, controllato sia nell’interfaccia sia, nel desktop, dal processo Rust.
- **Niente segreti in chiaro.** Chiavi, token e passphrase non vengono mai scritti in archivio, backup, URL o log. La passphrase dell’archivio cifrato non viene salvata da nessuna parte.
- **Cifratura dell’archivio sul drive**: AES-256-GCM con chiave derivata dalla passphrase (PBKDF2-SHA-256, 600.000 iterazioni), sul dispositivo e prima del caricamento.
- **Output dell’assistente.** È sempre una bozza: non sovrascrive le risposte, non cambia la completezza e non produce classificazioni AI Act.

I dettagli tecnici sono nel README del repository.
