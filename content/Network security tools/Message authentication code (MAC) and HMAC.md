==Il Message Authentication Code (MAC) anche conosciuto come cryptographic checksum è un meccanismo crittografico utilizzato per verificare l’integrità e l’autenticità di un messaggio==. In altre parole, serve a garantire che:
- Il messaggio non sia stato modificato durante la trasmissione (**integrità**).
- Il mittente sia autentico, cioè che il messaggio provenga davvero da chi dice di averlo inviato (**autenticità**).

> [!NOTE] La confidenzialità con il MAC
>  Il MAC, da solo, fornisce **autenticità** e **integrità**. La **riservatezza (confidentiality)** non è fornita dal MAC stesso, ma si ottiene aggiungendo un algoritmo di **cifratura** al sistema. Per combinare tutte e tre le proprietà (autenticità, integrità e riservatezza), le migliori pratiche di sicurezza (come lo schema **Encrypt-then-MAC** o E-t-M) prevedono che si usino **chiavi crittografiche distinte** (una chiave per il MAC e una chiave diversa per la cifratura) e che l'operazione di **cifratura** sia eseguita **prima** dell'operazione di MAC (cioè, il MAC viene calcolato sul testo _cifrato_), poiché questo ordine è generalmente considerato il più sicuro.

# Come funziona un MAC?

Un MAC viene calcolato con una **funzione crittografica** che prende in input:

- il messaggio da proteggere;
- una **chiave segreta condivisa** (nota solo a mittente e destinatario).

```
MAC = f(chiave, messaggio)
```

Il risultato è un valore di **lunghezza fissa** (il MAC), che viene **inviato insieme al messaggio**. Quando il destinatario riceve il messaggio, ricalcola il MAC con la **stessa chiave** e confronta il risultato con il MAC ricevuto:

- Se i due MAC coincidono, il messaggio è autentico e integro.
- Se sono diversi, il messaggio è stato alterato o proviene da una fonte non autorizzata.

![[Screenshot 2026-02-09 at 15.25.01.png]]

==Il MAC può anche essere integrato con crittosistemi che garantiscono la confidenzialità del messaggio. Nel MAC grazie al testo cifrato non abbiamo la possibilità di implementare l'attacco a compleanno come nell'hash, però dobbiamo tenere in considerazione che l'hash è estremamente più veloce rispetto al MAC.==

> [!NOTE] MAC non è firma digitale!
> Il MAC è basato sulla crittografia simmetrica, invece la firma digitale si basa sulla crittografia asimmetrica.

# HMAC

L'**HMAC** (Hash-based Message Authentication Code) è un meccanismo utilizzato per verificare contemporaneamente l'**integrità** e l'**autenticità** di un messaggio utilizzando una funzione di hash (come SHA-256) e una chiave segreta ($K$).

![[Screenshot 2026-02-06 at 09.45.40.png]]

In alto a sinistra vediamo $K \parallel 000..00$. Se la chiave segreta $K$ è più corta della dimensione del blocco della funzione di hash ($b$ bit), viene riempita con degli zeri a destra (padding) per raggiungere esattamente i $b$ bit. La chiave viene messa in XOR ($\oplus$) con una costante fissa composta dal byte `0x36` ripetuto. Il risultato è il **blocco $S_i$**. Questo blocco $S_i$ viene messo davanti al messaggio originale $m$ (che vediamo diviso in blocchi $Y_0, Y_1, \dots$). Il tutto viene passato nella funzione di hash $H$ (usando un valore di inizializzazione standard $IV$). Il risultato è un digest di $n$ bit: $H(S_i \parallel m)$.  La chiave originale (sempre con padding di zeri) viene messa in XOR con una diversa costante fissa composta dal byte `0x5C` ripetuto. Il risultato è il **blocco $S_o$**. Questo nuovo blocco $S_o$ viene concatenato con il risultato del primo hash ottenuto al punto precedente. Il blocco risultante viene passato nuovamente nella funzione di hash $H$. Il risultato finale è l'**$HMAC_K(m)$**.

> [!NOTE] Osservazione sull'architettura dell'HMAC
> Questo design a "doppio giro" serve a proteggere l'algoritmo da attacchi crittografici noti come **"length extension attacks"**, ai quali alcune funzioni di hash (come MD5 o SHA-1) sono vulnerabili se usate in modo semplice (ad esempio facendo solo l'hash di chiave + messaggio).
