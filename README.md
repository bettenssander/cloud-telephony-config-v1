# Cloud Telephony – configuratieformulier

Een webformulier in 6 stappen waarmee een klant (of een consultant samen met de klant) de configuratie van
**Cloud Telephony** invult. Op het einde wordt een **PDF-verslag** gemaakt in de Orange Business-huisstijl, met
tabellen én een tekening van de call-flow per telefoonnummer.

Er is **geen server en geen database**: alles gebeurt in de browser en de ingevulde gegevens verlaten het toestel
niet. Wie de pagina herlaadt, begint opnieuw.

## Bestanden

| Bestand | Doel |
|---|---|
| `index.html` | **Het formulier.** Alles zit in dit ene bestand (HTML, CSS, JavaScript, logo). Dit is het bestand dat je aanpast en publiceert. De naam `index.html` is nodig voor GitHub Pages. |
| `cloud-telephony-configuratie.html` | Identieke kopie van `index.html` (de versie in het Claude-artifact). |
| `call-flow-schema-test.html` | Losse testpagina van het blokkenschema (oorspronkelijk prototype). Niet nodig voor het formulier. |

Het formulier werkt **enkel online**: de PDF-bibliotheken (jsPDF 2.5.1 en AutoTable 3.8.2) worden bij het openen van
de pagina van cdnjs geladen. Zonder internet kan er geen PDF gemaakt worden.

## Gebruiken

Open `index.html` (of de gehoste link) in een browser. Vul de stappen in; de stappenbalk bovenaan laat toe om terug
te gaan zonder gegevens te verliezen. Vooruit springen controleert eerst de stappen ertussen.

1. **Klant** – bedrijfsnaam, adres, contactpersoon (verplicht).
2. **Nummers** – Belgische nummers of **nummerreeksen** (bv. `02 334 88 40` tot `49`, max. 1000 nummers).
   Nummers worden automatisch genoteerd als `+32 2 334 88 40`.
3. **Randapparatuur** – fax, parlofoon, alarm. Het nummer van fax en alarm kies je uit de lijst van stap 2.
4. **Nummerplan** – max. 3 gebruikers (intern nummer, rechtstreeks nummer, naam, PC/GSM-app, GSM, e-mail).
   Werkt als een Excel-tabel (pijltjes, Tab, Enter, plakken). Op een telefoon wordt elke rij een kaart.
5. **Call-flow** *(optioneel per nummer)* – kies eerst een nummer en bouw dan het schema met de `+`-knoppen:
   openingsuren, groep (met naam), keuzemenu (met "Geen keuze"), forward, boodschap, voicemail, inhaken.
   Klik op een blok voor de instellingen. Een gestart schema moet volledig zijn om verder te kunnen.
6. **Overzicht** – samenvatting en knop **PDF-verslag downloaden**.

## Publiceren (GitHub Pages)

1. Maak een publieke repository en upload `index.html` (en eventueel `README.md`).
2. *Settings → Pages → Deploy from a branch → `main` / `(root)`*.
3. De pagina staat op `https://<gebruiker>.github.io/<repository>/`.

Na elke wijziging: `index.html` opnieuw uploaden onder dezelfde naam. Omdat de gegevens enkel in de browser
blijven, hoeft de gehoste pagina geen persoonsgegevens te beheren.

## Aanpassen

Pas `index.html` aan en upload het opnieuw. Er is geen bouwstap, bundler of `package.json`. Pin bij een update van de
bibliotheken telkens een **exacte versie** in de twee `<script src>`-regels bovenaan het bestand.

## Codeoverzicht

De JavaScript staat onderaan het bronbestand en is opgedeeld in genummerde secties (zie de banners in de code).
De volgorde in het bestand is:

