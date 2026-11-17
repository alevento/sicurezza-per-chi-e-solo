# Lezione 6 — La truffa del bonifico e le altre storie ben raccontate

*Sicurezza informatica per chi è solo (o quasi) · Alessandro Manneschi · 17 novembre 2026*

> Qui non c'è nessun virus. C'è qualcuno che racconta una storia credibile alla persona giusta. È l'attacco che costa di più, e l'informatico da solo non lo ferma. Ma qualcosa da bloccare c'è, e un caso di questo mese lo mostra.

## Pretexting

Il **pretexting** è **costruire un pretesto/scenario credibile** per ottenere fiducia
e informazioni. *Esempio:* "Sono il tecnico dell'assistenza, mi confermi la sua
password per verificare l'account?". La forza non è tecnica: è la **storia** costruita
attorno alla richiesta.

## Attacchi fisici: baiting e tailgating

- **Baiting:** lasciare un'**esca** (chiavetta USB "Stipendi 2026" nel parcheggio)
  sperando che qualcuno la inserisca → esegue malware.
- **Tailgating** (o *piggybacking*): **seguire fisicamente** una persona autorizzata
  per **entrare in un'area riservata** (dietro qualcuno che apre la porta col badge,
  magari con le mani occupate da scatoloni).

## BEC — Business Email Compromise

La truffa **BEC** **impersona un dirigente o un fornitore via email** per indurre
**bonifici** o l'invio di **dati riservati**. Tipico: "Sono l'AD, fai urgentemente un
bonifico a questo IBAN, sono in riunione, non chiamarmi". Colpisce l'ufficio
amministrazione; i danni economici sono enormi. Non usa malware: sfrutta **fiducia e
gerarchia**.
## Un caso di questo mese

Nella prima settimana di novembre, in un'azienda che seguo, è arrivata una mail da un corriere con cui lavorano davvero. Oggetto: una richiesta di scarico. Il mittente era vero: la casella del corriere era già stata compromessa, e i controlli sul mittente erano tutti verdi. Nessun testo, nessun allegato. Solo un'immagine che imitava OneDrive, con un documento da aprire.

Il percorso dietro quel clic. I log della casella non c'erano, quindi questa è la ricostruzione più probabile, coerente con quello che si è visto, non un verbale:

1. L'immagine porta a un modulo Microsoft Forms, vero, su dominio Microsoft. Nessun filtro lo ferma.
2. Dal modulo si passa a una pagina ponte che controlla di non essere dentro un sistema di analisi automatica. Se a guardare è un filtro, mostra una pagina innocua. Se è una persona, va avanti.
3. L'ultimo passo è un proxy: una pagina che mostra il **vero** login Microsoft, gira tutto a Microsoft in tempo reale e si tiene la risposta. La persona scrive la password, riceve la notifica sul telefono, approva. Tutto come ogni mattina.
4. Il proxy ha in mano il cookie di sessione. Entra in Outlook sul web senza password e senza secondo fattore. Sul PC non c'è nessun virus: l'antivirus non ha niente da trovare.
5. Chi è dentro legge le conversazioni vere con clienti e fornitori, di solito crea regole nella casella per nascondere le risposte, e al momento giusto manda il «cambio IBAN» dentro una di quelle conversazioni, dal mittente giusto. Il bonifico parte.

Lo schema si chiama **adversary-in-the-middle** (AiTM, nel glossario) e i kit si affittano. Chi ha cliccato ha fatto esattamente quello che fa ogni giorno per lavoro: ha aperto un documento di un fornitore vero. È una vittima, non il colpevole. La stessa campagna gira da fine ottobre di vittima in vittima: ogni casella rubata manda la stessa mail ai propri contatti, e l'azienda successiva riceve una mail vera da un fornitore vero.

Quattro cose rendono il caso istruttivo. La mail era autentica, quindi i controlli sul mittente non potevano fermarla. Il secondo fattore c'era, ma era a notifica: il proxy l'ha inoltrato come il resto. I log della casella non erano conservati: quando è entrato, cosa ha letto, quando ha scritto, non si è potuto ricostruire. E il tempo: la mail è stata segnalata all'IT il giorno dopo, l'analisi è arrivata una settimana dopo, quando il bonifico era già partito. La misura che conta è quella fra la segnalazione e la revoca delle sessioni.

## La procedura, in cinque righe

