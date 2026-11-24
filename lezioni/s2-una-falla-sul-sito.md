# Speciale 2 — Una falla critica sul sito aziendale: cosa fai la settimana in cui esce

*Sicurezza informatica per chi è solo (o quasi) · Alessandro Manneschi · 24 novembre 2026*

> Il 21 ottobre è uscita PHP2Shell: due falle che fanno eseguire codice su qualunque sito in PHP, senza password. Ne esce una così quasi ogni anno, con un nome diverso. Questa lezione è la procedura per quella settimana.

## Cos'è successo, in breve

Il 21 ottobre 2026 PHP, il linguaggio su cui gira gran parte dei siti web, ha pubblicato le correzioni per due falle critiche, CVE-2026-109618 e CVE-2026-109619, battezzate insieme **PHP2Shell**. Chi le sfrutta fa eseguire codice sul server, una RCE, **senza doversi autenticare**: basta che il sito sia raggiungibile. Riguardano ogni installazione di PHP che risponde a richieste web, in qualunque modo il server la mandi in esecuzione (mod_php di Apache, che PHP indica come il caso più a rischio, PHP-FPM, php-cgi e gli altri). Quindi WordPress, Drupal, Joomla, i negozi online, i gestionali web e tutto quello che ci gira sopra.

Lo stesso giorno la CISA americana e l'ACN italiana hanno pubblicato un'allerta. Due giorni dopo ACN segnalava codice di attacco funzionante in giro; il 26 ottobre CISA ha messo le due falle nel catalogo KEV, quello delle vulnerabilità sfruttate in attacchi reali (lezione 3); il 28 ottobre ACN ha confermato lo sfruttamento in rete di entrambe. Una settimana, dalla correzione agli attacchi. Le versioni che chiudono le falle sono PHP 8.5.12, 8.4.27, 8.3.36 e 8.2.35. **Per la 8.1 e le precedenti la correzione non arriva**: sono fuori supporto, e CISA dice di spegnere quei siti finché non si aggiornano.

Questa lezione non è su PHP2Shell. È su cosa fare la settimana in cui esce una falla così, perché ne esce una quasi ogni anno: cambia il nome, non la procedura.

## Il sito è un server tuo, anche se non lo vedi

La prima reazione di quasi tutti è «il sito lo fa l'agenzia». È vero, e non cambia niente: il sito porta il nome dell'azienda, raccoglie i dati dei clienti, e se qualcuno ci entra lo usa per mandare mail a nome tuo o per arrivare al resto. Il punto non è chi lo fa. È che tu sappia **dove gira e chi aggiorna il software che c'è sotto**.

Il sito pubblico è il primo della lista, non l'unico. In una PMI di solito ci sono:

| Cosa | Dove gira di solito | Chi aggiorna PHP |
|---|---|---|
| Sito aziendale (WordPress o simili) | hosting condiviso o server dell'agenzia | l'hosting, non l'agenzia: vanno chiesti entrambi |
| Negozio online, portale clienti o fornitori | un server (VPS) dell'agenzia o tuo | chi amministra il server |
| Gestionale web, intranet, sistema ticket | un server in azienda o in cloud | il fornitore del gestionale, oppure tu |
| Webmail, strumenti del fornitore (monitoraggio, fotovoltaico) | dove li ha messi chi li ha installati | spesso nessuno |

L'elenco è lo stesso della lezione 2, con una colonna in più: per ogni applicazione web, **chi aggiorna il linguaggio sotto**. PHP è spesso un componente di qualcos'altro, e non si vede: la CISA lo scrive esplicitamente, e chiede per prima cosa l'inventario di tutto quello che lo usa.

## La settimana, in cinque passi

**Giorno 1 · Capire se ti riguarda.** Per ogni riga dell'elenco: gira in PHP? Che versione? È esposto a internet? Dove gestisci tu il server, la versione la leggi sulla macchina, non dal banner che il sito dichiara: sulle distribuzioni con le correzioni «all'indietro» (RHEL, AlmaLinux, Rocky, Oracle Linux) il numero non cambia dopo la correzione, e la prova sta nel changelog del pacchetto, come spiegato nella lezione 4. Dove gestisce un altro, scrivi subito le quattro domande del riquadro qui sotto. Se la risposta è «non lo so» o non arriva in giornata, tratta il sito come vulnerabile.

