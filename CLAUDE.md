# At Fielt Trimble Connect extensie — Project Dashboard

Deze extensie toont een overzicht van alle Trimble Connect projecten van de gebruiker, met rollen, leden-aantallen en activiteit. Geïntegreerd met de At Fielt Extension Hub voor toegangscontrole en usage-tracking.

## Werkwijze voor Claude Code sessies

Wanneer je deze extensie bewerkt:

- **Bewaar de hub-integratie blokken** tussen `╔══ AT FIELT EXTENSION HUB — NIET VERWIJDEREN ══╗` en `╔══ EINDE AT FIELT HUB BLOK ══╗` ongewijzigd, tenzij je het hub-contract aanpast.
- **De hub-script tag** bovenaan `<head>` niet verwijderen:
  ```html
  <script src="https://atfielt-extension-hub.tom-da0.workers.dev/client.js"></script>
  ```
- **De `[HUB] logEvent` calls** niet verwijderen — je mag wel het `data` object uitbreiden met extensie-specifieke tracking-velden.
- **Volledige integratie-referentie**: https://github.com/Tomtietom/atfielt-extension-hub/blob/main/HUB-INTEGRATION.md

## Publiceren

Na wijzigingen in `index.html`:
```bash
git add index.html
git commit -m "<korte beschrijving wat je wijzigde>"
git push
```

Wacht 1-3 min op GitHub Pages rebuild, dan verifieer:
1. Open extensie in Trimble Connect
2. Console (F12) → zoek `[Hub] Toegang verleend`
3. https://atfielt-extension-hub.tom-da0.workers.dev/admin → Events tab → nieuw `load` event verschijnt binnen 30s

## Deze extensie

- **Slug in hub-database**: `project-dashboard`
- **GitHub repo**: Tomtietom/trimble-project-dashboard
- **Manifest URL**: https://tomtietom.github.io/trimble-project-dashboard/
- **Hub URL**: https://atfielt-extension-hub.tom-da0.workers.dev
- **Admin dashboard**: https://atfielt-extension-hub.tom-da0.workers.dev/admin
- **Bijzonderheid**: deze extensie gebruikt een eigen proxy (tc-proxy.tom-da0.workers.dev) voor de meeste API-calls, maar roept Trimble `/users/me` direct aan (via client.js `check()`)

## Token-refresh patroon (2026-04-17)

Alle API-calls gaan via `apiFetch()` / `apiFetchPaginated()` / `fetchAllProjects()`. Deze gebruiken:
- **`fetchWithTimeout()`** — wrapt `fetch()` met 15-20s timeout via `AbortController`
- **`ensureValidToken()`** — checkt JWT `exp` claim (via `parseJWT()`, uitgebreid met `exp`) en refresht het token via `tcAPI.extension.getAccessToken()` als het verlopen is (60s marge)
- **401-retry** — bij 401 response wordt het token vernieuwd en de call herhaald

**Niet verwijderen of omzeilen.** JWT-tokens verlopen na ~1-2 uur; zonder deze helpers hangt de app voor eindgebruikers die het paneel lang open laten staan. Ditzelfde patroon is ook toegepast in tc-viewer, trimble-rechten-viewer en trimble-project-upload (daar met prefix `_` zoals `_ensureValidToken`).
