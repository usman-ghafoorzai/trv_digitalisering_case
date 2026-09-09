# TRV – innføring av arbeidstids- og ressursplanleggingssystem

Casebesvarelse utarbeidet av **Usman Ghafoorzai**.

Oppgaven gjelder innføring av et nytt tredjepartssystem for arbeidstid og ressursplanlegging, som skal fungere sammen med virksomhetens eksisterende ERP-system.

Jeg har behandlet caset som et lite forprosjekt: først forstå behov, roller og data, deretter vurdere informasjonsflyt og teknisk retning, før testing, pilot, produksjonssetting og oppfølging planlegges.

## Tilnærming

**Kartlegging → Use Case → Domenemodell → Krav → Informasjonsflyt → High-Level arkitektur → Tekniske avklaringer / PoC → Roadmap → Risiko og måloppnåelse**

## Sentrale avklaringer

Arbeidet avdekket noen spørsmål som må avklares før løsningen kan designes ferdig:

- **Systemansvar:** ERP støtter allerede timeføring. Det må avklares hvilke deler av prosessen som faktisk skal løses i tredjepartssystemet, slik at dobbeltregistrering og to ulike sannheter unngås.

- **Dataeierskap:** ERP er foreløpig kandidat som autoritativ kilde for ansatte, organisatorisk tilhørighet og prosjektstruktur. Tredjepartssystemet er kandidat som kilde for operative arbeidstidsregistreringer, rute/bemanning, oppfølging og godkjenning.

- **Prosjekt, delprosjekt og aktivitet:** Aktivitet beskrives som en ekstra dimensjon i ERP og bør derfor ikke automatisk behandles som et nivå under delprosjekt. Det må avklares hva aktivitet representerer, hvordan den brukes og om den skal deles med tredjepartssystemet.

- **Tilbakeføring til ERP:** Tredjepartssystemet skal ikke nødvendigvis kunne endre ERP-masterdata. Det relevante er å sende godkjent arbeidsgrunnlag tilbake til de ERP-prosessene som trenger det.

- **Dataomfang:** Ikke alle operative data trenger å deles. Rute-, bemannings- og detaljert avviksinformasjon kan bli værende i tredjepartssystemet dersom det ikke finnes et konkret behov i ERP.

- **Lønnsarter:** ERP har lønnsarter, men det er ikke gitt at tredjepartssystemet trenger dem. Det må avklares hvor eventuell klassifisering skal skje.

- **Rute og prosjektføring:** Rute/bemanning og prosjekt-/aktivitetsføring er ikke nødvendigvis samme prosess. En eventuell kobling mellom dem må avklares, ikke antas.

- **Integrasjon og datalag:** Systemene bør integreres gjennom leverandørstøttede grensesnitt. Integrasjonen skal ikke forutsette direkte tilgang til systemenes databaser; hvert system eier og oppdaterer sitt eget datalag.

- **Teknologivalg:** API, fil, batch, event/webhook og eventuell integrasjonsfunksjon må vurderes etter leverandørstøtte og faktisk behov.

---

## 1. Kartlegging og organisering

Jeg ville startet med et kort forprosjekt sammen med berørte fagområder og representative brukere.

Sentrale aktører er HR/lønn, økonomi, HMS/kvalitet, operativ ledelse, logistikk/ruteplanlegging, IT/digitalisering, systemleverandørene og operative sluttbrukere.

Målet er å forstå dagens prosess, ønsket arbeidsflyt, ansvar, dataeierskap og hva som faktisk må fungere ved første produksjonssetting.

### Use Case

Use Case-diagrammet brukes for å avklare **hvem som skal bruke løsningen og hvilke oppgaver de må kunne utføre**.

Det gir et første funksjonelt bilde av behovene før vi går videre til data, integrasjoner og teknisk design.

![Use Case](docs/02-krav-og-analyse/use-case.png)

[Forprosjektplan](docs/01-forprosjekt/forprosjektplan.pdf)  
[Kravdokumentasjon](docs/02-krav-og-analyse/kravdokumentasjon.pdf)

---

## 2. Data og integrasjoner

Når aktører og behov er tydeligere, må de viktigste begrepene og dataene i løsningen forstås.

### Domenemodell

Domenemodellen brukes for å etablere et **felles språk rundt sentrale begreper og sammenhenger**, uten å blande inn tekniske komponenter eller databaseutforming.

| Relasjon | Beskrivelse |
|---|---|
| Operativ ansatt → Arbeidsøkt | Den ansatte registrerer arbeidsdagen sin. |
| Operativ ansatt → Organisasjonsenhet | Den ansatte har en organisatorisk tilhørighet. |
| Operativ ansatt → Rute | Ansatte kan bemannes på operative ruter. |
| Arbeidsøkt → Arbeidstidsregistrering | En arbeidsøkt består av registreringer gjennom dagen. |
| Arbeidstidsregistrering → Prosjekt / delprosjekt / aktivitet | Registrert tid kan knyttes til relevant arbeidskontekst. |
| Operativ leder → Rute | Leder/ruteplanlegger planlegger og bemanner ruter. |
| Operativ leder → Arbeidsøkt | Leder kan følge opp og godkjenne arbeidstid. |
| Arbeidsøkt → Avvik | En arbeidsøkt kan ha avvik som må følges opp. |

