# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

Spendly is a personal expense-tracking web app built with Flask and server-rendered Jinja2 templates (no frontend framework, no build step, vanilla JS only). This is a course/tutorial-style scaffold: `app.py` and `database/db.py` contain routes and functions explicitly staged as numbered "Steps" for incremental implementation — several routes are intentionally unfinished placeholders (see Architecture below).

## Commands

```bash
# Activate the existing virtualenv (already created at ./venv)
./venv/Scripts/activate        # Windows (this repo is developed on Windows)

# Install dependencies
pip install -r requirements.txt

# Run the dev server (Flask debug mode, http://127.0.0.1:5001)
python app.py

# Run tests (pytest-flask is installed; no test files exist yet)
pytest
pytest path/to/test_file.py::test_name   # single test
```

There is no linter/formatter configured in this repo.

## Architecture

- **`app.py`** — single-file Flask app; all routes live here directly (no blueprints).
  - Fully implemented, template-rendering routes: `/` (landing), `/register`, `/login`, `/terms`, `/privacy`. These currently only render templates on `GET` — form submissions (e.g. `register.html`'s `POST /register`) are not yet handled by a matching route.
  - Placeholder routes that return a plain string instead of a template, each labeled with the course step that will implement it: `/logout`, `/profile`, `/expenses/add`, `/expenses/<id>/edit`, `/expenses/<id>/delete`. Don't "fix" these into full features unless asked — they're deliberately stubbed.
  - App runs on port **5001**, not Flask's default 5000.

- **`database/db.py`** — stub module for `get_db()`, `init_db()`, `seed_db()` (SQLite, row_factory + foreign keys). Not implemented yet; `expense_tracker.db` is gitignored and created at runtime once this is built out.

- **`templates/`** — Jinja2 templates all extend `base.html`, which defines the shared shell (navbar, `<footer>`, and the `title` / `head` / `content` / `scripts` blocks). Page-specific `<script>` tags go in a page's own `{% block scripts %}` rather than in `main.js`, so they only load on that page (see `landing.html` + `static/js/how-it-works-modal.js` for the pattern).

- **`static/css/style.css`** — the single stylesheet for the whole site. The entire design system (colors, fonts, spacing radii) is defined once as CSS custom properties on `:root` at the top of the file; every component reuses those variables (`--ink`, `--ink-muted`, `--paper`, `--paper-card`, `--accent`, `--accent-2`, `--border`, `--font-display`, `--font-body`, `--radius-*`, `--max-width`). There's no dark-mode/theme-switch media query — keep new styles on the same token set instead of hardcoding colors. Sections are organized under `/* ... */` banner comments (Hero, Mock window, Buttons, Features, CTA, Auth pages, Legal pages, Modal, Footer, Responsive) — add new rules under the matching banner rather than at the end of the file.

- **`static/js/main.js`** — currently an empty placeholder for site-wide JS added as features are built.
