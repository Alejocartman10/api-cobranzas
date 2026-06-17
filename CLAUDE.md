# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a single-file debt collection management system ("Sistema de Cobranza IA") for Colombian collectors, built as a browser-only React app. The entire application lives in `index.html` — there is no build step, no package.json, and no bundler.

## Running the App

Open `index.html` directly in a browser. No server or build process required. React 18 and Babel are loaded via CDN; JSX is transpiled in-browser by `@babel/standalone`.

## Architecture

**Single-file SPA** with flat screen routing via a `pantalla` state string in the root `App` component. Navigation is purely state-driven — no router library.

### Screen flow

```
login
  └─ selectEtapa (coordinadores with etapa="todas")
       ├─ panel (PanelSupervisor — KPI dashboard)
       └─ listado
            ├─ registros
            └─ gestion
                 └─ whatsapp
```

Regular asesores skip `selectEtapa` and land directly on `listado` for their assigned `etapa`.

### Key components (all in `index.html`)

| Component | Role |
|---|---|
| `PantallaLogin` | Auth via Supabase RPC `login_usuario` |
| `PantallaSelectEtapa` | Coordinators pick a debt stage |
| `PantallaListado` | Client list filtered by stage, Ley 2300 locks applied |
| `PantallaGestion` | Record collection result; AI suggestion via Claude API |
| `PantallaWhatsApp` | Simulated WhatsApp thread with AI-suggested replies |
| `PantallaRegistros` | Today's logged gestiones from Supabase |
| `PanelSupervisor` | Executive KPIs, portfolio breakdown, advisor productivity |

### Data sources

- **`CLIENTES_MOCK`** — hardcoded array of 12 clients; saldo, diasMora, etapa, etc.
- **Supabase `gestiones` table** — persisted collection records; fetched via direct REST calls through `sbFetch()`. The anon key and URL are hardcoded constants at the top of the script.
- **Anthropic API** (`https://api.anthropic.com/v1/messages`) — called directly from the browser in `pedirIA()` (gestión screen) and `pedirSugerencia()` (WhatsApp screen). These calls currently omit the `x-api-key` header; adding it is required for the AI features to work.

### Ley 2300 compliance

`verificarCumplimiento(clienteId, gestiones)` enforces Colombian debt-collection law: blocks contact on Sundays, public holidays (hardcoded `FESTIVOS` array), outside Mon–Fri 7:00–19:00 / Sat 8:00–15:00, and if the client was already contacted today. Every client card and the gestión screen check this before allowing action.

### User roles

Determined by the `etapa` field returned from `login_usuario`:
- `"todas"` → coordinador; sees `selectEtapa` and the supervisor panel
- Any specific stage string → asesor; goes directly to their assigned `listado`
