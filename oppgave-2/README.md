# Exercise 2 - CI/CD pipeline setup

I denne delen skal vi sette opp CI/CD-pipeline med Github Actions.

Vi skal innom:

- Oppsett og grunnleggende bruk av Github Actions
- Kjøre bygg av kode
- Utføre kodekvalitetssjekk
- Kjøring av tester

## 2.1 GitHub Actions

:book: Github Actions er en tjeneste fra Github som passer bra til å kjøre CI-steg og utføre enkle deployment-oppgaver.
Se quickstart-dokumentasjon om dette er nytt for deg: https://docs.github.com/en/actions/get-started/quickstart

## 2.2 GitHub Actions config

:book: GitHub Actions er "code as configuration", som betyr at vi konfigurerer bygg-, kvalitet-, og deploypipeline med å skrive kode. Github Actions kan vi skrive ved å bruke YAML.

:pencil2: Opprett en ny mappe i roten av prosjektet som heter `.github`

:pencil2: I `.github`, opprett en ny mappe som heter `workflows`.

:book: Mappestrukturen skal se slik ut:

````shell
.github
└── workflows
```The folder structure should look like this:

```shell
.github
└── workflows
````

### 2.2.1 GitHub Actions Hello World

:book: Før vi gjør noe nyttig i Github Actions, lager vi en liten test-workflow for å sjekke at alt kjører OK.

:pencil2: I `workflows`-mappen, opprett en ny fil `test.yml` med følgende innhold:

```yml
name: Github Actions Demo
run-name: Github Action run by ${{ github.actor }} 🚀
on: [push]
jobs:
  Test-GitHub-Actions:
    runs-on: ubuntu-latest
    steps:
      - run: echo "🎉 The job was automatically triggered by a ${{ github.event_name }} event."
      - run: echo "🐧 This job is now running on a ${{ runner.os }} server hosted by GitHub!"
      - run: echo "🔎 The name of your branch is ${{ github.ref }} and your repository is ${{ github.repository }}."
      - name: Check out repository code
        uses: actions/checkout@v4
      - run: echo "💡 The ${{ github.repository }} repository has been cloned to the runner."
      - run: echo "🖥️ The workflow is now ready to test your code on the runner."
      - name: List files in the repository
        run: |
          ls ${{ github.workspace }}
      - run: echo "🍏 This job's status is ${{ job.status }}."
```

:book: Filstruktur skal se slik ut:

```shell
.github
└── workflows
    └── test.yml
```

:pencil2: Lagre filen, commit og push endringene til Github.

Når du pusher endringene i en workshop fil til et remote repository på Github, trigges `push` event'en, som igjen kjører workshop'en.

:pencil2: På GitHub.com, gå til repositoriet ditt, og velg "Actions".

![](images/actions-tab.png)

:pencil2: På venstre side, klikk på action vi har kjørt (`Github Actions Demo`).

:pencil2: Fra listen over workflow runs, velg riktig run (`Github Action run by xxx 🚀`)

:book: Loggen viser hvor mange steg som ble kjørt. Du kan utvide hvert enkelt steg for å se videre detaljer.

### 2.4 Flyt for å bygge applikasjon

:book: Ett av de viktigste steget i en god CI-pipeline er _build_-steget, der vi kjører bygg/kompilering av koden for å sjekke at vi ikke har pushet kode som har ødelagt bygget ved hver endring.

:book: For å kjøre bygg i CI-pipelinen vår, kjører vi følgende kommandoer:

```bash
npm ci
npm run build
```

- :book: `npm ci` installerer alle dependencies. Denne kommandoen er lik `npm install`, men er ment til å brukes i automatiserte miljøer.
- :book: `npm run build` oppretter en `dist`-mappe (forkortelse for "distribution"), som inneholder et bygg av koden vår.

Disse kommandoene er spesifikke til app-typen vi lager. Dersom du skal bygge en annen type app, f.eks. Python, .NET, Java i pipelinen din, må du bruke spesifikke kommandoer for økosystemet du jobber i.

:pencil2: Slett `test.yml` vi laget tidligere i `.github\workflows`-mappen.

:pencil2: Opprett en ny fil `main.yml` i `.github\workflows`-mappen med følende innhold:

```yml
name: Build

on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: ./code
    steps:
      - uses: actions/checkout@v4
      - name: Use Node.js 22.x
        uses: actions/setup-node@v4
        with:
          node-version: 22.x
      - run: npm ci
      - run: npm run build
```

Oppsummering av hva workflow-filen gjør:

- `name:` - Definerer navn på workflow
- `on: [push]` - `on` definerer triggers for når workflow kjøres, her ved _push_ av commits.
- `jobs:` - Hvilke jobber vi ønsker å kjøre

  - `build:` - Navn på en jobb

    - `runs-on: ubuntu-latest` - Hvilken miljø vi kjører vi (her siste versjon av Ubuntu)
    - `defaults:` default settings for jobben
      - `run:` - settings for selve kjøring av jobbem
        - `working-directory: ./code` - sett mappe vi kjører jobben fra. Her definerer vi `./code`, som er der koden vår ligger
    - `steps:`

      - `- uses: actions/checkout@v4` - kloner ned repositoret (gjør koden tilgjengelig i action)
      - Install Node.js:

      ```yml
      - name: Use Node.js 22.x
        uses: actions/setup-node@v4
        with:
          node-version: 22.x
      ```

      - `- run: npm ci` - Installerer dependencies
      - `run: npm run build` - Bygger appen

