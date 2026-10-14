# Design

## Context

Vedi `proposal.md` (Why) e i delta in `specs/tenant-membership/spec.md` e `specs/shopping-lists/spec.md`.

Oggi il repo è un template: root npm senza workspaces, `PATTERNS.md` vuoto, nessun `apps/`, nessun Compose, nessuna spec principale. Vincoli che pesano sull'implementazione:

- file protetti: `package.json` (root e sub-package), `package-lock.json`, `.agent-guidelines/*`, `docker-compose.yml`, `.env*` (salvo `.env.example`);
- nessuna nuova dipendenza senza approvazione (elenco in Impact della proposal);
- test derivati dagli scenari, un test per scenario, nomi identici ai titoli.

## Goals / Non-Goals

**Goals:**

- Struttura monorepo e confini API/SPA tali che gli scenari delle due capability siano implementabili e testabili.
- Modello dati e isolamento Tenant applicato in un solo punto (query sempre filtrate sul Tenant della sessione).
- Sessione osservabile come cookie httpOnly, senza store esterno.
- Elenco esplicito di ciò che deve fare un umano (file protetti e dipendenze) rispetto a ciò che può fare l'agente nel worktree.

**Non-Goals:**

- Non ripetere i non-goal di prodotto della proposal.
- Non progettare ruoli, OAuth, realtime o un client HTTP tipizzato condiviso (`packages/`).
- Non introdurre un ADR: le scelte restano locali a questo change.

## Decisions

### 1. Layout monorepo npm workspaces, senza `packages/`

- `apps/api`: Fastify + TypeORM, porta `3000`.
- `apps/web`: Vite + React, porta `5173`, proxy `/api` verso l'API in sviluppo.
- Root: workspaces `apps/*`, script `lint` / `test` / `build` che delegano alle app.

**Alternative scartate:** pnpm/turbo (tooling extra, il template è già npm); `packages/shared` al primo giro (YAGNI finché web e api non condividono tipi); Next.js fullstack (scartato in analisi).

**File protetti:** creare/aggiornare `package.json` di root e delle app, e il lockfile, è lavoro umano nella PR. L'agente scrive il codice sotto `apps/*/src` (e test) dopo che i manifest esistono.

### 2. Persistenza TypeORM su PostgreSQL, `synchronize: false`

Entità (identificatori in inglese come da glossario):

| Entità | Campi rilevanti |
| --- | --- |
| `Tenant` | `id` UUID, `inviteCode` stringa unica |
| `Account` | `id` UUID, `email` unica, `passwordHash`, `tenantId` FK non nullable |
| `ShoppingList` | `id` UUID, `tenantId` FK, `name` |
| `ShoppingItem` | `id` UUID, `shoppingListId` FK, `name`, `quantity` nullable (`text`), `taken` boolean default false |

Relazioni: `Account` N:1 `Tenant`; `ShoppingList` N:1 `Tenant`; `ShoppingItem` N:1 `ShoppingList` con `onDelete: CASCADE` (elimina Lista → elimina Voci).

Migrazioni TypeORM versionate in `apps/api/src/migrations`. In test: database PostgreSQL dedicato (`DATABASE_URL` di test), schema applicato con le stesse migrazioni; mai `synchronize: true`.

Il Tenant creato in registrazione non ha un nome di dominio obbligatorio: si persiste solo `id` + `inviteCode`. Eventuale nome display è fuori spec.

**Alternative scartate:** Drizzle (scartato in analisi); Prisma (client pesante, meno allineato alla scelta TypeORM); SQLite nei test (diverge da Postgres su tipi e vincoli); `synchronize: true` (schema implicito, pericoloso in ogni ambiente).

**File protetti:** `docker-compose.yml` non può essere scritto dall'agente. In sviluppo l'API legge `DATABASE_URL`. Un umano può aggiungere Compose in un follow-up; `.env.example` documenta `DATABASE_URL` e `SESSION_SECRET`.

### 3. Sessione: cookie cifrato `@fastify/secure-session`, password `argon2`

- Login riuscito: cookie httpOnly, `Secure` in produzione, `SameSite=Lax`, payload minimo `{ accountId }`.
- Nessuna tabella sessioni e nessun Redis.
- Logout: cookie invalidato (scadenza / overwrite vuoto).
- Password: hash `argon2id` in scrittura, verify in login. Email normalizzata in lowercase prima di unique check e login.

**Alternative scartate:** `@fastify/session` + store TypeORM (stato server-side da pulire, più pezzi per v1); JWT in `localStorage` (non è cookie httpOnly); `bcrypt` (argon2id è il default moderno; bcrypt resta accettabile se argon2 è bloccato in CI per bind nativi — in quel caso `bcryptjs`).

### 4. Codice invito opaco, un valore corrente per Tenant

Un solo campo `Tenant.inviteCode`. Lettura e rigenerazione: qualsiasi Account autenticato, sempre sul **proprio** Tenant (deriva dalla sessione, mai da un id Tenant nel path).

Formato: 10 caratteri Crockford-base32 da `crypto.randomBytes` (digitabile, non indovinabile). Gli esempi di spec (`JOIN-CASA-01`) sono dati di test, non il formato di produzione. In test si può fissare il valore a seed.

Rigenerazione: `UPDATE` atomico del campo; il valore precedente non è più accettato in registrazione (lookup per uguaglianza).

**Alternative scartate:** UUID grezzo (scomodo da dettare); tabella di inviti con scadenza/multi-codice (fuori spec); codice nel path `/tenants/:id/invite` (permetterebbe di chiedere il codice di un altro Tenant, vietato dalla spec).

### 5. API REST sotto `/api`, Tenant solo dalla sessione

