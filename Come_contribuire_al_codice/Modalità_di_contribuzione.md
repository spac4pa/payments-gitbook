# Contribuzioni
Le pratiche di sviluppo seguono un modello gitflow semplificato e saranno documentate nel file CONTRIBUTING presente nei repository. Nel seguito si illustrano i punti più rilevanti

- Esistono esclusivamente 3 branch long lived: develop, uat, main
- A differenza del modello gitflow tradizionale, non esistono branch dedicate per le release, che sono gestite con una strategie di tagging su main. 
- Develop funge da trunk principale di sviluppo
- A differenza di un modello gitflow tradizionale, per minimizzare i conflitti, i feature branch devono essere incorporati in develop entro 3 giorni (strategia short-lived feature branch).
- Il collaboratore stacca i  feature branch esclusivamente da develop
- La pull request deve contenere il seguente insieme minimo di informazioni per semplificare il contributo del revisore e supportare i workflow automatici, altrimenti viene rigettata automaticamente.
   - Un titolo, che deve essere nella forma semantic_tag: [XXX-999] proposed change short description, dove il semantic_tag può essere fix, chore, feat o breaking in base alla natura della modifica,
   - Un descrizione esaustiva delle modifiche, 
   - Dei riferimenti alle modalità di testing che ne garantiscono la qualità,
   - dei riferimenti alla documentazione aggiornata.

- Il maintainer revisiona la pull request ed eventualmente l’approva. Altrimenti propone delle modifiche. Il collaboratore e il maintainer lavorano sulle modifiche finché non sono entrambi soddisfatti.
- Il maintainer incorpora i commit su develop con un’operazione di squash and merge. Contestualmente il feature branch viene eliminato.
- Se un collaboratore vuole sottoporre alla Comunità SPAC e al team di Quality Assurance - messo a disposizione da PagoPA - le feature più recenti propone una pull request da develop a uat. 
- Se un collaboratore vuole proporre il tagging di una nuova versione, propone una pull request da uat a main
- Il maintainer può incorporare i commit su uat  o main con un'operazione di merge commit della pull request
- La nuova versione corrisponde a un tag su main, i relativi artefatti e un change-log, generati da un workflow automatizzato. Il tag segue le specifiche del numero di versione semantico major.minor.patch (https://semver.org/) che viene incrementato automaticamente come segue:
   - se i commit contengono solo fix, c'è un incremento del patch
   - se i commit contengono nuove feature, c'è un incremento del minor
   - se i commit hanno prodotto una braking change, c'è un incremento del major


![Foto di esempio](spac4pa/payments-gitbook/Immagini/Workflow_Github.png)

# Ambienti
PagoPA mette a disposizione due ambienti per supportare il processo di sviluppo e controllo qualità:

- Un ambiente di sviluppo (o di dev) accessibile da tutti i collaboratori, utile per semplificare le pratiche di sviluppo;
- Un ambiente di User Acceptance Testing / Quality Assurance (o di uat) accessibile a tutti i membri della Comunità SPAC reload per valutare le feature e la qualità operativa.

Per tutti gli ambienti deve essere pienamente adottato l'approccio IaC (Terraform) al fine di poter sempre consentire alle amministrazioni riusanti di ricostruire l’ambiente in cui l’applicativo è operato.

# Qualità
Esistono processi di qualità operanti a diversi livelli:

- Qualità del Codice
- Qualità delle Feature
- Qualità Operativa

## Qualità del Codice
La qualità del codice è garantita da workflow automatici di test, controlli statici e scansioni di sicurezza come meglio dettagliato in seguito.

- Il codice presente nella pull request da feature branch a develop è sottoposto a test automatici di unità e di integrazione, che devono essere superati al 100%.

   - I test di unità devono garantire una branch coverage del 100% e una line coverage del 90%.

   - I test di unità sono basati sulle librerie junit/mockito e fanno un uso aggressivo del mocking.

   - La suite di unit test deve poter essere eseguibile in 10 minuti.

   - Il tool di riferimento per i test di integrazione è Postman.

   - I test di integrazione possono essere eseguiti automaticamente in un ambiente di sviluppo fornito da PagoPA (ambiente di dev) e allineato da pratiche di Continuous delivery con il branch develop.

   - Per agevolare le pratiche di test-automation, tutti i test vivono nello stesso repository dell’applicativo.

- Il codice differenziale presente nella pull request è sottoposto a scansione statica con SonarCloud, mediante un account messo a disposizione da PagoPA. La scansione ha l’obiettivo di rilevare e misurare con un rating le cattive pratiche di coding (es. mancanza di modularità, eccesso di ereditarietà, etc.), potenziali bug e potenziali problemi di sicurezza. Il rating è espresso con una lettera da A (rating eccellente) a E (rating pessimo). Il maintainer non può approvare la pull request se i rating non sono tutti A.

- Tutti i repository sono integrati con un security scan automatico (trivy) che opera in fase di pull request e periodicamente. La pull request può essere incorporata in develop se e solo se non esistono criticità di livello pari o superiore a HIGH.

### Qualità delle Feature
La qualità delle feature è garantita dall’uso di strategie di Behavior Driven Development e dal coinvolgimento della community, come meglio dettagliato in seguito.

- Il codice presente nella pull request da feature branch a develop è sottoposto a feature test automatici, che devono essere superati al 100%.

- I test di feature devono essere concordati prima dei relativi sviluppi (Behavior Driven Development).

- Il tool di riferimento per i test di feature è Behave/Cucumber.

- È compito del collaboratore proporre nel contesto della pull request i test di feature.

- I feature test sono eseguiti in un ambiente di sviluppo messo a disposizione da PagoPA (ambiente di dev) allineato con pratiche di Continuous Delivery con il branch develop.

- Se esiste un nuovo insieme di feature correlate su develop, un collaboratore può chiedere l’ingaggio del team di QA di PagoPA e della community aprendo una pull request da develop a uat.

- Al merge della pull request, ciò che esiste sul branch di uat viene rilasciato automaticamente in un ambiente di User Acceptance Testing and Quality Assurance (ambiente di uat).

- Tutti i membri della community possono accedere all’ambiente di uat per testare manualmente o automaticamente le nuove feature, in particolare l’integrazione con il SIL della PA.

- Il team di QA di PagoPA mantiene una propria suite di acceptance testing, anch’essa basata su Behave/Cucumber o Playwright, contenente test di acceptance che coinvolgono diverse feature e che viene lanciata automaticamente all’atto del rilascio in uat. L’esito dei test viene trasmesso ai membri della community.

- Tutti i membri della community, ricevuti gli esiti dei test, sono invitati a esprimere le loro valutazioni sulla qualità delle feature prodotte.

### Qualità Operativa
La community adotterà delle pratiche di supporto all’eccellenza operativa, garantendo che il sistema supporti correttamente i livelli di carico previsti e abbia il livello corretto di osservabilità.

A tal fine, il team di QA di PagoPA mantiene una suite di test di carico basata sul tool k6 che può essere lanciata a richiesta nell’ambiente di UAT e il cui esito è trasmesso ai membri della community.

Contestualmente ai test di carico, il team di PagoPA valuterà l’osservabilità del sistema e trasmetterà un feedback alla community.

## Security
- Ogni contribuzione deve rispettare le apposite security checklist (che verranno condivise).

- PagoPA mette a disposizione un team di Security Expert ingaggiabile a richiesta.
