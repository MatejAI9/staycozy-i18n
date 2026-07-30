# StayCozy — i18n rečnici za custom kod (SC.t)

Flat JSON rečnici za runtime prevod hardkodovanog teksta iz custom JS embeda i
per-page before-body koda. Engleski je no-op default: ključ = **tačan engleski
string** kako se pojavljuje u kodu, vrednost = prevod.

## Fajlovi
- `es.json` — Spanish (es)
- `pt-BR.json` — Brazilian Portuguese (pt-BR)
- `en.json` — identitetska mapa (ključ = vrednost); opciona, korisna za test/fallback

Sva tri fajla imaju **identičan skup od 188 ključeva** (garantovano build skriptom).

## Kako se koristi (predloženi ugovor u site-wide Head)
Engine detektuje locale iz `document.documentElement.lang`, fetch-uje odgovarajući
JSON sa jsDelivr-a (verzionisan URL zbog cache-a) i izlaže `SC.t`:

```js
SC.t("See Availability")            // "en": vraća isti string (no-op)
                                     // "/es": "Ver disponibilidad"
SC.t("Max {n}", { n: 16 })          // "Máx. 16"
SC.t("From {price}/night", { price: "$120" })  // "Desde $120/noche"
```

Za `en` engine je striktni no-op (ne fetch-uje, vraća ključ) — engleski live sajt
ostaje netaknut.

## Placeholderi (format stringovi)
Ključevi sa `{...}` su format stringovi — implementacija ubacuje vrednost preko
drugog argumenta `SC.t(key, params)`:
`{n}`, `{price}`, `{date}`, `{time}`, `{status}`, `{error}`.
Npr. `s"From " + cur + price + "/night"` → `SC.t("From {price}/night",{price:cur+price})`.

## Množina
Množinski oblici su odvojeni ključevi (jer engleski kod radi `n + " Night" + (n!==1?"s":"")`):
- `Night`/`Nights`, `night`/`nights` (mala i velika slova su različiti ključevi — koristi ih tačno)
- `Guest`/`Guests`, `guest`/`guests`
- `Bedroom`/`Bedrooms`, `Bed`/`Beds`, `Bath`/`Baths`, `Photo`/`Photos`, `Room`/`Rooms`, `Studio`/`Studios`, `Apartment`/`Apartments`

## Meseci i dani — PREPORUKA: Intl, ne rečnik
Meseci/dani SU u rečniku radi kompletnosti, ali se preporučuje render preko
`Intl.DateTimeFormat(locale)` / `SC.fmtDate`, jer:
- engleska skraćenica `May` == pun mesec `May` (kolizija ključa — u rečniku je zadržan pun oblik),
- pravila velikog slova i skraćivanja se razlikuju po jeziku,
- day-header labele (`Mo`,`Tu`,…) su 2-slovne u en; es/pt ekvivalenti mogu biti duži — proveriti prelamanje u kalendaru.

## Case-sensitive duplikati (namerno odvojeni ključevi)
`Clear all` ≠ `Clear All`, `Contact us` ≠ `Contact Us`, `Loading…` (elipsa) ≠ `Loading...` (tri tačke),
`Night` ≠ `night`, `Guest` ≠ `guest`. Uvek koristi tačan oblik iz koda.

## Šta NIJE u ovim rečnicima
- **JSON-LD** (`application/ld+json`) tekst polja (`name`, `description`, FAQ pitanja/odgovori,
  `Available Properties in {city}`, itd.) — to je zaseban sloj, lokalizuje se **per-locale u head-u**
  (statički ili run-time patch), ne kroz ovaj runtime fetch rečnik.
- **Proper nouns**: nazivi gradova (Miami, Philadelphia, London, Birmingham), naziv brenda
  (StayCozy), nazivi objekata, mere (`sqft`), e-mail/telefon.
- **Native Webflow tekst i form polja/placeholderi** — to Webflow Localization prevodi sam.
- **Dashboard** (interni admin alat) — van scope-a lokalizacije.
- **Klase, ID-jevi, `data-` atributi, API parametri, dataLayer event imena** — NETAKNUTO.

## Pokrivene komponente (odakle su stringovi izvučeni)
Global nav; search widget (desktop bar + mobilni bottom-sheet); homepage benefits slides;
newsletter popup; reels; unit gallery; amenities popup (+kategorije); reviews popup;
floor plan / virtual tour; buildings property filter; unit counter; property/unit kartice;
map prazna stanja; MBW booking widget (unit page) — uklj. cancellation policy labele/tekst,
coupon, pricing labele; checkout before-body (money path) — greške, coupon, telefon;
checkout terms popup; checkout success screen.
