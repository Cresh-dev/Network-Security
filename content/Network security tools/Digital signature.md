==La firma digitale è l'equivalente informatico della firma autografa su carta e ne ha lo stesso valore legale. Serve per garantire l'autenticità, l'integrità e il non ripudio di un documento digitale==. Il suo funzionamento si basa sulla [[Cryptosystem#Crittografia a chiave pubblica (asimmetrica)|crittografia asimmetrica]] (o a chiave pubblica) e si articola in due fasi principali: la **firma** e la **verifica**. A differenza del [[Message authentication code (MAC) and HMAC|MAC]], la firma digitale utilizza la crittografia asimmetrica su un messaggio [[Hash function|hashato]]. Di seguito è riportato uno schema dove sono presenti le viarie fasi che verranno approfondite successivamente.

![[Screenshot 2025-11-07 at 18.05.16.png]]

# Fasi della firma digitale

Possiamo dividere il processo della firma digitale in due macro-fasi: la **generazione** (chi firma) e la **verifica** (chi riceve).
## Fase di Firma

Il firmatario utilizza un sistema di chiavi crittografiche: una **chiave privata** (segreta, nota solo al firmatario) e una **chiave pubblica** (disponibile a tutti) correlate tra loro.

1. **Hashing del Documento:** Un algoritmo matematico (funzione di hash) elabora il contenuto del documento da firmare, generando un codice alfanumerico univoco e di lunghezza fissa chiamato **hash** o _message digest_. Qualsiasi minima modifica al documento originale produrrebbe un hash completamente diverso.
2. **Cifratura dell'Hash:** L'hash viene **cifrato** utilizzando la **chiave privata** del firmatario. 
3. **Creazione della Firma:** L'hash cifrato con la chiave privata è la **firma digitale**.
4. **Associazione:** La firma digitale (l'hash cifrato) e il **certificato digitale** del firmatario (che contiene la chiave pubblica) vengono **associati** in modo indissolubile al documento originale.

## Fase di Verifica

Il destinatario del documento utilizza la chiave pubblica del firmatario per assicurarsi che il documento sia originale e non sia stato alterato.

1. **Decifratura della Firma:** Il destinatario usa la **chiave pubblica** del firmatario (contenuta nel certificato digitale allegato al documento) per **decifrare** la firma digitale (l'hash cifrato), ottenendo l'**Hash A** originale.
2. **Nuovo Hashing del Documento:** Il software del destinatario calcola un **nuovo hash** del documento ricevuto (utilizzando la stessa funzione di hash), ottenendo l'**Hash B**.
3. **Confronto:** Il sistema **confronta** l'**Hash A** (quello decifrato dalla firma) con l'**Hash B** (quello calcolato sul documento ricevuto).

- **Se i due hash corrispondono**, la firma è **valida**. Ciò garantisce che il documento è stato firmato dal titolare della chiave privata (autenticità) e che non è stato modificato dopo la firma (integrità).
- **Se i due hash non corrispondono**, la firma non è valida e il documento è stato alterato o la firma non è autentica.