---
title: La Sicurezza di rete
---

La network security, o sicurezza della rete, è l'insieme di misure, tecnologie e politiche che proteggono le reti informatiche e i dati da minacce, accessi non autorizzati e attacchi informatici.

# Network security tools

==Il modello di sicurezza di rete è un insieme di politiche, tecnologie e procedure che proteggono l'integrità, la riservatezza e la disponibilità dei dati e delle infrastrutture di rete. Dobbiamo essere consapevoli che il canale per definizione è non sicuro e possono accedervi dei nodi malevoli==. 

![[Screenshot 2026-02-05 at 16.50.54.png]]

Il mittente e il destinatario devono cooperare per lo scambio di messaggi sicuri che può avvenire nei seguenti modi:
- Accordo su algoritmi;
- Accordo su "segreti" (condivisi);
- Con l'opzione di una terza parte fidata che distribuisce segreti e certificati ai peer.

Cominciamo con una panoramica sui seguenti network security tools:

- [[Cryptosystem]] -> **Confidentiality**
- [[Hash function]] -> **Data integrity**
- [[Message authentication code (MAC) and HMAC]] -> **Data integrity & Authentication**
- [[Digital signature]] -> **Data integrity & Authentication**
- [[ITU X.509 certificates]] -> **Authentication**

# Public Key Infrastructure

==L'infrastruttura a chiave pubblica (PKI) è definita come l'insieme di hardware, software, persone, politiche e procedure necessarie per creare, gestire, archiviare, distribuire e revocare i certificati a chiave pubblica basati sulla crittografia a chiave pubblica==. La CA (Certificate Authority) è un componente fondamentale della PKI. La CA è l'Autorità Fidata (Trusted Third Party) che rilascia e gestisce i certificati, convalida le informazioni dell'utente e gestisce la revoca e il rinnovo.

![[Screenshot 2025-11-11 at 10.37.32.png]]

- **Certification Authority (CA)**: Autorità attendibile che crea e assegna certificati di chiave pubblica. Necessaria per convalidare le informazioni dell’utente e verificare che possieda la chiave privata. Necessaria per gestire le CRL.
- **Registration Authority (RA)**: Autorità facoltativa che può agire per conto di una CA per convalidare le informazioni dell’utente e verificare la sua chiave privata.
- **Repository**: Database o directory utilizzata per archiviare e distribuire certificati a chiave pubblica e CRL.
- **Time Stamping Authority (TSA)**: Entità attendibile che aggiunge informazioni temporali non ripudiabili alle transazioni elettroniche o ai documenti firmati digitalmente.
- **Certificate Revocation List (CRL)**: Elenco dei certificati che sono stati revocati a causa della violazione da parte dei loro proprietari di una delle regole della policy dei certificati o della compromissione della loro chiave privata.
- **Certificate Practice Statement (CPS)**: Insieme di linee guida che una CA segue quando emette certificati.
- **Certificate Policy (CP)**: Insieme di regole che indicano come un certificato deve essere utilizzato da una comunità di utenti o da un insieme di applicazioni.

# Key Management Protocols

La **Gestione delle Chiavi (Key Management)** è un aspetto fondamentale della sicurezza di rete e della crittografia, definita come l'insieme delle tecniche e procedure che supportano la creazione e il mantenimento delle relazioni chiave tra le parti autorizzate. Il problema della gestione delle chiavi copre l'intero ciclo di vita delle chiavi, ossia **generare, distribuire, archiviare, utilizzare e revocare/distruggere** le chiavi in modo sicuro. La scelta della metodologia di gestione delle chiavi è complessa e non esiste un'unica "risposta giusta". Questa scelta è influenzata dalla topologia di rete, dai servizi crittografici richiesti (ad esempio, riservatezza o non ripudio) e dai meccanismi crittografici utilizzati (ad esempio, cifratura o firma digitale). Tipicamente si utilizza una gerarchia di chiavi:

