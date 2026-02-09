==Le VPN sono tecnologie che operano al livello 2 (Data Link/Ethernet) o al livello 3 (Network/IP) del modello OSI per creare canali sicuri attraverso reti pubbliche come Internet==. L'obiettivo principale è il **Tunneling**: questa tecnica consente ai mittenti di incapsulare i propri dati in pacchetti IP che nascondono l'infrastruttura di routing e switching sottostante, garantendo la sicurezza dei dati contro accessi indesiderati. Possiamo avere due scenari principali:
- **LAN-to-LAN:** Viene creato un tunnel tra due router di bordo (chiamati _concentrator_). Questo collega, ad esempio, sedi distaccate di un'azienda.
- **Host-to-LAN:** Viene creato un collegamento sicuro tra un singolo host (definito _roadwarrior_) e il router di bordo della LAN. Questo scenario è tipico per il lavoro remoto o mobile.

# Protocolli e Implementazione: Il Ruolo Centrale di IPSec

Abbiamo varie tipologie di VPN (come PPTP, L2TP, SSL/TLS e MPLS), nel nostro caso diamo ampio spazio a **[[Internet Protocol Security (IPSec)|IPSec]]** come tecnologia fondamentale per la sicurezza gateway-to-gateway e per le VPN. Per le VPN, IPSec utilizza specificamente la **Tunnel Mode** dove l'intero pacchetto IP originale viene [[Cryptosystem|crittografato]] e incapsulato e viene generato un nuovo header IP per ogni pacchetto. Questo rende il processo trasparente per gli utenti finali e nasconde gli indirizzi di destinazione della rete interna, proteggendo l'identità della rete stessa. All'interno di IPSec, il protocollo ESP è cruciale per le VPN perché fornisce confidenzialità (crittografia), oltre all'autenticazione e all'integrità. I pacchetti viaggiano dalla rete interna a un gateway; qui vengono criptati (inclusi gli header originali) e inviati attraverso Internet. Il gateway ricevente decifra il pacchetto e inoltra l'originale all'indirizzo interno di destinazione.

# Vantaggi

- **Riduzione dei costi:** Eliminano la necessità di linee dedicate costose (leased lines) sfruttando la connettività Internet pubblica.
- **Scalabilità e Mobilità:** Facilitano l'aggiunta di utenti e supportano il lavoro mobile.
- **Sicurezza:** Forniscono riservatezza, autenticità e integrità, nascondendo la topologia della rete interna.

# Svantaggi e Rischi

- **Single Point of Failure:** Un guasto nel gateway VPN può disconnettere l'intera rete o creare colli di bottiglia nelle prestazioni.
- **Opacità del traffico:** Il traffico nel tunnel è crittografato e quindi spesso non rilevabile dai sistemi di rilevamento intrusioni (IDS).
- **Compromissione del Gateway:** Se il gateway VPN viene compromesso, i dati protetti vengono esposti.
- **Complessità:** Protocolli come IPSec sono definiti "molto complessi", e la complessità è citata come "il più grande nemico della sicurezza".
- **Mancanza di standard:** Difficoltà nell'accomodare prodotti di venditori diversi