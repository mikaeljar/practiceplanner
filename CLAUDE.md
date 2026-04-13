# HHC Träningsplanerare — Projektbeskrivning för Claude Code

## Vad är det här?
Ett planeringsverktyg för ishockeytränare att hantera trupp, närvaro och gruppsättning inför en träning. Används i planeringsstadiet, inte under träningen.

## Teknisk grund
- Fristående HTML-fil (`hockey-practice-planner.html`), ingen backend, inga beroenden att installera
- All data sparas i `localStorage` under nyckeln `hockey-planner-v5`
- PWA med inline service worker för offline-stöd
- Hostad på GitHub Pages: https://mikaeljar.github.io/practiceplanner
- Google Drive-integration via OAuth 2.0 (opt-in)

---

## Datamodell

### Spelare `players[]`
```js
{
  id,           // uid() — stort tal som sträng, jämför alltid med String(x.id)===String(id)
  name,
  number,       // tröjnummer, sträng
  level,        // 'a'|'b'|'c'|'d'|'e'
  attendance,   // 'yes'|'no'|'unknown'
  position,     // 'forward'|'center'|'back'|'goalie'|null
  status,       // 'active'|'injured'|'longterm'
  guest,        // boolean — gäster sparas EJ i trupp-sparningar
  jerseyColour  // hex-sträng, sätts vid gruppsättning
}
```

### Ledare `coaches[]`
```js
{ id, name, coachType, attendance }
// coachType: 'on-ice'|'off-ice'|'goalie-coach'
```

### Grupper `groups[]`
```js
{ colour, colour2, fullColour, players[] }
```

### Övriga tillståndsvariabler
- `poolPlayers[]` — spelare utan grupp
- `goaliesInGroup[]` — ID:n för spelare i målvaktszonen
- `numStations` — antal stationer (1–8)
- `stations[]` — `{ label, drill, coaches[], fixed, hasDrill }`
- `usePositions` — boolean
- `practiceImageData` — base64-bild

---

## Viktiga designval och fallgropar

### ID-jämförelse
Använd **alltid** `String(x.id) === String(id)` — aldrig `Number()`. ID:n är stora tal från `uid()`.

### Touch-bug i närvaro
Närvaro-listan renderas **inte** om helt vid enskilda ändringar — bara toggle-knappar och `.absent`-klass uppdateras inline. Detta förhindrar en touch-bugg på mobil.

### Målvakt vs goalie
`position === 'goalie'` är det korrekta fältet. Legacy-data med `level === 'goalie'` migreras vid laddning: `level → 'b'`, `position → 'goalie'`.

### goaliesInGroup
Innehåller **ID:n**, inte player-objekt.

### Gäster
- `guest: true` på spelaren
- Sparas i tränings-snapshots men **filtreras bort** i `getRosterData()`
- Rensas automatiskt vid "Ny träning"
- Läggs automatiskt i pool eller målvaktszon om grupper redan finns

### Status (skadad/långtidsfrånvaro)
Vid "Ny träning" → `injured`/`longterm` sätts till `attendance: 'no'` istället för `'unknown'`

### Drill-historik
Sparas **vid blur** (när man lämnar fältet), inte vid varje tangenttryckning.

---

## localStorage-nycklar
| Nyckel | Innehåll | Max |
|--------|----------|-----|
| `hockey-planner-v5` | Aktuell träning (auto-save) | — |
| `hockey-planner-v5_img` | Träningsbild (base64, separat pga storlek) | — |
| `hhc-traning-list` | Manuellt sparade träningar | 15 |
| `hhc-roster-list` | Sparade truppar | 15 |
| `hhc-upplagg-list` | Sparade upplägg | 15 |
| `hhc-drill-history` | Övningshistorik per station | 8 per station |
| `hhc-drive-folder-id` | Google Drive mapp-ID | — |
| `hhc-theme` | `'dark'`\|`'light'` | — |
| `hhc-drive-consent` | Flagga för Drive-samtycke | — |
| `hhc-group-history` | Snapshots av genererade grupper (sparas vid träningskort) | 10 |
| `hhc-hold-apart` | Spelarpar som ska hållas isär vid generering | — |

Drive-token sparas i `sessionStorage` (försvinner när fliken stängs).

---

## Flikar och funktioner

### Planera träning (huvudflik)
- **Närvaro** — markera Ja/Nej/Okänd per spelare och ledare. "+ Lägg till gäst"-knapp under sammanfattningsraden.
- **Upplägg** — träningsbild + stationer med övningshistorik och ledarstilldelning
- **Grupper** — generera grupper, drag & drop, målvaktszon, otilldelad pool. Checkbox **"Använd historik"** viktar par som ofta hamnat ihop negativt (60 swap-försök, nivåbalans prioriteras). **Håll isär**-par är alltid aktiva (vikt 1000).

### Trupp (separat flik)
- Kollapsabara "Lägg till"-formulär för spelare och ledare
- Spara/Ladda/Rensa-band överst
- Status per spelare: Aktiv / Skadad / Långtidsfrånv.
- Toggle "Använd positioner"

### Inställningar (separat flik)
- **Grupphistorik** — lista över de 10 senaste träningskorten (sparas när träningskortet visas). Radera enskilda poster eller rensa all historik.
- **Håll isär** — spelarpar som algoritmen försöker separera vid gruppgenerering. Om separation ej möjlig pga nivåbalans → garanterat samma färg (samma lag). Alltid aktivt, oberoende av historik-checkbox.

---

## Spara/ladda-system
Tre typer med samma modal-mönster:

| Typ | Lokal nyckel | Drive-prefix |
|-----|-------------|--------------|
| Träning | `hhc-traning-list` | `traning-` |
| Trupp | `hhc-roster-list` | `trupp-` |
| Upplägg | `hhc-upplagg-list` | `upplägg-` |

- Lokala listor: radera-knapp per rad
- Drive-listor: paginering — träning/trupp 5 per sida, upplägg 10 per sida

---

## Google Drive
- OAuth Client ID: `602701212439-j7oep5soiafabgqiek28d659fm1fkoa5.apps.googleusercontent.com`
- Scope: `https://www.googleapis.com/auth/drive`
- Alla ledare delar samma mapp-ID
- Drive-knappen i headern: "Anslut" utloggad, förnamn inloggad

---

## UI-konventioner
- **Typsnitt**: Georgia (rubriker/body), Helvetica Neue (UI-element)
- **SVG-ikoner** genomgående — inga emoji i knappar
- **Tema**: ljust/mörkt, sparas i localStorage, fallback till systempreferens
- **Mobil**: long-press (500ms) → action sheet för spelarmovning i grupper; edit-sheet vid tryck på spelare/ledare i Trupp
- **Desktop**: dropdowns direkt på raden i Trupp

## Underhåll av denna fil
När ny funktionalitet läggs till eller befintlig ändras väsentligt ska denna fil uppdateras i samma session — innan commit. Det gäller:
- Nya fält i datamodellen
- Nya localStorage-nycklar
- Nya flikar, vyer eller UI-mönster
- Viktiga designbeslut eller fallgropar som tillkommer

---

## CSS-variabler (tema-medvetna)
```
--bg, --surface, --surface2, --border, --border-strong
--text, --text-muted, --text-faint
--accent, --accent-light, --accent-text
--a-bg/txt, --b-bg/txt, --c-bg/txt, --d-bg/txt, --e-bg/txt
--goalie-bg/txt, --coach-bg/txt
--on-bg/txt, --off-bg/txt
--danger-bg/txt, --warn-bg/txt
```
