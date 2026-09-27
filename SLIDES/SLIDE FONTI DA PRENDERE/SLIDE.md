### Slide 1

Buongiorno a tutti. Siamo il team Struffoli: Valerio, Alfredo, Samuele e Luca. Vi presentiamo l'**AI Race Engineer Copilot**, un assistente per l'ingegnere di pista della Scuderia Tor Vergata, costruito sulla piattaforma **OCI Enterprise AI**. `[→]`
# Slide 2: Obiettivo del Progetto
*Un chatbot operativo a supporto di scuderia Torvergata.*

*   **Contesto Real-Time**
    Analisi immediata dei dati di gara e del regolamento.
*   **Nessuna Allucinazione**
    Esposizione esclusiva di dati e numeri effettivamente recuperati.
*   **Incertezza Dichiarata**
    Assenza di informazioni notificata esplicitamente all'utente.

---

# Slide 3: Fonti e Dati

### Telemetria 
*   **Contenuto:** Dati dinamici sulle condizioni del veicolo a ogni pit di gara.
*   JSON 

### Regolamento 
*   **Dettaglio:** Dossier tecnico e normativo per le verifiche dei veicoli in gara.
*   PDF

---

# Slide 4: Task Progettuali
*L'architettura del copilota sviluppata in tre task.*

1.  **Task 1 — Server MCP**
    Sviluppo tool di estrazione per analizzare i dati JSON della telemetria.
2.  **Task 2 — RAG Normativo**
    Chunking dei PDF e verifica della conformità tramite ricerca semantica.
3.  **Task 3 — App Unificata**
    Linking del chatbot con server e RAG, sviluppo di una web app tramite streamlit per porre query all'assistente.

---

## Slide 5 — Un LLM da solo non basta

**Layout:** testo a sinistra (frase, le due fonti, tre riquadri rosso chiaro con icona accanto al titolo), disegno grande a destra
**Occhiello:** PERCHÉ UN SISTEMA
**Sottotitolo:** Perché non mettere tutto nel prompt?

**Frase centrale**
> Abbiamo **due fonti di informazione molto diverse**: metterle tutte nel prompt renderebbe il sistema poco efficiente e poco controllabile.

**Le due fonti**
- *icona segnale (activity)* — **TELEMETRIA**
- *icona documento* — **DOCUMENTI**

**Tre riquadri (rosso chiaro: sono i problemi)**
- *icona gauge* — **Token sprecati** — Context window piena di dati inutili
- *icona volante* — **Poco controllo** — Il modello cerca da solo nel contesto
- *icona avviso* — **Valori inventati** — Calcoli approssimativi e regole non verificate

**Immagine:** assistente di gara ↔ LLM, senza nulla attorno
![[Pasted image 20260927104729.png|172]]

**Copione / Discorso (Luca):**
> "La domanda sorge spontanea: per rispondere al problema non sarebbe sufficiente utilizzare un semplice LLM a cui facciamo delle domande sulle varie telemetrie, magari mettendo direttamente nel system prompt tutto il contesto che deve avere, come dati di gara, regolamento e informazioni tecniche?
> 
> La risposta è no, o comunque non sarebbe l’approccio migliore. Abbiamo infatti **due fonti di informazione molto diverse** e soprattutto mettere tutto dentro al prompt produrrebbe un approccio poco efficiente e poco controllabile.
> 
> L’obiettivo è:
> - ridurre l’uso inutile di token;
> - far lavorare il modello solo con le informazioni che effettivamente gli servono;
> - evitare di riempire la context window con dati non rilevanti.
> 
> Trattare tutto allo stesso modo significherebbe chiedere al modello di cercare da solo l’informazione utile dentro un grande blocco di contesto.
> 
> Inoltre non vogliamo che il modello inventi valori, faccia calcoli approssimativi o interpreti una regola basandosi soltanto sulla propria conoscenza generale.
> 
> Vogliamo invece che recuperi i dati reali da una fonte precisa e che, quando possibile, i calcoli vengano eseguiti da **codice deterministico**.
> 
> Il valore aggiunto quindi non è semplicemente dare al modello più contesto possibile, ma costruire un sistema che gli permetta di recuperare **l’informazione giusta, dalla fonte giusta, nel momento giusto**."

