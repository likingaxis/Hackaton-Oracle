### SLIDE 1 Perché non usare semplicemente un LLM?
La domanda sorge spontanea: per rispondere al problema non sarebbe sufficiente utilizzare un semplice LLM a cui facciamo delle domande sulle varie telemetrie, magari mettendo direttamente nel system prompt tutto il contesto che deve avere, come dati di gara, regolamento e informazioni tecniche?

La risposta è no, o comunque non sarebbe l’approccio migliore. Abbiamo infatti **due fonti di informazione molto diverse** e soprattutto mettere tutto dentro al prompt produrrebbe un approccio poco efficiente e poco controllabile.

L’obiettivo è:

- ridurre l’uso inutile di token;
- far lavorare il modello solo con le informazioni che effettivamente gli servono;
- evitare di riempire la context window con dati non rilevanti.

Trattare tutto allo stesso modo significherebbe chiedere al modello di cercare da solo l’informazione utile dentro un grande blocco di contesto.

Inoltre non vogliamo che il modello inventi valori, faccia calcoli approssimativi o interpreti una regola basandosi soltanto sulla propria conoscenza generale.

Vogliamo invece che recuperi i dati reali da una fonte precisa e che, quando possibile, i calcoli vengano eseguiti da **codice deterministico**.

Il valore aggiunto quindi non è semplicemente dare al modello più contesto possibile, ma costruire un sistema che gli permetta di recuperare **l’informazione giusta, dalla fonte giusta, nel momento giusto**
![[Pasted image 20260927104729.png|172]]

#### SLIDE 2 la soluzione
**Costruire un harness attorno all’LLM**

Il modello non viene più usato come un semplice chatbot, ma diventa un **orchestratore** che ha il compito di ragionare e interpretare i dati che recupera tramite gli strumenti che gli mettiamo a disposizione.

Noi gli forniamo strumenti specializzati:

- uno per accedere ai **dati di gara** e ai calcoli sulla telemetria;
- uno per recuperare informazioni dai **documenti** tramite ricerca semantica.

In questo modo separiamo i compiti:

- l’LLM si occupa di **interpretare e ragionare**;
- i tool si occupano di **recuperare dati reali ed eseguire operazioni precise**;
- il sistema di retrieval si occupa di **recuperare solo il contesto documentale utile** 
![[Pasted image 20260927104714.png|223]]

#### SLIDE 3 ARCHITETTURA EFFETTIVA
![[Pasted image 20260927104658.png|519]]

**L’ingegnere di gara interagisce con una User Interface, che comunica con un client incaricato di costruire la richiesta da inviare a OCI Generative AI tramite la Responses API.**

Alla richiesta non passiamo solamente il prompt dell’utente: rendiamo disponibili al modello anche gli strumenti che può utilizzare durante il ragionamento.

Nel nostro caso i due percorsi principali sono:

- **File Search**, per recuperare informazioni dai documenti indicizzati nell’OCI Vector Store;
- **MCP**, per invocare i tool esposti dal nostro server e accedere alla telemetria.

Una volta ricevuta la richiesta, il modello può decidere autonomamente quali strumenti utilizzare e può effettuare più chiamate ai tool all’interno dello stesso flusso prima di produrre la risposta finale.

Se servono informazioni normative o tecniche dai documenti, utilizza `file_search` sul Vector Store. Se invece servono dati di gara o calcoli sulla telemetria, utilizza i tool del server MCP.

Il server MCP gira localmente sulla nostra macchina, mentre OCI Generative AI è nel cloud. Per renderlo raggiungibile dall’esterno abbiamo quindi utilizzato **ngrok**, che crea un tunnel HTTPS pubblico verso il server locale

#### SLIDE 5 SPIEGAZIONE NEL DETTAGLIO DI MCP E OCI VECTOR STORE
a sinistra una card con la seguente foto dove si spiega cosa è un MCP server
a destra una card con la seguente foto dove si spiega cosa è un vector store
ESEMPIO DI STILE (NON METTERE NELLE SLIDE):
![[Pasted image 20260927115603.png|463]]



![[Pasted image 20260927115007.png|248]]
- Un MCP(Model context protocol) Server espone una serie di tool che permettono al modello di accedere in modo controllato a dati esterni, nel nostro caso telemetria salvata in JSON e JSONL. La comunicazione con OCI Generative AI avviene tramite il Model Context Protocol, che definisce in modo standardizzato come descrivere, invocare e restituire i risultati dei tool

![[embedding.jpg|256]]
- **Un Vector Store è una struttura progettata per memorizzare embedding e recuperare contenuti semanticamente simili a una query.**
	- ogni testo viene trasformato in un **vettore multidimensionale** che ne rappresenta il significato;
	- testi con significati simili tendono a essere vicini nello spazio degli embedding;
	- questo permette di effettuare **ricerca semantica**, non basata solo sulla corrispondenza esatta delle parole;
	- nel nostro progetto, indicizzazione e retrieval sono gestiti dal servizio **OCI Vector Store**, interrogato tramite `file_search`
