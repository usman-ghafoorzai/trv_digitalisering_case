# TRV – innføring av arbeidstids- og ressursplanleggingssystem

Casebesvarelse utarbeidet av **Usman Ghafoorzai**.

Oppgaven gjelder innføring av et nytt tredjepartssystem for arbeidstid og ressursplanlegging, som skal fungere sammen med virksomhetens eksisterende ERP-system.

Jeg har behandlet caset som et lite forprosjekt. Jeg ville først forstått arbeidsprosessene, brukerne, dataene og systemansvaret, før jeg valgte API eller tegnet en ferdig arkitektur.

## Tilnærming

**Forstå prosessen → Aktører og behov → Domeneforståelse → Dataeierskap og informasjonsflyt → Kandidatarkitektur → Teknisk verifikasjon → Pilot og innføring → Måle effekten**

Jeg viser en rekkefølge her, men kartlegging, Use Case, krav og domenemodell ville utviklet seg i takt med nye avklaringer.

## Sentrale avklaringer

Underveis la jeg særlig merke til fem forhold som påvirker løsningsdesignet:

- **ERP støtter allerede timeføring.** Hvorfor trengs et nytt system når ERP kan registrere og kategorisere timer mot prosjekt, delprosjekt og aktivitet? Arbeidshypotesen min er at tredjepartssystemet skal dekke den operative arbeidsprosessen bedre: stempling, løpende registrering, rute/bemanning, oppfølging og godkjenning. Dette må valideres, slik at vi unngår dobbeltregistrering og to ulike sannheter.

- **Toveis dataflyt betyr ikke toveis eierskap til alle data.** ERP er foreløpig kandidat som autoritativ kilde for ansatte, organisatorisk tilhørighet og prosjektstruktur, mens tredjepartssystemet er kandidat som kilde for de operative dataene. Retur til ERP gjelder godkjent arbeidsgrunnlag til prosessene som trenger det, ikke vedlikehold av ERP-masterdata.

- **Aktivitet omtales som en «ekstra dimensjon».** Derfor vil jeg ikke automatisk modellere aktivitet som et nivå under delprosjekt. Hva begrepet representerer i ERP, må avklares før modell og mapping låses.

- **Ikke alle data trenger å flyttes.** Rute-, bemannings- og detaljert avviksinformasjon kan bli værende i tredjepartssystemet. Hvis et avvik påvirker arbeidstiden, kan det korrigerte og godkjente resultatet være det ERP trenger, fremfor hele historikken.

- **Integrasjonen må gå gjennom støttede grensesnitt.** Den skal ikke skrive direkte til systemenes databaser. Hvert system eier og oppdaterer sitt eget datalag.

---

## 1. Kartlegging og organisering

Jeg ville først kartlagt hvordan timer registreres i dag, hvor de manuelle stegene og problemene oppstår, og hva som må fungere ved første produksjonssetting.

Det krever innspill fra operative ansatte, operativ ledelse, HR/lønn, økonomi, HMS/kvalitet, logistikk/ruteplanlegging, IT og leverandørene. Forprosjektet skal redusere usikkerheten før vi begynner å bygge.

### Use Case

Jeg bruker Use Case for å avklare **hvem som faktisk skal gjøre hva i den nye arbeidsprosessen**. Roller og ansvar gir grunnlag for kravene, før data og teknologi bestemmes.

Operativ leder er en naturlig kandidat for godkjenning av arbeidstid, men det endelige ansvaret må avklares med virksomheten.

![Use Case](docs/02-krav-og-analyse/use-case.png)

[Forprosjektplan](docs/01-forprosjekt/forprosjektplan.pdf)\
[Kravdokumentasjon](docs/02-krav-og-analyse/kravdokumentasjon.pdf)

---

## 2. Data og integrasjoner

Når vi vet hvem som gjør hva, trenger vi et felles språk for de viktigste forretningsbegrepene. Kravene konkretiserer samtidig hva løsningen må støtte.

### Domenemodell

Hovedkjeden jeg har brukt er:

**Operativ ansatt → Arbeidsøkt → Arbeidstidsregistrering → relevant prosjekt/delprosjekt/aktivitet**

Modellen er konseptuell og beskriver begreper og sammenhenger, ikke database- eller API-design. Aktivitet må forstås før modellen detaljeres. Jeg har heller ikke antatt at en rute automatisk tilhører et prosjekt; en eventuell kobling må avklares.

![Domenemodell](docs/02-krav-og-analyse/domenemodell.png)

### Informasjonsflyt

Her spør jeg: **Hvor finnes dataene, hvem eier dem, og hvor må de videre?** Figuren viser denne foreløpige arbeidsdelingen:

**ERP → tredjepartssystem:** relevante grunndata\
**Tredjepartssystem:** operativ registrering, rute/bemanning, oppfølging og godkjenning\
**Tredjepartssystem → ERP:** relevant godkjent arbeidsgrunnlag

