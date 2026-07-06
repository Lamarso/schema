# HANDOFF – XXL Dagsschema

Kontext för nästa Claude Code-session. Läs denna fil först, sedan `index.html`.

## Vad detta är
Ett interaktivt dagsschemaläggningsverktyg för personalen på en XXL-butik. Byggt åt
Ammar, används av hans fru Rebecca som sätter dagsschemat. Allt ligger i **en enda fil**:
`index.html` (HTML + CSS + vanilla JS, ingen build, inga beroenden, inga externa scripts).

- Hostas på GitHub Pages: **https://lamarso.github.io/schema/**
- Repo: `https://github.com/Lamarso/schema` (branch `main`, fil `index.html` i root)
- Pages bygger om sig automatiskt ~1 min efter push till `main`.
- Språk i UI och kod-kommentarer: **svenska**.

## Arbetsflöde
1. Ändra direkt i `index.html`.
2. Efter varje redigering: kör en syntaxkontroll (se nedan) INNAN commit.
3. Committa + pusha till `main` först när Ammar godkänt. Pages uppdateras av sig självt.
4. Ammar beskriver ändringar på svenska i vanligt språk. Han föredrar EN genomtänkt
   lösning framför flera alternativ, minimal förklaring, och att man fångar logiska fel
   och undviker överkonstruktion.
5. Uppdatera **denna fil** ("Senaste läge") efter varje avslutad ändring, så nästa
   session kan fortsätta utan att behöva läsa hela konversationshistoriken.

### Syntaxkontroll före commit (gör alltid detta)
```bash
node -e "const c=require('fs').readFileSync('index.html','utf8').match(/<script>([\s\S]*)<\/script>/)[1]; new Function(c); console.log('JS-syntax OK');"
```
För logiktester: mocka `document`/`window.storage` och exponera interna funktioner genom
att ersätta `})();` i slutet av IIFE med `window.__t={...};})();`. Se git-historiken för
exempel – detta mönster har använts genomgående för att verifiera schemaläggningslogiken
mot Ammars riktiga personallista utan en webbläsare.

## KÄNSLIGA DELAR – rör bara på uttrycklig begäran
Två block högst upp i scriptet är kritiska för att sparning och förifyllt schema ska
fungera både lokalt och på Pages. Ändra dem inte utan att Ammar ber om det:

- **Lagringsadaptern** (`const store = ...`) direkt efter `"use strict";`. Väljer
  claude.ai-lagring om den finns, annars webbläsarens `localStorage`. Alla anrop i koden
  går via `store.get` / `store.set` – aldrig `window.storage` direkt.

Andra saker att inte råka förstöra:
- **Inga browser-storage-API:er utöver adaptern**, och inga externa CDN-scripts – filen
  ska fungera helt offline/fristående.
- Artefakter/Pages tål inte `localStorage` i vissa sandlådor; adaptern hanterar det.
  Nedladdning av bild sker via en overlay (se `showPngOverlay`) eftersom programmatisk
  `a.click()`-nedladdning blockeras i vissa inbäddade miljöer.

## Domänregler (affärslogiken – ändra inte utan begäran)
Avdelningar (DEPTS): Kläder, LY-Skor, Kassa, SRT-Sport, SRT/OH-Skor, SB & OH, Verkstad,
Lager, Möte, Uppgift, Lunch. (Cykel finns INTE – den ingår i SB & OH.)

- **Öppettider** styr när avdelningar måste vara bemannade (default 10–20). Lager och
  Verkstad kräver ingen täckning (jobbar ej kväll, slutar oftast 14–15).
- **Min-bemanning per avdelning** (0–3, ställbart). Skor ska ofta vara 2.
- **Skuldertimmar:** första och sista öppettimmen räcker alltid 1 person, oavsett min.
  Se `minAt(d,i)`.
- **Lunch under min-bemanning:** när någon från en avdelning är på lunch sänks kravet till
  1 den halvtimmen (den kvarvarande får vara ensam), men avdelningen får aldrig stå HELT
  tom – då ordnas en täckare. Se `reqAt(d,i)`.
- **Lunch:** 30 min för alla. Fönster default 12–14. Max N samtidigt i hela huset
  (inställbart, default 2). Behövs för pass ≥ 6 h. Personer kan bockas ur ("noLunch") om de
  inte behöver lunch en viss dag (t.ex. chefer som tar lunch när de vill).
- **Lunchordning:** slutar tidigt → äter tidigt; slutar sent → äter sent.
- **Lunchplacering är kostnadsmedveten** (se `planLunches` → `tryBlocks`): prioritet är
  1) fönstret utan täckare, 2) fönstret med täckare, 3) reservtid utan täckare,
  4) reservtid med täckare. Gratis lösningar vinner alltid över lösningar som flyttar folk.
  Reservtid ligger närmast fönstret, tidigast 2 h in i passet, senast 1 h före slut.
- **Vana-regeln:** all AUTOMATIK (Auto-placera, Fixa luckor, lunchtäckare) placerar ALDRIG
  någon på en avdelning de inte är vana vid (`home`). Förslagsknapparna i varningspanelen
  får däremot föreslå vem som helst (vana-personer först) eftersom Rebecca klickar själv.
- **Manuellt placerade rutor** (`state.manual`) markeras med prick och rörs ALDRIG av
  automatiken. Sudda för att låsa upp.

