# 📋 Foreningsplanner

Et simpelt, browserbaseret kanban-værktøj til foreninger og frivillige organisationer — inspireret af Microsoft Planner, men uden behov for M365-licenser eller IT-opsætning.

Alle i foreningen tilgår det samme board via én URL. Ingen app. Ingen login. Ingen abonnement.

---

## Funktioner

- **Flere boards** — ét per udvalg eller arrangement, hver med sin egen farve
- **Kanban kolonner** — standard: *Ikke startet, I gang, Til review, Færdig* — egne kan tilføjes frit
- **Opgavekort** med titel, beskrivelse, ansvarlig person og deadline
- **Deadline-farvekodning** — grøn (god tid), gul (≤3 dage), rød (overskredet)
- **Drag & drop** — flyt kort mellem kolonner
- **Delt i realtid** — alle brugere ser det samme board via en cloud-database
- **Forbindelsesstatus** — grøn/rød indikator i headeren viser om databasen er tilgængelig

---

## Kom i gang på 15 minutter

Løsningen kræver to gratis konti: **Turso** (database) og **Cloudflare Pages** (hosting).

### Trin 1 — Opret Turso database

Turso er en gratis, hosted SQLite-database. [Opret konto på turso.tech](https://turso.tech), installer derefter CLI:

```bash
# macOS / Linux
curl -sSfL https://get.tur.so/install.sh | bash

# Windows (PowerShell)
powershell -c "irm https://get.tur.so/install.sh | iex"
```

Opret din database og hent dine credentials:

```bash
turso auth login
turso db create foreningsplanner
turso db show foreningsplanner --url        # kopiér denne — starter med https://
turso db tokens create foreningsplanner    # kopiér dette token
```

### Trin 2 — Indsæt dine credentials

Åbn `planner.html` i en teksteditor og find de to linjer øverst i `<script>`-blokken:

```js
const TURSO_URL        = "TURSO_DATABASE_URL";
const TURSO_AUTH_TOKEN = "TURSO_AUTH_TOKEN";
```

Erstat med dine egne værdier:

```js
const TURSO_URL        = "https://foreningsplanner-xxx.turso.io";
const TURSO_AUTH_TOKEN = "eyJhbGc...dit-token...";
```

> ⚠️ URL skal starte med `https://` — ikke `libsql://`

### Trin 3 — Deploy til Cloudflare Pages

Pak filen i en zip (Cloudflare kræver zip-upload):

```bash
zip planner-upload.zip planner.html
```

1. Gå til [pages.cloudflare.com](https://pages.cloudflare.com) og opret en gratis konto
2. Klik **"Create a project"** → **"Direct upload"**
3. Giv projektet et navn, fx `foreningsplanner`
4. Upload `planner-upload.zip`
5. Del URL'en med foreningen — fx `foreningsplanner.pages.dev`

Databasetabellen oprettes automatisk første gang siden åbnes. Du er klar.

---

## Opdater til ny version

```bash
zip planner-upload.zip planner.html
```

Gå til dit projekt på Cloudflare Pages → **"Create new deployment"** → upload ny zip. URL'en forbliver den samme.

---

## Arkitektur

```
Browser → Cloudflare Pages (planner.html)
               ↓
          Turso HTTP API
               ↓
          SQLite database
```

Ingen server. Ingen backend. `planner.html` taler direkte med Turso via deres REST API. Al data gemmes som ét JSON-objekt i tabellen `planner_state`.

Begge services er **gratis** til dette formål og vil ikke nå deres grænser for en forenings brug.

---

## Sikkerhed

Turso auth-tokenet ligger i sidens kildekode, da der ikke er noget backend-lag. Det er acceptabelt til intern foreningsbrug med ikke-sensitiv data. Et reelt loginflow er planlagt som fremtidig forbedring.

---

## Fremtidige forbedringer

- [ ] Loginflow med brugernavn/password og roller (admin / læse-only)
- [ ] Læse-only visningsside til deling med hele foreningen
- [ ] Notifikationer ved deadline

---

## Licens

MIT — brug det frit, tilpas det, del det videre.