Prefisso `/api`. JSON. Errori `{ "error": "<codice>" }` con HTTP:

| Codice | Quando |
| --- | --- |
| 400 | body invalido (email/password/nome Lista/Voce) |
| 401 | assente/invalida sessione, login fallito |
| 403 | risorsa di un altro Tenant (id Lista/Voce noto ma non del Tenant) |
| 404 | Lista/Voce inesistente **nel** Tenant, oppure Codice invito sconosciuto in registrazione |
| 409 | email già registrata |

Risorse:

- `POST /api/auth/register` `{ email, password, inviteCode? }` → Account + cookie sessione
- `POST /api/auth/login` `{ email, password }` → cookie
- `POST /api/auth/logout`
- `GET /api/me` → `{ id, email, tenantId }`
- `GET /api/tenant/invite-code` → `{ inviteCode }`
- `POST /api/tenant/invite-code/regenerate` → `{ inviteCode }`
- `GET\|POST /api/lists`
- `PATCH\|DELETE /api/lists/:listId`
- `GET\|POST /api/lists/:listId/items`
- `PATCH\|DELETE /api/lists/:listId/items/:itemId`

Ogni handler autenticato carica l'`Account` della sessione e usa `tenantId` in **tutte** le query. Un id di Lista/Voce di un altro Tenant → 403 (stesso esito visibile della spec: modifica rifiutata). Elenco Liste: `WHERE tenantId = session.tenantId` (isolamento in lettura).

CORS: in sviluppo origine `http://localhost:5173` con `credentials: true`. In produzione stessa policy sull'origine della SPA. Cookie `Path=/`.

**Alternative scartate:** tRPC/GraphQL (fuori proposal); mettere `tenantId` nel path (duplica la sessione e apre IDOR); 404 per risorse di altri Tenant (nasconde meglio, ma 403 rende esplicito il rifiuto richiesto dagli scenari di isolamento).

### 6. SPA React: fetch con `credentials: 'include'`, CSS Modules

Pagine: registrazione (email, password, Codice invito opzionale), login, elenco Liste, dettaglio Lista (Voci: nome, quantità, toggle presa, elimina), pannello Codice invito (mostra + rigenera).

Stato: React `useState`/`useEffect` + fetch; niente Redux/React Query in v1.

CSS Modules co-locati ai componenti. Vite proxy `/api` → `http://localhost:3000` così in dev il cookie è same-origin sul path `/api`.

**Alternative scartate:** Tailwind/shadcn (fuori proposal); React Query (overhead); cookie letto da JS (contraddice httpOnly).

### 7. Test Vitest, un test per scenario

- `apps/api`: Vitest + chiamata HTTP in-process a Fastify (`inject` o `app.listen` su porta efimera) contro Postgres di test. Ogni test si chiama **esattamente** come il titolo dello scenario.
- `apps/web`: test di integrazione API-first per gli scenari (il comportamento è l'API). Test di componente solo se un scenario non è osservabile senza DOM — in questo change tutti gli scenari sono osservabili via HTTP, quindi i test vivono in `apps/api`.

Seed degli esempi di spec (`anna@example.com`, `CorrectHorse1`, `JOIN-CASA-01`, `casa-verde` come etichetta di test) in factory, non come fixture globali condivise tra file.

**Alternative scartate:** Jest (due runner); Playwright obbligatorio in v1 (nessuno scenario è “l'utente clicca X in UI” come unico osservabile); mock in-memory del repository al posto di Postgres (non verifica vincoli unique/FK).

### 8. PATTERNS.md

Dopo l'approvazione, un umano (file protetto) scrive in `.agent-guidelines/PATTERNS.md` lo stack deciso e la mappa `apps/api` / `apps/web`, poi `npm run sync-agents`. L'agente non modifica `.agent-guidelines/`.

## Risks / Trade-offs

- [Dipendenze non ancora approvate] → la PR di spec elenca i pacchetti; l'implementazione parte solo dopo ok esplicito e dopo che i `package.json` esistono.
- [File protetti bloccano workspaces, Compose, PATTERNS] → task umani separati nella stessa PR di implementazione, o PR `chore/` precedente che sblocca i manifest.
- [Cookie cifrato non revocabile server-side prima della scadenza] → accettabile in v1; logout cancella il cookie sul client; se serve kill-session globale, si aggiunge una tabella in un change successivo.
- [argon2 bind nativo in CI] → se la build fallisce, sostituire con `bcryptjs` senza cambiare le spec.
- [IDOR se una query dimentica `tenantId`] → wrapper repository / helper `forTenant(tenantId)` e test di isolamento già previsti dalle spec.
- [Postgres assente in locale] → falliscono i test; documentare `DATABASE_URL` in `.env.example`; Compose è follow-up umano.

## Migration Plan

Repo verde, nessun dato di produzione.

1. Umano: workspaces, dipendenze, `.env.example`, eventuale Compose.
2. Codice API/web e migrazioni TypeORM.
3. `npm test` / `npm run build` dalla root.
4. Deploy: applicare migrazioni (`typeorm migration:run`), avviare api e web statico (o dev).

Rollback: fermare i processi; `migration:revert` o drop dello schema di sviluppo. Nessuna migrazione di dati esistenti.

## Open Questions

- Approvazione esplicita dei pacchetti: `fastify`, `@fastify/cookie`, `@fastify/secure-session`, `@fastify/cors`, `typeorm`, `pg`, `reflect-metadata`, `argon2`, `react`, `react-dom`, `vite`, `vitest` (e plugin TS/React Vite). Senza questo ok l'implementazione non installa nulla.
- Chi scrive i file protetti (`package.json`, lockfile, `PATTERNS.md`, eventuale `docker-compose.yml`): sviluppatore nella PR di implementazione, non l'agente.
