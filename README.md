# Esercitazione Firebase

## Introduzione

Firebase è una piattaforma, gestita da Google, per lo sviluppo di applicazioni mobili e Web.
La piattaforma offre una varietà di funzionalità, tra cui sistemi di tracciamento utenti, autenticazione, messaggistica, notificazioni e banca dati.
In questa esercitazione faremo alcune semplici prove con il database che Firebase offre, in modo da avere un sistema per la memorizzazione di informazioni strutturate “nel cloud”.

## Requisiti

Firebase offre una serie di librerie utilizzabili da diverse piattaforme di sviluppo (Android, iOS, Web, etc.).
In generale la maggior parte delle funzionalità è comunque disponibile tramite semplici interfacce Web, basate su HTTP e il [paradigma RESTful](https://sites.google.com/site/richardgennaro/home/development/software_architecture_design/programming-paradigms/paradigma-rest---restful).

Serviranno:

* Account Google (personale, l’account Google legato alla credenziali presso UniUrb non è abilitato ad accedere a Firebase),
* Qualsiasi client&nbsp;HTTP per fare richieste al back-end di Firebase (nella traccia si farà uso di cURL, per cui conviene usare Linux oppure installare [WSL](https://docs.microsoft.com/en-us/windows/wsl/install-win10) su Windows&nbsp;10).

## Traccia

### Creazione del database

Fare login nella [console Firebase](https://console.firebase.google.com/) con il proprio account Google.
Creare un **nuovo progetto**, assegnando un nome qualsiasi e “Italy” come regione.
Durante la creazione, utilizzare le **regole di accesso** di test (aperto a tutti) per semplicità.

Nel pannello di controllo, accedere alla sezione “Database”, che mostrerà l’URL radice del proprio database ed i contenuti dello stesso (al momento ancora vuoti).

### Caricamento di dati

Firebase utilizza una struttura gerarchica per memorizzare i nostri dati, andando a formare una struttura che è identica a quella di un grande blocco in formato JSON.

È possibile usare cURL per caricare qualsasi tipo di blocco di dati, sempre utilizzando il formato JSON.
Ad esempio:

```bash
curl -X PUT -d '{"age":31,"first_name":"Jamie","house":"Lannister","last_name":"Lannister","nick_name":"Kingslayer"}' 'https://PROJECT-ID.firebaseio.com/characters/jaime.json'
```

Perché la chiamata abbia successo, è necessario sostituire la stringa `PROJECT-ID` con il proprio ID di progetto, come riportato nell’interfaccia Web di Firebase, in modo da accedere al proprio database.

Dopo l’esecuzione del comando mostrato sopra, si può verificare nell’interfaccia Web che è stato generato un nuovo nodo `characters`, con un unico nodo figlio, `jaime`, che presenta 5&nbsp;campi con dei dati primitivi&nbsp;(stringhe e numeri).

È possibile sfruttare una richiesta di tipo `GET` per scaricare il nodo in particolare:

```bash
curl 'https://PROJECT-ID.firebaseio.com/characters/jaime.json'
```

### Aggiornamento dati

Con richieste&nbsp;HTTP con verbo `PUT` è possibile *sostituire* (o sovrascrivere) del tutto un nodo del proprio database (quindi, nell’esempio sopra, scrivendo sul nodo `/characters/jaime` si va a rimpiazzare eventuali dati già presenti).
Per *integrare* i dati, è possibile utilizzare il verbo&nbsp;HTTP `PATCH`:

```bash
curl -X PATCH -d '{"nick_name":"Kingslayer"}' 'https://PROJECT-ID.firebaseio.com/characters/jaime.json'
```

Andando a scaricare nuovamente il nodo, si noterà che l’informazione è stata aggiunta a quelle pre-esistenti.
Nel caso ci sia una collisione (campi di un nodo che hanno lo stesso nome), le informazioni esistenti vengono aggiornate.

Con il verbo&nbsp;`PATCH` è anche possibile caricare più nodi come figli di un nodo esistente:

```bash
curl -X PATCH -d '{"daenerys":{"age":13,"first_name":"Daenerys","house":"Targaryen","last_name":"Targaryen"},"ned":{"age":34,"first_name":"Eddard","house":"Stark","last_name":"Stark","nick_name":"Ned"},"jon":{"first_name":"Jon","last_name":"Snow","age":14,"house":"Stark"}}' 'https://PROJECT-ID.firebaseio.com/characters.json'
```

Grazie al fatto che, a differenza del metodo&nbsp;`PUT`, il nodo non viene rimpiazzato, ma le informazioni del nodo `characters` vengono integrate, possiamo aggiungere due nuovi personaggi con una singola richiesta&nbsp;HTTP.

Per visualizzare l’intero nodo dei personaggi con i dati aggiornati, è sufficiente effettuare questa richiesta:

```bash
curl 'https://PROJECT-ID.firebaseio.com/characters.json'
```

### Rimozione dati

Con delle richieste con verbo&nbsp;`DELETE` è possibile rimuovere interi nodi del database (*spoiler per chi non ha visto S01E09*):

```bash
curl -X DELETE 'https://PROJECT-ID.firebaseio.com/characters/ned.json'
```

### Indicizzazione e ordinamento

Dopo ogni operazione è possibile scaricare l’intero blocco di personaggi all’indirizzo `https://PROJECT-ID.firebaseio.com/characters.json`.
Tuttavia, una volta raggiunto un certo numero di dati inseriti nel database, questo non è più né comodo né efficiente: per questo Firebase offre la possibilità di **indicizzare** i propri dati (ossia, definire delle proprietà dei dati che verranno sfruttate per ordinare e filtrare l’accesso ai dati).

Per aggiungere un indice, è necessario andare a modificare le “regole” del proprio database.
È possibile farlo direttamente tramite l’interfaccia Web, aprendo la scheda “Rules” nella pagina del database (accando alla scheda di visualizzazione dei dati).

Le regole esistenti dovrebbero avere questo aspetto:

```json
{
  "rules": {
    ".read": true,
    ".write": true
  }
}
```

(In particolare, queste regole fanno sì che il database sia aperto pubblicamente in lettura e scrittura, come è stato specificato durante la creazione del database.)

È possibile aggiungere delle regole per ogni nodo dei dati su cui operiamo.
Per aggiungere una serie di indici al nodo dei personaggi dovremmo avere la seguente struttura finale:

```json
{
  "rules": {
    ".read": true,
    ".write": true,
    "characters": {
      ".indexOn": ["last_name", "age", "house"]
    }
  }
}
```

In questo caso abbiamo aggiunto un indice sulle 3&nbsp;proprietà `last_name`, `age` e `house`.

A questo punto possiamo ordinare i nostri personaggi nelle nostre richieste&nbsp;HTTP, ad esempio ordinandoli per età:

```bash
curl 'https://PROJECT-ID.firebaseio.com/characters.json?orderBy="age"'
```

È anche possibile visualizzare solo il personaggio più giovane:

```bash
curl 'https://PROJECT-ID.firebaseio.com/characters.json?orderBy="age"&limitToFirst=1'
```

o il più vecchio:

```bash
curl 'https://PROJECT-ID.firebaseio.com/characters.json?orderBy="age"&limitToLast=1'
```

Provare ad utilizzare anche gli altri indici per effettuare l’ordinamento.
L’utilizzo di una proprietà su cui non è definito un indice generarà un errore.

### Aggiunta di dati in lista

Finora abbiamo utilizzato dati strutturati in nodi di cui conoscevamo il nome e la posizione all’interno della gerarchia.
In alcuni casi è necessario aggiungere un blocco di dati senza che se ne conosca la posizione finale, ma inserendolo in una lista di dati già presenti.

Creeremo un nuovo nodo, con nome&nbsp;`messages`, che conterrà una sequenza di messaggi in ingresso (potrebbe essere una lista di messaggi in input ad un bot o qualcosa di simile), sfruttando il verbo&nbsp;HTTP di tipo&nbsp;`POST`:

```bash
curl -X POST -d '{"timestamp": 123456, "message":"Hello world!"}' 'https://PROJECT-ID.firebaseio.com/messages.json'
curl -X POST -d '{"timestamp": 123457, "message":"Hello again!"}' 'https://PROJECT-ID.firebaseio.com/messages.json'
curl -X POST -d '{"timestamp": 123458, "message":"Bye bye!"}' 'https://PROJECT-ID.firebaseio.com/messages.json'
```

Dopo aver eseguito le 3&nbsp;richieste, troveremo un nodo `messages` con 3&nbsp;nodi sotto di esso, ognuno identificato da una stringa casuale.
Firebase ci assicura che la stringa casuale che identifica ogni nodo sarà unico e che quindi sarà sempre possibile inserire un nuovo nodo sotto al nodo `messages`.

Grazie a richiesta di tipo&nbsp;`GET` è possibile scaricare l’intera lista di messaggi, ma nella maggior parte dei casi sarà più utile ordinare e filtrare i risultati utilizzando degli indici, come visto sopra.
Provare a creare un indice sul campo `timestamp` e accedere solo all’ultimo messaggio inserito.

## Consegna

Niente da consegnare in questa esercitazione.&nbsp;🙂

È possibile (e consigliato) provare a sfruttare Firebase per memorizzare conversazioni ed altre informazioni durante l’[esercitazione con il bot Telegram](https://github.com/DigiPlatMOOC/pdgt-esercitazione-bot-telegram).