**Giorno 1 · Proteggere nel frattempo.** Tre cose, nell'ordine. Un **backup** fresco del sito e della sua banca dati, messo da parte, prima di toccare qualsiasi cosa. Poi ridurre l'esposizione: se c'è un **WAF**, un filtro davanti al sito, farlo attivare con le regole per la falla (la CISA dice che «può dare qualche protezione» finché non si aggiorna, e niente di più); limitare l'area di amministrazione ai soli indirizzi dell'ufficio; applicare il tampone di configurazione indicato da CISA e ACN, se chi gestisce il sito conferma che non lo rompe. E la domanda scomoda: **se il sito è su una versione che non si può correggere, lo spegni?** Per un sito vetrina la risposta è quasi sempre sì, finché non è aggiornato. Per un negozio online è una decisione del titolare, e va presa il primo giorno, non il quinto.

**Giorno 2-3 · Aggiornare, o far aggiornare.** Chi gestisce il server applica la correzione, prima sui sistemi esposti a internet. Si chiude solo con una prova: la versione del pacchetto, il changelog, o la conferma scritta del fornitore con la versione installata. «Fatto» a voce non è una prova.

**Giorno 3-5 · Controllare se sei già stato colpito.** Aggiornare chiude la porta, non fa uscire chi è già entrato. Fra la correzione e gli attacchi è passata una settimana, e tu l'hai saputo dopo. Si guardano quattro cose, e chi gestisce il server le sa trovare: file PHP creati o modificati dal 21 ottobre in cartelle dove non dovrebbero cambiare (upload, cache, temi); utenti amministratori nuovi nel CMS e nella banca dati; processi del sito che hanno lanciato una shell o scaricato qualcosa (è quello che la CISA dice di sorvegliare); collegamenti in uscita dal server verso indirizzi mai visti. Se trovi anche una sola di queste cose, è un incidente: si isola il server, si cambiano tutte le credenziali (amministratore del sito, banca dati, FTP, chiavi), si ripristina dal backup precedente al 21 ottobre e si segnala al CSIRT Italia. Pulire a mano un sito con una webshell dentro non funziona quasi mai.

**Dopo · Scrivere chi fa cosa.** La settimana finisce con una pagina: per ogni applicazione web, chi aggiorna, in quanto tempo da un'allerta critica, come ti avvisa. Se il fornitore non lo scrive nel contratto, lo scrivi tu in una mail e ti fai rispondere «confermo». Alla prossima falla parti da lì.

## Le quattro domande al fornitore

Vanno bene per l'hosting, per l'agenzia, per chi ti dà il gestionale web. Corte, per iscritto, il primo giorno:

> Buongiorno, il 21 ottobre è uscita PHP2Shell (CVE-2026-109618 e CVE-2026-109619), sfruttata in rete. Per il nostro sito [nome] vi chiedo: 1) gira in PHP, e con quale versione e pacchetto installato oggi? 2) la correzione è già applicata? Se no, quando? 3) nel frattempo c'è una protezione davanti al sito (WAF, regole)? 4) potete verificare se ci sono file modificati o accessi anomali dal 21 ottobre? Grazie, resto in attesa.

La risposta buona ha quattro numeri o quattro date. La risposta «siamo al sicuro, non vi preoccupate» senza una versione dentro non è una risposta: si richiede.

## Se il sito è su una versione fuori supporto

È il caso più frequente di quanto si creda: PHP 8.1 o più vecchio, perché il tema di WordPress o il gestionale «non gira con le versioni nuove». La correzione non arriverà. Le strade sono tre, e la quarta non esiste:

1. **Aggiornare PHP e far sistemare il sito.** Costa qualche ora di sviluppo, si prova prima su una copia. È l'unica che chiude il problema.
2. **Tenere il sito dietro un WAF e limitare quello che è esposto**, come misura provvisoria dichiarata, con una data per il punto 1.
3. **Spegnerlo**, se è una vetrina che non porta ordini.