1. Ogni **cambio di IBAN** di un fornitore si verifica **telefonando** al numero già in anagrafica. Non si risponde alla mail, non si usa il numero scritto nella mail. Il primo pagamento sul nuovo IBAN aspetta due giorni: le urgenze vere sopravvivono a un'attesa, le truffe no.
2. Sopra una cifra decisa insieme (per esempio cinquemila euro), e sempre al primo bonifico dopo un cambio di IBAN, il pagamento lo **approvano due persone**.
3. Le richieste **urgenti** seguono la stessa procedura. Se non c'è tempo per una telefonata, non c'è tempo per un bonifico.
4. Il **titolare non è esentato**. La richiesta più pericolosa arriva a nome suo, quindi è proprio lui che deve volere la verifica.
5. La procedura è **scritta** e sta nel cassetto di chi fa i pagamenti, non nella testa di qualcuno.

## Quello che si può bloccare, oltre alla procedura

La procedura ferma il bonifico. Le misure qui sotto fermano, o fanno vedere, quello che viene prima: il clic, il furto della sessione, le settimane in cui qualcuno legge la posta. Sono scritte per Microsoft 365, perché è quello che trovo nelle aziende che seguo; su Google Workspace esistono quasi tutte, con nomi diversi. Per ogni misura: cosa fa, dove si imposta, e cosa avrebbe cambiato nel caso raccontato sopra. Le licenze Business Premium comprendono tutto quello che serve; con le licenze base alcune voci mancano, e lo segnalo.

### Se è già successo: il contenimento, nell'ordine giusto

1. **Revocare le sessioni, prima di tutto.** Nel centro di amministrazione, sull'utente: «Disconnetti da tutte le sessioni» (in Entra ID: «Revoca sessioni»). Il ladro è dentro con un cookie, non con la password: se cambi solo la password, il cookie continua a valere finché non scade. Nel caso sopra, è il passo che avrebbe chiuso la porta il giorno della segnalazione.
2. **Poi la password, e i metodi del secondo fattore.** Cambiata la password, si guardano i metodi di autenticazione registrati sull'utente: un attaccante dentro la sessione può aver aggiunto un telefono suo. Si tolgono quelli che la persona non riconosce.
3. **Regole, inoltri, deleghe, app.** In Outlook sul web, impostazioni, «Regole»: una regola che sposta in archivio o cancella i messaggi che contengono «fattura», «IBAN» o il nome di un cliente è la firma di questo attacco. Poi l'inoltro automatico, le deleghe sulla casella, e in Entra ID le «Applicazioni aziendali» con consenso dell'utente: un'app sconosciuta con permesso di leggere la posta è un'altra porta di servizio.
4. **Salvare i log, subito.** Nel portale Defender o Purview, «Ricerca nel log di controllo» sull'utente, scaricato in un file prima che la finestra di conservazione scada. È il materiale con cui si ricostruisce quando è entrato, cosa ha letto, a chi ha scritto. Nel caso sopra non c'era: la cronologia della frode è rimasta probabile, non accertata.
5. **Avvisare chi scrive a quella casella.** Clienti e fornitori con cui la persona era in conversazione ricevono un avviso, per telefono o da un altro indirizzo: «se avete ricevuto un cambio di IBAN da noi, non è nostro». La stessa campagna, da quella casella, sta già partendo verso i vostri contatti.

### Sulla posta: quello che si imposta in un pomeriggio

