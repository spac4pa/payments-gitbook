# Modalità di contribuzione

Le pratiche di sviluppo seguono un modello gitflow semplificato e saranno documentate nel file **CONTRIBUTING** presente nei repository. Di seguito si illustrano i punti più rilevanti:

1. **Branch long-lived**
   - Esistono esclusivamente tre branch long-lived: `develop`, `uat`, e `main`.
   - A differenza del modello gitflow tradizionale, non ci sono branch dedicati per le release, che vengono gestite tramite una strategia di tagging su `main`.
   - `develop` funge da trunk principale di sviluppo.

2. **Feature branch short-lived**
   - Per minimizzare i conflitti, i feature branch devono essere incorporati in `develop` entro 3 giorni (**strategia short-lived feature branch**).
   - Il collaboratore stacca i feature branch esclusivamente da `develop`.

3. **Pull request**
   - Ogni pull request deve contenere il seguente insieme minimo di informazioni per facilitare la revisione e supportare i workflow automatici, pena il rigetto automatico:
     - un titolo nella forma `semantic_tag: [XXX-999] descrizione breve della modifica`, dove `semantic_tag` può essere `fix`, `chore`, `feat` o `breaking`, in base alla natura della modifica;
     - una descrizione esaustiva delle modifiche;
     - riferimenti alle modalità di testing che ne garantiscono la qualità;
     - riferimenti alla documentazione aggiornata.

4. **Revisione e approvazione**
   - Il maintainer revisiona la pull request e, se necessario, propone modifiche. Il collaboratore e il maintainer lavorano sulle modifiche finché entrambi sono soddisfatti.
   - Il maintainer incorpora i commit su `develop` con un’operazione di **squash and merge**. Contestualmente, il feature branch viene eliminato.

5. **Deploy e tagging**
   - Per sottoporre le feature più recenti alla comunità SPAC e al team di Quality Assurance (messo a disposizione da PagoPA), il collaboratore propone una pull request da `develop` a `uat`.
   - Per il tagging di una nuova versione, il collaboratore propone una pull request da `uat` a `main`.
   - Il maintainer incorpora i commit su `uat` o `main` con un'operazione di **merge commit** della pull request.
   - La nuova versione corrisponde a un tag su `main`, generato automaticamente insieme agli artefatti e al changelog. Il tag segue le specifiche semantiche **major.minor.patch** ([semver.org](https://semver.org/)), incrementato come segue:
     - Se i commit contengono solo fix, si incrementa il `patch`.
     - Se i commit contengono nuove feature, si incrementa il `minor`.
     - Se i commit hanno prodotto una breaking change, si incrementa il `major`.
