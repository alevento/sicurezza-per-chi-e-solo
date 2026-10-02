# Lezione 1 — Cosa stai proteggendo, e da cosa

*Sicurezza informatica per chi è solo · Alessandro Manneschi · 6 ottobre 2026*

> Prima di parlare di firewall e antivirus conviene mettersi d'accordo su quattro parole. Le uso in tutte le lezioni, e le sento confondere in quasi tutte le aziende.

## La sicurezza in tre parole: la triade CIA

Ogni misura di sicurezza serve a proteggere almeno una di **tre** proprietà:

- **C — Confidentiality (Riservatezza):** solo chi è autorizzato accede all'informazione.
  *Esempio:* cifratura del disco, permessi sui file, HTTPS.
- **I — Integrity (Integrità):** l'informazione non viene alterata senza autorizzazione.
  *Esempio:* hash/checksum, firme digitali, log append-only.
- **A — Availability (Disponibilità):** il servizio è raggiungibile quando serve.
  *Esempio:* ridondanza, backup, protezione anti-DDoS.

> Un attacco si può leggere sempre come **violazione** di C, I o A: un furto di dati
> viola C; un ransomware viola I e A; un DDoS viola A.

## Il vocabolario del rischio

Quattro parole che spesso si confondono:

| Termine | Significato | Esempio |
|---------|-------------|---------|
| **Minaccia** (threat) | *chi/cosa* potrebbe farti danno | un gruppo ransomware |
| **Vulnerabilità** | una *debolezza* sfruttabile | un software non aggiornato |
| **Exploit** | il *codice/tecnica* che sfrutta la vulnerabilità | il programma che attiva il bug |
| **Rischio** | probabilità × impatto | quanto è probabile e quanto farebbe male |

**Vulnerabilità ≠ exploit:** la vulnerabilità è la *porta aperta*, l'exploit è
*passarci attraverso*.

**Zero-day:** una vulnerabilità **sfruttata prima** che sia nota al produttore o che
esista una correzione (patch). È pericolosa perché non c'è ancora difesa ufficiale.

## Lunedì mattina, se sei l'unico informatico

1. Scrivi su un foglio i cinque sistemi senza i quali l'azienda si ferma. Di solito sono gestionale, posta, file server, produzione e centralino.
2. Per ognuno segna cosa conta di più: che nessuno lo legga, che nessuno lo modifichi, o che funzioni. Per il gestionale di solito è che funzioni, per le buste paga che nessuno le legga.
3. Tieni il foglio: nelle prossime lezioni ti dice dove mettere il primo backup e il primo secondo fattore.

---

[Indice](../README.md) · Lezione 2: *Leggere un bollettino di sicurezza senza perdersi* (in uscita il 13 ottobre 2026)
