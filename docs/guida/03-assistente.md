# Farsi aiutare dall’assistente

L’assistente usa un modello di intelligenza artificiale per spiegare le domande e preparare bozze. Puoi scegliere tra Gemini, il servizio di Google, e un **modello locale**, che gira sul tuo computer o su un server dell’ente. È facoltativo: il percorso, le regole di completezza e i risultati funzionano allo stesso modo anche senza.

## Prima di cominciare

Ti serve una **chiave API Gemini**, creata in Google AI Studio. Un abbonamento all’app Gemini non basta. Se la chiave te la fornisce l’ente, chiedi a chi l’ha configurata; i dettagli sono nel capitolo [Installare e configurare](09-installare.md).

## Attivare l’assistente

1. Apri «Metodo e impostazioni» e scorri fino a «Assistente AI (facoltativo)».
2. Spunta «Attiva l’assistente durante la compilazione», lascia selezionato «Gemini (Google)» e scegli il modello. Quello consigliato va bene nella maggior parte dei casi.
3. Incolla la chiave in «Chiave API Gemini» e premi «Usa per questa sessione» (versione web) o «Salva nel portachiavi» (versione desktop).
4. Premi «Verifica connessione».

[Vai alle impostazioni](#app-settings)

Nella versione web l’app tiene la chiave solo nella scheda aperta e la dimentica quando ricarichi la pagina. Per non doverla incollare ogni volta, puoi farla ricordare al gestore delle password del browser: dopo «Usa per questa sessione» il browser ti propone di salvarla. La volta successiva premi «Usa la chiave salvata nel browser» (in Chrome, Edge e negli altri browser basati su Chromium) oppure scegli la chiave dal suggerimento che compare nel campo. Fallo solo sul tuo computer personale, mai su un computer condiviso. Nella versione desktop la chiave è conservata nel portachiavi del sistema operativo. Con «Elimina chiave e configurazione» cancelli chiave e preferenze.

## Usare un modello locale

Con un modello locale i dati non escono verso servizi esterni: vanno solo al server che indichi. È la scelta adatta per informazioni riservate, se il server è sotto il controllo dell’ente.

1. Installa un server di modelli, ad esempio Ollama, sul tuo computer, oppure chiedi all’ente l’indirizzo del suo server.
2. In «Assistente AI (facoltativo)» scegli «Modello locale».
3. In «Tipo di server» scegli Ollama, oppure «Compatibile OpenAI» per llama.cpp, vLLM, LM Studio e server simili.
4. Scrivi l’«Indirizzo del server», ad esempio `http://localhost:11434` per Ollama sul tuo computer, e premi «Salva l’indirizzo».
5. Premi «Aggiorna l’elenco dei modelli» e scegli il «Modello».
6. Se il server richiede una chiave, inseriscila in «Chiave di accesso (facoltativa)».
7. Premi «Verifica connessione».

Nell’app desktop funziona subito. Nel browser il server deve accettare le richieste dalla pagina dell’app: per Ollama va avviato con la variabile `OLLAMA_ORIGINS` che include l’indirizzo del sito, e il browser può chiedere il permesso di accedere alla rete locale.

> **Modelli piccoli.** Le risposte devono seguire un formato preciso. Alcuni modelli piccoli non ci riescono e l’app lo segnala: in quel caso usa un modello più grande.

I pulsanti e gli avvisi indicano sempre dove vanno i dati: con Gemini «Invia a Gemini», con un modello locale «Invia al modello locale» e l’indirizzo del server.

## Le tre azioni

Con l’assistente attivo, sotto ogni domanda compaiono tre pulsanti:

- «Spiegami la domanda»: che cosa chiede, perché conta per il tuo progetto, quali informazioni servono;
- «Aiutami a formulare una risposta»: una bozza basata solo su ciò che invii;
- «Che cosa manca per rispondere?»: le informazioni mancanti e chi può fornirle.

## Scegliere che cosa inviare

Prima di ogni richiesta decidi tu che cosa parte. La domanda corrente è sempre inclusa; il resto lo spunti uno per uno: la descrizione del progetto, la risposta già scritta, le risposte precedenti, gli estratti dei riferimenti.

![La scelta del contesto da inviare](img/assistente.png)

Apri «Anteprima esatta del testo inviato» per leggere parola per parola che cosa riceverà Google. Poi premi «Invia a Gemini». Se ci ripensi, «Annulla richiesta» la interrompe.

Durante l’attesa l’app mostra il modello a cui è andata la richiesta, da quanti secondi aspetta e l’attesa massima: 90 secondi per Gemini, 5 minuti per un modello locale. Una risposta può richiedere anche più di un minuto, soprattutto nelle ore di traffico. Se succede spesso, scegli un modello più veloce in «Metodo e impostazioni».

> **Attenzione.** Non inviare dati personali, dati riservati o documenti interni dell’ente. Gli allegati non vengono mai inviati come file e i collegamenti non vengono aperti.

## Usare la proposta

La risposta di Gemini è una proposta da verificare. Ogni affermazione è etichettata:

- **dichiarato**: viene da ciò che hai scritto tu;
- **documentato**: viene dall’estratto di un riferimento;
- **ipotesi da confermare**: non ha una fonte tra quelle inviate.

Se una citazione non corrisponde a ciò che hai inviato, l’app la segnala con «citazione non corrispondente al contesto inviato».

Puoi modificare il testo nella «Proposta modificabile» e poi:

- «Inserisci come bozza», se la risposta è vuota;
- «Aggiungi in coda come bozza», per tenere ciò che hai già scritto;
- «Sostituisci la risposta…», che chiede una conferma esplicita.

In ogni caso la risposta torna allo stato di bozza e conserva l’indicazione del modello e della data. Spetta a te confermarla.

> **Sull’AI Act.** Per la domanda sull’inquadramento normativo l’assistente organizza le informazioni e i punti da verificare, ma non indica categorie di rischio né conformità: quella valutazione va fatta con un esperto legale.

Prosegui con [Risultato ed esportazioni](04-risultato.md).