---

## Slide 6 — Costruire un harness attorno all'LLM

**Layout:** tre riquadri in colonna a sinistra (fondo verde chiaro, icona accanto al titolo verde, testo scuro), disegno dell'harness a destra
**Occhiello:** LA SOLUZIONE
**Sottotitolo:** Il modello non è più un chatbot.

**Tre riquadri (verde chiaro: sono la soluzione)**
- *icona cervello* — **Orchestratore** — L'LLM coordina gli strumenti e ragiona
- *icona chiave* — **Tool dedicati** — MCP + File Search
- *icona frecce che si dividono* — **Separazione dei compiti** — LLM e tool con responsabilità distinte

**Immagine:** lo stesso LLM, ora dentro l'harness
![[Pasted image 20260927104714.png|223]]

**Copione / Discorso (Luca):**
> "Costruire un harness attorno all’LLM.
> 
> Il modello non viene più usato come un semplice chatbot, ma diventa un **orchestratore** che ha il compito di ragionare e interpretare i dati che recupera tramite gli strumenti che gli mettiamo a disposizione.
> 
> Noi gli forniamo strumenti specializzati:
> - uno per accedere ai **dati di gara** e ai calcoli sulla telemetria;
> - uno per recuperare informazioni dai **documenti** tramite ricerca semantica.
> 
> In questo modo separiamo i compiti:
> - l’LLM si occupa di **interpretare e ragionare**;
> - i tool si occupano di **recuperare dati reali ed eseguire operazioni precise**;
> - il sistema di retrieval si occupa di **recuperare solo il contesto documentale utile**."

---

## Slide 7 — Una richiesta, due percorsi (Architettura)

**Layout:** quattro riquadri su fondo verde chiaro (icona + titolo verde + testo scuro, senza bordi colorati) a sinistra, schema dell'architettura grande a destra
**Occhiello:** ARCHITETTURA

**Immagine:** architettura completa
![[Pasted image 20260927104658.png|519]]

**Quattro riquadri (accanto allo schema)**
- *icona cervello* — **OCI Generative AI** — Riceve dal client, via Responses API, il prompt dell'utente e gli strumenti disponibili.
- *icona chiave* — **Tool calling** — Il modello decide quali strumenti utilizzare, anche più volte nello stesso flusso. Per la telemetria invoca i tool esposti dal server MCP.
- *icona database* — **`file_search`** — Tool di retrieval: recupera dall'OCI Vector Store i passaggi più rilevanti per la richiesta.
- *icona percorso* — **ngrok** — Tunnel HTTPS: rende il server MCP locale raggiungibile da OCI Generative AI nel cloud.

**Copione / Discorso (Luca):**
> "L’ingegnere di gara interagisce con una User Interface, che comunica con un client incaricato di costruire la richiesta da inviare a OCI Generative AI tramite la Responses API.
> 
> Alla richiesta non passiamo solamente il prompt dell’utente: rendiamo disponibili al modello anche gli strumenti che può utilizzare durante il ragionamento.
> 
> Nel nostro caso i due percorsi principali sono:
> - **File Search**, per recuperare informazioni dai documenti indicizzati nell’OCI Vector Store;
> - **MCP**, per invocare i tool esposti dal nostro server e accedere alla telemetria.
> 
> Una volta ricevuta la richiesta, il modello può decidere autonomamente quali strumenti utilizzare e può effettuare più chiamate ai tool all’interno dello stesso flusso prima di produrre la risposta finale.
> 
> Se servono informazioni normative o tecniche dai documenti, utilizza `file_search` sul Vector Store. Se invece servono dati di gara o calcoli sulla telemetria, utilizza i tool del server MCP.
> 
> Il server MCP gira localmente sulla nostra macchina, mentre OCI Generative AI è nel cloud. Per renderlo raggiungibile dall’esterno abbiamo quindi utilizzato **ngrok**, che crea un tunnel HTTPS pubblico verso il server locale."

---

## Slide 8 — I due strumenti: MCP e OCI Vector Store

