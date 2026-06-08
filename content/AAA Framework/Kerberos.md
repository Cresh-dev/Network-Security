Kerberos è un protocollo di autenticazione di rete progettato per permettere a client e server di provare la loro identità reciproca in modo sicuro su una rete non sicura. È la base di come funzionano i login in molti sistemi aziendali (come Windows Active Directory). L'idea fondamentale è: **non inviare mai la password attraverso la rete**. Invece, si usano dei "biglietti" (tickets) crittografati.

## L'architettura di Kerberos

- **Realm:** L'ambiente Kerberos costituito da utenti, server e KDC.
- **Principals:** Gli utenti o servizi registrati, identificati nella forma `primary/instance@REALM` (es. `alessio/admin@POLIBA.IT`)
- **KDC:** Il centro di distribuzione chiavi che include due funzioni logiche distinte: l'Authentication Server (AS) e il Ticket Granting Server (TGS)

## Il Funzionamento del Protocollo (V4)

![[Screenshot 2025-12-09 at 09.42.48.png]]

- **Fase 1**: Autenticazione Iniziale: _Succede una volta sola quando accendi il computer_
	- **Passaggio 1:** L'utente si siede alla workstation e inserisce il nome utente. La workstation invia una richiesta all'**Authentication Server (AS)** dicendo: "Ciao, sono l'Utente X e vorrei accedere".
	- **Passaggio 2:** L'AS controlla nel database se l'utente esiste. Se sì, crea due cose:
	    1. Una **Chiave di Sessione** temporanea.
	    2. Un **Ticket-Granting Ticket (TGT)**.
	    - L'AS [[Cryptosystem|cripta]] tutto questo usando una chiave derivata dalla **password dell'utente** (che l'AS conosce). Questo pacchetto viene inviato indietro alla workstation.
	- **Passaggio 3 (prima parte):** La workstation riceve il pacchetto. Chiede all'utente la password. Usa la password digitata per decriptare il messaggio. Se la password è giusta, il pacchetto si apre e la workstation ottiene il TGT. _Se la password fosse sbagliata, il pacchetto non si aprirebbe._
- **Fase 2**: Richiesta del Servizio: _Succede ogni volta che vuoi usare un nuovo servizio, es. la mail o la stampante_
	- **Passaggio 3 (seconda parte):** Ora l'utente vuole usare un servizio (es. un File Server). La workstation invia al **Ticket-Granting Server (TGS)**:
	    1. Il **TGT** (che prova che l'utente si è già autenticato).
	    2. Il nome del servizio richiesto.
	    3. Un "Autenticatore" (con orario e nome utente, per sicurezza).
	- **Passaggio 4:** Il TGS decripta il TGT (perché è stato criptato con una chiave che solo il TGS conosce). Controlla che sia valido. Se tutto è ok, crea un **Ticket per il Servizio** specifico richiesto e lo invia alla workstation.
- **Fase 3**: Accesso al Servizio: _Succede quando ti connetti effettivamente_
	- **Passaggio 5:** La workstation ora ha il biglietto specifico per quel server. Lo invia al **Server** di destinazione insieme a un nuovo autenticatore. 
	- **Passaggio 6:** Il Server riceve il biglietto, verifica che sia autentico (perché criptato con una chiave che il Server condivide con Kerberos) e che l'autenticatore corrisponda.
	    - Se tutto corrisponde, il Server **garantisce l'accesso**.


> [!Autenticatore]
> L'autenticatore è un piccolo pacchetto di dati che la workstation genera e che contiene principalmente due informazioni:
> - Il **Nome Utente** (Client ID).
> - L'**Orario esatto** (Timestamp) in cui si sta mandando il messaggio (es. 09:30:05).
> 
> Questo pacchetto viene **criptato** con la _Chiave di Sessione_ (una chiave segreta che conoscono solo la workstation e il server, grazie al passaggio precedente).

### Flusso dei messaggi

![[Screenshot 2025-12-09 at 10.01.44.png]]

#### (a) Fase di Autenticazione (Ottenere il TGT)

Qui è dove si fa il login.

- **(1) $C \rightarrow AS$**: Il client invia il suo $ID$ e l'orario ($TS_1$) in chiaro. Dice: "Sono io, voglio entrare".
- **(2) $AS \rightarrow C$**: L'AS risponde con un messaggio cifrato usando la chiave $K_c$ (derivata dalla nostra password).
    - **Cosa c'è dentro il pacchetto?** Una nuova **chiave di sessione** ($K_{c,tgs}$) da usare per parlare col TGS dopo, e il **Ticket TGT** ($Ticket_{tgs}$).

> [!Note] Cifratura del ticket _tgs_
>  Il $Ticket_{tgs}$ è cifrato con la chiave del TGS ($K_{tgs}$). Il Client non può leggerlo, lo conserva e basta. L'AS dà una "chiave per il prossimo passaggio" e una "busta chiusa" (il Ticket) che solo il TGS può aprire.

#### (b) Fase TGS (Ottenere il Ticket per il Servizio)

Qui usi il TGT per chiedere l'accesso a un servizio specifico (chiamato $V$).

- **(3) $C \rightarrow TGS$**: Inviamo tre cose:
    1. L'ID del servizio che vogliamo ($ID_v$).
    2. Il **Ticket TGT** ($Ticket_{tgs}$) che abbiamo ricevuto prima (ancora cifrato).
    3. L'**Authenticator**: Contiene il nome e l'orario, cifrati con la chiave di sessione ($K_{c,tgs}$) che ha dato l'AS nel passaggio 2.
- **(4) $TGS \rightarrow C$**: Il TGS decifra il ticket, vede che è valido, decifra l'autenticatore, controlla l'orario. Se tutto è ok, ci invia:
    - Una nuova chiave di sessione specifica per quel servizio ($K_{c,v}$).
    - Il **Ticket per il Servizio** ($Ticket_v$), cifrato con la chiave del server V ($K_v$).

#### (c) Fase Client/Server (Accedere al Servizio)

Finalmente la connessione alla risorsa (es. la stampante o il file server).

- **(5) $C \rightarrow V$**: Inviamo il **Ticket per il Servizio** ($Ticket_v$) e un nuovo **Authenticator** (cifrato con la nuova chiave $K_{c,v}$).
- **(6) $V \rightarrow C$**: **Mutua Autenticazione**.
    - Il server $V$ deve provare a noi che è quello vero.
    - Prende il nostro orario ($TS_5$) dall'autenticatore, **aggiunge 1**, lo cifra e te lo rimanda.
    - Perché $TS_5 + 1$? Dimostra che il server è stato in grado di decifrare il nostro messaggio, leggere l'orario e modificarlo. Solo il vero server (che possiede la chiave $K_v$) poteva aprire il Ticket per ottenere la chiave di sessione necessaria per leggere il nostro orario.

#### Le 3 chiavi fisse

**Chiave Utente ↔ Authentication Server (AS)**:

- **Cos'è:** È la chiave derivata dalla **password dell'utente**.
- **A cosa serve:** Quando ci si logga, l'AS usa questa chiave per criptare la prima risposta che invia. Solo il destinatario (che conosce la password) può decifrarla. Dimostra che "il destinatario è chi dice di essere" senza inviare la password al server.

**Chiave AS ↔ Ticket-Granting Server (TGS)**:

- **Cos'è:** Una chiave segreta condivisa tra i due componenti principali del KDC (Key Distribution Center).
- **A cosa serve:** Serve a proteggere il **TGT (Ticket Granting Ticket)**. Quando l'AS rilascia un TGT, lo cripta con questa chiave. Poiché l'utente non conosce questa chiave, non può manomettere il ticket, può solo consegnarlo al TGS così com'è.

**Chiave TGS ↔ Server Remoto (Service Key)**:

- **Cos'è:** Una chiave condivisa tra il sistema Kerberos e il servizio finale (es. un database, un server di file o una stampante).
- **A cosa serve:** Quando chiediamo di accedere a un servizio specifico, il TGS ci dà un "Ticket di Servizio" criptato con questa chiave. Solo il server di destinazione può aprirlo, confermando che il TGS ci ha effettivamente autorizzato.

#### Riassunto della "Matrioska"

Osserviamo che ci sono chiavi dentro chiavi:

1. Usiamo la **Password** ($K_c$) per sbloccare la **Chiave TGS** ($K_{c,tgs}$).
2. Usiamo la **Chiave TGS** per sbloccare la **Chiave Servizio** ($K_{c,v}$).
3. Usiamo la **Chiave Servizio** per parlare col Server finale.

![[Screenshot 2026-02-08 at 11.43.44.png]]

### Kerberos v4 con multirealm

In presenza di più realm Kerberos (scenario multi-realm) può sorgere una problematica specifica. Questa si manifesta quando un utente deve accedere a un server che appartiene a un realm differente dal proprio. Per gestire tale situazione, lo scenario viene realizzato estendendo i meccanismi già descritti, introducendo ulteriori interazioni tra i diversi realm, come illustrato di seguito.

![[Screenshot 2026-02-08 at 11.50.17.png]]

La differenza principale risiede nell’interazione (4), nella quale la chiave effimera utilizzata non è la **K<sub>c,v</sub>**, condivisa tra il client e il server remoto, bensì una nuova chiave effimera **K<sub>c,tgs</sub>**, condivisa tra il client e il Ticket Granting Server presente nel dominio di destinazione del server.

![[Screenshot 2026-02-08 at 11.56.42.png]]

## Struttura del protocollo V5

**Limiti della Versione 4:**

- **Dipendenza dall'algoritmo:** Richiedeva l'uso esclusivo del DES sulle chiavi fisse.
- **Dipendenza dal protocollo IP:** Usava indirizzi IP specifici, rendendolo rigido.
- **Lifetime limitato:** La durata dei ticket era codificata in unità di 5 minuti su 8 bit, limitando la validità a circa 21 ore.
- **Scalabilità:** In ambienti multi-realm, richiedeva scambi di chiavi complessi (N(N−1)/2) per far interoperare tutti i domini

**Miglioramenti nella Versione 5:**

- **Indipendenza crittografica:** Il testo cifrato è etichettato con un identificatore del tipo di crittografia, permettendo l'uso di qualsiasi algoritmo.
- **Standardizzazione:** Usa ASN.1 e BER per definire la struttura dei messaggi, eliminando ambiguità sull'ordinamento dei byte.
- **Flessibilità temporale:** I ticket includono tempi di inizio e fine espliciti, permettendo durate arbitrarie.
- **Delegation:** Permette l'inoltro delle credenziali (forwarding), consentendo a un client di far agire un altro host per suo conto.
- **Efficienza:** Elimina la doppia cifratura (presente nella V4) e introduce la negoziazione di chiavi di sotto-sessione per prevenire replay attack in connessioni successive.

|**Caratteristica**|**Kerberos v4**|**Kerberos v5 (Il tuo schema)**|
|---|---|---|
|**Crittografia**|Solo DES (Insicuro oggi)|Flessibile (AES, 3DES, RC4, ecc.)|
|**Durata Ticket**|Max ~21 ore (fisso)|Flessibile (Definita da Start/End time)|
|**Rinnovo Ticket**|Non possibile|Supportato (Renewable)|
|**Relazione tra Regni**|Solo diretta|Transitiva e Gerarchica|
|**Dipendenza Rete**|Fortemente legato a IP|Indipendente (usa ASN.1)|
|**Sicurezza Password**|Vulnerabile ad attacchi dizionario|Supporta Pre-Authentication (per mitigare attacchi)|

### Flusso dei messaggi

![[Screenshot 2025-12-09 at 10.52.50.png]]

#### Fase (a): Autenticazione Iniziale (AS Exchange)

**Obiettivo:** L'utente fa il login e ottiene un "Passpartout" (chiamato TGT - Ticket Granting Ticket).

1. **$C \rightarrow AS$:** Il Client dice all'AS: "Ciao, sono l'utente $ID_c$. Voglio accedere al servizio TGS". Invia anche un numero casuale ($Nonce_1$) per sicurezza.
    - _**Nota**:_ Qui non viene inviata la password.
2. **$AS \rightarrow C$:** L'AS controlla nel suo database se l'utente esiste. Se sì, risponde con due cose importanti:
    - Una **Chiave di Sessione ($K_{c,tgs}$)** cifrata con la password dell'utente ($K_c$). L'utente deve digitare la password corretta per decifrare questo pacchetto e ottenere la chiave.
    - Il **Ticket TGS ($Ticket_{tgs}$)**. Questo è il "Passpartout". È cifrato con la chiave segreta del TGS, quindi l'utente non può leggerlo né modificarlo, può solo conservarlo.

#### Fase (b): Richiesta del Servizio (TGS Exchange)

**Obiettivo:** Usare il "Passpartout" per ottenere il biglietto specifico per il servizio desiderato (es. il server file $V$).

3. **$C \rightarrow TGS$:** Il Client invia al TGS:
    - Il nome del servizio che vuole ($ID_v$).
    - Il **Ticket TGS** (ricevuto prima).
    - Un **Authenticator**: Un piccolo pacchetto cifrato con la chiave di sessione che contiene l'ID e l'ora corrente ($Times$), per provare che è proprio lui a usare il ticket e non un hacker che lo ha rubato.
4. **$TGS \rightarrow C$:** Il TGS decifra il ticket, verifica l'Authenticator e, se tutto è corretto, invia:
    - Un **Ticket per il Servizio V ($Ticket_v$)**, cifrato con la chiave del server V (così solo il server V può leggerlo).
    - Una nuova chiave di sessione ($K_{c,v}$) per parlare in modo sicuro con il server V.

#### Fase (c): Accesso al Servizio (Client/Server Exchange)

**Obiettivo:** Accedere finalmente al server o alla risorsa.

5. **$C \rightarrow V$:** Il Client contatta il Server $V$ e invia:
    - Il **Ticket per V ($Ticket_v$)**.
    - Un nuovo **Authenticator** cifrato con la nuova chiave di sessione ($K_{c,v}$).
6. **$V \rightarrow C$:** Il Server $V$ decifra il ticket (usando la sua chiave segreta $K_v$), estrae la chiave di sessione e verifica l'Authenticator.
    - Se il messaggio (6) è presente (come nell'immagine), significa che c'è **Autenticazione Mutua**: il server risponde al client confermando l'orario ($TS_2$) per dire "Sì, sono veramente il server che cercavi e ho accettato il tuo ticket".

## Kerberos con crittografia a chiave pubblica

Il punto di partenza è che il protocollo Kerberos classico ha un tallone d'Achille: la sua sicurezza dipende interamente dalla robustezza delle password scelte dagli utenti. Se la password è debole, tutto il sistema è vulnerabile. Per risolvere questo problema, è stata introdotta l'estensione **PKINIT**, che cambia radicalmente il modo in cui avviene il primo contatto tra l'utente e il server. Invece di digitare una password, l'utente dimostra la propria identità utilizzando la crittografia a chiave pubblica, spesso tramite certificati digitali (come quelli che si trovano sulle Smart Card). Questo toglie all'utente il peso di dover gestire password complesse e si integra meglio con i moderni sistemi di sicurezza aziendali. Il funzionamento pratico è una sorta di dialogo sicuro. Quando un client vuole accedere, invia al server centrale (il KDC) il suo certificato digitale firmato. Il server controlla che il certificato sia autentico e valido. Una volta accertata l'identità, il server deve consegnare al client una "chiave di sessione" per poter comunicare in futuro. Per proteggere questo passaggio delicato, il server cifra questa chiave usando la chiave pubblica dell'utente (o un metodo matematico chiamato Diffie-Hellman), in modo che solo l'utente vero, che possiede la corrispondente chiave privata, possa "aprire il pacchetto" e leggerla. Tutto questo processo complesso e matematicamente pesante serve solo per l'ingresso iniziale. Appena l'utente ha decifrato la chiave di sessione, il sistema mette da parte la crittografia a chiave pubblica (che è lenta) e torna a usare la crittografia simmetrica standard, che è molto più veloce e performante, per tutto il resto della sessione di lavoro.