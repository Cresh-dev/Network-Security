==IPsec (Internet Protocol Security) è un insieme di protocolli che serve a proteggere le comunicazioni IP (IPv4 e IPv6)==. È usato soprattutto per creare **VPN sicure**, ma può proteggere qualunque traffico IP. IPsec lavora a **livello 3 (Rete)** del modello OSI, quindi può proteggere tutto ciò che viaggia sopra (TCP, UDP, applicazioni, ecc. IPsec garantisce quattro cose fondamentali:

1. **Confidenzialità** → cifra i dati (nessuno può leggerli).
2. **Integrità** → verifica che i dati non siano stati modificati.
3. **Autenticazione** → verifica l’identità delle parti coinvolte.
4. **Anti-replay** → impedisce di riutilizzare pacchetti vecchi per attacchi.

Prima di introdurre modalità e protocolli dobbiamo parlare della **Security Association (SA)** la quale è una _connessione logica monodirezionale_ tra un mittente e un destinatario, utilizzata in IPsec per applicare servizi di sicurezza al traffico di rete (es. autenticazione, cifratura). 

![[Screenshot 2026-02-07 at 09.41.40.png]]

IPsec ha due protocolli principali:

- **AH (Authentication Header)** → autentica il pacchetto e garantisce integrità. 
- **ESP (Encapsulating Security Payload)** → cifra il contenuto e può anche autenticare.

Una **SA può usare solo uno dei due**, non entrambi contemporaneamente:

- SA per AH **oppure**
- SA per ESP

Poiché un’SA è _monodirezionale_, se due host devono comunicare in entrambe le direzioni in modo sicuro si crea prima **un’SA per il traffico A → B** e poi **un’altra SA per il traffico B → A**. Quindi, la comunicazione sicura completa richiede **almeno due SA**. Il funzionamento di IPsec si basa su due database principali:

1. **SPD (Security Policy Database):** Contiene le regole che decidono **cosa fare** con i pacchetti (se proteggerli con IPsec, scartarli o inviarli senza protezione).
2. **SAD (Security Association Database):** Memorizza i parametri tecnici di ogni connessione sicura (**SA**).

Ogni **Security Association (SA)** è identificata univocamente da tre elementi:

- **Indirizzo IP di destinazione.**
- **Protocollo di sicurezza** (AH o ESP).
- **SPI (Security Parameters Index):** Un identificativo a 32 bit che permette al destinatario di selezionare l'esatta SA per elaborare correttamente il pacchetto ricevuto.

## Modalità operative di IPsec

IPsec può funzionare in due modi:

- **Modalità Transport**:
	- Protegge _solo il payload IP_, non l’intero pacchetto.
	- Il vero indirizzo IP del mittente rimane visibile.
	- Usata tra **host e host** (es.: server to server).
- **Modalità Tunnel**:
	- Incapsula l’intero pacchetto IP dentro un nuovo pacchetto IPsec.
	- Indirizzi originali _nascosti_ e protetti.
	- Usata nelle **VPN site-to-site** o **client-to-site**.

## Componenti principali di IPsec

IPsec usa tre blocchi principali:

- **Protocollo AH (Authentication Header)**:
	- Garantisce **integrità e autenticazione**.
	- **Non cifra** i dati.
	- Oggi è usato raramente (quasi tutto è fatto con ESP).
- **Protocollo ESP (Encapsulating Security Payload)**:
	- Cifra i dati (**confidenzialità**).
	- Garantisce integrità e autenticazione.
	- È quello usato quasi sempre nelle VPN.
- **IKE (Internet Key Exchange)**:
	- Negozia algoritmi
	- Scambia chiavi (Diffie–Hellman)
	- Stabilisce le SA (Security Association)

Versioni:

- **IKEv1** (vecchio)
- **IKEv2** (attuale, più sicuro e semplice)

## AH

**AH (Authentication Header)** è il protocollo IPsec che fornisce **autenticazione, integrità e protezione antireplay**, ma **non cifra** il contenuto dei pacchetti. Garantisce che il pacchetto non sia stato modificato durante il tragitto. Usa **numeri di sequenza** e una finestra scorrevole per prevenire ritrasmissioni malevole.

> [!Limiti]
> - **NON offre confidenzialità** (nessuna crittografia)
> - Problemi con **NAT**
> - Quasi sempre sostituito da ESP

Questo protocollo garantisce l'integrità dei dati tramite il **campo ICV** (Integrity check value) che è l'impronta crittografica del pacchetto. Questo campo garantisce che il pacchetto non sia stato manomesso e utilizza una [[Cryptosystem|crittografia]] a chiave simmetrica che viene calcolata su campi non modificabili. 

![[Screenshot 2026-02-07 at 10.01.30.png]]

> [!NOTE] Come AH garantisce l'Integrità (L'impronta digitale)
> - **Il calcolo del MAC:** Prima di inviare il pacchetto, il mittente prende quasi tutto il pacchetto (Header IP + Header AH + Payload) e lo passa attraverso un algoritmo di hashing (come SHA-256) insieme a una **chiave segreta** condivisa.
> - **Il risultato (ICV):** Il risultato è un numero fisso chiamato **ICV (Integrity Check Value)**, che viene scritto dentro l'header AH.
> - **La verifica:** Quando il destinatario riceve il pacchetto, rifà lo stesso calcolo. Se il suo risultato è identico a quello scritto nell'header AH, significa che **nessun bit** del pacchetto è stato cambiato durante il viaggio. Se anche solo un bit del mittente fosse stato alterato, il calcolo non tornerebbe.