## Funktioner som finns
- Personaltabell: namn, start, slut, Lunch-kryssruta, avdelningsvana (klickbara taggar),
  ta bort. Ändringar sparas direkt (`persist()`), plus manuell "💾 Spara personal".
- Auto-placera alla / Fixa luckor / Planera luncher (knappar i header).
- Målning: välj färg i paletten, klicka & dra i schemat. Lunch och Sudda finns i paletten.
- Ångra (↩ eller Ctrl/Cmd+Z), upp till 20 steg. Snapshot före varje förstörande handling.
- Täckningstabell per timme (grön/gul/röd mot kravet).
- Varningar & förslag: luckor med "Placera"-knappar, lunch saknas/utanför fönster/överfull.
- Export till gruppchatt: "Skapa bild" (PNG i overlay: Kopiera bild / Ladda ner / spara via
  håll-in/högerklick) och "Kopiera chatt-text".
- Utskrift (liknar det ursprungliga Excel-arket).
- Datumväljare: byter dag; sparad dag laddas, annars hämtas personalregistret och schemat
  börjar tomt (seed läggs in om tomt). Personalregistret (`xxl-personal`) delas mellan dagar.
- Autospar via `persist()` / `scheduleAutosave()`. Status visas i headern ("✓ Sparat HH:MM").

## State-modell (i `index.html`)
```
state = {
  staff: [{id, name, start "HH:MM", end "HH:MM", home:[deptId...], noLunch?}],
  assignments: { staffId: { slotIdx: deptId } },   // slotIdx 0..28, 30-min steg från 06:00
  manual:      { staffId: { slotIdx: true } },     // Rebeccas egna, orörbara av auto
  mins:        { deptId: 0..3 },                    // min-bemanning
  openStart, openEnd, lunchStart, lunchEnd, lunchLen(=30), maxLunch,
  date "YYYY-MM-DD", seeded?
}
```
Tid ↔ slot: `slotStart(i) = 6*60 + i*30`. Dagen är 06:00–20:30, 29 slots (NSLOTS).
`migrate()` uppgraderar äldre sparad data (t.ex. `required`→`mins`, `cykel`→`sboh`).

## Kända begränsningar / öppna punkter
- **localStorage är per webbläsare och enhet** – schema gjort på datorn syns inte på mobilen.
  Räcker för nuvarande arbetsflöde (gör schema på ett ställe, dela bild). Äkta synk mellan
  enheter kräver en liten backend – större bygge, gör bara om Ammar ber om det.
- **Lunchplaneraren är girig** (en person i taget, ingen backtracking). Har inte gett fel
  resultat mot Ammars riktiga lista, och olösliga fall FLAGGAS i varningarna istället för
  att lösas tyst. Om ett verkligt fall dyker upp kan en global/optimerande lösare övervägas.
- **Strukturella luckor** (t.ex. sista timmen: färre personer än avdelningar) kan inget
  schema lösa – de flaggas medvetet så Rebecca beslutar (offra en avdelning / bredda vana /
  måla manuellt). Detta är avsiktligt, inte en bugg.

## Senaste läge (var vi slutade)
- Verktyget färdigt och deployat på GitHub Pages via `index.html`.
- Senaste ändringar i tur och ordning: lunch max-antal inställbart; vana-regel för
  automatik; skuldertimmar; lunch sänker min-bemanning till 1 (men aldrig tom); noLunch
  per person; kostnadsmedveten lunchplacering (fixade "Amanda hoppade in i onödan medan
  Oskar kunde täckt"-buggen); PNG-overlay istället för blockerad nedladdning; lagrings-
  adapter för lokal körning.
- Inga öppna buggar kända. Nästa steg är löpande justeringar på Ammars begäran.
- 2026-07-06: Repo klonat lokalt till denna arbetskatalog, GitHub-autentisering (PAT via
  osxkeychain) konfigurerad så push till `main` fungerar direkt. HANDOFF.md tillagd i
  repo-roten (denna fil) för att hållas uppdaterad löpande framöver.
- 2026-07-06: Tog bort hela SEED-mekanismen (`const SEED = {...}` + `applySeedIfEmpty()`,
  samt anropet av den vid start). En ny enhet/tom `localStorage` startar nu med ett helt
  tomt schema istället för att fyllas med det gamla testschemat för de 14 ursprungs-
  personerna. Personalregistret (`xxl-personal`) hämtas fortfarande som vanligt.
- 2026-07-06: Notiser (`toast`) syns längre (5–12 s beroende på längd, varningar extra)
  och kan klickas bort. Man hann inte läsa dem förut.
- 2026-07-06: Mobilanpassning (allt i `@media (max-width:640px)` + touch-hantering,
  desktop/mus helt orört). Schemats `touchstart` preventDefault-målning ersatt: svep
  skrollar schemat sidleds, en tap (utan att flytta fingret >8px) målar EN ruta. Mobil-
  layout: mindre padding, kompakta header-knappar, högre rutor (40px) för tap-mål,
  smalare namnkolumn med ellipsis, ram runt scrollytan, fullbredds-notiser.
- 2026-07-06: La till "↩ Ångra"-knapp i paletten direkt efter "✕ Sudda" (anropar samma
  `undo()`), så ångra finns nära där man målar – praktiskt på mobil vid feltryck.
