==I certificati X.509 sono lo strumento centrale che la PKI (public-key-certificates) utilizza per fornire un metodo sicuro, affidabile e scalabile per la distribuzione delle chiavi pubbliche, garantendo segretezza, correttezza e verifica del mittente==. Il certificato X.509 ha il ruolo cruciale di "legare" (bind) il proprietario alla chiave pubblica. Al centro dello schema X.509 si trova il certificato a chiave pubblica associato a ciascun utente, che si assume sia creato da una **Certification Authority (CA)** fidata. Un certificato lega l'identità alla chiave pubblica, includendo anche altre informazioni come il periodo di validità e i diritti di utilizzo. La sicurezza del certificato è garantita dal fatto che esso è autenticato tramite una **crittografia asimmetrica** (ad esempio RSA) del suo hash.

![[ns01 baseline security tools 2.png]]

> [!NOTE] Resistenza all'attacco Man-in-the-Middle
>  Qualora si trasmettesse unicamente una stringa di testo dichiarando che essa rappresenta la propria chiave pubblica, il destinatario non avrebbe alcun mezzo per verificare l’autenticità del mittente. Di conseguenza, non sarebbe possibile stabilire se la comunicazione provenga effettivamente dal soggetto legittimo oppure da un attaccante che tenta di intercettare e alterare le comunicazioni, come avviene in un attacco di tipo _Man-in-the-Middle_.

# Struttura del certificato 

![[ns01 baseline security tools 1.png]]

# Come ottenere un certificato

|**Passaggio**|**Azione del Richiedente (Es. Server)**|**Azione dell'Autorità di Certificazione (CA)**|**Risultato**|
|---|---|---|---|
|**1. Generazione Chiavi**|Genera una **Chiave Privata** (segreta) e una **Chiave Pubblica** (pubblica).|Nessuna.|Coppia di chiavi generate.|
|**2. Richiesta (CSR)**|Crea una **Certificate Signing Request (CSR)** che include la **Chiave Pubblica** e i dati identificativi (es. nome del dominio).|Nessuna, riceve la richiesta.|File CSR inviato alla CA.|
|**3. Verifica e Firma**|Attende.|**Verifica** l'identità del richiedente e, se l'identità è confermata, **firma** il CSR usando la sua chiave privata.|Certificato X.509 firmato.|
|**4. Installazione**|Riceve il certificato firmato e lo **installa** sul server, accoppiandolo con la sua chiave privata segreta.|Nessuna.|Il server può ora stabilire connessioni sicure (es. HTTPS).|

# Gerarchia dei certificati

La gerarchia X.509 permette a due utenti in parti diverse del mondo di fidarsi l'uno dell'altro, a patto che esista un **percorso ininterrotto di fiducia** che colleghi le rispettive Autorità di Certificazione, risalendo fino a un punto comune e fidato (la radice o un'altra CA comune).