### Come funziona la sliding window

Immaginiamo una finestra numerata, ad esempio di **64 slot** (valore tipico ma configurabile). Questa finestra rappresenta un intervallo di numeri di sequenza **accettabili** in un dato momento. Il sistema controlla:

- **se è già stato ricevuto** → RIFIUTATO (probabile replay) 
- **se non è stato ricevuto** → ACCETTATO e il relativo bit nella finestra viene segnato come _ricevuto_

![[Screenshot 2026-02-07 at 10.03.41.png]]

> [!NOTE] Resistenza all'attacco di replay
> Ogni pacchetto inviato riceve un numero progressivo (1, 2, 3...). Il destinatario tiene traccia dei numeri già ricevuti tramite una "finestra scorrevole" nel **SAD**. Se l'attaccante rimanda il pacchetto n. 1 quando il destinatario è già arrivato al n. 50, il pacchetto viene **scartato immediatamente** perché considerato un duplicato (replay).

## ESP – Encapsulating Security Payload

**ESP (Encapsulating Security Payload)** è il componente più utilizzato di IPsec.  
Fornisce **confidenzialità**, **integrità**, **autenticazione opzionale** e **protezione antireplay**.

### Funzioni principali

- **Crittografia per nascondere i dati, tramite algoritmi come**:
	- AES-CBC
	- AES-GCM (modernissimo: include autenticazione integrata)
	- 3DES (deprecato)
- **Integrità (opzionale in alcune modalità)**: Tramite HMAC-SHA1/SHA2 o integrità integrata in AES-GCM.
- **Autenticazione**: Verifica il mittente, spesso tramite un MAC (Message Authentication Code).
- **Protezione antireplay**: Come AH, usa numeri di sequenza.

![[Screenshot 2026-02-07 at 10.13.41.png]]

> [!NOTE] Come ESP garantisce la Riservatezza
> - **Cifratura:** Il payload originale (ad esempio un segmento TCP contenente una mail) viene preso e trasformato in un ammasso di dati illeggibili usando un algoritmo di cifratura simmetrica (come **AES**) e una chiave segreta.
> - **Incapsulamento:** Questo blocco cifrato viene messo "in mezzo" tra un nuovo **ESP Header** (che serve a gestire la connessione) e un **ESP Trailer** (che serve a dare la giusta lunghezza ai dati).
> - **Autenticazione:** Infine, ESP aggiunge in coda un campo di autenticazione (simile ad AH) per assicurarsi che nessuno abbia manomesso il "lucchetto".

