==Il crypto-system è un sistema di cifratura che utilizza degli algoritmi matematici e chiavi per proteggere dati, rendendoli illeggibili a chi non è autorizzato==. Partiamo con il definire il seguente schema base per capire come è strutturato questo sistema:

![[Screenshot 2025-10-16 at 17.57.38.png]]

> [!Funzionamento dei Cryptosystem]
> Possiamo osservare dallo schema che abbiamo un messaggio ($m$) che vogliamo trasferire ad un altra persona, per far sì che il nostro messaggio non venga letto da nessun'altro utilizziamo una chiave ($k_1$) per cifrare tramite un algoritmo il messaggio ($c=E_{k1}(m)$) e successivamente lo trasmettiamo sul canale. Una volta arrivato a destinazione, il destinatario non può leggere il messaggio (perché è cifrato) finché non utilizza una chiave ($k_2$) che permette di decifrarlo tramite un algoritmo ($m=D_{k2}(c)$).

Nei Cryptosystem se $k_1$ è diversa da $k_2$ parliamo di crittografia asimmetrica, se sono uguali ci riferiamo alla crittografia simmetrica. A seconda di come viene gestito il flusso del messaggio $m$, la trasformazione del testo in chiaro è un fattore che categorizza i cryptosystem in due tipi principali che sono: la **cifratura a blocchi** (l'elaborazione avviene su blocchi di bit) e la **cifratura a flusso** (l'elaborazione avviene su flusso di bit quindi un bit o byte alla volta). Le principali operazioni utilizzati in questi sistemi sono:
- Sostituzione;
- Trasposizione;
- Prodotto.

# Sicurezza crittografica

Un obiettivo al quale vogliamo aspirare è ottenere la segretezza perfetta che non permette al nodo malevolo di poter leggere il messaggio (m).

## Perfect secrecy

==Secondo il teorema di Shannon la segretezza perfetta può essere ottenuta assicurandosi che il messaggio in chiaro (m) e il messaggio cifrato (c) siano completamente non correlati. Un algoritmo che assicura questa perfetta segretezza è chiamato OTP (one time pad)==, ed è implementato dal seguente schema:

![[Screenshot 2025-10-16 at 18.13.17.png]]

> [!Funzionamento dello schema OTP]
> Purtroppo questo algoritmo non può essere implementato, perché:
> - Non è possibile generare numeri realmente casuali;
> - Non avendo un canale sicuro la chiave in quale modo può essere comunicata alla destinazione ?

## Sicurezza pratica

Come detto in precedenza, nella pratica non si può raggiungere la segretezza perfetta, per cui abbiamo:
- **Sicurezza incondizionata**: il testo cifrato non fornisce informazioni a sufficienza per calcolare il corrispondente messaggio in chiaro, in maniera tale che il cipher non può essere rotto.
- **Sicurezza computazionale**: il costo per rompere il cipher supera il valore dell'informazione e il tempo richiesto per rompere il cipher supera il tempo di vita dell'informazione.

# Cryptoanalysis

==La cryptoanalysis è il processo che ha come obiettivo di scoprire il testo in chiaro o la chiave di un cryptosystem. ==

## Approcci

Questa analisi si può fare tramite due approcci:
- **Cryptanalytic attack**: è un attacco che sfrutta la struttura interna e le proprietà matematiche. 
- **Brute-force attacks**: è un attacco che ignora la struttura dell'algoritmo e si basa unicamente sulla prova sistematica di ogni chiave possibile fino a trovare quella corretta.

## Contrasto alla cryptoanalysis

Per contrastare questi approcci di cryptoanalysis dobbiamo:
- Utilizzare chiavi più lunghe;
- Selezionare un cryptosystem robusto;
- Garantire l'effetto valanga (Quando un input varia leggermente, l'output deve variare in maniera significativa).

# Crittografia Simmetrica