1. **Inoltro automatico verso l'esterno spento per tutti.** Exchange, «Posta in uscita», criteri di posta indesiderata in uscita: inoltro automatico «Disattivato». È la misura che costa meno e toglie all'attaccante il modo più comodo di leggere la posta anche dopo che gli hai chiuso la sessione.
2. **Un avviso quando qualcuno crea una regola sospetta.** Nel portale Defender, «Criteri di avviso»: gli avvisi «Creazione di regola di inoltro/reindirizzamento» e «Regole di manipolazione della posta sospette» esistono già, basta accenderli e mandarli al tuo indirizzo. Nel caso sopra, l'avviso sarebbe arrivato il giorno del clic.
3. **Link controllati al clic, senza scappatoia.** Defender per Office 365, «Collegamenti sicuri» (Safe Links): i link vengono riscritti e verificati nel momento in cui si cliccano, non quando la mail arriva. Due impostazioni contano: il controllo al clic attivo, e «consenti agli utenti di passare al sito originale» disattivato, altrimenti la persona clicca «continua comunque». La pagina ponte del caso sopra era nuova e pulita all'arrivo della mail; al clic, giorni dopo, poteva già essere nota. Serve Defender per Office 365, compreso in Business Premium.
4. **Allegati in detonazione e ritiro a posteriori.** «Allegati sicuri» (Safe Attachments) apre gli allegati in un ambiente isolato prima di consegnarli; ZAP (Zero-hour Auto Purge) ritira dalle caselle una mail già consegnata quando viene riconosciuta come malevola dopo. Nel caso sopra non c'erano allegati, ma ZAP avrebbe tolto la mail dalle altre caselle che l'avevano ricevuta.
5. **Protezione dall'impersonificazione.** Nel criterio anti-phishing: i nomi del titolare, dell'amministrazione e dei fornitori principali come «utenti protetti», i vostri domini e quelli dei fornitori come «domini protetti», e l'intelligence per le cassette postali attiva, che impara con chi scrivete di solito. Ferma la versione classica della truffa, quella con il nome del titolare da un dominio simile. Non ferma il caso sopra, dove il mittente era vero: per quello servono le misure sull'accesso.
6. **Una regola per i link a moduli esterni.** Exchange, «Regole del flusso di posta»: se il mittente è esterno e il corpo contiene un link a `forms.office.com` o `forms.microsoft.com`, aggiungi un avviso in testa al messaggio o mettilo in quarantena. I filtri non bloccano un dominio Microsoft; una regola tua sì. Nello stesso posto: il contrassegno «mittente esterno» sulle mail da fuori, e un avviso sulle mail che contengono una sola immagine e nessun testo.
7. **Conservazione dei log accesa, con la durata massima.** Purview, «Log di controllo»: la registrazione deve essere attiva (sui tenant nuovi lo è, su quelli vecchi va verificato) e la conservazione impostata al massimo che la licenza permette: 180 giorni nelle licenze Business, un anno e oltre con le licenze superiori. È la cosa che mancava nel caso sopra, e si accende prima dell'incidente, non durante.
8. **DMARC del vostro dominio, in tre tappe.** Prima `p=none` con l'indirizzo dei rapporti, un mese a leggerli, poi `p=quarantine`, poi `p=reject`. Non ferma il caso sopra, dove la mail era autentica, ma impedisce che la stessa mail parta a nome vostro verso i vostri clienti, dal dominio vostro, senza passare dalla vostra casella. Come si fa, passo per passo, è nella lezione 7.
9. **Gli indicatori bloccati nel tenant.** Defender, «Elenco consentiti/bloccati del tenant»: i domini usa-e-getta della campagna, appena li conosci, vanno nella lista dei bloccati. I domini, non gli indirizzi IP: la pagina ponte stava dietro un servizio condiviso da milioni di siti, e bloccare l'IP avrebbe bloccato anche quelli.

### Sull'accesso: il punto in cui l'attacco sarebbe fallito

