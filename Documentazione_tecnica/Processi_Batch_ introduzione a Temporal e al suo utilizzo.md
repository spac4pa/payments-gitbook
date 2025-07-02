# Processi Batch: introduzione a Temporal e al suo utilizzo

# Introduzione

[Temporal.io](http://temporal.io/) è una piattaforma open source progettata per orchestrare workflow distribuiti e affidabili. Si distingue per fornire gestione dello stato, retry automatici, timeout, e recupero da errori per codice asincrono distribuito, senza sacrificare la semplicità del modello di programmazione.  
Un workflow, in estrema sintesi, rappresenta sequenza di task.  
Temporal ne permette le seguenti modalità di esecuzione:

* On-demand  
* Schedulati secondo un certo delay  
* Temporizzati per essere eseguiti con una certa frequenza.

   
Per maggiori dettagli, si rimanda alla documentazione ufficiale:

[Temporal Platform Documentation | Temporal Platform Documentation](https://docs.temporal.io/)

## Vantaggi

* Affidabilità e tolleranza ai guasti  
  Garantisce che i workflow siano resilienti ai fallimenti. Riprova automaticamente i task falliti e mantiene lo stato dei workflow, assicurando che essi possano recuperare da crash o problemi di rete senza perdere progressi.  
* Scalabilità  
  Può gestire un gran numero di workflow contemporaneamente, rendendolo adatto per applicazioni che richiedono alta scalabilità. Distribuisce il carico di lavoro su più worker, permettendo un utilizzo efficiente delle risorse. La distribuzione del carico può avvenire sia sui workflow che sulle singole attività.  
* Gestione dello stato  
  Fornisce una gestione dello stato integrata, permettendo agli sviluppatori di concentrarsi sulla logica applicativa piuttosto che sulla persistenza e il recupero dello stato. Il framework assicura che lo stato sia mantenuto in modo consistente, possa essere interrogato in qualsiasi momento e possa essere cifrato applicativamente.  
* Workflow basati su eventi  
  Fornisce supporto nativo per workflow basati su eventi (modello push e non solo pull).  
  In tal caso Temporal assume anche il ruolo di event broker.  
* Definizione semplificata dei workflow  
  Permette di definire i workflow utilizzando codice (in linguaggio Java nel nostro caso), rendendo più facile creare, mantenere e gestire le versioni di workflow complessi.  
* Timeout e retry  
  Fornisce supporto integrato per timeout e retry, permettendo agli sviluppatori di specificare quanto tempo un task dovrebbe durare e quante volte dovrebbe essere riprovato in caso di fallimento. Ciò permette di sviluppare workflow robusti e molto resilienti.  
* Supporto per più linguaggi  
  Supporta diversi linguaggi di programmazione, tra cui Java, Go e TypeScript, permettendo agli sviluppatori di utilizzare il loro linguaggio preferito per definire workflow e attività.  
  * Integrato con Spring Boot  
    Si integra con Spring Boot grazie ad un apposito [starter](https://github.com/temporalio/sdk-java/tree/master/temporal-spring-boot-autoconfigure).  
* Visibilità e monitoraggio  
  Offre strumenti per monitorare e visualizzare i workflow, rendendo più facile tracciare i progressi e diagnosticare problemi. Tutto ciò aiuta a mantenere la visibilità operativa e ad assicurare che i workflow stiano funzionando nella maniera prevista.  
* Separazione della logica di workflow da quella applicativa  
   Separa l'orchestrazione del workflow dalla logica di business, rendendo il codice più modulare e più facile da mantenere.  
* Deploy indipendente  
  Non presenta vendor lock-in verso Cloud Service Provider: è possibile installarlo ovunque, anche on-premises.  
* Facilità di test  
  I workflow sono semplici da eseguire e monitorare: altamente modulari, per cui ogni loro task è riutilizzabile e testabile in modo isolato.

# Architettura

Temporal si compone di 2 componenti principali:

1. La propria applicazione, che tramite l’SDK di Temporal vi si integra registrandosi dinamicamente come suo Worker  
2. I servizi di Temporal

Per maggiori dettagli, si rimanda alla documentazione ufficiale:  
[Understanding Temporal | Temporal Platform Documentation](https://docs.temporal.io/evaluate/understanding-temporal)

## Worker

Rappresentano 1 o più applicativi che si registrano dinamicamente verso i servizi di Temporal allo scopo di integrarvisi e quindi essere in grado di:

* Comportarsi da client e quindi:  
  * Ispezionare lo stato di Temporal e dei suoi workflow  
  * Eseguire o schedulare nuovi workflow  
* Eseguire i Task che i servizi di Temporal commissionano

### Tipologie di Worker

I Task commissionati da Temporal si suddividono in:

1. Workflow Task  
   La logica di orchestrazione, ossia l’invocazione delle singole Activity e/o l’attesa di determinati eventi.  
2. Activity Task  
   Le unità di lavoro effettive (chiamate API, operazioni su DB, invio email, ecc.).

   
Di conseguenza anche i Worker si suddividono a loro volta in:

1. Workflow Worker  
2. Activity Worker

   
Tuttavia un singolo Worker può registrarsi per entrambe le tipologie.

## Servizi di Temporal

Temporal si compone di una serie di servizi che insieme prendono il nome di Temporal Server, progettati per essere scalabili orizzontalmente, ciascuno avente un ben preciso ruolo nell'orchestrazione dei workflow.  
Open image-20250530-164141.png

### Frontend Service

Rappresenta l’interfaccia principale che, data una richiesta da parte di un client:

1. Vi applica il rate limiting;  
2. La autentica;  
3. Infine la smista internamente.

### Matching Service

Associa le attività ai worker disponibili e fa da coda per le attività in attesa di esecuzione.

### History Service

Tiene traccia dello stato e della cronologia del workflow:

1. Ogni evento (inizio attività, completamento, errore, timeout, ecc.) viene registrato come evento immutabile.  
2. Usa event sourcing per garantire la riproducibilità e consistenza.

### Worker Service

Per l’esecuzione di workflow interni, gestire timeouts, decisioni, ecc.

### Persistenza

Temporal fa uso di un database persistente (PostgreSQL, MySQL o Cassandra) per:

* Salvare la cronologia degli eventi.  
* Conservare lo stato dei workflow.  
* Gestire le code e task.

### Elasticsearch

Opzionalmente Temporal può usare un server Elasticsearch al fine estendere le possibilità di ricerca dei workflow, ad esempio:

* Workflow in stato Running, Completed, Failed, ecc.  
* Workflow per nome, tipo, namespace.  
* Workflow che hanno una custom search attribute (es. CustomerID \= "12345").  
* Filtri per tempo (startTime, closeTime, ecc.).

# Multi-tenancy

Tramite la definizione dei namespace, Temporal è in grado di isolare logicamente i workflow, in modo tale che una singola installazione sia in grado di supportare molteplici progetti.

# Monitoraggio

Temporal fornisce 2 strumenti utili per il monitoraggio.

## Temporal UI

Un'interfaccia grafica web che permette di:

1. Osservare lo stato dei workflow ed i dettagli delle loro esecuzioni;  
2. Terminarli;  
3. Resettarli, in modo da farli ripartire da un ben preciso evento;  
4. Eseguirli fornendo nuovi parametri (tramite questa interfaccia è possibile solamente eseguire workflow che accettano un solo parametro, anche se questo può a suo volta essere un oggetto complesso).

Open image-20250530-171613.png

## Temporal CLI

Tramite la quale è possibile eseguire le stesse operazioni del tool grafico, ma non si è soggetti al limite sul numero di parametri tramite i quali è possibile avviare un workflow.

# Deployment

È possibile avere a disposizione un’installazione di Temporal secondo le seguenti modalità:

* Temporal Cloud: [il servizio SaaS ufficiale](https://docs.temporal.io/cloud/get-started/).  
* Temporal Self-hosted: [installandolo ed erogandolo da sè](https://docs.temporal.io/self-hosted-guide/deployment) (ad esempio su Kubernetes tramite Helm chart).

# Utilizzo in Piattaforma Unitaria

All’interno di Piattaforma unitaria è stata creata una libreria, payhub-activities, allo scopo di avere una unica code base dove implementare le Activity e poterle riusare all’occorrenza all’interno di più applicativi che si possano integrare con Temporal.

* A meno delle annotazioni necessarie a Temporal, questa libreria non presenta ulteriori utilizzi del suo SDK: le Activity ivi implementate di fatto rappresentano dei puri bean di Spring utilizzabili anche al di fuori di Temporal.

   
Sulla base di questa libreria, sono stati realizzati 2 microservizi che si integrano con Temporal:

1. workflow-hub:  
   * Si registra su Temporal sia come Workflow Worker, sia come Activity Worker per attività più elementari, spesso legate alla mera invocazione di ulteriori workflow.  
   * Rappresenta una Spring Boot application che nasconde l’uso di Temporal agli altri microservizi per mezzo di API che traducono le richieste in ingresso in comandi verso Temporal.  
   * Fa uso della libreria payhub-activities per avere a disposizione le interfacce mediante le quali richiedere tramite l’SDK l’esecuzione delle Activity.  
2. workflow-worker:  
   * Si registra su Temporal unicamente come Activity worker.  
   * Si presenta come un applicativo corredato della sola classe main, le classi di configurazioni di Temporal e del file application.yml tramite il quale viene configurata l’integrazione verso Temporal e la registrazione dinamica delle Activity.  
     Per il resto, la logica applicativa è unicamente contenuta nella libreria payhub-activities.

   
I workflow creati si possono suddividere nelle seguenti categorie:

* Gestione delle Posizioni Debitorie  
* Sincronizzazione delle Posizioni Debitorie, stampa avvisi massiva e notifica IO  
* Workflow custom per la gestione delle Posizioni Debitorie  
* Import dei dati  
* Export dei dati  
* Classificazione  
* Integrazione con SEND  
* Accertamenti

