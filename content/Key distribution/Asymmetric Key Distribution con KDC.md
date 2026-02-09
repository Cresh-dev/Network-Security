Come visto in precedenza, nella crittografia asimmetrica abbiamo la chiave pubblica e la chiave privata. Annunciando pubblicamente la propria chiave pubblica abbiamo una debolezza di sicurezza alta, quindi si è sensibili a manomissioni. Le tecniche che vengono utilizzate per la distribuzione della chiave pubblica sono le seguenti:

- **Diffie-Hellman (DH):** Negoziazione **reciproca**. A e B si scambiano le proprie chiavi pubbliche per generare insieme un segreto condiviso ($K$).
- **RSA:** Negoziazione **unidirezionale**. Il client (A) genera un numero casuale e lo invia al server (B) criptandolo con la chiave pubblica di quest'ultimo. Solo B può decifrarlo.
- **Public-key authority**: ![[Screenshot 2025-11-11 at 12.33.37.png]]
- **Public-key certificates**: ![[Screenshot 2025-11-11 at 12.33.55.png]]

> [!NOTE] Collo di bottiglia del sistema con KDC
> La soluzione definitiva è rappresentata dalla **Certification Authority (CA)**. A differenza del KDC, la CA rilascia **certificati digitali** che vengono memorizzati direttamente sui dispositivi dei peer. In questo modo, l'autenticazione avviene una sola volta all'inizio e i peer possono scambiarsi i certificati in autonomia, eliminando la necessità di essere sempre collegati all'autorità centrale.

