# Tasks

## 1. Test derivati dagli scenari

Sessione separata: solo test, devono fallire. Ogni `it`/`test` usa il titolo esatto dello scenario. File in `apps/api` (HTTP in-process, Postgres di test). Non implementare produzione.

- [ ] 1.1 Scrivere i test di `tenant-membership` con questi nomi e verificare che `vitest` li elenchi e falliscano: «Registrazione senza codice crea Tenant e Account», «Registrazione con email già usata è rifiutata», «Registrazione con Codice invito valido entra nel Tenant», «Registrazione con Codice invito sconosciuto è rifiutata», «Account appartiene a un solo Tenant dopo la registrazione», «Login con credenziali corrette apre una sessione», «Login con password errata è rifiutato», «Membro legge il Codice invito del proprio Tenant», «Membro rigenera il Codice invito e l'altro smette di valere», «Account di un altro Tenant non legge il Codice invito»
- [ ] 1.2 Scrivere i test di `shopping-lists` con questi nomi e verificare che `vitest` li elenchi e falliscano: «Membro crea una Lista della spesa nel Tenant», «Tenant può avere più Liste», «Membro dello stesso Tenant vede le Liste», «Account di un altro Tenant non vede le Liste», «Account di un altro Tenant non modifica una Lista», «Membro rinomina una Lista del Tenant», «Membro elimina una Lista e le sue Voci», «Membro aggiunge una Voce con quantità», «Membro aggiunge una Voce senza quantità», «Membro segna una Voce come presa», «Membro rimuove una Voce dalla Lista», «Account di un altro Tenant non modifica una Voce»



## 2. Manifest e file protetti (umano)

Dopo approvazione dipendenze. L'agente non modifica `package.json`, lockfile, `.agent-guidelines/`, `docker-compose.yml`.

- [x] 2.1 Umano: workspaces `apps/*` nei `package.json` (root, `apps/api`, `apps/web`) con le dipendenze approvate (`fastify`, `@fastify/cookie`, `@fastify/secure-session`, `@fastify/cors`, `typeorm`, `pg`, `reflect-metadata`, `argon2`, `react`, `react-dom`, `vite`, `vitest` e plugin TS/React) e script root `lint` / `test` / `build`; verificare con `npm ci` dalla root
- [x] 2.2 Umano: `.env.example` con `DATABASE_URL` e `SESSION_SECRET`; verificare che il file esista e non committi segreti
- [x] 2.3 Umano: `.agent-guidelines/PATTERNS.md` con stack e mappa `apps/api` / `apps/web`, poi `npm run sync-agents`; verificare che `npm run sync-agents -- --check` passi



## 3. API: schema e app Fastify

- [ ] 3.1 Entità TypeORM `Tenant`, `Account`, `ShoppingList`, `ShoppingItem` e migrazione iniziale con `synchronize: false`; verificare che `migration:run` su `DATABASE_URL` di test crei tabelle e FK (incluso cascade Voci)
- [ ] 3.2 App Fastify con prefisso `/api`, CORS credenziali, `@fastify/secure-session`, helper `forTenant`; verificare con `inject` su `GET /api/me` senza cookie → 401



## 4. tenant-membership

- [ ] 4.1 `POST /api/auth/register` (crea Tenant o join con Codice invito, email unica lowercase, hash argon2id, cookie sessione); verificare verdi i test «Registrazione senza codice crea Tenant e Account», «Registrazione con email già usata è rifiutata», «Registrazione con Codice invito valido entra nel Tenant», «Registrazione con Codice invito sconosciuto è rifiutata», «Account appartiene a un solo Tenant dopo la registrazione»
- [ ] 4.2 `POST /api/auth/login`, `POST /api/auth/logout`, `GET /api/me`; verificare verdi «Login con credenziali corrette apre una sessione» e «Login con password errata è rifiutato» (cookie httpOnly)
- [ ] 4.3 `GET /api/tenant/invite-code` e `POST /api/tenant/invite-code/regenerate` sul Tenant della sessione; verificare verdi «Membro legge il Codice invito del proprio Tenant», «Membro rigenera il Codice invito e l'altro smette di valere», «Account di un altro Tenant non legge il Codice invito»



## 5. shopping-lists

- [ ] 5.1 CRUD Liste `GET|POST /api/lists`, `PATCH|DELETE /api/lists/:listId` filtrate per `tenantId` sessione (403 cross-tenant); verificare verdi «Membro crea una Lista della spesa nel Tenant», «Tenant può avere più Liste», «Membro dello stesso Tenant vede le Liste», «Account di un altro Tenant non vede le Liste», «Account di un altro Tenant non modifica una Lista», «Membro rinomina una Lista del Tenant», «Membro elimina una Lista e le sue Voci»
- [ ] 5.2 CRUD Voci `GET|POST /api/lists/:listId/items` e `PATCH|DELETE .../:itemId` (nome, quantità testo opzionale, `taken`); verificare verdi «Membro aggiunge una Voce con quantità», «Membro aggiunge una Voce senza quantità», «Membro segna una Voce come presa», «Membro rimuove una Voce dalla Lista», «Account di un altro Tenant non modifica una Voce»



## 6. SPA

- [ ] 6.1 Pagine registrazione, login, proxy Vite `/api` → `3000`, fetch `credentials: 'include'`; verificare a mano o con curl verso il proxy che login imposti il cookie e `GET /api/me` torni l'Account
- [ ] 6.2 Elenco/dettaglio Liste, Voci (toggle presa, quantità, elimina) e pannello Codice invito con CSS Modules; verificare nel browser i flussi principali allineati alle spec (stesso Tenant vs altro Tenant)



## 7. Integrazione

- [ ] 7.1 Dalla root `npm test` e `npm run build` verdi; tutti i 22 scenari in 1.1–1.2 passano
- [ ] 7.2 Summary in `docs/summaries/stack-tecnologico.md` con sezioni Cosa è stato modificato / non modificato / domande aperte



## Workflow follow-up

- PR `spec/stack-tecnologico` (se non già mergiata) poi worktree `ai/stack-tecnologico` per i task 1.x e 3.x–7.x.
- I task 2.x restano dello sviluppatore (file protetti).
- Prima del merge della PR di implementazione: `npx openspec archive stack-tecnologico --yes`.

