il protocollo **RADIUS** (Remote Authentication Dial-In User Service) emerge come lo standard _de facto_ per l'autenticazione remota e l'implementazione del framework **AAA** (Authentication, Authorization, Accounting). 

# Architettura e Ruolo nel Framework AAA

RADIUS utilizza **UDP** come protocollo di trasporto. Questa scelta è dettata dalla necessità di gestire tempistiche diverse rispetto al TCP e dalla natura _stateless_ del protocollo (client e server possono apparire e scomparire rapidamente). RADIUS implementa il modello **Client-Server**, ma con una distinzione fondamentale rispetto all'interazione utente classica:

- **Client RADIUS:** Non è l'utente finale, bensì il **NAS** (Network Access Server), ovvero il dispositivo che funge da punto di accesso alla rete (es. un access point o un server [[Virtual Private Networks (VPNs)|VPN]]).
- **Server RADIUS:** È l'entità che gestisce il database delle credenziali e le policy di accesso.

![[Screenshot 2026-02-07 at 16.57.15.png]]

l protocollo supporta tutte e tre le funzioni del framework AAA:

1. **Authentication:** Verifica l'identità dell'utente tramite diversi metodi (PAP, CHAP, EAP).
2. **Authorization:** Fornisce i parametri per il servizio (es. indirizzo IP, maschera di rete, tempo di sessione) tramite i pacchetti _Access-Accept_.
3. **Accounting:** Traccia l'attività dell'utente (inizio, fine e aggiornamenti intermedi della sessione) per scopi di auditing o fatturazione, utilizzando una porta UDP distinta (1813) rispetto all'autenticazione (1812).

> [!NOTE] Utilizzo di crittografia simmetrica
> Tutti i messaggi scambiati nel framework RADIUS sono [[Cryptosystem|crittografati]] con una chiave simmetrica.

# Metodi di Autenticazione Supportati

**PAP (Password Authentication Protocol):** È un metodo di autenticazione debole. La password è vulnerabile a intercettazioni e replay attack se non protetta da un canale sicuro sottostante. Il NAS riceve le credenziali dell'utente, ma non possedendo il database degli utenti, non sa se le credenziali sono corrette. Impacchetta le credenziali in un messaggio tramite una funzione [[Hash function|hash]] MD5 chiamata Access-Request che invia al Server. Il server fornisce in risposta il messaggio di accettazione o rifiuto.

![[Screenshot 2026-02-07 at 17.04.03.png]]

**CHAP (Challenge Handshake Authentication Protocol):** Più sicuro del PAP, utilizza un meccanismo di _challenge-response_ a tre vie. La password non viene trasmessa; il client dimostra di conoscerla rispondendo a una sfida del server con un hash MD5. Tuttavia, richiede che il server conservi le password in chiaro (o in modo reversibile) per verificare la risposta. Abbiamo le seguenti fasi:

- **Richiesta Iniziale:** L'utente invia le proprie credenziali al **NAS** (Network Access Server).
- **Inoltro al Server RADIUS:** Il NAS invia una richiesta di accesso (**Access-Request**) al server RADIUS contenente lo username e una password temporanea convenzionale (es. "Challenge").
- **Emissione della Challenge:** Il server RADIUS genera e invia una **"Sfida" (Challenge)** al NAS, che a sua volta la inoltra all'utente.
- **Risposta dell'Utente:** L'utente calcola e invia la **risposta** alla sfida ricevuta.
- **Validazione Finale:** Il NAS crea un nuovo messaggio di Access-Request includendo la risposta dell'utente. Il server RADIUS verifica la validità della risposta e decide se **autorizzare o negare** l'accesso.

![[Screenshot 2026-02-07 at 17.07.26.png]]

# Pacchetto RADIUS

![[Screenshot 2026-02-07 at 17.20.53.png]]

Il pacchetto ha un'intestazione (**header**) fissa di **20 byte**, seguita da un payload variabile. I campi principali sono:

- **Codice:** Definisce il tipo di operazione (es. richiesta di accesso o accettazione).
- **Identificatore:** Serve per accoppiare correttamente ogni richiesta alla sua risposta.
- **Lunghezza:** Indica la dimensione totale del pacchetto.
- **Autenticatore:** Un campo di 16 byte usato per la sicurezza (spiegato sotto).
- **Attributi:** Contengono i dati specifici (utente, password, permessi) in formato **TLV** (Type-Length-Value).

## Meccanismi di Sicurezza

Il protocollo non invia mai le informazioni sensibili in chiaro, utilizzando un **Segreto Condiviso ($S$)** noto solo al client (NAS) e al server.

### Protezione della Password

La password dell'utente ($P$) viene cifrata prima dell'invio tramite un'operazione di XOR ($\oplus$):

$$\text{User-Password} = P \oplus \text{MD5}(S + RA)$$

- Il NAS genera un numero casuale chiamato **Request Authenticator ($RA$)**.
- Viene creato un hash MD5 unendo il segreto e il numero casuale.
- Il risultato viene combinato con la password. In questo modo, la password cambia "aspetto" a ogni sessione anche se l'utente usa sempre la stessa.

### Autenticatore di Risposta

Per garantire che la risposta provenga davvero dal server autorizzato e non sia stata manomessa, il server calcola un codice di verifica:

$$\text{Auth}_{Resp} = \text{MD5}(\text{Codice}|\text{ID}|\text{Lunghezza}|RA|\text{Attributi}_{Resp}|S)$$

Questo valore concatena tutti i dati del pacchetto con il segreto condiviso, rendendo impossibile per un attaccante falsificare la risposta senza conoscere $S$.

# RADIUS accounting procedure

![[Screenshot 2026-02-07 at 17.41.55.png]]

## Fase di Login (Authentication & Authorization)

1. **Access-Request**: L'utente prova a connettersi. Il NAS invia le credenziali (cifrate) al server.
2. **Access-Accept**: Se le credenziali sono corrette, il server risponde positivamente inviando anche i parametri di configurazione (indirizzo IP, maschera di rete, tempo massimo di sessione).
## Fase di Accounting (Il "Registro")

3. **Accounting-Request (Start)**: Appena l'utente inizia a navigare, il NAS comunica al server che la sessione è ufficialmente iniziata.
4. **Accounting-Response**: Il server conferma di aver registrato l'inizio.
5. **Accounting-Request (Stop)**: Quando l'utente si disconnette, il NAS invia un messaggio di "Stop" contenente i dati finali (es. durata totale, byte trasmessi).
6. **Accounting-Response**: Il server conferma la ricezione e chiude il log.
# Scalabilità e Roaming (Proxy RADIUS)

Un tema rilevante trattato è l'uso di RADIUS in scenari distribuiti complessi tramite il meccanismo di **Proxy**. Invece di avere ogni rete locale che autentica direttamente gli utenti, si utilizza un **server remoto centrale** (Home Network) che detiene le credenziali reali. Questo permette a un utente (nell'esempio "Alice") di accedere a una rete esterna ("Foreign Network") usando le proprie credenziali di casa.

![[Screenshot 2026-02-07 at 17.43.21.png]]

1. Alice invia le credenziali al **NAS**.
2. Il NAS invia un pacchetto **Access-Request** al Proxy RADIUS locale.
3. Il Proxy inoltra l'**Access-Request** al Server RADIUS della Home Network.
4. Il server di casa verifica le credenziali e invia un **Reply** (Accept o Reject) al Proxy.
5. Il Proxy inoltra la risposta al NAS, che permette o nega l'accesso ad Alice.