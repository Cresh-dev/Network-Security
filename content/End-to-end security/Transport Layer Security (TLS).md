SSL e TLS sono dei protocolli [[Cryptosystem|crittografici]] che forniscono l'autenticazione e la crittografia dei dati tra server, macchine e applicazioni che operano su una rete. SSL è il predecessore di TLS che è subentrato nel 1999. Questo protocollo è composto da 2 livelli:
- **TLS Handshake Protocol**: Negozia gli algoritmi e le modalità di protezione dei dati permettendoci di ottenere un canale sicuro per la comunicazione.
- **TLS Record Protocol**: Protegge i dati dopo e mette in pratica gli algoritmi negoziati nell'handshake.

![[Screenshot 2025-11-11 at 14.27.19.png]]

> [!Differenza tra sessione e connessione]
> La sessione è il **contesto di sicurezza** duraturo. Contiene le chiavi crittografiche e i parametri. Una singola Sessione viene stabilita una sola volta, per **evitare la rinegoziazione costosa** della sicurezza. La connessione è il **canale di comunicazione** temporaneo. Ogni nuova connessione **riutilizza lo stato di sicurezza** della Sessione esistente, rendendo i trasferimenti di dati successivi estremamente più veloci.

![[Screenshot 2025-11-11 at 14.14.36.png]]

## TLS 1.2 Handshake protocol

![[Screenshot 2026-02-06 at 17.27.55.png]]

- **Scambio dei Messaggi "Hello" e Accordo sugli Algoritmi:**
    - **Cliente e Server si salutano** ("Client Hello" e "Server Hello").
    - Si scambiano informazioni sulle **capacità crittografiche** supportate da ciascuno (ad esempio, quali algoritmi di cifratura, [[Hash function|hash]] e scambio di chiavi conoscono).
    - **Si accordano** su una suite crittografica comune da usare per la sessione.
    - Scambiano **valori casuali** (random values) che saranno usati in seguito per generare le chiavi di sessione.
    - Verificano se la sessione può essere **ripristinata** (session resumption) se si sono connessi di recente.
- **Scambio dei Parametri Crittografici Necessari:**
    - In questa fase, vengono scambiati i dati che permetteranno al client e al server di calcolare un **"premaster secret"** (un valore segreto provvisorio).
    - A seconda dell'algoritmo di scambio delle chiavi scelto (ad esempio, RSA o Diffie-Hellman), può essere scambiata la chiave pubblica del server o i parametri necessari per il calcolo.
- **Scambio di Certificati e Autenticazione:**
    - Il server invia il suo [[ITU X.509 certificates|certificato digitale]] al client. Questo certificato contiene la chiave pubblica del server e la verifica della sua identità da parte di un'Autorità di Certificazione (CA).
    - Il client verifica l'autenticità del certificato.
    - Questo passaggio permette al client e al server di **autenticarsi** reciprocamente (se è richiesta anche l'autenticazione del client, il client invia anch'esso il suo certificato).
- **Generazione del "Master Secret":**
    - Sia il client che il server, usando il **"pre-master secret"** (generato nel punto 2) e i **valori casuali** scambiati nel punto 1, calcolano in modo indipendente lo stesso **"master secret"**.
    - Il "master secret" è la chiave principale da cui verranno derivate tutte le chiavi di crittografia e autenticazione della sessione.
- **Fornitura dei Parametri di Sicurezza al Livello Record:**
    - Dal "master secret", vengono derivate le **chiavi simmetriche** (le "security parameters") che verranno effettivamente utilizzate per crittografare e decrittografare tutti i dati dell'applicazione che seguiranno (il "record layer").
- **Verifica e Conclusione:**
    - Client e server si inviano a vicenda un messaggio finale (**"Finished"**) che è crittografato usando le chiavi appena generate e contiene un hash di tutti i messaggi precedenti dell'handshake.   
    - Se l'altro lato riesce a decifrare il messaggio e l'hash corrisponde, significa che:
        - Ha calcolato le **stesse chiavi di sessione**.
        - L'intero handshake è avvenuto **senza manomissioni** (tampering) da parte di un attaccante.
    - A questo punto, l'handshake è completo, e la connessione è considerata **sicura**. Tutte le comunicazioni successive saranno crittografate con le nuove chiavi simmetriche.

### Stabilire il pre-master secret