## AH + ESP

Per ottenere simultaneamente la confidenzialità (tipica di ESP) e la protezione dell'intero header IP (tipica di AH), è necessario concatenare più SA in una sequenza definita "iterated layering". Utilizzando la **Transport Adjacency**, si applica AH sopra ESP in modalità trasporto, estendendo l'autenticazione all'intestazione IP originale per prevenire attacchi di spoofing. Nel **Transport-tunnel bundle**, invece, si inserisce un'autenticazione end-to-end (AH in modalità trasporto) all'interno di un tunnel cifrato (ESP in modalità tunnel) stabilito tra gateway, garantendo così sia la sicurezza della tratta intermedia che l'autenticità della sorgente originale.

## IKE

**IKE (Internet Key Exchange)** è il protocollo che "gestisce" la sicurezza in IPsec.

Serve per:

- autenticare le parti (client e server)
- negoziare parametri di sicurezza
- scambiare chiavi crittografiche
- creare e gestire le **SA (Security Associations)**

IKE è un **protocollo a due fasi**. Il suo obiettivo finale non è trasportare i dati (quello lo fanno ESP/AH), ma costruire un canale sicuro affinché i router possano mettersi d'accordo sulle chiavi senza che nessuno li ascolti.

### Le fasi del protocollo IKE

**Fase 1 in modalità principale**: Creare il canale di gestione. In questa fase, i due dispositivi (es. due firewall) non si fidano ancora l'uno dell'altro.

- **Obiettivo:** Autenticarsi e creare un primo tunnel sicuro criptato _solo per parlare tra loro_.
- **Cosa succede:**
    1. Si scambiano le proposte (algoritmi, durata vita, gruppo DH).
    2. Eseguono lo scambio Diffie-Hellman per generare le chiavi.
    3. Si autenticano (tramite Pre-Shared Key o Certificati Digitali).
- **Risultato:** Viene creata una **IKE SA** (Security Association). È un tunnel bidirezionale usato _solo_ per negoziare la fase successiva.

![[Screenshot 2026-02-07 at 10.47.01.png]]

**Fase 1 con modalità aggressiva**: nel caso in cui due peer si conoscano già, è possibile semplificare la procedura scambiando solo 3 messaggi invece dei precedenti sei. Tuttavia, vi è anche una negoziazione limitata degli algoritmi crittografici e il rischio di un man in the middle.

**Fase 2**: Creare i tunnel per i dati. Ora che i firewall hanno un canale sicuro (creato nella Fase 1), possono mettersi d'accordo su come proteggere i dati veri e propri degli utenti.

- **Obiettivo:** Negoziare le SA per **ESP** o **AH**.
- **Cosa succede:** All'interno del tunnel protetto della Fase 1, i dispositivi negoziano rapidamente quali algoritmi usare per i dati (es. "Per i dati usano AES-256 e SHA-256").
- **Risultato:** Vengono create le **IPsec SA**.
    - A differenza della Fase 1, queste sono **unidirezionali**. Ne serve una per l'andata e una per il ritorno.

![[Screenshot 2026-02-07 at 10.50.28.png]]

## NAT e IPSec

Con un NAT nella rete, IPsec non può funzionare direttamente, il problema nasce dal fatto che il NAT modifica gli indirizzi IP dei pacchetti, mentre IPsec nasce per proteggerli: il protocollo AH vede la modifica come una manomissione e si blocca, mentre ESP fatica a gestire il ricalcolo dei checksum. La soluzione è il **NAT-Traversal (NAT-T)**, che risolve il conflitto incapsulando i dati IPsec dentro pacchetti **UDP sulla porta 4500**. Durante la fase di negoziazione (IKE), i due dispositivi si scambiano degli hash per rilevare se ci sia un NAT nel percorso; se lo trovano, iniziano a "impacchettare" tutto il traffico dentro UDP, permettendo così al router NAT di modificare le porte e gli IP senza corrompere la sicurezza della VPN.