1. **Chiave Principale (Master Key):** Utilizzata per cifrare le chiavi di sessione. È condivisa tra l'utente e un Key Distribution Center (KDC).
2. **Chiave di Sessione (Session Key):** È una chiave temporanea, usata per cifrare i dati tra gli utenti per una singola sessione logica, e viene scartata al termine della sessione.

Come visto in precedenza possiamo distribuire chiavi simmetriche e asimmetriche. A tal proposito possiamo osservare due tipi di distribuzioni:
- [[Symmetric key distribution con KDC]]
- [[Asymmetric Key Distribution con KDC]]

# End-to-end security

==La Sicurezza End-to-End (E2E) è un approccio fondamentale nella sicurezza delle reti che mira a stabilire un canale sicuro== per la comunicazione, sia con un server web remoto, sia tra due dispositivi che comunicano attraverso Internet.

## Obiettivi e Requisiti di un Canale Sicuro E2E

Per affrontare le minacce alla sicurezza, le soluzioni tecniche E2E devono mirare contemporaneamente a:

1. **Autenticazione**.
2. **Gestione delle chiavi**.
3. **Riservatezza della comunicazione**.
4. **Autorizzazione e controllo degli accessi**.

Inoltre, è fondamentale garantire l'uso di **crittografia all'avanguardia** e l'aggiornamento di _firmware_ e _software_.

Il significato di "canale sicuro" in un contesto E2E è ben definito: ==il canale deve fornire sia la riservatezza che l'integrità==. A tal fine, le parti (Alice e Bob) devono:

- **Concordare la** **cipher suite**, che include i protocolli crittografici, la modalità di cifratura e le lunghezze delle chiavi.
- **Concordare le chiavi di sessione**.
- Avere la capacità di rilevare messaggi mancanti e **attacchi di** **replay**.
- Essere in grado di mantenere e terminare la connessione.

## Strumenti e Protocolli E2E

1. **[[Transport Layer Security (TLS)]]**: Questo protocollo crittografico (successore di SSL) ha come obiettivo primario quello di fornire **riservatezza e integrità dei dati** tra due applicazioni comunicanti. TLS opera sul Secure Transport Layer.
2. **[[Internet Protocol Security (IPSec)]]**: IPSec **non è realmente una soluzione end-to-end** perché l'autenticazione è troppo generica (basata sull'host) ed è troppo complesso per essere utilizzato dalle applicazioni. IPSec è utilizzato principalmente da alcuni fornitori di VPN.
3. **[[Virtual Private Networks (VPNs)]]**: Le VPN implementano spesso la sicurezza a livello IP (IPSec) per fornire un canale sicuro.

# Framework di Sicurezza AAA

Il Framework di Sicurezza AAA è un'architettura fondamentale per la gestione della sicurezza di rete, definita specificamente attraverso tre componenti principali: Autenticazione, Autorizzazione e Accounting.

1. **[[Authentication]]:** Questo servizio verifica che l'utente sia effettivamente chi dichiara di essere. I metodi di autenticazione possono includere l'uso di password, token card, ID chiamante, o schemi di _challenge-response_. Le fasi tipiche dell'autenticazione includono l'ottenimento dei dati di autenticazione (es. password sottoposta ad hash, certificato digitale, informazioni biometriche), la verifica di tali dati confrontando le credenziali ricevute con i riferimenti memorizzati, e l'associazione con il principale per confermare l'identità dichiarata.
2. **Authorization (Autorizzazione):** Questo servizio determina se l'utente autenticato è autorizzato ad accedere a specifiche risorse o servizi. L'autorizzazione si basa su profili utente, database di policy o liste di controllo degli accessi (ACL).
3. **Accounting (Contabilità):** Questo servizio registra le attività dell'utente a scopo di audit o fatturazione. Le metriche registrate includono il tempo di connessione, i byte inviati/ricevuti, i servizi a cui si è avuto accesso e i file scaricati.