La quarta, «lasciarlo com'è e sperare», è quella che gli attacchi di fine ottobre andavano a cercare.

## Gli errori che vedo più spesso

- **Fidarsi del banner.** «Il sito dichiara 8.2.20, quindi siamo vulnerabili», oppure il contrario. Il numero dichiarato non è la prova: lo è il pacchetto installato (lezione 4).
- **«Lo gestisce l'agenzia», e il contratto non dice in quanto tempo.** Allora lo gestisce nessuno, finché non chiedi tu.
- **Aspettare la mail del fornitore.** Se arriva, arriva dopo. Le allerte di CISA e ACN sono pubbliche e gratuite: ci si iscrive una volta.
- **Aggiornare senza backup.** La correzione va quasi sempre liscia; la volta che non va, senza backup il sito resta giù.
- **Chiudere la porta e non guardare dentro.** La patch chiude l'ingresso. Chi è entrato prima, resta.


## Con l'AI, per questa lezione

**Uso speciale · Leggere l'allerta, scrivere al fornitore, controllare le risposte.** Nella settimana della falla il tempo manca, e l'assistente serve a non saltare passi: traduce l'allerta in cose da fare per i tuoi sistemi, scrive la mail, e poi legge le risposte dei fornitori cercando quello che manca.

**A · Dall'allerta al tuo elenco.** Incolla il testo dell'allerta ufficiale (CISA o ACN) e il tuo elenco delle applicazioni web, senza indirizzi né nomi di clienti:

> Questa è un'allerta di sicurezza: [incolla]. Questo è l'elenco delle nostre applicazioni web, con dove gira ciascuna e chi la gestisce: [incolla]. Dimmi per ogni riga se l'allerta la riguarda, cosa devo verificare per esserne sicuro, e cosa posso fare oggi stesso per ridurre il rischio mentre aspetto la correzione. Se per qualche riga non hai abbastanza informazioni, dimmi quale domanda fare e a chi.

Controlla una cosa: se l'assistente ti dice «sei vulnerabile» o «sei al sicuro» sulla base del solo numero di versione, ti ha rifatto l'errore del banner. La prova è il pacchetto installato o la conferma del fornitore.

**B · Le risposte dei fornitori.** Quando le risposte arrivano, incollale tutte insieme:

> Ho scritto a quattro fornitori queste quattro domande: [le quattro domande]. Queste sono le risposte: [incolla, senza nomi]. Per ogni fornitore dimmi a quali domande ha risposto davvero con un dato verificabile (una versione, una data, un sì o no motivato) e a quali ha risposto in modo vago o non ha risposto. Scrivimi la riga da mandare a chi è stato vago, cordiale e con una scadenza.

**C · Il controllo sul server, se lo gestisci tu.** Chiedigli i comandi per trovare i file PHP modificati dopo una data nelle cartelle del sito, e per vedere i processi del server web che hanno aperto collegamenti in uscita. Fatti spiegare ogni comando prima di lanciarlo, lancialo tu, e poi incollagli l'output (senza indirizzi) chiedendo cosa è normale e cosa no. Se segnala un file sospetto, non lo cancelli: lo copi da parte e chiami chi fa risposta agli incidenti. Da quel momento è un incidente, non più manutenzione.

## Lunedì mattina, se sei l'unico informatico

1. Fai l'elenco dei siti e delle applicazioni web dell'azienda raggiungibili da internet, e per ognuno scrivi chi lo gestisce, dove gira e chi aggiorna il software che c'è sotto.
2. Scrivi al fornitore di ciascuno le quattro domande di questa lezione e metti le risposte nell'elenco. Dove la risposta è «non lo so», hai trovato il primo lavoro.
3. Stampa la procedura dei cinque passi e mettila con l'inventario: la prossima falla critica non si gestisce da capo, si segue la pagina.

## Glossario

Le sigle nuove di questa lezione, con due righe ciascuna: la prima per capirsi, la seconda per capire. Poi, in breve, quelle delle lezioni precedenti.

