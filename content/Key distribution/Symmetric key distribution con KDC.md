Lo schema simmetrico richiede che entrambi le parti condividano una chiave segreta, l'idea da cui partire è condividere una chiave master con il KDC (key distribution center) che ci permette di creare a sua volta delle chiavi di sessione che saranno distrutte una vola terminata la sessione. Utilizzando questo approccio noto che non c'è l'utilizzo di chiavi a lungo termine e se viene compromessa la chiave di sessione comunque la master rimane sicura.

![[Screenshot 2025-11-11 at 12.22.52.png]]


> [!NOTE] Problemi del Symmetric key distribution con KDC
> Questo approccio è vulnerabile agli attacchi MITM perché il KDC può esporre le chiavi di sessione ad altri. Inoltre, rappresenta il punto di errore e questo non può essere accettato perché deve essere online per supportare connessioni sicure.
