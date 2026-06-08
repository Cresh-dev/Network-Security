L'autenticazione può essere **one-way** (solo un'entità prova la propria identità) o **mutual** (entrambe le entità si autenticano a vicenda). Il processo tipico prevede l'ottenimento di credenziali (password, certificati, biometria), la loro verifica rispetto a un riferimento memorizzato e la conferma dell'identità prima di procedere all'autorizzazione.

## Fattori e Metodologie di Autenticazione

Possiamo classificare i meccanismi di autenticazione in tre categorie principali, basate su ciò che l'individuo "sa", "possiede" o "è":

- **Ciò che sai:** Si basa su segreti come password o PIN.
	-  _Problemi:_ Le password non devono essere salvate in chiaro ma come hash. Sono vulnerabili ad attacchi a dizionario (mitigabili con l'uso del "salt" per aumentare lo spazio degli input dell'hash).
- **Ciò che possiedi:** Include token, smart card o chiavi crittografiche.
	-  _One-Time Password (OTP):_ Una password valida per una sola sessione, utilizzata per prevenire attacchi di _replay_.
- **Ciò che sei:** Caratteristiche fisiche come impronte digitali o voce.
	- _Limiti:_ Possibilità di falsi positivi/negativi e difficoltà nella revoca delle credenziali (non puoi cambiare la tua impronta digitale come una password).

## Sicurezza nell'Autenticazione e Gestione delle Password

Questa analisi copre le tecniche per proteggere le credenziali degli utenti (mitigazione) e le vulnerabilità intrinseche dei protocolli di autenticazione "Challenge-Response".

### Strategie di Mitigazione per le Password

Per aumentare la sicurezza rispetto alla semplice memorizzazione delle password, si adottano tre livelli di mitigazione:

#### Prima Mitigazione: Hashing e Salting

Non si deve **mai** memorizzare la password in chiaro, né il suo semplice hash. La pratica corretta prevede l'uso di un **Salt** (un numero casuale $S$).

- **Meccanismo:** Nel database viene salvato il risultato della funzione: $H(password, S)$.
- **Funzionamento:** Quando l'utente inserisce la password in chiaro, il server recupera il _Salt_ associato a quell'utente, ricalcola l'hash e lo confronta con quello memorizzato.
- **Vantaggio:** Questo metodo rende il sistema molto più robusto contro gli **attacchi a dizionario** e le _rainbow tables_, poiché costringe l'attaccante a ricalcolare gli hash per ogni singolo _salt_.

#### Seconda Mitigazione: Complessità

Imporre l'utilizzo di una **password complessa** (lunghezza, caratteri speciali, ecc.) per rendere più difficile indovinarla tramite _brute-force_.

#### Terza Mitigazione: Autenticazione a Due Fattori (2FA)

Implementare un secondo livello di verifica oltre alla password (es. un codice temporaneo o un token fisico).
### Protocolli di Autenticazione

Esistono diversi protocolli standard per verificare l'identità di un utente, con diversi livelli di sicurezza:

- **PAP (Password Authentication Protocol):**
    - _Livello:_ Basso.
    - _Funzionamento:_ La password viene inviata in **chiaro** sulla rete. È vulnerabile allo sniffing.
- **CHAP (Challenge Handshake Authentication Protocol):**
    - _Livello:_ Medio (migliore di PAP, ma considerato "debole" rispetto a standard moderni avanzati).
    - _Funzionamento:_ La password **non** viene mai inviata. Si utilizza un meccanismo di _Challenge-Response_ (sfida-risposta).

### Il Meccanismo Challenge-Response e le sue Vulnerabilità

L'approccio più sicuro per l'autenticazione crittografica è il **Challenge-Response**.

- **Concetto base:** Un peer (es. il Server) invia una "sfida" (un numero casuale cifrato o meno) all'altro peer (es. il Client). Il destinatario deve decifrare o firmare la sfida e rispedirla indietro per dimostrare di possedere la chiave segreta, senza mai trasmettere la chiave stessa.

Tuttavia, questo meccanismo è soggetto a diversi attacchi specifici se non implementato correttamente:

#### Attacco Replay (Ripetizione)

Un attaccante intercetta la "sfida" e la "risposta" corretta di un utente legittimo. In futuro, se il server ripropone la stessa sfida, l'attaccante può inviare la vecchia risposta registrata per autenticarsi senza conoscere la chiave.

- **Soluzione:** Utilizzare **Timestamp**, **Nonce** (numeri usati una sola volta) o **Numeri di Sequenza** per garantire che ogni sfida sia unica e non riutilizzabile.

#### Attacco Chosen-Ciphertext (Testo cifrato scelto)

Il nodo malevolo invia un testo cifrato casuale alla vittima per analizzare come questa risponde o decifra il messaggio (crittoanalisi sulla risposta correlata).

![[Screenshot 2026-02-07 at 16.07.17.png]]

- **Soluzione:** Evitare l'uso di numeri casuali aggiuntivi prevedibili o semplici; il protocollo deve essere strutturato in modo che il server invii un "testimone" (es. hash del numero casuale) per provare di essere l'effettivo generatore del messaggio cifrato.

#### Attacco Reflection (Riflessione)

L'attaccante apre sessioni parallele con il server. Quando il server invia una sfida all'attaccante, quest'ultimo (non conoscendo la chiave) invia la stessa sfida _indietro_ al server (o a un altro client legittimo) in una seconda sessione. Il server risolve la sfida per lui, e l'attaccante usa quella risposta per autenticarsi nella prima sessione.

![[Screenshot 2026-02-07 at 16.08.33.png]]

- **Contesto:** Spesso accade quando il primo messaggio è in chiaro e si richiede una risposta cifrata.
- **Soluzione:** Il server deve includere prove della propria identità (testimone) o usare chiavi diverse per le due direzioni di comunicazione.

#### Attacco Interleaving (Intreccio)

Si tratta di un'evoluzione sofisticata dell'attacco _Reflection_. L'obiettivo dell'attaccante è dimostrare di essere autentico sfruttando un client legittimo (A) e un server (B).

![[Screenshot 2026-02-07 at 16.09.10.png]]

1. Il dispositivo maligno riceve una sfida da B.
2. Non sapendo risolverla, la inoltra al client A (che crede di parlare col server).
3. Il client A risponde correttamente.
4. L'attaccante raccoglie la risposta e la invia al server B, ma manipolando l'ordine o scambiando due numeri casuali.
5. Il server accetta l'autenticazione perché riceve un testo cifrato che contiene validi numeri casuali (il proprio e un altro).

- **Soluzione:** Combinare i nonce (numeri casuali) nella risposta in ordine differente. 