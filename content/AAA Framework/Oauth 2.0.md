Introduciamo OAuth 2.0 partendo da un problema di sicurezza nel modello client-server tradizionale: per permettere a un'applicazione di terze parti di accedere a risorse protette, l'utente (resource owner) doveva condividere le proprie credenziali (username e password) con l'applicazione terza. Questo è definito come una metodologia insicura. OAuth 2.0 (standardizzato nella **RFC 6749**) risolve questo problema introducendo un livello di autorizzazione che **separa il ruolo del client da quello del proprietario della risorsa**. In sintesi, permette alle applicazioni terze di agire per conto dell'utente senza che queste debbano conoscere l'identità dell'utente stesso o possedere le sue credenziali. 

## Architettura e Ruoli

Definiamo quattro attori principali nel framework OAuth 2.0:

- **Resource Owner:** L'entità (spesso l'utente finale) in grado di concedere l'accesso a una risorsa protetta.
- **Resource Server:** Il server che ospita le risorse protette e che accetta richieste contenenti _access token_.
- **Client:** L'applicazione che richiede l'accesso alle risorse protette per conto del proprietario.
- **Authorization Server:** Il server che rilascia gli _access token_ al client dopo aver autenticato con successo il proprietario della risorsa e averne ottenuto l'autorizzazione.

**Esempio Pratico**: Un'app di stampa foto

1. **Client:** L'app "StampaFoto".
2. **Resource Owner:** Noi.
3. **Resource Server:** Google Photos (dove sono le immagini).
4. **Authorization Server:** Google Accounts (dove fai il login).



![[Screenshot 2026-02-09 at 09.23.54.png]]

### Fase A e B: La Concessione (The Grant)

In questa fase, il Client non riceve dati, ma solo un **"titolo di autorizzazione"** (Authorization Grant).

- **Dettaglio:** Esistono diversi modi per ottenere questa concessione (chiamati _Grant Types_). Il più comune è l'**Authorization Code**, dove l'utente viene reindirizzato su una pagina del fornitore (es. login di Facebook), inserisce lì le credenziali e viene riportato indietro con un codice temporaneo.
- **Sicurezza:** L'app (Client) non vede mai la password dell'utente.

### Fase C e D: Lo Scambio (The Exchange)

Il Client scambia il codice ottenuto nella fase B con un **Access Token**.

- **Dettaglio:** Questa chiamata avviene spesso "back-channel" (da server a server), invisibile all'utente. Il Client deve solitamente identificarsi con una sua password segreta (_Client Secret_) per dimostrare che è proprio lui a chiedere il token.
- **Risultato:** Il Client ora possiede l'**Access Token**, una stringa (spesso in formato JWT - JSON Web Token) che contiene informazioni su chi è il client, chi è l'utente e cosa può fare.

### Fase E e F: L'Accesso (The API Call)

Il Client usa finalmente la chiave per aprire la porta.

- **Dettaglio:** Il token viene inviato nell'intestazione HTTP della richiesta (Header: `Authorization: Bearer <token>`).
- **Verifica:** Il Resource Server non chiede all'utente "chi sei?", ma chiede al token "cosa ti è permesso fare?". Se il token è valido e non è scaduto, i dati vengono inviati.

## Tipi di Autorizzazione (Grant Types)

A seconda del tipo di applicazione, OAuth 2.0 offre diversi metodi per ottenere il token:

- **Authorization Code:** Il più sicuro e diffuso, usato per applicazioni server-side.
- **Implicit:** Flusso semplificato per app browser-based (oggi largamente deprecato).
- **Client Credentials:** Usato quando l'app accede a risorse proprie (non per conto di un utente).
- **Resource Owner Password Credentials:** L'utente dà le sue credenziali direttamente all'app (metodo legacy, sconsigliato).
## Evoluzioni e integrazioni

Per dispositivi come smart TV o stampanti che non hanno un browser, esiste un "device flow" (RFC 8628) che usa un secondo dispositivo (es. smartphone) per completare l'autorizzazione. Mentre OAuth 2.0 gestisce l'**autorizzazione** (cosa possiamo fare), OpenID Connect aggiunge uno strato di **identità** sopra di esso per gestire l'**autenticazione** (chi siamo) tramite un nuovo tipo di token chiamato **ID Token** (in formato JWT).