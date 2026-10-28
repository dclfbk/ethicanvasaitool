# Salvare e spostare i dati

L’app salva tutto sul dispositivo, ma una copia di sicurezza la fai tu: con un file da conservare, in una cartella sincronizzata oppure sul tuo drive personale.

## Una copia in un file

1. Premi «Esporta una copia» nella barra laterale, oppure «Esporta archivio» in «Metodo e impostazioni».
2. Ottieni un file JSON con tutti i progetti, le risposte e gli allegati.
3. Conservalo dove l’ente conserva i documenti di lavoro.

Per riportare i progetti, nell’archivio premi «Importa archivio» e scegli il file. L’importazione crea copie con il nome «· importato» e non sovrascrive i progetti già presenti.

> **Nel browser.** I dati del browser possono essere cancellati dalla pulizia della cronologia o dalle regole del sistema. In «Metodo e impostazioni» puoi premere «Richiedi persistenza browser» per chiedere al browser di non eliminarli, ma la copia in un file resta la protezione migliore.

## In una cartella sincronizzata

È il modo più semplice per avere una copia sempre aggiornata, anche su un altro computer, senza account aggiuntivi né registrazioni. Usa una cartella che il computer tiene già sincronizzata con il programma di Google Drive, OneDrive, Nextcloud o simili.

In «Metodo e impostazioni», nel riquadro «Archivio sul tuo drive»:

![L’archivio in una cartella sincronizzata](img/drive.png)

1. Premi «Scegli la cartella» e seleziona la cartella sincronizzata.
2. Lascia spuntato «Cifra l’archivio con una passphrase prima di caricarlo» e scrivi una passphrase di almeno 12 caratteri, poi ripetila in «Ripeti la passphrase».
3. Premi «Salva nella cartella». L’app scrive un solo file, «ethicanvas-archivio.json»: a copiarlo sugli altri computer pensa il programma di sincronizzazione.

[Vai alle impostazioni](#app-settings)

> **Non perdere la passphrase.** Non viene salvata da nessuna parte e non si può recuperare: senza di essa il backup è illeggibile. Con la cifratura, il servizio di sincronizzazione riceve solo dati cifrati.

Nell’app desktop la cartella resta ricordata. Nel browser funziona con Chrome ed Edge, che possono chiederti di nuovo il permesso a ogni sessione; con altri browser usa la copia in un file. Con «Cambia cartella» ne scegli un’altra, con «Dimentica la cartella» l’app smette di usarla: il file resta dove si trova.

## Su Google Drive o OneDrive, con l’account

Se chi ha installato l’app ha registrato i servizi, nel riquadro compaiono anche le schede di Google Drive e di OneDrive di Microsoft 365. In questo caso l’app usa direttamente il tuo account, senza un programma di sincronizzazione sul computer.

1. Scegli il servizio e premi il pulsante **Collega**. Si apre la pagina di accesso di Google o Microsoft: entra con il tuo account e concedi il permesso.
2. Imposta la cifratura come sopra e premi «Salva sul drive». Il file finisce nella cartella «EthiCanvas AI» del tuo drive.

Per registrare i servizi vedi [Installare e configurare](09-installare.md).

## Ripristinare

1. Sull’altro computer scegli la stessa cartella sincronizzata, o collega lo stesso account.
2. Premi «Ripristina dalla cartella» o «Ripristina dal drive». Se l’archivio è cifrato, inserisci la passphrase e premi «Decifra e ripristina».

Il ripristino unisce i progetti e non cancella nulla:

- i progetti nuovi vengono aggiunti;
- quelli cambiati solo nella copia salvata vengono aggiornati;
- se un progetto è cambiato in entrambi i posti, vengono conservate tutte e due le versioni: la più recente con il nome originale, l’altra con l’aggiunta «copia locale in conflitto» o «copia dal drive in conflitto». Confrontale e tieni quella giusta.

Prima di sovrascrivere un archivio salvato da un altro dispositivo, l’app ti chiede conferma e ti suggerisce di ripristinarlo prima.

## Accesso e disconnessione

Con un account, nel browser l’accesso vale solo per la scheda aperta e ricaricando la pagina va ripetuto; nell’app desktop resta nel portachiavi del sistema. Con «Scollega» lo elimini.

Hai finito la parte per chi compila. Se coordini il lavoro di un gruppo, prosegui con [Versioni, revisioni e riesame](06-versioni.md).
