# Glossario di dominio

Applicazione web per gestire liste della spesa in uno spazio condiviso tra Account.

## Linguaggio

**Tenant** (`Tenant`):
Uno spazio condiviso che isola dati e Liste dagli altri spazi. Può corrispondere a una famiglia o a un altro gruppo, ma nel dominio è solo lo spazio.
_Evitare_: famiglia (come sinonimo del concetto), organizzazione, workspace, account

**Account** (`Account`):
Una persona che può accedere all'applicazione ed è membro di un Tenant.
_Evitare_: utente, user (nel linguaggio di dominio)

**Lista della spesa** (`ShoppingList`):
Una lista nominata di cose da acquistare, appartenente a un Tenant e visibile a tutti gli Account di quel Tenant.
_Evitare_: carrello, ordine

**Codice invito** (`InviteCode`):
Un codice che permette a un Account in registrazione di entrare in un Tenant già esistente.
_Evitare_: link di invito, token (nel linguaggio di dominio)

**Voce** (`ShoppingItem`):
Una riga in una Lista della spesa, con nome, quantità opzionale in testo libero e stato presa/non presa.
_Evitare_: prodotto, item, articolo
