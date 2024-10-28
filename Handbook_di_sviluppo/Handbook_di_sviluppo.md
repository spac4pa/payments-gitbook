# Pratiche di Sviluppo con Modello Gitflow Semplificato

Le pratiche di sviluppo seguono un modello **gitflow semplificato**, documentato nel file `CONTRIBUTING` presente nei repository. Di seguito, i punti più rilevanti:

## Branch Principali

Sono presenti esclusivamente 3 branch a vita lunga (*long-lived*):
- **develop**: trunk principale di sviluppo.
- **uat**: per test e QA.
- **main**: branch di produzione.

### Note sul Flusso di Lavoro
- **Niente Branch di Release**: Le release sono gestite tramite **tagging** direttamente su `main`, a differenza del modello gitflow tradizionale.
- **Feature Branch a Breve Termine**: Per minimizzare i conflitti, i **feature branch** devono essere incorporati in `develop` entro **3 giorni** (strategia short-lived).
- **Creazione dei Branch**: Ogni collaboratore crea i feature branch a partire da `develop`.

## Apertura di una Pull Request

Le pull request devono includere le seguenti informazioni minime per supportare il revisore e i workflow automatici; in caso contrario, sono rifiutate automaticamente:
1. **Titolo**: formato `semantic_tag: [XXX-999] breve descrizione della modifica`, dove `semantic_tag` può essere:
   - `fix`, `chore`, `feat`, o `breaking`, in base alla natura della modifica.
2. **Descrizione**: dettagli esaustivi delle modifiche.
3. **Test**: riferimenti alle modalità di testing per garantirne la qualità.
4. **Documentazione**: riferimenti alla documentazione aggiornata.

### Revisione della Pull Request

Il maintainer:
1. Revisiona la pull request e, se necessario, propone modifiche.
2. Approva la pull request una volta soddisfatto, e incorpora i commit su `develop` con **squash and merge**, eliminando il feature branch contestualmente.

## Strategie di Merge e Tagging

1. **Sottomettere una Feature per QA**: Se una feature è pronta per il QA della Comunità SPAC e del team di Quality Assurance di PagoPA, si apre una pull request da `develop` a `uat`.
2. **Proporre una Nuova Versione**: Per proporre il tagging di una nuova versione, si apre una pull request da `uat` a `main`.
3. Il maintainer incorpora i commit su `uat` o `main` utilizzando un **merge commit** per la pull request.

### Versionamento e Tagging

Il tag della nuova versione, con relativi artefatti e change-log, è generato da un workflow automatizzato, seguendo le specifiche del **versionamento semantico** `major.minor.patch` ([semver.org](https://semver.org/)):

- **Patch**: incremento se i commit contengono solo `fix`.
- **Minor**: incremento se i commit includono nuove feature.
- **Major**: incremento se i commit hanno prodotto una breaking change.
