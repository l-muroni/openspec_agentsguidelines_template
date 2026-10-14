# Spec Delta

## Purpose

Copre creazione e gestione delle Liste della spesa e delle Voci visibili a tutti gli Account dello stesso Tenant.

## ADDED Requirements

### Requirement: Tenant owns multiple ShoppingLists
Un Account autenticato MUST poter creare una Lista della spesa nel proprio Tenant indicando un nome. Il Tenant MUST poter contenere più Liste. Ogni Lista MUST appartenere a quel Tenant e MUST essere visibile a tutti gli Account dello stesso Tenant.

#### Scenario: Membro crea una Lista della spesa nel Tenant
- **WHEN** l'Account `anna@example.com` è autenticato e appartiene al Tenant `casa-verde`
- **AND** crea una Lista della spesa con nome `Spesa settimanale`
- **THEN** esiste una Lista della spesa `Spesa settimanale` appartenente al Tenant `casa-verde`

#### Scenario: Tenant può avere più Liste
- **WHEN** il Tenant `casa-verde` ha già una Lista della spesa `Spesa settimanale`
- **AND** `anna@example.com` autenticato crea una Lista della spesa con nome `Festa di sabato`
- **THEN** il Tenant `casa-verde` ha entrambe le Liste `Spesa settimanale` e `Festa di sabato`

#### Scenario: Membro dello stesso Tenant vede le Liste
- **WHEN** `anna@example.com` ha creato la Lista `Spesa settimanale` nel Tenant `casa-verde`
- **AND** `bruno@example.com` appartiene a `casa-verde` ed è autenticato
- **AND** `bruno@example.com` elenca le Liste del proprio Tenant
- **THEN** l'elenco include `Spesa settimanale`

### Requirement: ShoppingLists are isolated between Tenants
Il sistema MUST NOT mostrare né modificare Liste della spesa di un Tenant ad Account di un altro Tenant.

#### Scenario: Account di un altro Tenant non vede le Liste
- **WHEN** il Tenant `casa-verde` ha la Lista `Spesa settimanale`
- **AND** `dario@example.com` appartiene a un Tenant diverso ed è autenticato
- **AND** `dario@example.com` elenca le Liste del proprio Tenant
- **THEN** l'elenco non include `Spesa settimanale`

#### Scenario: Account di un altro Tenant non modifica una Lista
- **WHEN** il Tenant `casa-verde` ha la Lista `Spesa settimanale`
- **AND** `dario@example.com` appartiene a un Tenant diverso ed è autenticato
- **AND** `dario@example.com` tenta di rinominare quella Lista in `Lista rubata`
- **THEN** la modifica è rifiutata e la Lista resta `Spesa settimanale`

### Requirement: Members can rename and delete ShoppingLists
Qualsiasi Account autenticato del Tenant MUST poter rinominare o eliminare una Lista della spesa del proprio Tenant. L'eliminazione MUST rimuovere anche tutte le Voci di quella Lista.

#### Scenario: Membro rinomina una Lista del Tenant
- **WHEN** il Tenant `casa-verde` ha la Lista `Spesa settimanale`
- **AND** `bruno@example.com` autenticato, membro di `casa-verde`, la rinomina in `Spesa del lunedì`
- **THEN** quella Lista ha nome `Spesa del lunedì` e non esiste più una Lista `Spesa settimanale` in `casa-verde`

#### Scenario: Membro elimina una Lista e le sue Voci
- **WHEN** la Lista `Spesa settimanale` del Tenant `casa-verde` contiene una Voce con nome `Latte`
- **AND** `bruno@example.com` autenticato elimina quella Lista
- **THEN** `casa-verde` non ha più la Lista `Spesa settimanale`
- **AND** non esiste una Voce `Latte` associata a quella Lista

### Requirement: ShoppingItem has name quantity and taken state
Una Voce MUST appartenere a una Lista della spesa e MUST avere un nome, una quantità opzionale in testo libero e uno stato presa/non presa. Il sistema MUST NOT richiedere categorie, prezzi o unità tipizzate.

#### Scenario: Membro aggiunge una Voce con quantità
- **WHEN** `anna@example.com` è autenticato e il Tenant `casa-verde` ha la Lista `Spesa settimanale`
- **AND** aggiunge una Voce con nome `Latte` e quantità `2 l`
- **THEN** quella Lista contiene una Voce `Latte` con quantità `2 l` e stato non presa

#### Scenario: Membro aggiunge una Voce senza quantità
- **WHEN** `anna@example.com` è autenticato e il Tenant `casa-verde` ha la Lista `Spesa settimanale`
- **AND** aggiunge una Voce con nome `Pane` senza quantità
- **THEN** quella Lista contiene una Voce `Pane` senza quantità e con stato non presa

### Requirement: Members can update and remove ShoppingItems
Qualsiasi Account autenticato del Tenant MUST poter segnare una Voce come presa o non presa, aggiornarne nome o quantità, e rimuoverla dalla Lista.

#### Scenario: Membro segna una Voce come presa
- **WHEN** la Lista `Spesa settimanale` di `casa-verde` ha una Voce `Latte` non presa
- **AND** `bruno@example.com` autenticato, membro di `casa-verde`, la segna come presa
- **THEN** quella Voce ha stato presa

#### Scenario: Membro rimuove una Voce dalla Lista
- **WHEN** la Lista `Spesa settimanale` di `casa-verde` ha una Voce `Pane`
- **AND** `bruno@example.com` autenticato rimuove quella Voce
- **THEN** la Lista non contiene più una Voce `Pane`

#### Scenario: Account di un altro Tenant non modifica una Voce
- **WHEN** la Lista `Spesa settimanale` di `casa-verde` ha una Voce `Latte` non presa
- **AND** `dario@example.com` appartiene a un Tenant diverso ed è autenticato
- **AND** `dario@example.com` tenta di segnare quella Voce come presa
- **THEN** la modifica è rifiutata e la Voce resta non presa