- **RCE** *(nuova)* — Una falla che permette a chi attacca di eseguire i propri comandi su un server da remoto (Remote Code Execution).
  *Per capirla meglio:* È la categoria più grave, perché chi la sfrutta non ruba un dato: prende il controllo della macchina. Quando è «non autenticata», come PHP2Shell, non serve nemmeno una password: basta raggiungere il servizio. Per questo le RCE sui sistemi esposti a internet si correggono per prime, e il KEV della lezione 3 ne è pieno.
- **WAF** *(nuova)* — Un filtro che sta davanti a un sito e blocca le richieste che somigliano a un attacco (Web Application Firewall).
  *Per capirla meglio:* Serve come tampone: con una regola per la falla del giorno ferma gli attacchi più semplici finché non si aggiorna. Non sostituisce la correzione, perché le regole si aggirano. Molti hosting lo offrono già: spesso basta chiedere di accenderlo.
- **CMS** *(nuova)* — Il programma con cui si costruisce e si aggiorna un sito senza scrivere codice: WordPress, Drupal, Joomla (Content Management System).
  *Per capirla meglio:* Il CMS gira sopra PHP e ha i suoi aggiornamenti, oltre a quelli dei temi e delle estensioni che ci si installano. Quando esce una falla in PHP il CMS non c'entra, ma ne subisce le conseguenze: chi aggiorna il CMS non è detto che aggiorni PHP, e viceversa.
- **webshell** *(nuova)* — Un piccolo file lasciato sul server da chi è entrato, che gli permette di tornare e dare comandi dal browser.
  *Per capirla meglio:* È la cosa che si cerca nei giorni 3-5: un file PHP nuovo in una cartella di upload o di cache, con un nome innocuo. Finché c'è, l'attaccante rientra anche dopo che la falla è stata corretta. Per questo aggiornare non basta: si controlla, e in caso si ripristina dal backup precedente.
- **banner** *(nuova)* — La riga con cui un servizio dichiara nome e versione a chi si collega.
  *Per capirla meglio:* Su alcune distribuzioni il numero di versione non cambia dopo la correzione, perché la correzione arriva all'indietro: lo scanner che legge il banner può dire «vulnerabile» a un sistema già corretto, o il contrario. Da solo non prova niente: la prova è il pacchetto installato e il suo changelog.
- **VPS** *(nuova)* — Un server virtuale preso in affitto, su cui si installa quello che si vuole (Virtual Private Server).
  *Per capirla meglio:* È dove di solito girano i negozi online e i portali fatti dalle agenzie. La differenza con l'hosting condiviso sta in chi aggiorna il sistema: sul VPS lo fa chi lo amministra, cioè l'agenzia o tu, non il fornitore dell'infrastruttura. Se nessuno lo sa, non lo fa nessuno.
- **FTP, SFTP** *(nuova)* — I modi con cui si caricano i file su un server web: il primo in chiaro, il secondo cifrato (File Transfer Protocol; SFTP passa dentro SSH, cifrato).
  *Per capirla meglio:* Le credenziali FTP del sito sono fra quelle da cambiare dopo un incidente, insieme a quelle della banca dati e dell'amministratore del CMS. E FTP in chiaro non dovrebbe più esistere: la password passa leggibile sulla rete.

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
- **BEC** — La truffa in cui qualcuno si finge il titolare o un fornitore via email per far fare un bonifico (Business Email Compromise). *(lezione 6)*
- **AD (amministratore delegato)** — Il capo dell'azienda. Nelle mail di truffa è lui che «scrive» chiedendo un bonifico urgente. *(lezione 6)*
- **AiTM** — Un attacco in cui una pagina finta sta in mezzo fra te e il vero login, e si tiene la sessione (Adversary-in-the-Middle). *(lezione 6)*
- **cookie di sessione** — Il gettone che il browser conserva dopo il login per non chiedere la password a ogni pagina. *(lezione 6)*
- **passkey** — Un modo di entrare senza password: la chiave sta nel telefono, nel PC o in una chiavetta, e si sblocca con l'impronta o il PIN. *(lezione 6)*

---

← [Lezione 6](06-la-truffa-del-bonifico.md) · [Indice](../README.md) · Lezione 7: *Difendere le persone: le misure che funzionano* (in uscita il 1 dicembre 2026)
