# Definition of done
Un Task/User Story si considera chiuso se e solo se è in produzione ed è stata staccata una nuova release, fatta eccezione per i task di sperimentazione o laboratorio.
- **Test Coverage**: Overall (line + branch) > 90%. Vedi qui per delle indicazioni sullo sviluppo dei test.
- **Branch Coverage**: 100%
- **Feature test**: superati con successo
- **Controlli statici del codice**: (es. Sonar) non sono disabilitati e presentano la valutazione A per tutte le voci
- **Criticità di sicurezza**: non ci sono criticità di livello maggiore o uguale a HIGH

# Gestione del repository
- Ogni Pull Request deve essere revisionata da almeno un altro membro del team di sviluppo. Questo evita lo sviluppo autoreferenziale.
- Modello di branching Git-flow, con l’uso di tre branch long-lived: `develop`, `uat`, `main`
- Per evitare errori in fase di creazione delle pull-requests, il branch default è `develop`.
- Il merge da feature branch a `develop` deve essere fatto tramite squash and merge, in modo da riportare nel commit il titolo della pull-request, che sarà usato per costruire un change-log automatico e per determinare la versione da rilasciare.
- Il merge tra ambienti, quindi da `develop` a `uat` e da `uat` a `main`, deve avvenire tramite merge commit.
- Dopo l’incorporazione in `develop`, i feature-branch possono essere eliminati.

# Pull-request, rilascio e versionamento

## Ciclo di vita Pull Request
Ogni PR deve essere legata a un ticket Jira tramite la naming convention riportata in seguito.

Si adotta una strategia short lived feature branch, per cui una PR deve soddisfare alcuni requisiti:
- **Durata**: dall’apertura della PR, deve essere chiusa entro 48h
- **Dimensione**: il numero di righe modificate deve essere significativamente basso (20-30 righe, esclusi test per ogni PR)
- **Superficie di impatto**: pochi file modificati
- **Recupero debito e refactor**: è lecito effettuare modifiche non inerenti al ticket a cui la PR è collegato, a patto che le modifiche siano facilmente separabili e distinguibili in fase di review. In ogni caso, se le modifiche impattano tante righe di codice, vanno spostate in una PR dedicata.
- **Approvazione**: per i semantic tag `feat` e `breaking`, occorre la review e l’approvazione di un membro di `p4pa-admin` o di un gruppo equivalente.

Nel caso in cui non sia possibile rispettare questi requisiti, occorre fare il possibile per dividere la PR in più sotto-task.

