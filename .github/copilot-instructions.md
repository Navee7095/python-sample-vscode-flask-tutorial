## Quick context for AI coding agents

This repository is a minimal Flask tutorial sample. Keep changes small and localized to the `hello_app` module unless the task explicitly requires altering deployment files (Dockerfile, `uwsgi.ini`, `startup.py`). The project is intentionally simple for tutorial/demo purposes — prefer minimal, easily reviewable edits.

Key facts (where to look)
- Application package: `hello_app/`
  - App instance: `hello_app/__init__.py` defines `app = Flask(__name__)` (single Flask app object)
  - Routes & handlers: `hello_app/views.py` registers routes via decorators and uses relative imports
  - App entry for Flask CLI: `hello_app/webapp.py` imports `app` and `views` for side-effects (routes)
- Deployment shim: `startup.py` imports `app` from `hello_app.webapp` so Gunicorn on Azure can use `startup:app`
- Production servers/config: `uwsgi.ini` (uWSGI) and `Dockerfile` (containerization)
- Static data: `hello_app/static/data.json` is returned by the `/api/data` route (see `views.get_data`)
- Tests: `test_test1.py` (pytest)

Recommended agent behavior
- Preserve the module structure: many components rely on import side-effects. For example, `hello_app/webapp.py` depends on `from . import views` to register routes.
- When adding routes or templates: place route handlers in `hello_app/views.py` and add templates under `hello_app/templates/`.
- If you need to modify app configuration, change `hello_app/__init__.py` (it owns the Flask instance). Avoid creating a second, conflicting app object.

Local developer workflows (explicit commands)
- Run app locally (PowerShell):
  - cd into app folder: `cd hello_app`
  - set environment variable: `$env:FLASK_APP = "webapp"`
  - start dev server: `flask run`
  - (On cmd.exe) `set FLASK_APP=webapp` then `flask run`.
- Run tests: from repo root run `pytest` (the sample has `test_test1.py`).

Deployment notes
- Azure App Service (non-container): use the `startup.py` shim so you can point Gunicorn to `startup:app`. This avoids the "Attempted relative import in non-package" error.
- uWSGI: `uwsgi.ini` uses `module = hello_app.webapp` and `callable = app` — keep module/callable names in sync if moving files.
- Docker / container images: the `Dockerfile` in repo root is present for containerized deployments; prefer editing Dockerfile only when necessary.

Patterns and small examples
- Add a route (canonical pattern):
  - Put the handler in `hello_app/views.py` using `@app.route(...)`.
  - Use `render_template('your_template.html', ...)` for HTML responses; templates live under `hello_app/templates/`.
- Return static JSON via Flask: `return app.send_static_file('data.json')` reads `hello_app/static/data.json`.

What NOT to change without explicit reason
- Do not move the app object out of `hello_app/__init__.py` without adjusting `startup.py`, `uwsgi.ini`, and `Dockerfile` accordingly.
- Avoid changing the import-side-effect pattern (that is, `webapp.py` importing `views`) — many small tutorial changes assume this behavior.

If you need more context
- Inspect these files: `hello_app/__init__.py`, `hello_app/views.py`, `hello_app/webapp.py`, `startup.py`, `uwsgi.ini`, `Dockerfile`, `requirements.txt`, `test_test1.py`.

After making changes
- Run `pytest` and run the app locally (PowerShell steps above). If modifying deployment wiring (startup, uWSGI, Dockerfile), include a short note in your change explaining the reason and how to replicate locally.

Questions / feedback
- If any of these conventions need clarifying or you want me to add examples for adding blueprints, logging, or tests, tell me which area to expand.
