# Spec: Registration

## Overview

Implement account creation so that visitors can turn the existing `register.html` form into a real signup flow. This is the second step on the Spendly roadmap, immediately after the database layer (Step 1). It adds the `POST /register` handler that validates input, hashes the password, and inserts a new row into `users`. It does **not** implement login/session handling — that is Step 3 (`logout` is already stubbed "coming in Step 3" in `app.py`), so after a successful registration the user is redirected to `/login` rather than being logged in automatically.

## Depends on

- Step 1 — Database setup (`database/db.py`: `get_db()`, `init_db()`, `users` table). Must already be complete and working.

## Routes

- `POST /register` — validate and create a new user account — public
  - On validation/uniqueness failure: re-render `register.html` with an `error` message and HTTP 400, preserving the submitted name/email so the user doesn't retype them.
  - On success: redirect (302) to `/login`.
- `GET /register` — already implemented, unchanged.

## Database changes

No database changes. The `users` table (`database/db.py`) already has every column needed: `name`, `email` (unique), `password_hash`, `created_at`. `POST /register` only performs `INSERT`s and `SELECT`s against the existing schema.

## Templates

- **Create:** none.
- **Modify:** `templates/register.html` — when re-rendered after a failed submission, prefill `name` and `email` inputs with the previously submitted values (`value="{{ name or '' }}"`, etc.) so the user doesn't lose their input. The existing `{% if error %}` block already renders server-supplied error messages, so no markup changes are needed there.

## Files to change

- `app.py` — replace the current `GET`-only `/register` route with one that accepts `GET` and `POST`, and implements the logic described above.
- `templates/register.html` — prefill `name`/`email` values on re-render after an error.

## Files to create

None.

## New dependencies

No new dependencies.

## Rules for implementation

- No SQLAlchemy or ORMs.
- Parameterised queries only — no string formatting in SQL.
- Passwords hashed with `werkzeug.security.generate_password_hash` before insert; never store or log a plaintext password.
- Validate server-side (do not rely solely on the HTML `required`/`type=email` attributes):
  - `name`, `email`, and `password` are all non-empty after stripping whitespace.
  - `password` is at least 8 characters (matches the form's placeholder text "Min. 8 characters").
  - `email` is not already present in `users` — check with a `SELECT` before inserting, and also handle the `sqlite3.IntegrityError` from the `UNIQUE` constraint as a fallback in case of a race, converting it into the same friendly `error` message rather than a 500.
- Use CSS variables — never hardcode hex values (only relevant if any new styling is touched).
- All templates extend `base.html` (already true for `register.html`; do not change this).
- Close every `get_db()` connection (use `try`/`finally` or a `with` block) so connections aren't leaked on error paths.

## Definition of done

- [ ] Submitting the registration form with a new, valid name/email/password (≥8 chars) creates a row in `users` with a bcrypt-style werkzeug password hash (not plaintext) and redirects to `/login`.
- [ ] Submitting with an email that already exists in `users` re-renders `register.html` with an error message, returns HTTP 400, and does not create a duplicate row.
- [ ] Submitting with a missing name, missing email, missing password, or a password under 8 characters re-renders `register.html` with an error message and does not touch the database.
- [ ] After a failed submission, the previously entered name and email are still shown in the form fields.
- [ ] `GET /register` still renders the empty form exactly as before.
- [ ] The app starts and runs with no errors (`python app.py`, port 5001).