## Rilascio e Versionamento
Il versionamento dei progetti segue il versionamento semantico `major.minor.patch` (e.g., `v1.2.1` - [Versionamento Semantico 2.0.0](https://semver.org/lang/it/)), e valgono le seguenti regole:
- Un `fix` produce un incremento della patch
- Un set di `feature` produce un incremento della minor
- Un `breaking change` produce un incremento della major

Il rilascio delle versioni è gestito tramite la GitHub Action `cycjimmy/semantic-release-action@v2`, che provvederà, interpretando il semantic tag nei diversi commit contenuti nella pull-request, a staccare la versione nel momento in cui si effettua il merge di promozione verso il branch `main`.

Il semantic tag dei commit deriva dai titoli delle pull-requests che hanno subito uno squash and merge verso `develop`, che ha l'effetto di unificare tutti i commit, denominando il risultante con il titolo della pull-request. Di conseguenza, ogni pull-request verso `develop` deve obbligatoriamente avere un titolo che descriva chiaramente l’oggetto della pull-request e l’impatto dei cambiamenti proposti sul numero di versione.

In particolare, lo sviluppatore deve strutturare il titolo della pull request secondo lo schema `semantic_tag: [XXX-999] proposed change description`, utilizzando i seguenti semantic tag convenzionali:
- `fix` se lo sviluppatore ritiene che i cambiamenti debbano produrre un incremento di patch
- `feat` se lo sviluppatore ritiene che i cambiamenti debbano produrre un incremento della minor
- `breaking` se lo sviluppatore ritiene che i cambiamenti debbano produrre un incremento della major
- Se, invece, lo sviluppatore ritiene che i cambiamenti proposti siano abilitanti per feature future ma non debbano avere effetti sul numero di versione, può utilizzare il semantic-tag `chore`

Il nuovo numero di versione è pilotato dal commit con il semantic tag più rilevante. Se il tag più rilevante è `fix`, la action aumenterà il numero di patch della versione; se è `feat` aumenterà la minor; se è `breaking` aumenterà la major. Se i commit sono taggati con `chore` o non hanno nessun tag, non sarà staccata una nuova versione.

Il processo di versionamento è interamente basato sui titoli delle PR e non dipende dal numero di versione riportato nel `pom.xml` o nel `build.gradle`, che dovrà essere allineato manualmente.

## Template Pull Request
Il template nella descrizione delle pull request è descritto dal file .github/PULL_REQUEST_TEMPLATE.md

# Api Rest
- https://learn.microsoft.com/en-us/azure/architecture/best-practices/api-design
- https://roy.gbiv.com/untangled/2008/rest-apis-must-be-hypertext-driven
- https://en.wikipedia.org/wiki/Richardson_Maturity_Model

Distinguiamo due tipologie di **openapi**:

- **interno**: associabile alle API di backend esposte da un microservizio.
- **esterno**: generalmente non associabile a un singolo microservizio; è la specifica esterna che viene esposta su APIM.

# Generazione openapi interno
In un approccio **API-first** si tende a preferire la scrittura delle specifiche di un servizio prima della sua implementazione interna.

Le specifiche di un servizio definiscono l’interfaccia con cui gli altri sistemi possono interagire con quel servizio. Lo standard per scrivere una specifica è **openapi** in formato YAML.

Esistono due approcci possibili:
- Scrivere il file di specifica openapi manualmente e autogenerare il codice che dovrà servire quelle interfacce.
- Scrivere le interfacce utilizzando il framework/linguaggio preferito e autogenerare il file di specifiche openapi.

I due approcci sono funzionalmente equivalenti, ma viene preferito lo **swagger**, in quanto è generalmente l’artefatto prodotto per primo (ad esempio come output DR).

---

## Gestione openapi
Ogni microservizio che espone delle API deve mantenere, all’interno del repository, il suo **openapi interno** (è indifferente se viene autogenerato dal controller o viceversa).

Se viene utilizzato un **API Gateway** come APIM di Azure, allora esisterà anche un **openapi esterno**, ovvero le API a cui accedono i client. Tale openapi viene utilizzato dal repository infra contenente il codice Terraform con cui si crea l’infrastruttura.

La proposta è mantenere gli openapi esterni in un unico repository, diverso da quello del microservizio e da quello dell’infra, es. `p4pa-api-spec`, in cui è possibile raggruppare tutte le interfacce di APIM e le potenziali policy.

Operativamente, la risorsa Terraform API referenzierà l’openapi del repository tramite un link.

Perché l’openapi interna ed esterna possono differire?
Possono esserci vari motivi:
- In un’architettura a microservizi, la struttura interna non deve trasparire all’esterno del sistema. Eventuali cambiamenti all’architettura interna (ad esempio il merge o lo split di due o più servizi) è preferibile che non abbiano impatti sullo swagger esterno (mentre ce li avranno sugli swagger interni).
- Cambio di parametri/header per fini di autenticazione o autorizzazione.
- Modifiche alle modalità di passaggio di un parametro (cambio da query param a body a path variable).
- Orchestrazione da parte di APIM tramite chiamate ad altri servizi.

Il repository `p4pa-api-spec` (o equivalente) seguirà un branching model identico a quello dei repository dei microservizi (**git flow**). Esistono quindi tre branch long-lived con versioni diverse in base all’ambiente.

---

## Async API
Anche le **API async** devono essere modellate e documentate opportunamente.

Per API async si intendono, ad esempio, i broker di messaggistica come Kafka o Rabbit.

Attraverso il tool **asyncapi** si può creare un file di specifica molto simile a openapi in cui si va a specificare se il servizio funge da **consumer**, **producer** e lo schema dell’evento. Tale file va mantenuto nel repository dell’applicazione in quanto documentazione inerente.

---

## Principi e Pattern
Aderire ai principi di programmazione:
- **DRY** (Don’t Repeat Yourself)
- **KISS** (Keep It Simple, Stupid)
- **YAGNI** (You Aren’t Gonna Need It)
- **SOLID**


**REF.** 
- https://gist.github.com/wojteklu/73c6914cc446146b8b533c0988cf8d29
- https://gist.github.com/wojteklu/73c6914cc446146b8b533c0988cf8d29


# Formattazione del codice
## Java 
Adottiamo la Google Java Style Guide (https://google.github.io/styleguide/javaguide.html)
Link alla configurazione per i principali IDE (https://github.com/google/styleguide/blob/gh-pages/intellij-java-google-style.xml) o (https://code.visualstudio.com/docs/java/java-linting)

# Pre-commit scripts

Per garantire la pulizia dei file inclusi nei commit utilizziamo alcuni hook di pre-commit.

Per installarlo sul proprio Mac è possibile utilizzare Brew:

`brew install pre-commit`

Se il file .pre-commit-config.yaml è già presente nella root del repository allora è sufficiente eseguire il comando:
`pre-commit install`

Se il file .pre-commit-config.yaml non fosse presente nella root del repository allora è necessario crearne uno prendendo come esempio il seguente:

`repos: - repo: https://github.com/pre-commit/pre-commit-hooks rev: v4.4.0 hooks: - id: check-case-conflict - id: check-executables-have-shebangs - id: check-json - id: check-xml - id: check-yaml - id: detect-private-key - id: end-of-file-fixer exclude_types: [sql] - id: mixed-line-ending args: [ --fix=lf ] exclude_types: [sql] - id: pretty-format-json - id: trailing-whitespace args: [--markdown-linebreak-ext=md] exclude_types: [sql]`

Se si tratta di codice Python è possibile concatenare le seguenti righe alle precedenti:Se si tratta di codice Terraform è possibile concatenare le seguenti righe alle precedenti:

Dopo di che eseguire:

`pre-commit install`
Fatto ciò, prima di ogni commit (tramite CLI o tramite GUI dell'IDE) viene eseguito il pre-commit.

Gli script `end-of-file-fixer` e `trailing-whitespace` non falliscono ma apportano direttamente le modifiche ai file prima del commit.
In caso di fallimento di uno step il commit non viene effettuato.

Per eseguire gli hook senza fare un commit è sufficiente lanciare:
`pre-commit run -a`
Per aggiornare gli script eseguire:
`pre-commit autoupdate`

# Microservizio

Un microservizio corrisponde a un unico applicativo Spring. Il relativo progetto è organizzato con le seguenti directory/packages rilevanti:

- `config`
- `controller`
- `dto`
- `model`
- `repository`
- `service`
- `exception`
- `utils`
- `event` (handler messaggi nel caso in cui il servizio sia connesso a una coda)

## Consigliato l’uso di:

- Lombok

L’evoluzione degli schemi degli eventuali DB relazionali è gestita con Liquibase.

## Pratiche sconsigliate

In un’architettura a microservizi è fondamentale mantenere i servizi disaccoppiati per garantire l'indipendenza del ciclo di vita di sviluppo e rilascio.

Sono deprecate librerie comuni e package condivisi tra microservizi. I servizi che utilizzano lo stesso stack tecnologico devono condividere soltanto il framework su cui sono basati, ad esempio Spring, Quarkus, Micronaut, ecc.

Partendo dal framework e dalle sue astrazioni vanno sviluppate tutte le funzionalità dell’applicazione limitando allo stretto indispensabile l’aggiunta di layer di astrazione o la riscrittura di parti del framework.

## refs:
- https://medium.com/@shanthi.shyju/shared-libraries-in-microservices-avoiding-an-antipattern-c9a3161276e
- https://medium.com/duda/shared-libraries-design-and-best-practices-710774ae0bdc
- https://stackoverflow.com/questions/48961000/why-shared-libraries-between-microservices-are-bad


# Stack reattivo vs stack tradizionale

Valutare lo stack reattivo con Spring e Reactor soltanto nei casi in cui le performance sono un fattore critico per il sistema. 
In tutti gli altri casi preferire lo stack tradizionale in quanto esiste una knowledge base più diffusa ed è più semplice manutenere l’applicativo nel lungo periodo.

## Tool di Build

I tool attualmente accettati sono Maven e Gradle. Gradle viene preferito in quanto consente di pinnare l’hash delle dipendenze per cui aderisce agli standard di security. È inoltre preferibile utilizzare il linguaggio Kotlin per la scrittura del file `build.gradle.kts`.

## CI/CD
- Utilizzo di helm chart per deploy su AKS, come previsto da pratiche infra.  
- Sonar Cloud
- Esclusioni Sonar: classi configurazione, test, enumerazioni
- La promozione tramite PR da DEV a UAT innesca una suite di test e2e (postman o k6)
- Possibilità di deployare in DEV da qualsiasi branch (rimuovere filtri sul nome del branch)


 
Lo strumento preferito per eseguire le pipeline di CI è **Github Actions**.
Lo strumento preferito per eseguire le pipeline di CD è **Azure DevOps**.

# Naming Pattern

Micro-servizi: `p4pa-xxxx-yyyy`

### Naming Branch e PR

Per innescare l’automazione Pull Requests <--> Jira e consentire l’aggiornamento automatico dello stato dei task, è necessario adottare le seguenti pratiche:

- **Denominazione branch:** `IDTASK-descrizione-feature` es. `ISB-180-develop-controller-for-onboarding-workflow-api`
- **Titolo PR:** `[IDTASK] Titolo PR` es. `[ISB-180] Develop Spring Boot Controller of APIs for Citizen Onboarding Workflow`
- **Messaggio commit:** `"[IDTASK] messaggio di commit"`

### Container non-root

È best practice far girare i container con un utente non-root. A grandi linee occorre effettuare questi passaggi:

1. nel dockerfile
   - creare un utente non root
   - correggere i permessi su eventuali file o java agent che l’applicazione deve leggere
   - impostare il nuovo utente come utente corrente dell’immagine docker
2. nei values dell’helm template
   - impostare il security context per girare con l’utente non root

Vedi qui per qualche snippet di esempio.

### Pinning dipendenze

Best practice per qualunque dipendenza importata in un progetto è pinnare lo sha del commit da cui si clona. Questa pratica riguarda: le dipendenze del sistema di build (gradle, in maven non è supportato il pinnning), le immagini docker, le action github e qualunque altro software di terzi parti. Vedi qui per degli snippet di esempio su come pinnare.

### Configurazione Azure EventHub

Vedi il documento HB Dev - Configurazione Azure Eventhub per le configurazioni raccomandate da Microsoft nel caso di utilizzo del loro broker managed.

### Github Packages

È necessario seguire le linee guida di Technology per mantenere un livello di sicurezza elevato quando si utilizzano i github packages per gestire le immagini docker dei microservizi. Le linee guida complete qui.

### Github Secrets

Linee guida di Technology qui.

### Affinity e anti-affinity

Per i prodotti in produzione è necessario rispettare determinati livelli di availability. Per aumentare questa caratteristica su k8s è possibile impostare il deployment per schedulare i pod su nodi diversi per tutelare il funzionamento del prodotto se uno dei nodi cade. Linee guida devops qui.

esempio concreto di implementazione su ms TAE:
- https://github.com/pagopa/rtd-ms-file-reporter/pull/113
- https://github.com/pagopa/rtd-ms-file-register/pull/172