Il framework AAA può essere implementato tra due entità o con l'aiuto di una terza parte fidata (TTP). Il modello è supportato da standard come **RFC 2903** (Generic AAA Architecture) e **RFC 2904** (AAA Authorization Framework). Come protocolli più famosi analizzeremo i seguenti:

- [[Radius]]
- [[Kerberos]]
- [[Oauth 2.0]]

# Perimeter security

==Il firewall è un'entità (hardware o software) posta a protezione di un'infrastruttura ICT per renderla scalabile e robusta. Invece di aggiornare singolarmente migliaia di server per ogni falla scoperta, il firewall permette una protezione centralizzata==. Il firewall può essere configurato con diverse politiche di sicurezza in diversi livelli dello stack ISO-OSI. Quando un'azienda vuole offrire servizi al mondo esterno (come un sito web o un server email), non può permettere che gli utenti internet entrino direttamente nella propria rete privata. Sarebbe troppo rischioso. Si crea quindi una **"organizzazione di rete differente"** tramite una DMZ. 

## DMZ

La DMZ viene definita come una **rete semi-pubblica**. Si trova a metà strada tra la rete interna (sicura) e quella esterna (non sicura). All'interno abbiamo server pubblici (Siti Web, server FTP per i file, DNS, server di posta). In DMZ non si mettono mai dati "critici" o sensibili dell'azienda. Gli utenti possono entrare nella DMZ per usare i servizi (es. guardare il sito web), ma il firewall impedisce loro di andare oltre. Le risorse interne rimangono protette nell'area privata e sono totalmente invisibili e inaccessibili dall'esterno.

### Two Single-Homed Firewalls

![[Screenshot 2026-02-09 at 10.25.39.png]]

Qui vengono utilizzati **due firewall distinti** per creare i confini:

- **Firewall Esterno:** Si trova tra Internet e la DMZ. È la prima linea di difesa.
- **Firewall Interno:** Si trova tra la DMZ e la rete privata (Intranet).
- **Vantaggio:** È molto sicuro. Se un hacker riesce a superare il primo firewall, si trova comunque bloccato dal secondo prima di poter toccare i dati aziendali sensibili.

### Single Firewall "Three Legged"

![[Screenshot 2026-02-09 at 10.26.40.png]]

In questo caso si usa un **unico firewall più potente** che ha almeno tre "gambe" (interfacce di rete):

1. Una connessa a **Internet**.
2. Una connessa alla **DMZ**.
3. Una connessa alla **Rete Privata**.

> [!NOTE] Vantaggio del Single Firewall
> È più economico e semplice da gestire rispetto alla soluzione con due macchine separate, poiché devi configurare un solo dispositivo.

## Tipologie di Firewall e Livelli OSI

I firewall operano a diversi livelli della pila OSI:

| **Tipo di Firewall**    | **Livello OSI**  | **Caratteristiche Principali**                                                                              |
| ----------------------- | ---------------- | ----------------------------------------------------------------------------------------------------------- |
| **Packet Filtering**    | Network (L3)     | Filtra in base a IP sorgente/destinazione, porte e protocollo . Veloce ma non traccia le connessioni.       |
| **Stateful Inspection** | Transport (L4)   | Traccia lo stato delle connessioni TCP (SYN, ACK, ecc.) in una tabella di stato.                            |
| **Application Gateway** | Application (L7) | Comprende protocolli specifici (HTTP, FTP) e applica policy granulari, ma è esigente in termini di risorse. |

## Oltre i Firewall Convenzionali: NGFW e WAF

I firewall classici non bastano contro attacchi complessi come virus nelle email o vulnerabilità negli script CGI.

- **Web Application Firewall (WAF)**: Protegge specificamente le applicazioni web da attacchi come **SQL Injection**, **Cross-Site Scripting (XSS)** e **Brute Force**.
- **Next Generation Firewall (NGFW)**: Integra Deep Packet Inspection (DPI), sistemi di prevenzione delle intrusioni (IPS) e identificazione degli utenti/applicazioni indipendentemente dalla porta usata.