![Domenemodell](docs/02-krav-og-analyse/domenemodell.png)

### Informasjonsflyt

Informasjonsflyten viser **hvilke data som må bevege seg mellom ERP og tredjepartssystemet, i hvilken retning og til hvilket formål**.

Den skiller mellom:

- **ERP-eide grunndata** som ansatte, organisatorisk tilhørighet og prosjektstruktur
- **operative data** som oppstår i tredjepartssystemet
- **godkjent arbeidsgrunnlag** som kan sendes tilbake til ERP

En foreløpig arbeidshypotese er derfor:

**ERP → tredjepartssystem:** relevante grunndata  
**Tredjepartssystem:** registrering, rute/bemanning, oppfølging og godkjenning  
**Tredjepartssystem → ERP:** relevant godkjent arbeidsgrunnlag

Ikke alle operative data trenger å returneres til ERP.

![Informasjonsflyt](docs/03-data-og-integrasjoner/informasjonsflyt.png)

---

## 3. Teknisk tilnærming

Når informasjonsbehovet er tydeligere, kan vi vurdere hvordan systemene teknisk kan kobles sammen.

Før teknologi velges ville jeg undersøkt hva leverandørene faktisk støtter, blant annet API-er, fil/batch, events/webhooks, autentisering, dataformat, testmiljø og mekanismer for feil og drift.

### High-Level kandidatarkitektur

Arkitekturfiguren viser **hvordan informasjonsflyten kan realiseres teknisk på et overordnet nivå**.

Den er en kandidatarkitektur, ikke en ferdig løsning.

Systemene bør kommunisere gjennom leverandørstøttede grensesnitt. En egen integrasjonsfunksjon er først aktuell dersom det er behov for eksempelvis mapping, validering, retry, logging eller løsere kobling.

Hvert system eier og oppdaterer sitt eget datalag. Integrasjonen skal derfor ikke forutsette direkte tilgang til systemenes databaser.

![High-Level arkitektur](docs/04-teknisk-tilnaerming/high-level-arkitektur.png)

Før arkitekturen låses ville jeg verifisert noen representative flyter i testmiljø, for eksempel synkronisering av ansatt/prosjekt og retur av godkjent arbeidsgrunnlag.

[Teknologier og tekniske avklaringer](docs/04-teknisk-tilnaerming/teknologier-og-tekniske-avklaringer.pdf)

---

## 4. Gjennomføring og innføring

Gjennomføringen går fra kartlegging og krav, via tekniske avklaringer og integrasjon, til testing, pilot, opplæring og produksjonssetting.

Testing må dekke både teknisk dataflyt og den faktiske arbeidsprosessen. Før full utrulling ville jeg brukt en avgrenset pilot med representative brukere og operative scenarioer.

### Overordnet tidsplan

Gantt-diagrammet gir et overordnet bilde av **hovedfaser og fremdrift**.

Det er ikke ment som en detaljert eller låst prosjektplan. Konkret varighet, avhengigheter og bemanning må justeres etter kartlegging, leverandøravklaringer og valgt omfang.

![Gantt](docs/01-forprosjekt/gantt.png)

[Roadmap – gjennomføring og innføring](docs/05-gjennomforing/roadmap.pdf)

---

## 5. Risiko og måloppnåelse

De viktigste risikoene ligger ikke bare i teknologien.

Sentrale områder er uklart system- og dataeierskap, dobbeltregistrering, feil arbeidsgrunnlag til ERP, leverandørbegrensninger, integrasjonsfeil, manglende sporbarhet og lav brukeradopsjon.

Prosjektet er heller ikke vellykket bare fordi løsningen er satt i produksjon.

Effekten bør blant annet vurderes ut fra:

- kvalitet og sporbarhet i arbeidstidsdata
- færre manuelle korrigeringer
- mindre dobbeltregistrering
- korrekt grunnlag til ERP
- bedre støtte for rute- og ressursplanlegging
- faktisk brukeradopsjon
- stabil drift og tydelig forvaltningsansvar

Der det er mulig bør dagens situasjon måles før innføring, slik at effekten kan vurderes mot en reell baseline.

[Risiko og måloppnåelse](docs/06-risiko-og-maal/risiko-og-maaloppnaaelse.pdf)

---

## Dokumentasjon

Underlagsdokumentene er samlet i repositoryet:

- [Forprosjektplan](docs/01-forprosjekt/forprosjektplan.pdf)
- [Kravdokumentasjon](docs/02-krav-og-analyse/kravdokumentasjon.pdf)
- [Teknologier og tekniske avklaringer](docs/04-teknisk-tilnaerming/teknologier-og-tekniske-avklaringer.pdf)
- [Roadmap](docs/05-gjennomforing/roadmap.pdf)
- [Risiko og måloppnåelse](docs/06-risiko-og-maal/risiko-og-maaloppnaaelse.pdf)

DOCX-versjonene ligger i de samme mappene som redigerbare kildefiler.