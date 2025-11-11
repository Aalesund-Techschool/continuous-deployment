# 3 - Continuous deployment

Hva skal vi innom:
- Sette opp Github Pages
- Bruke Github Actions til å deploye en applikasjon til Github Pages

I denne delen skal vi jobbe med å få web-applikasjonen vår deployet ut til Github Pages.

GitHub Pages er en gratis webhosting-tjeneste fra GitHub som lar deg publisere statiske nettsider direkte fra et GitHub-repository.

## 3.1 - Oppsett av Github Pages

I repoet, gå inn i "Settings" i topp-fanen, og velg "Pages". 

Du vil få opp denne siden: 
![Enable github pages](pages.png)


Under "Source", endre fra "Deploy from branch" til "Github Actions"
![Enable Github Action](pages_enable_gha.png)

Vi har nå mulighet til å deploye kode til Github Pages fra Github Actions :tada:

## 3.2 - Deploy av web-applikasjon til Pages

I YAML-filen vi har jobbet i før, legg til ett nytt steg som vist under. Dette steget tar artifakten vi bygget i forrige steg og deployet dette til Github Pages.

```diff
name: Build and deploy

on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
+    permissions:
+      id-token: write
+      pages: write
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
      - name: Upload static files as artifact
        uses: actions/upload-pages-artifact@v3 
        with:
          path: code/dist
+      - name: Deploy to GitHub Pages
+        id: deployment
+        uses: actions/deploy-pages@v4
```

:pencil2: Sjekk at koden din er tilgjengelig på internett. Du finner lenke til Github Pages ved å besøke "Settings" og videre til "Pages". 
![alt text](pages_link.png)

:tada: Du har nå deployet noe ut på internett, etter å ha kjørt automatiske sjekker på koden! 

## 3.3 - Splitt CI / CD pipeline og bruk av Github Releases

Nå har vi en felles jobb from kjører både bygg og deploy. Som regel ønsker vi å skille disse to, der vi har kontinuerlig kjøring av CI-prosesser (bygg, linting og testing), mens vi har et aktivt forhold til når vi deployer koden. 

GitHub Releases lar deg markere viktige milepæler i prosjektet ditt ved å tagge spesifikke versjoner av koden (f.eks. `v1.0.0`, `v2.1.3`). Når du lager en release, kan du legge ved en changelog som beskriver hva som er nytt, endret eller fikset i denne versjonen. Dette gjør det enkelt for brukere og utviklere å følge med på utviklingen av prosjektet og forstå hva som har endret seg mellom ulike versjoner.

Vi kan også trigge actions ved å lage en release. Dette fungerer bra, med at vi kan generere en changelog over koden vår, tagge commit vi deployer og deretter deploye koden vår ut til et miljø. 

## 3.3.1 - Egen jobb for kvalitetssjekker

:pencil2: Rename `main.yaml` til `ci.yaml`. Denne filen skal nå kun inneholde kodesjekker, og ikke utføre deploy. 
Den oppdaterte filen kan se slik ut: 

```
name: Build
on:
  push:
    branches:
      - '**'
    tags-ignore:
      - '**'
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
      - run: npm run lint
      - run: npm test
```

Denne filen tar nå å sjekker ut koden for å så kjøre kvalitetssjekker. 

Vi har oppdatert reglene for triggers, ved å si at den skal trigge på alle branches, men ikke trigge på tagging av commits.

### 3.3.2 - Trigger deploy fra Github Releases

:pencil2: Opprett en ny actions fil som heter `deploy.yaml`. Legg følgende innhold inn:

```
name: Deploy
on:
  release:
    types: [created]
jobs:
  deploy:
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: ./code
    permissions:
      pages: write     
      id-token: write
    steps:
      - uses: actions/checkout@v4
      - name: Use Node.js 22.x
        uses: actions/setup-node@v4
        with:
          node-version: 22.x
      - run: npm ci
      - run: npm run build
      - name: Upload static files as artifact
        uses: actions/upload-pages-artifact@v3 
        with:
          path: code/dist
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

#### Hva gjør deploy-pipelineen?

Deploy-pipelineen trigges kun når en release opprettes (`on: release: types: [created]`). Dette gir deg full kontroll over når koden din deployes til produksjon.

Når pipelineen kjører, henter den ut koden fra den taggede releasen, installerer avhengigheter og bygger prosjektet. De kompilerte filene fra `code/dist`-mappen lastes deretter opp som et artifact til GitHub Pages med `upload-pages-artifact@v3`.

Til slutt deployer `deploy-pages@v4` artifaktet til GitHub Pages, slik at den nye versjonen av applikasjonen blir tilgjengelig på internett. På denne måten sikrer du at kun godkjente og taggede versjoner av koden din når produksjon.


:pencil2: Gå til Github.com og opprett en Release for å trigge deploy-pipeline. 
I roten av repositoriet har du en lenke til "Releases" og "Create new release".
![](create_new_release.png)

Under "Tag"-dropdown, velg deg et tag-navn (Forslag: Bruk en versjonstag, f.eks. v1.0.0, v1 etc). Velg et passende navn på release. 
![](release_info.png)


Trykk på "Publish release" nederst i skjermbildet. Sjekk at Action din kjører og at ny deploy går ut.
![](publish_release.png)


### 3.3.3 - Testing av flyten

:pencil2: Utfør endringer i koden. Kjør CI-pipeline, så deploy-pipeline, og sjekk at endringene dine blir publisert ut. 

### [Gå til oppgave 4 (Bonus) :arrow_right:](../oppgave-4/README.md)