| Sectie | Inhoud |
|---|---|
| Algemeen | `val`, `cond` (vervolgvragen met `data-if="veld=waarde"`) |
| Stap 2 | `addNr`, `syncRng` – nummers en nummerreeksen toevoegen |
| Stap 4 | `addP`, `fillRn`, toetsenbord en plakken uit Excel |
| Call-flow: model | `mk` (nieuw blok), `slots` (aansluitpunten), constanten `NAME`, `SHORT`, `IC`, `COL`, `HELP` |
| Call-flow: formulieren | object `F` – instellingen per bloktype |
| Call-flow: tekst | `sub`, `own`, `issues`, `desc`, `outline` – beschrijving en controle |
| Call-flow: tekenen | `tile`, `plus`, `openMenu`, `nodeEl`, `panel`, `render`, `tabs` |
| Nummers | `fmtBE`, `fmtNr`, `expandNums`, `allNums`, `fillNrSel` |
| Gegevens | `initFlow`, `ppl`, `nrsList`, `gather`, `check` (validatie per stap) |
| PDF | `pdfFlow` (tekening), overzicht, PDF-download |
| Navigatie | `show`, `goto`, stappenbalk |

### Gegevensmodel van de call-flow

Een call-flow is een **boom van blokken**. Elk blok is een object met een type `t`:

```js
{ t: "nr",   nr: "+32 2 334 88 44", kid: <blok> }              // startblok
{ t: "oh",   hrs, days: [...], open: <blok>, closed: <blok>, pause: <blok> }
{ t: "grp",  name, sel: ["r1", "r2"], mode: "Simultaan"|"Cascade", wacht: "Ja"|"Nee", noans: <blok> }
{ t: "menu", keys: [{ k: "1"|…|"Geen keuze", c: <blok> }] }
{ t: "ext",  num }   { t: "msg", next: <blok> }   { t: "vm", vmu: ["r1"] }   { t: "hang" }
```

`slots(blok)` geeft de aansluitpunten van een blok. Alle functies die door de boom lopen (tekenen, controleren,
tekstschema, PDF-tekening) gebruiken enkel `slots()`. Een nieuw blok hoeft dus op één plaats beschreven te worden.
Gebruikers worden bijgehouden met een stabiel ID (`"r" + rijnummer`), zodat een gewijzigde naam in het nummerplan
automatisch in de call-flow doorwerkt.

### Een nieuw bloktype toevoegen

1. Voeg het type toe aan `NAME`, `SHORT`, `IC`, `COL` en `HELP`, en aan `ADD` als het als vervolgstap gekozen mag
   worden.
2. Geef het startwaarden in `mk()` en eventuele aansluitpunten in `slots()`.
3. Maak het instellingenformulier in `F` (of voeg het toe aan `NOSET` als er geen instellingen zijn).
4. Vul `sub()`, `own()` (controle) en `desc()` (tekst voor overzicht en PDF) aan.

De tekening in de PDF (`pdfFlow`) en het tekstschema nemen het nieuwe blok automatisch mee.

## Belgische nummers

`fmtBE` herkent `+32…`, `0032…` en nationale nummers. Geldig zijn vaste nummers (zones 02, 03, 04, 09 en de
tweecijferige zones in `Z2`) en GSM-nummers (045x–049x). Nummers zoals `0800` worden geweigerd. Een nummerreeks
(`fmtNr`) moet in dezelfde zone liggen, oplopend zijn en maximaal 1000 nummers tellen.

## Bekende beperkingen

- Gegevens worden niet bewaard na herladen of sluiten van de pagina.
- De PDF gebruikt het standaardlettertype van jsPDF; daarom staat in het tekstschema `>` in plaats van een pijl.
- In Claude gebruikt de PDF-knop de download-functie van het artifact (`window.claude.use("downloads")`); in een
  gewone browser valt hij terug op een gewone download-link.
- Een nummerreeks heeft **één** call-flow voor de hele reeks, geen aparte per nummer binnen de reeks.
- De functionaliteit is getest met geautomatiseerde scripts (jsdom) en de PDF is als afbeelding gecontroleerd.
  Controleer de opmaak zelf op uw eigen browser en telefoon.
