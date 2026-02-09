# Sicurezza di Rete (Appunti)

Questa repository contiene i file sorgente dei miei appunti universitari riguardanti le architetture e i protocolli per la protezione delle infrastrutture ICT.

🌐 **Leggi gli appunti formattati qui:** 👉 [https://Cresh-dev.github.io/Network-Security/](https://Cresh-dev.github.io/Network-Security/)

---

## Contenuti

### 🔐 Infrastruttura a Chiave Pubblica (PKI)

- **Concetti Base:** Gestione del ciclo di vita dei certificati digitali tramite hardware, software e procedure dedicate.
- **Componenti:** Ruoli della **Certificate Authority (CA)** come ente fidato, della **Registration Authority (RA)** per la verifica delle identità e del **Repository** per la memorizzazione di certificati e CRL.
- **Standard X.509:** Utilizzo dei certificati per garantire l'autenticazione tramite crittografia a chiave pubblica.
- **Policy:** Differenza tra **Certificate Policy (CP)** e **Certificate Practice Statement (CPS)** per le linee guida operative.

### 🛡️ Sicurezza Perimetrale

- **Firewalling:** Sistemi di protezione per il controllo degli accessi tra reti con diversi livelli di fiducia.
- **Architetture DMZ:** Configurazione di zone demilitarizzate per esporre servizi pubblici (Web, Mail) in sicurezza.
- **Modelli Implementativi:** Analisi delle soluzioni **Three-Legged Firewall** (economica) e **Two Single-Homed Firewalls** (massima sicurezza tramite isolamento fisico).
- **Tipologie di Filtro:** Analisi dei filtri a livello Network (Packet Filtering), Transport (Stateful Inspection) e Application (Gateway).

### 🤝 Framework AAA e Sicurezza E2E

- **Modello AAA:** Implementazione di **Authentication**, **Authorization** e **Accounting** per il controllo granulare degli accessi.
- **Protocolli di Sicurezza:** Approfondimento su standard come Radius, Kerberos, TLS e IPsec.
- **Canali Sicuri:** Tecniche per garantire riservatezza e integrità dei dati end-to-end.
- **Gestione delle Chiavi:** Gerarchia e distribuzione delle chiavi di sessione e chiavi master.