:pencil2: Commit workflow'en og push til GitHub. Workflow'en bør trigges automatisk.

:pencil2: Åpne GitHub Actions workflow-oversikten og klikk på den nye `Build`-workflowen på venstre side. Klikk på workflow-kjøringen på høyre side for å se flere detaljer.

:book: En workflow-kjøring vil enten lykkes eller feile, avhengig av om noen av jobbstegene feiler.

:pencil2: Prøv å gjør en endring i koden for å få bygget til å feile (f.eks. ved å skrive ugyldig syntax). Sjekk at bygget feiler i Github. Reverter endring og se at bygget blir grønt igjen. 

## 2.5 Forbedring av Continuous-integration-flyt

:book: Husk at Continuous Integration handler om å sikre at koden vår er god nok til å deployes. Så langt gjør vi ikke mye for å bevise dette. Vi sørger for at appen kan bygges, men det er omtrent alt. La oss introdusere noen flere kvalitetssjekker.

## 2.5.1 Linting

Linting bruker vi for å verifisere at koden vår følger visse beste praksis og kodekonvensjoner. Vi bruker verktøyet _[ESLint](https://eslint.org/)_ til å gjøre dette for oss. ESLint analyserer koden din statistisk for raskt å finne problemer.

:pencil2: I et terminalvindu, gå inn i `code`-mappen og kjør `npm run lint` for å kjøre ESLint. Kommandoen skal ta noen sekunder, og så avslutte uten feil.

:pencil2: Åpne `code/src/main.js` og legg til følgende innhold:

```javascript
const unusedVariable = 3;
```

:pencil2: Lagre filen og kjør `npm run lint`. Du bør se en feilmelding som ser lik feilen som vist under.

```shell
<...>/code/src/main.js
  15:7  error  'unusedVariable' is assigned a value but never used  no-unused-vars

✖ 1 problem (1 error, 0 warnings)
```

Dette er en av kvalitetssjekene vi får med linting. Vi har en ubrukt variabel som ikke brukes noe sted (og som ikke tilfører noe verdi), så linteren vil gi oss tilbakemelding om at koden ikke følger beste praksis.

Dette er et eksempel på hvordan linting hjelper oss med å håndheve god kodingspraksis.

:pencil2: For å automatisk kjøre linting, kan vi legge inn et ekstra jobb-steg i `.github/workflows/main.yml`:

```diff
name: Build

on: [push]

jobs:
  build:

    runs-on: ubuntu-latest

    defaults:
      run:
        working-directory: ./code

    steps:
      - uses: actions/checkout@v4
      - name: Use Node.js 22.x
        uses: actions/setup-node@v4
        with:
          node-version: 22.x
      - run: npm ci
      - run: npm run build
+     - run: npm run lint
```

:pencil2: Commit og push endringene. Om du fremdeles har linting-feilen vi la til i forrige oppgave, vil action feile. Fjern denne, commit og push, og sjekk at pipeline kjører med linting.

### 2.5.2 Testing

:book: Det finnes allerede et oppsett for å kjøre tester i `package.json`.

:pencil2: Kjør `npm test`. Hvis terminalen sier "No tests found related to files changed since last commit" eller noe lignende, trykk `a`-tasten for å få den til å kjøre alle tester uansett. Trykk `q`-tasten for å avslutte.

:pencil2: For å kjøre tester automatisk i CI-pipeline, må vi legge til testing som et jobb-steg `.github/workflows/main.yml`:

```diff
name: Build

on: [push]

jobs:
  build:

    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4
      - name: Use Node.js 22.x
        uses: actions/setup-node@v4
        with:
          node-version: 22.x
      - run: npm ci
      - run: npm run build
      - run: npm run lint
+     - run: npm test
```

:pencil2: Commit endringene, og push til Github. Sjekk at testene kjører.

:star: Bonusoppgave: Prøv å endre testene i `code/src/clock.test.js`. Kan du få dem til å feile? Eller til og med lage nye tester som kjøres i CI-pipelinen?

## 2.6 Creating a build artifact

Før vi kan begynne å deploye noe til internett, trenger vi noe å deploye. Så langt har vi sjekket at koden vår bygger og at den består våre kvalitetssjekker gjennom linting og testing.

For å få et artifakt (artifakt = applikasjon som har blitt bygget), kan vi ta resultatet fra vårt byggsteg og lagre det som en fil relatert til Github Action-workflowen som vi nettopp har kjørt.

`npm build` vil produsere en mappe kalt `dist` i vår Github Action Runner. Vi kan ta den filen og tilknytte den til workflowen vår og bruke den i senere steg når vi skal deploye applikasjonen.

Modifiser workflow-filen din for å inkludere `Upload static files as artifact`-steget som vist

```diff
name: Build and deploy

on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: ./code
    steps:
      - uses: actions/checkout@v4

      - name: Use Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 22.x
      - run: npm ci
      - run: npm run build
      - run: npm run lint
      - run: npm run test
+      - name: Upload static files as artifact
+        uses: actions/upload-pages-artifact@v3 
+        with:
+          path: code/dist
```

Commit og push endringen din til Github, og du skal se følgende output nederst på workflow-siden din. Dette er applikasjonen vår som har blitt bygget og er klar til å deployes.

![](artifact_example.png)
---

I denne oppgaven har vi laget en fullstendig CI-pipeline. I neste oppgave skal vi lage opplegg for deploy av koden til Github Pages.

### [Gå til oppgave 3 :arrow_right:](../oppgave-3/README.md)