**Layout:** due card affiancate su fondo verde chiaro (senza bordi, titoli verdi, testo scuro), ognuna con disegno schematico + spiegazione
**Occhiello:** TECNOLOGIE

### Card sinistra — 01 · DATI DI GARA — MCP Server
**Immagine:**
![[Pasted image 20260927115007.png|248]]

**Frase:** Espone tool per accedere in modo **controllato** a dati esterni ed eseguire calcoli deterministici.
- Telemetria in **JSON / JSONL**
- Standard per descrivere e invocare tool
- Chiamato da **OCI Generative AI** tramite MCP
- Calcoli deterministici eseguiti via codice

### Card destra — 02 · DOCUMENTI — OCI Vector Store
**Immagine:**
![[embedding.jpg|256]]

**Frase:** Embedding e ricerca semantica sui documenti
- Ogni chunk diventa un **vettore multidimensionale**
- Significati simili → vettori **vicini nello spazio**
- Ricerca semantica tramite `file_search`
- Indicizzazione e retrieval gestiti da OCI

**Copione / Discorso (Luca):**
> "Un MCP (Model context protocol) Server espone una serie di tool che permettono al modello di accedere in modo controllato a dati esterni, nel nostro caso telemetria salvata in JSON e JSONL. La comunicazione con OCI Generative AI avviene tramite il Model Context Protocol, che definisce in modo standardizzato come descrivere, invocare e restituire i risultati dei tool.
> 
> Un Vector Store è una struttura progettata per memorizzare embedding e recuperare contenuti semanticamente simili a una query:
> - ogni testo viene trasformato in un **vettore multidimensionale** che ne rappresenta il significato;
> - testi con significati simili tendono a essere vicini nello spazio degli embedding;
> - questo permette di effettuare **ricerca semantica**, non basata solo sulla corrispondenza esatta delle parole;
> - nel nostro progetto, indicizzazione e retrieval sono gestiti dal servizio **OCI Vector Store**, interrogato tramite `file_search`."

## SLIDE 9:
Titolo slide: I tool MCP

Inserisci questo snippet di codice sullla sinistra:
`@mcp.tool()`
`def get_lap_window(start_lap: int, end_lap: int) -> Dict[str, Any]:`
    `"""Return a list of lap records between start_lap and end_lap, inclusive."""`
    `meta = _load_json("race_session_telemetry.json")`

    `result = []`

    `for lap in meta["laps"]:`
        `if start_lap <= lap["lap"] <= end_lap:`
            `result.append(lap)`

    `return {`
        `"start_lap": start_lap,`
        `"end_lap": end_lap,`
        `"laps_found": len(result),`
        `"laps": result`
    `}`
"Implementazione dell'mcp tool richiesto dal task 1: get_lap_window"
Inserisci Questo snippet di codice sulla destra:

`@mcp.tool()`
`def recommend_pit_strategy(current_lap: int) -> Dict[str, Any]:`
    `"""Analizza strategia e usura per raccomandare il momento`
    `ottimale del pit stop e stimarne l'impatto sulla gara."""`
    `...`
    `laps_to_limit = (95.0 - current_wear) / deg_rate   # calcolo deterministico`
    `...`
    `return {"recommendation": "PIT_IN_WINDOW", "target_pit_lap": 27, ...}`
"Implementazione di un mcp tool che abbiamo aggiunto per offrire funzionalità all'interfaccia grafica del task 3: get_lap_window"

## SLIDE 10:
Titolo Slide: l'OCI vector Store

Inserisci Questo snippet di codice sulla sinistra:
`chunks = chunk_pdf_per_ai(pdf_path, chunk_size=1500, overlap=200)`
`for i, chunk in enumerate(chunks):`
    `f = client.files.create(file=to_file(chunk), purpose="user_data")`
    `client.vector_stores.files.create(`
        `vector_store_id=VECTOR_STORE_ID, file_id=f.id,`
        `attributes={"original_file": name, "chunk_index": i})`

Sulla destra il testo: "I file vengono divisi in chunks di dimensione fissa, per evitare di superare i limiti di memoria per file imposti dal vector store"

## SLIDE 11:
Titolo Slide: Prompt Engineering

