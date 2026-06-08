Le Funzioni Hash (Hash Functions) sono uno strumento fondamentale nel modello di sicurezza di rete, essenziali nel contesto più ampio dell'**Integrità dei Dati**. Una Funzione Hash è un meccanismo che genera un **digest** (riassunto) di una stringa di bit seguendo un algoritmo specifico. ==Non fa l'utilizzo di chiavi, infatti l'hash function singolarmente non è un secure tool==, ma essa è spesso utilizzata nei crypto mechanisms. Per trasformare un semplice controllo di integrità in uno strumento di sicurezza autenticato, l'hash deve essere protetto tramite crittografia simmetrica o asimmetrica.

## Usi comuni 

Le hash functions sono spesso utilizzate per: 
- Memorizzare l'hash della password e non la password in chiaro;
- Per la verifica di manomissione dei file.

## Funzionamento

==Prende in input un messaggio M di lunghezza variabile e restituisce in output un digest H(M) di lunghezza fissa, senza l’uso di alcuna chiave segreta==. Per garantire l’autenticità del messaggio, il digest viene trasmesso insieme al messaggio stesso: il destinatario ricalcola l’hash del messaggio ricevuto e lo confronta con quello allegato, verificando così che il digest corrisponda e che il messaggio non sia stato alterato.

## Requisiti di Sicurezza (per l'Integrità)

Una funzione di hash può essere applicata a un blocco di dati di qualsiasi misura, producendo un output di lunghezza fissa. Questa funzione ha un insieme di requisiti che sono:
- **Facilità di Calcolo**: H(x) deve essere semplice da calcolare sia in hardware che software;
-  **Proprietà Unidirezionale**: Per qualsiasi valore di h, deve essere computazionalmente irrealizzabile trovare x tale che H(x) = h;
- **Resistenza alla Debole Collisione**: Per un dato blocco x, deve essere computazionalmente infattibile trovare un y tale che H(y)=H(x);
- **Resistenza alla Forte Collisione**: Deve essere computazionalmente infattibile trovare una qualsiasi coppia (x,y) tale che H(x)=H(y).

### Il Birthday attack

Il **Birthday attack** sfrutta lo stesso principio del famoso “paradosso del compleanno”. La funzione hash produce **n** bit, ci sono $2^n$ possibili valori. Il Birthday attack prova tante varianti di messaggi finché non trova **due messaggi diversi** che danno lo stesso hash (una _collisione_). Per trovare una collisione non servono $2^n$ tentativi: grazie al paradosso del compleanno bastano circa $2^{n/2}$.

#### Funzionamento

- L’attaccante genera molte varianti di messaggi (o file) e calcola l’hash di ognuno.
- Tiene una tabella degli hash e dei messaggi corrispondenti.
- Appena trova due messaggi con lo stesso hash, ha ottenuto una collisione che può usare a fini malevoli.

#### Contromisure

Per evitare questa tipologia di attacco possiamo adottare alcune contromisure che sono:
- Usare funzioni hash con output lungo (SHA-256 o superiore) — perché la lunghezza degli output sposta la difficoltà: per esempio SHA-256 richiede $2^{128}$ tentativi per una collisione (praticamente impraticabile oggi);
- Usare [[Message authentication code (MAC) and HMAC|HMAC]] o altri schemi che non si basano solo sull’hash semplice quando serve autenticazione.

## Merkle–Damgård Structure

La **struttura Merkle–Damgård** è un modo per **costruire una funzione di hash** sicura da una funzione di compressione di base (cioè una funzione che prende input di dimensione fissa e restituisce output di dimensione fissa).

![[Screenshot 2026-02-05 at 18.34.02.png]]

> [!Idea di base]
> Poiché i messaggi possono essere lunghi arbitrariamente, li dividiamo in **blocchi** di dimensione fissa e li elaboriamo uno alla volta, mantenendo uno stato intermedio (chiamato **chaining value**).

### Esempio

Come prima cosa si allunga il messaggio in modo che la sua lunghezza sia un multiplo della dimensione del blocco. Si parte da un valore fisso chiamato **IV** (Initial Value) e poi per ogni blocco del messaggio `M_i`, si calcola $H_i​=f(H_{i−1}​,M_i​)$ dove $f$ è la funzione di compressione.
Supponiamo che il messaggio venga diviso in 3 blocchi: `M1`, `M2`, `M3`:

```
IV → f(IV, M1) → H1
H1 → f(H1, M2) → H2
H2 → f(H2, M3) → H3 = output hash
```