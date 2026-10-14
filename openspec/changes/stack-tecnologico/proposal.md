# Proposal

## Why

Il repository è un template senza stack applicativo né comportamento di prodotto: non c'è monorepo, né API, né UI, né persistenza. Serve fissare lo stack e introdurre le capacità minime per un'applicazione web di liste della spesa condivise in un Tenant, così che implementazione e test abbiano un contratto chiaro.

## What Changes

- Introdurre un monorepo npm workspaces con `apps/api` (Fastify + TypeScript + TypeORM + PostgreSQL) e `apps/web` (React + Vite + TypeScript + CSS Modules).
- Documentare stack e pattern in `.agent-guidelines/PATTERNS.md` (e ridistribuire le guidelines).
- Esporre un'API REST+JSON con autenticazione Account (email + password, session cookie httpOnly).
- Consentire registrazione che crea un Tenant oppure entra in uno esistente tramite Codice invito; ogni Account appartiene a un solo Tenant; i membri possono vedere e rigenerare il Codice invito (nessun ruolo distinto in v1).
- Consentire CRUD di Liste della spesa e Voci appartenenti al Tenant e visibili a tutti gli Account di quel Tenant.
- Aggiungere script `lint` / `test` / `build` e test con Vitest allineati allo stack.

## Capabilities

### New Capabilities

- `tenant-membership`: registrazione e autenticazione Account, appartenenza a un Tenant, gestione del Codice invito.
- `shopping-lists`: creazione e gestione di Liste della spesa e delle relative Voci nell'ambito di un Tenant.

### Modified Capabilities

- Nessuna (non esistono ancora capability in `openspec/specs/`).

## Non-goals

- OAuth / identity provider esterni.
- Ruoli (admin vs membro) o permessi granulari sulle Liste.
- Liste o dati condivisi tra Tenant diversi.
- Liste multiple private per Account (le Liste sono del Tenant).
- Categorie, prezzi o unità tipizzate sulle Voci.
- UI kit (MUI, shadcn, ecc.) o Tailwind.
- GraphQL, tRPC o realtime (WebSocket).
- Backend Go o frontend Next.js / fullstack unificato.
- App mobile nativa.

## Impact

- **Codice**: nuova struttura `apps/api`, `apps/web`; root `package.json` con workspaces; aggiornamento di `.agent-guidelines/PATTERNS.md` e file generati via `npm run sync-agents`.
- **API**: nuove risorse REST per Account/sessione, Tenant/Codice invito, Liste e Voci.
- **Dati**: schema PostgreSQL via TypeORM (migrazioni da definire in design).
- **Dipendenze** (domande aperte — da approvare prima dell'implementazione):
  - runtime/API: `fastify`, `typeorm`, `pg`, libreria session/cookie e hashing password;
  - web: `react`, `react-dom`, `vite`, toolchain TypeScript React;
  - qualità: `vitest` (e relativi adapter/plugin per api e web);
  - eventuale tooling TypeScript/ESLint condiviso in root o per app.
- **Operatività**: PostgreSQL in sviluppo (es. via Compose o istanza esterna — dettaglio in design); variabili d'ambiente per DB e secret di sessione.
