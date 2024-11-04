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

```bash
brew install pre-commit
Se il file .pre-commit-config.yaml è già presente nella root del repository allora è sufficiente eseguire il comando:
pre-commit install

Se il file .pre-commit-config.yaml non fosse presente nella root del repository allora è necessario crearne uno prendendo come esempio il seguente:


Puoi copiare e incollare questo testo in un file Markdown!