Metti questo blocco di codice sulla sinistra:

`"You are an AI Race Engineer Copilot for Scuderia Tor Vergata.`
`You combine two main capabilities:`
1. `Race telemetry and strategy analysis through MCP tools.`
2. `Regulatory compliance and technical dossier analysis through file_search.`
`...`
`Do not guess.`
`Use MCP tools for structured race data.`
`Use file_search for regulations, rulebooks, technical dossiers, component specifications, and compliance evidence.`
`If evidence is missing, incomplete, ambiguous, or unavailable, say so explicitly.`
`Never invent:`
- `telemetry values`
- `lap data`
`...`
`For every answer, you must explicitly state which evidence source was used.`
`You must classify the answer source as one of the following:`
- `MCP only`
- `file_search only`
- `MCP + file_search`
- `No tool evidence available`
`...`
`For each compliance or mixed compliance answer, provide a documentation support score.`
`This score is a percentage estimate of how strongly the answer is supported by the provided documentation.`
`Important:`
- `This percentage does NOT mean how legally compliant the component or action is.`
- `It means how strongly the available documents support your answer.`
- `Use lower percentages when information is missing, ambiguous, indirect, or only partially retrieved.`
`...`

Sulla destra metti la seguente lista:
* Non inventare dati quando non sei sicuro della risposta
* Affidati ai tool MCP e file_search
* Dichiara la percentuale di informazioni presa da ogni fonte
* Mostra il ragionamento step by step che farebbe un ingegnere di gara
### SLIDE 12: Task 3 — Interfaccia del Race Engineer
Una dashboard Streamlit per consultare i dati della gara, interrogare l'assistente e caricare documenti tecnici.

- Video: VIDEO 2 (demo dashboard)
- Snippet dei grafici:
```python
tab1, tab2, tab3 = st.tabs(["Lap time", "Energy", "Position"])

with tab1:
    if "lap_time_s" in df.columns and df["lap_time_s"].notna().any():
        chart_df = df.set_index("lap")[["lap_time_s"]]
        st.line_chart(chart_df, height=180)
```


### SLIDE 13 Dalla quick action alla risposta
I dati della gara vengono recuperati tramite MCP e trasformati in una risposta operativa.
**Video:** VIDEO 1 (quick action)
**Pipeline:**
- **ANDATA:** `Streamlit` → `client Python` → `OCI Responses API · modello` → `strumenti MCP`
- **RITORNO:** `risultati MCP` → `OCI Responses API · modello` → `client Python` → `chat Streamlit`


### SLIDE 14 Verifica documentale BAT-X9
Un secondo percorso per il modello: file_search apre l'accesso ai documenti.
**Video:** VIDEO 3
**Pipeline:**
- **ANDATA:** `Streamlit` → `client Python` → `OCI Responses API · modello` → `file_search → documenti`
- **RITORNO:** `passaggi recuperati` → `OCI Responses API · modello` → `client Python` → `chat Streamlit`


### SLIDE 15 La fonte dipende dalla domanda
Una sola Responses API, due percorsi di evidenza.
| Analisi dello pneumatico e pit stop             | Verifica BAT-X9                                                        |
| ----------------------------------------------- | ---------------------------------------------------------------------- |
| **MCP 100%** · `file_search` 0%                 | MCP 0% · **`file_search` 100%**                                        |
| **Dato:** usura 61,4% contro 43,1% atteso       | **Documento trovato:** dossier BAT-X9                                  |
| **Risposta:** sosta proposta tra i giri 24 e 27 | **Risposta:** conformità non confermata; norma decisiva non recuperata |


### SLIDE 16 Dalla telemetria al post social
I dati in gara alimentano anche la comunicazione.
**Video:** `media3`, dall'inizio alla fine.

Sotto il video, tre passaggi brevi:
- `get_session_overview` → ultimo giro
- `get_lap_window` → ultimi 5 giri
- **Dati di gara → bozza del post**
### SLIDE 17 
Dalla simulazione alla decisione, fino alla comunicazione: questo è il nostro AI Race Engineer Copilot.
Grazie.
![[image9.jpeg|515]]