1. **Un secondo fattore che il proxy non può inoltrare.** La notifica sul telefono e il codice a sei cifre sono cose che la persona *dà* alla pagina, e la pagina le gira al servizio vero. Una passkey, una chiave fisica (FIDO2) o Windows Hello for Business sono legati al dominio vero: sulla pagina finta, anche identica, non rispondono. In Entra ID si impone con i «Livelli di autenticazione» (authentication strengths) dentro un criterio di accesso condizionale: «MFA resistente al phishing» almeno per gli amministratori e per chi fa i pagamenti, poi per tutti. Richiede Entra ID P1, compreso in Business Premium. È il passo 3 del caso sopra, quello dove la persona ha approvato la notifica: con una passkey non c'era niente da approvare, e l'attacco finiva lì.
2. **Accesso solo da dispositivi aziendali riconosciuti.** Accesso condizionale: la posta si apre solo da un dispositivo registrato in Intune e conforme (disco cifrato, antivirus attivo, aggiornamenti). Un cookie rubato, usato dal computer dell'attaccante, non passa: il dispositivo non è vostro. È la seconda rete di sicurezza, se la passkey non c'è ancora.
3. **Sessioni corte, e legate al dispositivo.** Nello stesso criterio: «Frequenza di accesso» a poche ore per chi sta fuori dai dispositivi aziendali, «sessione persistente» disattivata, e dove disponibile la «protezione dei token», che lega il cookie al dispositivo che l'ha ottenuto. Un cookie rubato vale meno se scade in fretta e non funziona altrove.
4. **Il rischio di accesso come segnale.** Identity Protection (Entra ID P2, non in tutte le licenze) segna come rischioso l'accesso da un paese nuovo, da un indirizzo anonimo, o un «viaggio impossibile» (Firenze alle 9, un altro continente alle 9:10). Con il criterio giusto, l'accesso a rischio medio chiede di nuovo il secondo fattore, quello alto blocca. Senza P2 resta la versione a mano: le «posizioni denominate» con i vostri uffici e le vostre VPN, e un criterio che blocca o avvisa sugli accessi da fuori.
5. **Le porte vecchie chiuse.** Accesso condizionale: «Blocca autenticazione legacy» (i protocolli vecchi, POP, IMAP, SMTP di base, non sanno cosa sia un secondo fattore). Entra ID, «Consenso utente»: gli utenti non possono autorizzare app di terze parti a leggere la posta, lo fa l'amministratore. Un attaccante dentro la sessione usa spesso proprio questo per restare dopo che la sessione è chiusa.
6. **Il browser e il DNS che frenano prima.** SmartScreen attivo e non disattivabile dall'utente (criterio Intune o GPO): la pagina finta, quando è già nota, viene bloccata nel browser. Un filtro DNS (quello del firewall, o un servizio) che blocca i domini registrati da meno di trenta giorni: i kit vivono su domini nati ieri, e il dominio della pagina ponte lo era. Sui PC, le regole di riduzione della superficie di attacco di Defender, almeno quelle contro gli script scaricati e i contenuti eseguibili dalla posta.
7. **Una caccia ogni tanto, con i log accesi.** Una volta al mese, nel portale Defender, tre domande ai log: chi ha creato regole di posta negli ultimi trenta giorni, chi si è collegato da un paese nuovo, quali app hanno ricevuto consenso. Con i log conservati le risposte ci sono; senza, no. È il motivo per cui la conservazione viene prima di tutto.
8. **Un account di emergenza fuori dalle regole.** Un amministratore «break-glass», escluso dall'accesso condizionale, con una password lunga chiusa in cassaforte e un avviso a ogni suo uso. Serve il giorno in cui la passkey si rompe o il criterio chiude fuori anche te. Senza, la tentazione è non stringere mai i criteri.

### Sui pagamenti: in più alla procedura