Queste procedure servono a far sì che client e server condividano un primo valore segreto (il pre-master secret) in modo sicuro. Una volta ottenuto il pre-master secret tramite uno di questi metodi, sia il client che il server lo useranno (insieme ai numeri casuali scambiati all'inizio) per calcolare autonomamente il **master secret**.

- **Anonymous Diffie-Hellman**: Client e server scambiano parametri matematici per creare il segreto senza autenticarsi. È rischioso perché un attaccante può inserirsi nel mezzo senza farsi scoprire.
- **Fixed Diffie-Hellman**: I parametri per il calcolo del segreto sono fissi e salvati nei certificati. Questo genera una chiave segreta costante tra le due parti.
- **Ephemeral Diffie-Hellman (DHE)**: Le chiavi sono temporanee e valide per una sola sessione. Per sicurezza, vengono firmate con la chiave RSA del mittente così da garantirne l'autenticità.
- **RSA**: Il client genera il segreto, lo cripta con la chiave pubblica del server (presa dal certificato) e glielo spedisce direttamente.

### Generazioni delle chiavi

Il processo di generazione delle chiavi in TLS avviene in tre fasi sequenziali:

1. **Generazione del Pre-master Secret:** Durante l'handshake, client e server stabiliscono un valore segreto iniziale. La natura di questo valore dipende dall'algoritmo di scambio chiavi utilizzato.
2. **Calcolo del Master Secret:** Il pre-master secret viene elaborato dalla **PRF** (Pseudo-Random Function) insieme ai valori "random" scambiati in chiaro tra client e server. Il risultato è un segreto univoco di 48 byte chiamato **master_secret**.
3. **Espansione del Key Material:** Il master secret viene nuovamente processato dalla PRF (operazione di _key expansion_) per generare un blocco continuo di dati. Questo blocco viene suddiviso per ottenere le chiavi operative finali:
    - **Chiavi di cifratura (Write Keys):** Per criptare e decriptare i dati.
    - **MAC Secrets:** Per garantire l'integrità e l'autenticità dei messaggi.
    - **IV e Numeri di sequenza:** Per la gestione tecnica dei cifrari a blocchi e per prevenire attacchi di tipo "replay".

Lo schema **P_hash** visibile nell'immagine rappresenta proprio il funzionamento iterativo della PRF, che utilizza l'algoritmo HMAC per produrre un flusso di dati della lunghezza necessaria a coprire tutte queste chiavi.

![[Screenshot 2026-02-06 at 17.54.34.png]]

## TLS record protocol

==Il TLS Record Protocol è uno dei componenti fondamentali del protocollo TLS (Transport Layer Security). Si occupa di incapsulare, proteggere e trasportare i dati applicativi in modo sicuro tra client e server.==

![[Screenshot 2026-02-06 at 18.03.03.png]]

### Operazioni del record protocol 

È lo strato più basso di TLS, responsabile di:

1. **Spezzare i dati** provenienti dai livelli superiori (es. HTTP) in record di dimensione gestibile.
2. **Comprimerli** (opzionale, oggi quasi sempre disattivo).
3. **Autenticarli e cifrarli**, secondo gli algoritmi negoziati nell’handshake TLS.
4. **Trasmetterli** in rete in modo sicuro.
5. **Verificarli, decifrarli e ricomporli** sul lato ricevente

## TLS alert protocol

==Il TLS Alert Protocol è uno dei protocolli logici che compongono TLS insieme al Record Protocol e all’Handshake Protocol. Serve a segnalare condizioni di errore o cambi di stato durante una connessione TLS==. È un protocollo molto piccolo ma fondamentale perché permette alle due parti (client e server) di comunicare in modo standard cosa sta succedendo quando qualcosa non va o quando si vuole chiudere la connessione in modo corretto.

## TLS 1.3

Le innovazioni introdotte in **TLS 1.3** rappresentano un significativo passo avanti rispetto al suo predecessore, TLS 1.2, specialmente in termini di **sicurezza** e **efficienza**. ==TLS 1.3 è più sicuro di TLS 1.2 grazie all'adozione di suite di cifratura più robuste e sicure e all'eliminazione di funzionalità insicure. Un'innovazione fondamentale è l'introduzione obbligatoria della perfetta segretezza in avanti (Perfect Forward Secrecy, PFS)==:

- **Problema in TLS 1.2:** TLS 1.2 non forniva PFS completa, specialmente se si utilizzava l'accordo chiave basato su RSA, il quale non offre una modalità di chiave effimera (temporanea). Se le chiavi pubbliche/private venivano compromesse, un attaccante che aveva archiviato i messaggi cifrati in precedenza era in grado di calcolare la material key e decrittare la sessione passata.
- **Soluzione in TLS 1.3:** In TLS 1.3, la material key viene derivata tramite **Diffie-Hellman effimero (ephemeral DH)**. La conoscenza di una chiave a lungo termine (tipicamente conservata nei certificati X.509) non consente di calcolare le chiavi di sessione generate in passato. Questo è possibile perché la negoziazione delle chiavi e l'autenticazione sono gestite separatamente.