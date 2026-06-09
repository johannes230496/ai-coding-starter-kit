# PROJ-2: User Authentication

## Status: Planned
**Created:** 2026-06-09
**Last Updated:** 2026-06-09

## Dependencies
- None (Basis für alle anderen Features)

## User Stories
- Als neuer Nutzer möchte ich mich mit E-Mail und Passwort registrieren können, damit ich mein persönliches Logbook anlegen kann.
- Als bestehender Nutzer möchte ich mich einloggen können, damit ich meine gespeicherten Daten wiederfinden kann.
- Als eingeloggter Nutzer möchte ich mich ausloggen können, damit mein Account auf gemeinsam genutzten Geräten sicher bleibt.
- Als Nutzer, der sein Passwort vergessen hat, möchte ich einen Reset-Link per E-Mail erhalten, damit ich wieder Zugang bekomme.
- Als Nutzer möchte ich meinen Account-Namen und meine E-Mail-Adresse in den Einstellungen sehen und bearbeiten können.

## Acceptance Criteria
- [ ] Registrierungsseite `/register`: Felder E-Mail, Passwort, Passwort-Bestätigung + Unternehmensname (optional)
- [ ] Login-Seite `/login`: Felder E-Mail + Passwort, Link "Passwort vergessen"
- [ ] Passwort-Reset-Flow: E-Mail eingeben → Link zugesendet → neues Passwort setzen
- [ ] Nach Login: Weiterleitung auf `/dashboard`
- [ ] Nicht-eingeloggte Nutzer, die eine geschützte Route aufrufen, werden zu `/login` weitergeleitet
- [ ] Eingeloggte Nutzer können sich über einen Button in der Navigation ausloggen
- [ ] Supabase Auth als Provider (kein OAuth im MVP)
- [ ] Passwort-Validierung: mind. 8 Zeichen
- [ ] E-Mail-Bestätigung via Supabase (Confirm-E-Mail wird gesendet)
- [ ] Profil-Seite `/settings/profile`: E-Mail und Anzeigename bearbeitbar

## Edge Cases
- Was wenn die E-Mail bereits registriert ist? → Fehlermeldung "Diese E-Mail ist bereits registriert"
- Was wenn der Reset-Link abgelaufen ist? → Hinweis mit erneutem Anfragelink
- Was wenn der Nutzer den Confirm-Link nicht angeklickt hat? → Hinweis im Dashboard + erneut senden
- Was wenn die Supabase-Session abläuft? → Automatisches Refresh oder Redirect zum Login
- Ungültige Token in der URL (Passwort-Reset) → Fehlermeldung + Redirect

## Technical Requirements
- Supabase Auth (Email/Password Provider)
- Row Level Security (RLS) auf allen Tabellen: Nutzer sehen nur eigene Daten
- Session wird via Supabase SSR Client gehandhabt (Next.js Middleware)
- Middleware schützt alle `/dashboard/*` und `/app/*` Routen

---

## Tech Design (Solution Architect)

### Architektur-Übersicht
Supabase Auth als fertige Auth-Lösung (E-Mail/Passwort, kein OAuth im MVP). Next.js Middleware schützt alle geschützten Routen serverseitig. Sessions werden via `@supabase/ssr` sowohl server- als auch clientseitig korrekt gehandhabt.

### Komponenten-Struktur
```
App Shell
+-- Middleware (route guard — runs on every request)
|   +-- Unprotected: /login, /register, /auth/*
|   +-- Protected: /dashboard/*, /app/*, /settings/*
|
+-- /login
|   +-- LoginForm (Email, Passwort, "Passwort vergessen"-Link)
|
+-- /register
|   +-- RegisterForm (Email, Unternehmensname optional, Passwort, Bestätigung)
|
+-- /auth/reset-password
|   +-- ResetPasswordForm (Email eingeben → Link senden)
|
+-- /auth/update-password
|   +-- UpdatePasswordForm (Neues Passwort nach E-Mail-Link)
|
+-- /settings/profile
    +-- ProfileForm (Anzeigename, E-Mail, Logout)
```

### Datenmodell
- **auth.users** (Supabase built-in): E-Mail, Passwort (verschlüsselt), Bestätigungsstatus, Sessions
- **profiles** Tabelle (1:1 mit auth.users): Anzeigename (optional), Unternehmensname (optional), created_at, updated_at
- Wird via Supabase Trigger automatisch bei Registrierung angelegt
- RLS: Nutzer sehen und bearbeiten nur ihre eigene Zeile

### Tech-Entscheidungen
| Entscheidung | Warum |
|---|---|
| Supabase Auth | Fertige Lösung: E-Mail/Passwort, Bestätigungs-E-Mail, Password-Reset — kein Custom-Code |
| Next.js Middleware | Schützt Routen serverseitig vor dem Rendering — sicher und performant |
| @supabase/ssr | Sessions korrekt für Next.js App Router (Server + Client) |
| React Hook Form + Zod | Bereits im Stack, saubere Validierung mit Fehlermeldungen |
| Sonner Toasts | Bereits installiert, für User-Feedback (Fehler, Erfolg) |
| Kein OAuth (MVP) | Reduziert Komplexität — Social Login als P1 möglich |

### Dependencies
- `@supabase/supabase-js` — Supabase Client
- `@supabase/ssr` — Server-Side Session Handling für Next.js

## QA Test Results
_To be added by /qa_

## Deployment
_To be added by /deploy_