1. **La verifica del beneficiario in banca.** Molte banche mostrano, prima del bonifico, se il nome del beneficiario corrisponde all'intestatario dell'IBAN (Verification of Payee, obbligatoria nell'area euro dal 2025). Un IBAN nuovo intestato a una persona, o a una società con un nome diverso dal fornitore, si vede prima di confermare.
2. **L'anagrafica fornitori bloccata.** Nel gestionale, la modifica dell'IBAN di un fornitore la fa una sola persona, diversa da chi dispone i pagamenti, e resta nel registro delle modifiche. Nell'home banking, la doppia firma sopra la soglia che avete scelto.
3. **La verifica registrata.** Ogni telefonata di conferma di un cambio di IBAN lascia una riga: chi ha chiamato, a che numero, quando, chi ha risposto. Se il bonifico parte lo stesso, si sa dove si è rotta la procedura.
4. **Una riga nelle vostre fatture.** «Non cambiamo mai le coordinate bancarie per email: in caso di comunicazioni diverse, chiamateci al numero in anagrafica.» Costa zero e protegge i vostri clienti dalla stessa truffa fatta a nome vostro.

## Le altre storie ben raccontate

- **Il tecnico al telefono.** «Sono dell'assistenza del gestionale, mi può dare la password che le aggiorno la licenza?» Nessuna assistenza seria chiede password. Si riattacca e si richiama il numero del contratto.
- **La chiavetta trovata.** Una chiavetta USB nel parcheggio con scritto «stipendi» finisce in un PC entro un'ora. In azienda si decide prima: le chiavette trovate si portano a te, non si inseriscono.
- **La persona che entra dietro di te.** Con gli scatoloni in mano, nessuno le chiede chi è. Dove c'è un badge, la regola è una persona per passaggio, e vale anche per chi è gentile.

## Gli errori che vedo più spesso

- **La procedura c'è, ma il titolare la salta.** Allora non c'è.
- **La verifica si fa rispondendo alla stessa mail.** Risponde l'attaccante, e conferma.
- **Nessun limite sopra il quale servono due firme.** Un solo errore di una sola persona basta.
- **Il secondo fattore a notifica creduto sufficiente.** Contro un proxy non lo è.
- **La password cambiata senza chiudere le sessioni.** Il ladro resta dentro.
- **I log scoperti dopo.** La conservazione si accende prima dell'incidente, non durante.
- **Pensare che sia un problema dell'amministrazione.** È un problema di come l'azienda decide di pagare, e la decisione la prende il titolare.

## Con l'AI, per questa lezione

**Uso 6 di 12 · Sapere cosa sa fare l'AI di chi attacca.** Qui l'AI non la usi tu: la usa chi chiama l'amministrazione con la voce del titolare. Con pochi secondi di audio preso da un video o da un messaggio vocale, oggi si imita una voce in modo credibile. La difesa non è tecnica, è una procedura, e all'assistente puoi chiedere di scriverla e di provarla.

> Scrivi la procedura per i pagamenti ai fornitori di un'azienda di [N] persone: verifica telefonica di ogni cambio di IBAN al numero già in anagrafica, doppia approvazione sopra [cifra] euro, nessuna eccezione per le urgenze, titolare compreso, e una parola d'ordine concordata a voce per le richieste di pagamento che arrivano per telefono o messaggio vocale. Cinque righe, da stampare.

Poi: «Elenca le situazioni in cui questa procedura rischia di essere saltata, e per ognuna dimmi cosa scrivere nella procedura per evitarlo». La lista che viene fuori è quella da discutere con il titolare, perché le eccezioni di solito partono da lui.

## Lunedì mattina, se sei l'unico informatico

1. Mettiti d'accordo con l'amministrazione e scrivilo: ogni cambio di IBAN di un fornitore si verifica richiamando il numero che avevate già in anagrafica, non quello nella mail. Sopra una certa cifra, due persone. Nessuna eccezione per le urgenze, titolare compreso: la richiesta più pericolosa arriva a nome suo.
2. Su ogni casella spegni l'inoltro automatico verso l'esterno e guarda le regole esistenti: una regola che sposta o cancella messaggi, e che nessuno ricorda di aver fatto, è un segnale.
3. Accendi la conservazione dei log della posta, oggi. E segna in agenda il secondo fattore che non si inoltra, passkey o chiave fisica: ci torniamo nella lezione 7.

## Glossario

Le sigle nuove di questa lezione, con due righe ciascuna: la prima per capirsi, la seconda per capire. Poi, in breve, quelle delle lezioni precedenti.

- **BEC** *(nuova)* — La truffa in cui qualcuno si finge il titolare o un fornitore via email per far fare un bonifico (Business Email Compromise).
  *Per capirla meglio:* Non usa virus: usa fiducia e gerarchia. Nella versione più efficace la casella del fornitore è davvero compromessa, e la fattura è vera con l'IBAN cambiato. La difesa che conta è una procedura: verifica telefonica di ogni cambio di IBAN e doppia approvazione sopra una cifra. Il resto, sulla posta e sull'accesso, fa vedere prima chi è entrato.
- **AD (amministratore delegato)** *(nuova)* — Il capo dell'azienda. Nelle mail di truffa è lui che «scrive» chiedendo un bonifico urgente.
  *Per capirla meglio:* La sigla è la stessa di Active Directory (la gestione degli utenti di Windows), che compare in altre lezioni: il contesto dice quale delle due. Nelle truffe, il nome del titolare è la leva: per questo la procedura dei pagamenti vale anche per lui, soprattutto per lui.
- **AiTM** *(nuova)* — Un attacco in cui una pagina finta sta in mezzo fra te e il vero login, e si tiene la sessione (Adversary-in-the-Middle).
  *Per capirla meglio:* La pagina mostra il login vero, gira password e secondo fattore al servizio vero e conserva il cookie di sessione: da quel momento entra senza password né codice. I controlli sul mittente non lo fermano, e il codice a sei cifre o la notifica nemmeno. Lo fermano una passkey o una chiave fisica legate al dominio vero.
- **cookie di sessione** *(nuova)* — Il gettone che il browser conserva dopo il login per non chiedere la password a ogni pagina.
  *Per capirla meglio:* Chi lo ruba è già dentro, senza password e senza secondo fattore, finché la sessione non scade o non viene revocata. Per questo, dopo un furto, prima si revocano le sessioni e poi si cambia la password.
- **passkey** *(nuova)* — Un modo di entrare senza password: la chiave sta nel telefono, nel PC o in una chiavetta, e si sblocca con l'impronta o il PIN.
  *Per capirla meglio:* La chiave risponde solo al dominio vero: su una pagina finta, anche identica, non funziona. È il secondo fattore che un proxy non può inoltrare. Si chiama anche FIDO2 o WebAuthn; Windows Hello for Business è la versione di Microsoft.

Dalle lezioni precedenti (la spiegazione completa è nella lezione in cui sono comparse):

- **CIA** — Le tre cose da proteggere: riservatezza, integrità, disponibilità (in inglese Confidentiality, Integrity, Availability). Niente a che vedere con l'agenzia americana. *(lezione 1)*
- **ransomware** — Un programma che cifra i file dell'azienda e chiede un riscatto per la chiave. Non è una sigla, ma è la parola che torna più spesso. *(lezione 1)*
- **DDoS** — Attacco che manda a un servizio più richieste di quante ne regga, finché smette di rispondere. Sta per Distributed Denial of Service. *(lezione 1)*
- **HTTPS** — La versione cifrata di HTTP, il protocollo delle pagine web. È il lucchetto nel browser. *(lezione 1)*
- **PLC** — Il computer industriale che comanda una macchina o una linea di produzione (Programmable Logic Controller). *(lezione 1)*
- **IBAN** — Il codice del conto corrente, quello che si scrive per fare un bonifico (International Bank Account Number). *(lezione 1)*
- **AI** — Intelligenza artificiale. Qui vuol dire gli assistenti con cui si parla scrivendo, e i modelli che ci stanno dietro. *(lezione 1)*
- **CVE** — Il codice univoco di una vulnerabilità nota, nel formato CVE-anno-numero (Common Vulnerabilities and Exposures). *(lezione 2)*
- **CWE** — La categoria di una debolezza del software, per esempio «SQL injection» (Common Weakness Enumeration). *(lezione 2)*
- **CVSS** — Il punteggio di gravità di una vulnerabilità, da 0 a 10 (Common Vulnerability Scoring System). *(lezione 2)*
- **NVD** — La banca dati pubblica dei CVE, gestita dal NIST americano (National Vulnerability Database). *(lezione 2)*
- **NIST** — L'ente americano degli standard (National Institute of Standards and Technology). Gestisce l'NVD e pubblica le linee guida più usate. *(lezione 2)*
- **CNA** — Gli enti autorizzati ad assegnare i codici CVE: grandi produttori, CERT nazionali, ricercatori accreditati (CVE Numbering Authority). *(lezione 2)*
- **MITRE** — L'organizzazione americana senza scopo di lucro che coordina i CVE e gestisce ATT&CK. *(lezione 2)*
- **CERT** — Una squadra che risponde agli incidenti informatici, nazionale o di settore (Computer Emergency Response Team). *(lezione 2)*
- **EPSS** — Una stima, in percentuale, della probabilità che una vulnerabilità venga sfruttata nei prossimi trenta giorni (Exploit Prediction Scoring System). *(lezione 2)*
- **AV, AC, PR, UI** — Le quattro lettere principali del vettore CVSS: da dove si attacca, quanto è difficile, se servono privilegi, se serve un'azione dell'utente. *(lezione 2)*
- **SQL** — Il linguaggio con cui si interrogano le banche dati (Structured Query Language). *(lezione 2)*
- **XSS** — Un difetto di un sito che permette di far eseguire del codice nel browser di chi lo visita (Cross-Site Scripting). *(lezione 2)*
- **VPN** — Un collegamento cifrato da fuori verso la rete aziendale, come se si fosse in ufficio (Virtual Private Network). *(lezione 2)*
- **NAS** — Un disco di rete condiviso, spesso usato per i file e per i backup (Network Attached Storage). *(lezione 2)*
- **API** — Un modo per far parlare due programmi fra loro, senza passare da una pagina web (Application Programming Interface). *(lezione 2)*
- **KEV** — Il catalogo delle vulnerabilità che vengono già sfruttate in attacchi reali, tenuto da CISA (Known Exploited Vulnerabilities). *(lezione 3)*
- **CISA** — L'agenzia americana per la sicurezza informatica e delle infrastrutture (Cybersecurity and Infrastructure Security Agency). *(lezione 3)*
- **FIRST** — L'associazione mondiale delle squadre di risposta agli incidenti, che calcola e pubblica l'EPSS (Forum of Incident Response and Security Teams). *(lezione 3)*
- **CSV** — Un file di testo con i dati in colonne separate da virgole (Comma-Separated Values). Si apre con un foglio di calcolo. *(lezione 3)*
- **ATT&CK** — Il catalogo, tenuto da MITRE, delle tattiche e tecniche che gli attaccanti usano davvero. *(lezione 4)*
- **TA0004, T1053** — I codici delle tattiche (TA) e delle tecniche (T) nel catalogo ATT&CK. *(lezione 4)*
- **C2** — Il canale con cui chi attacca comanda da remoto un computer infetto (Command and Control). *(lezione 4)*
- **OWASP** — Una comunità mondiale che pubblica materiale gratuito sulla sicurezza delle applicazioni web (Open Web Application Security Project). *(lezione 4)*
- **USB** — La presa a cui si attaccano chiavette, mouse, tastiere (Universal Serial Bus). *(lezione 4)*
- **SMS** — I messaggi di testo del telefono (Short Message Service). *(lezione 5)*
- **QR** — Il quadrato a punti che si inquadra con il telefono per aprire un indirizzo (Quick Response code). *(lezione 5)*
- **OTP** — Il codice usa e getta che arriva per SMS o che genera un'app, da inserire oltre alla password (One-Time Password). *(lezione 5)*
- **SPID** — L'identità digitale pubblica italiana, quella con cui si entra nei servizi della pubblica amministrazione (Sistema Pubblico di Identità Digitale). *(lezione 5)*
- **OT** — La tecnologia che comanda cose fisiche: frigoriferi, macchine, cancelli, impianti (Operational Technology). L'informatica che elabora dati si chiama IT. *(speciale 1)*
- **ICS, SCADA** — I sistemi che controllano impianti industriali (Industrial Control Systems), e il software di supervisione che li comanda da un posto solo (Supervisory Control And Data Acquisition). *(speciale 1)*
- **HMI** — Il pannello, fisico o web, da cui una persona legge e comanda una macchina o un impianto (Human-Machine Interface). *(speciale 1)*
- **Modbus** — Un protocollo industriale degli anni Settanta, ancora ovunque, con cui gli apparati si scambiano valori e comandi (porta 502). *(speciale 1)*
- **BACnet** — Il protocollo degli impianti dell'edificio: condizionamento, caldaie, luci, con cui i controllori si scambiano temperature, orari e allarmi (porta 47808). *(speciale 1)*
- **OPC UA** — Il protocollo moderno con cui supervisori e macchine recenti si scambiano i dati dell'impianto (porta 4840). *(speciale 1)*
- **SNMP** — Il protocollo con cui si leggono, e a volte si cambiano, le impostazioni degli apparati di rete e dei dispositivi (Simple Network Management Protocol, porta 161). *(speciale 1)*
- **UPnP** — Una funzione dei router domestici e di molti aziendali con cui un dispositivo interno si fa aprire una porta da fuori da solo, senza che nessuno la configuri (Universal Plug and Play). *(speciale 1)*
- **Shodan** — Un motore di ricerca che scandaglia tutti gli indirizzi di internet e tiene in memoria cosa risponde su ogni porta: banner, modello, a volte la schermata del pannello. Censys fa lo stesso. *(speciale 1)*
- **IEC 62443** — La norma internazionale della sicurezza dei sistemi industriali e OT, scritta per costruttori, integratori e chi gestisce l'impianto. *(speciale 1)*
- **NVR** — Il videoregistratore di rete a cui arrivano le immagini delle telecamere (Network Video Recorder). *(speciale 1)*
- **Firmware** — Il software che sta dentro un apparato e lo fa funzionare: router, telecamere, controllori, stampanti. *(speciale 1)*

---

← [Speciale 1](s1-ot-le-macchine-in-rete.md) · [Indice](../README.md) · Speciale 2: *Una falla critica sul sito aziendale: cosa fai la settimana in cui esce* (in uscita il 24 novembre 2026)