La crittografia simmetrica si basa sull'utilizzo di **un'unica chiave segreta** per entrambe le operazioni di crittografia e decrittografia. La chiave e l'algoritmo devono essere **condivisi** tra il mittente e il destinatario. Questo approccio si basa sulla "condivisione del segreto" (_Sharing secrecy_). È considerato quasi impossibile decifrare un messaggio se sono noti solo l'algoritmo e il testo cifrato. Questo tipo di crittografia è **molto veloce** rispetto alla crittografia asimmetrica.

> [!Svantaggio della crittografia simmetrica]
> Il principale svantaggio della crittografia simmetrica è il **problema dello scambio della chiave** (_Key exchange problem_). Per poter comunicare in modo sicuro, le due parti devono distribuire la chiave segreta in modo che non sia intercettata da nodi malevoli.

## Classificazione in Base all'Elaborazione del Testo in Chiaro

La crittografia simmetrica può essere suddivisa in base al modo in cui elabora il testo in chiaro:
- **Cifratura a Blocchi (Block Cipher)**;
- **Cifratura a Flusso (Stream Cipher)**.

### Cifratura a Blocchi

I cifrari a blocchi possono essere utilizzati in diverse modalità operative, ognuna con caratteristiche specifiche in termini di velocità, resilienza alla crittoanalisi e propagazione degli errori.

| **Modalità**                              | Vantaggi Principali                                                                                                                                                                                          | Svantaggi/Problemi                                                                                                                                                             |
| ----------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Electronic Code Book (ECB)**            | Molto semplice da implementare.                                                                                                                                                                              | Gli stessi input generano gli stessi output, rendendolo vulnerabile alla crittanalisi. Non garantisce l'integrità dei dati (i blocchi possono essere cancellati o riordinati). |
| **Cipher Block Chaining (CBC)**           | Il testo cifrato dipende dal passato; gli stessi input producono output diversi. La propagazione degli errori è limitata (un testo cifrato errato genera solo due testi in chiaro errati in decrittografia). | Problemi di sincronizzazione. Non è veloce.                                                                                                                                    |
| **Cipher FeedBack (CFB)**                 | Fa apparire il cifrario a blocchi come un cifrario a flusso. Buona resilienza alla crittanalisi e auto-sincronizzazione rapida.                                                                              | Non è veloce. È più complesso.                                                                                                                                                 |
| **Output FeedBack (OFB) & Counter (CTR)** | Sono molto veloci, con possibilità di **parallelizzare il processo**. Fanno apparire il cifrario a blocchi come un cifrario a flusso. Propagazione degli errori molto limitata.                              | Non sono semplici. I problemi di sincronizzazione sono demandati ai livelli superiori (_high-level protocols_).                                                                |

# Crittografia a chiave pubblica (asimmetrica)

==A differenza della crittografia simmetrica (dove si usa la stessa chiave per cifrare e decifrare), la crittografia asimmetrica utilizza lo stesso algoritmo sia per cifrare che per decifrare, ma impiega due chiavi distinte==. Mittente e destinatario devono possedere ognuno una coppia di chiavi che si accoppiano con quelle dell'altro

Le due chiavi sono:
- **Chiave pubblica**: può essere conosciuta da tutti e può essere usata per criptare messaggi e verificare la firma;
- **Chiave privata**: deve essere conosciuta solo dal proprietario, ed è usata per decifrare il messaggio e per firmarlo.

## Usi e Applicazioni

Questa tipologia di crittografia è utilizzata per:
- **Distribuzione delle chiavi**: Assicura comunicazioni sicure con una chiave personale senza la necessità di un centro di distribuzione delle chiavi (KDC) o di dover fidarsi del comportamento altrui.
- **Firma digitale**: Permette di verificare che un messaggio provenga intatto dal mittente dichiarato.

Le due chiavi possono essere usate in maniera complementare per la cifratura e la decifratura, offrendo funzionalità differenti:
- **Confidenzialità dei dati:** La cifratura con la chiave pubblica garantisce la confidenzialità;
- **Autenticazione dei dati:** La cifratura con la chiave privata garantisce l'autenticazione.

Con un mix di queste tecniche possiamo offrire **l'integrità dei dati**.

![[Screenshot 2026-02-05 at 17.53.52.png]]