Aktivitet må avklares nærmere, og lønnsarter deles ved behov. Arbeidsgrunnlaget tilbake kan omfatte ansattreferanse, dato/periode, godkjente timer og relevant prosjekt-/aktivitetsinformasjon.

**Målet er ikke å holde to komplette systemkopier synkronisert, men å etablere tydelig dataeierskap og flytte den informasjonen mottakssystemet faktisk trenger.**

![Informasjonsflyt](docs/03-data-og-integrasjoner/informasjonsflyt.png)

---

## 3. Teknisk tilnærming

Først når informasjonsflyten er forstått, ville jeg vurdert hvordan den kan realiseres teknisk. API, fil, batch eller event/webhook må velges ut fra leverandørstøtte og oppdateringsbehov. REST er ikke et forhåndsvalg.

### High-Level kandidatarkitektur

Figuren viser en **konseptuell kandidatarkitektur**, ikke en ferdig løsning. Applikasjonene eier prosessene sine, integrasjonen bruker leverandørstøttede grensesnitt, og datalaget representerer logisk dataeierskap. Pilene må forstås gjennom applikasjonen:

**Integrasjon → støttet grensesnitt → applikasjon → systemets eget datalag**

Jeg ville først vurdert direkte leverandørintegrasjon dersom den dekker behovet sikkert og er forvaltbar. En egen integrasjonsfunksjon bør gi tydelig verdi, for eksempel mapping, validering, retry, logging, overvåking eller løsere kobling.

![High-Level arkitektur](docs/04-teknisk-tilnaerming/high-level-arkitektur.png)

### Tekniske avklaringer og PoC

Dokumentasjon og antakelser er ikke nok til å låse arkitekturen. Jeg ville først testet representative flyter i leverandørenes testmiljø: én testansatt og prosjekt/aktivitet til tredjepartssystemet, og ett godkjent arbeidsgrunnlag tilbake til ERP. Det kan verifisere autentisering, mapping, feilrespons, duplikater og sporbarhet.

[Teknologier og tekniske avklaringer](docs/04-teknisk-tilnaerming/teknologier-og-tekniske-avklaringer.pdf)

---

## 4. Gjennomføring og innføring

**Kartlegging → Teknisk PoC → Integrasjon → Testing → Pilot/opplæring → Produksjon og oppfølging**

Jeg ville ikke gått direkte fra teknisk test til full utrulling. En representativ pilot må vise at både den operative arbeidsprosessen og grunnlaget videre til ERP fungerer. Opplæringen må tilpasses rollene.

Etter produksjonssetting trengs stabilisering, overvåking, avstemming og brukeroppfølging, før løsningen overtas av forvaltningen.

### Overordnet tidsplan

Gantt er en overordnet illustrasjon av fremdrift. Varighet, avhengigheter og bemanning må avklares etter kartlegging og leverandørdialog; detaljplanen er ikke låst.

![Gantt](docs/01-forprosjekt/gantt.png)

[Roadmap – gjennomføring og innføring](docs/05-gjennomforing/roadmap.pdf)

---

## 5. Risiko og måloppnåelse

Jeg ville særlig fulgt opp uklart system- og dataeierskap, feil arbeidsgrunnlag til ERP, leverandørbegrensninger, integrasjonsfeil og manglende sporbarhet. Brukeradopsjon er like viktig: løsningen må fungere i den operative arbeidshverdagen.

**Prosjektet har ikke nødvendigvis lykkes bare fordi løsningen er satt i produksjon.**

Der det er mulig ville jeg etablert en baseline før innføring og målt manuelle korrigeringer, dobbeltregistrering, kvalitet i ERP-grunnlaget, brukeradopsjon og stabil drift. Da kan vi vurdere om situasjonen faktisk er blitt bedre.

[Risiko og måloppnåelse](docs/06-risiko-og-maal/risiko-og-maaloppnaaelse.pdf)

---

## Dokumentasjon

Den røde tråden i arbeidet er å:

- forstå prosessen før teknologien
- avklare system- og dataeierskap før integrasjonen bygges
- verifisere teknisk og organisatorisk før full utrulling, og måle effekten etterpå

Underlagsdokumentene gir detaljene:

- [Forprosjektplan](docs/01-forprosjekt/forprosjektplan.pdf)
- [Kravdokumentasjon](docs/02-krav-og-analyse/kravdokumentasjon.pdf)
- [Teknologier og tekniske avklaringer](docs/04-teknisk-tilnaerming/teknologier-og-tekniske-avklaringer.pdf)
- [Roadmap](docs/05-gjennomforing/roadmap.pdf)
- [Risiko og måloppnåelse](docs/06-risiko-og-maal/risiko-og-maaloppnaaelse.pdf)

DOCX-versjonene ligger i de samme mappene som redigerbare kildefiler.
