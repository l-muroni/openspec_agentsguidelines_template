# Spec Delta

## Purpose

Copre registrazione e autenticazione degli Account, l'appartenenza a un solo Tenant e la gestione del Codice invito.

## ADDED Requirements

### Requirement: Account registration creates a Tenant
Un Account in registrazione senza Codice invito MUST essere creato con email e password e MUST appartenere a un Tenant nuovo. L'email MUST essere unica nel sistema.

#### Scenario: Registrazione senza codice crea Tenant e Account
- **WHEN** un visitatore si registra con email `anna@example.com` e password `CorrectHorse1` senza Codice invito
- **THEN** esiste un Account con quella email, appartiene a un Tenant nuovo, e non esiste un altro Account con la stessa email

#### Scenario: Registrazione con email già usata è rifiutata
- **WHEN** esiste già un Account con email `anna@example.com`
- **AND** un visitatore tenta di registrarsi con email `anna@example.com` e password `CorrectHorse1` senza Codice invito
- **THEN** la registrazione fallisce e non viene creato un secondo Account con quella email

### Requirement: Account registration joins a Tenant with InviteCode
Un Account in registrazione con Codice invito valido MUST appartenere al Tenant associato a quel codice. Un Codice invito inesistente o non più valido MUST far fallire la registrazione.

#### Scenario: Registrazione con Codice invito valido entra nel Tenant
- **WHEN** il Tenant `casa-verde` ha Codice invito `JOIN-CASA-01`
- **AND** un visitatore si registra con email `bruno@example.com`, password `CorrectHorse1` e Codice invito `JOIN-CASA-01`
- **THEN** l'Account `bruno@example.com` appartiene al Tenant `casa-verde`

#### Scenario: Registrazione con Codice invito sconosciuto è rifiutata
- **WHEN** un visitatore tenta di registrarsi con email `carla@example.com`, password `CorrectHorse1` e Codice invito `NO-SUCH-CODE`
- **THEN** la registrazione fallisce e non esiste un Account con email `carla@example.com`

### Requirement: Account belongs to exactly one Tenant
Ogni Account MUST appartenere a esattamente un Tenant. Il sistema MUST NOT consentire a un Account di appartenere a due Tenant contemporaneamente.

#### Scenario: Account appartiene a un solo Tenant dopo la registrazione
- **WHEN** l'Account `anna@example.com` è stato creato creando un Tenant
- **THEN** quell'Account appartiene a esattamente un Tenant

### Requirement: Session authentication with email and password
Un Account MUST autenticarsi con email e password. Una sessione riuscita MUST essere rappresentata da un cookie httpOnly. Credenziali errate MUST essere rifiutate senza creare una sessione.

#### Scenario: Login con credenziali corrette apre una sessione
- **WHEN** esiste un Account con email `anna@example.com` e password `CorrectHorse1`
- **AND** un visitatore invia quelle credenziali
- **THEN** la risposta include un cookie di sessione httpOnly e le richieste successive con quel cookie sono autenticate come `anna@example.com`

#### Scenario: Login con password errata è rifiutato
- **WHEN** esiste un Account con email `anna@example.com` e password `CorrectHorse1`
- **AND** un visitatore invia email `anna@example.com` e password `WrongPassword1`
- **THEN** l'autenticazione fallisce e non viene emesso un cookie di sessione

### Requirement: Members can view and regenerate InviteCode
Qualsiasi Account autenticato del Tenant MUST poter leggere il Codice invito corrente del proprio Tenant e MUST poter richiederne uno nuovo. Il codice precedente MUST smettere di funzionare dopo la rigenerazione. Il sistema MUST NOT distinguere ruoli tra i membri.

#### Scenario: Membro legge il Codice invito del proprio Tenant
- **WHEN** l'Account `anna@example.com` appartiene al Tenant il cui Codice invito è `JOIN-CASA-01`
- **AND** `anna@example.com` è autenticato e chiede il Codice invito
- **THEN** riceve `JOIN-CASA-01`

#### Scenario: Membro rigenera il Codice invito e l'altro smette di valere
- **WHEN** il Tenant di `anna@example.com` ha Codice invito `JOIN-CASA-01`
- **AND** `anna@example.com` autenticato rigenera il Codice invito
- **THEN** il Tenant ha un Codice invito diverso da `JOIN-CASA-01`
- **AND** una registrazione con `JOIN-CASA-01` fallisce

#### Scenario: Account di un altro Tenant non legge il Codice invito
- **WHEN** `anna@example.com` appartiene al Tenant `casa-verde` con Codice invito `JOIN-CASA-01`
- **AND** `dario@example.com` appartiene a un Tenant diverso ed è autenticato
- **AND** `dario@example.com` chiede il Codice invito di `casa-verde`
- **THEN** la richiesta è rifiutata e non riceve `JOIN-CASA-01`