## Diffie-Hellman

L'algoritmo Diffie-Hellman che è un protocollo crittografico che permette a due parti di generare una chiave segreta condivisa su un canale di comunicazione pubblico, senza che un attaccante possa ricavarne il segreto.

L'obiettivo è: Permettere a due persone (Alice e Bob) di creare una **chiave segreta condivisa**, anche se comunicano su un canale **non sicuro**. Di seguito abbiamo i passaggi dell'algoritmo:

1. **Scelta pubblica di base:**
    - Alice e Bob scelgono **due numeri pubblici**:
        - un numero primo grande `p`
        - un generatore `g` (minore di `p`)
2. **Scelte private:**
    - Alice sceglie un numero segreto `a`
    - Bob sceglie un numero segreto `b`
3. **Calcoli pubblici:**
    - Alice calcola `A = g^a mod p` e lo invia a Bob
    - Bob calcola `B = g^b mod p` e lo invia ad Alice
4. **Calcolo della chiave condivisa:**
    - Alice calcola: `K = B^a mod p`
    - Bob calcola: `K = A^b mod p`

Entrambi ottengono **la stessa chiave `K`**, che ora possono usare per cifrare i messaggi!

> [!Attenzione allo scambio di chiavi]
> Lo scambio non autenticato di chiavi può portare ad un attacco main-in-the-middle. Un sistema a chiave pubblica (asimmetrica) di distribuzione delle chiavi consente a due utenti di scambiarsi una chiave su un canale non sicuro. Nel nostro esempio le due entità non hanno nessuna garanzia sull'identità reciproca, quindi non si deve usare un DH anonimo ma bisogna usare i **certificati** !

![[Screenshot 2026-02-05 at 17.59.21.png]]

## RSA

La sicurezza di RSA si basa sulla difficoltà di invertire certe funzioni matematiche. Perché questo principio regga, i numeri utilizzati devono essere enormemente grandi. Se vogliamo evitare che qualcuno rompa il codice provando tutte le combinazioni (Brute Force), dobbiamo usare chiavi superiori a **2048 bit**. 

### Chosen Ciphertext Attack

- C'è un attaccante, **Trudy**, che ha intercettato un messaggio cifrato ($c$) e vuole leggerlo, ma non ha la chiave privata.
- Trudy usa un trucco: "maschera" il messaggio moltiplicandolo per un numero casuale. Ora il messaggio sembra solo una stringa di dati senza senso ($y$).
- Trudy invia questo dato e convince a **firmarlo** (magari spacciandolo per una verifica tecnica).
- **Qui sta l'errore:** In RSA, l'operazione matematica per _firmare_ è identica a quella per _decifrare_. Quindi, firmando quel dato mascherato, involontariamente lo si sta decifrando.
- Trudy prende la firma, rimuove matematicamente la "maschera" che aveva aggiunto all'inizio e si ritrova il messaggio originale in chiaro ($m$).


> [!NOTE] Firmare solo l'hash (footprint)
> Non dobbiamo **MAI** usare RSA per firmare un messaggio grezzo o sconosciuto, perché potrebbe nascondere una trappola. La soluzione è firmare sempre e solo l'**Hash** (l'impronta digitale) del messaggio. L'Hash rompe la struttura matematica che permette questo attacco, rendendolo impossibile.

## ECC

La crittografia basata sulle curve ellittiche (ECC) rappresenta un'evoluzione fondamentale rispetto ai sistemi tradizionali come RSA. Il principio cardine risiede nella geometria: anziché basarsi sulla difficoltà di scomporre numeri molto grandi in fattori primi, l'ECC utilizza le proprietà di specifiche curve matematiche. 

> [!NOTE] Vantaggi delle ECC
> Il motivo principale della diffusione dell'ECC è l'efficienza. A parità di livello di sicurezza, le chiavi utilizzate sono significativamente più corte. Questo comporta un minor dispendio di potenza di calcolo, una riduzione del consumo di batteria nei dispositivi mobili e un minor utilizzo di banda durante lo scambio di dati.
