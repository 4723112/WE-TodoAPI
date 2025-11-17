<!-- .github/copilot-instructions.md - guidance for AI coding agents -->
# TODO API — Copilot instructions

This repository is a small FastAPI + Tortoise ORM TODO service. Keep instructions short and actionable — focus on the concrete patterns used here.

Key facts (read before editing code)
- App entry: `main.py` creates a FastAPI app and registers Tortoise via `register_tortoise(...)`.
- DB model: `models.py` (Tortoise `Model` subclass `TodoItem`). The FastAPI code imports model objects from `model`/`models` (see `main.py` import).
- Pydantic usage: `pydantic_model_creator` is used to generate request/response models from Tortoise models in `main.py` (see `TodoItem_Pydantic`, `TodoItemCreate_Pydantic`, `TodoItemUpdate_Pydantic`).
- DB backend: SQLite in-file (`sqlite://db.sqlite3`) configured in `register_tortoise(...)`.

Project-specific patterns and gotchas
- The code mixes an in-memory `todos` list and Tortoise-backed endpoints; the authoritative data layer is Tortoise ORM (endpoints use `await TodoItem.*`). Ignore the in-memory `todos` list when adding persistence-related changes.
- `pydantic_model_creator(..., exclude=(...))` is used to shape incoming/outgoing schemas. When changing model fields update these creators accordingly.
- `models.py` uses nullable `description` (CharField null=True) — treat missing descriptions as None, not empty string.
- Route handlers are async and use Tortoise ORM's async API (`await TodoItem.all()`, `.get()`, `.create()`, `.save()`). Keep endpoints async and avoid blocking calls.

Developer workflows (how to run and test locally)
- Install: python -m pip install -r requirements.txt
- Run dev server: `uvicorn main:app --reload` (project root). The app will auto-create SQLite schema because `register_tortoise(..., generate_schemas=True)`.
- DB file: `db.sqlite3` is created in the project root. To reset DB, stop server and delete `db.sqlite3`.

Code conventions and examples
- Keep endpoint response models tied to Pydantic creators. Example GET all TODOs uses `response_model=List[TodoItem_Pydantic]` and returns `await TodoItem.all()`.
- When creating a new todo use the `TodoItem.create(...)` Tortoise method and return the created model (the file already does this in POST `/todos`).
- For updates, fetch the model, set fields only when request values are not None, then `await todo.save()` (see PUT `/todos/{todo_id}`).

Integration points & dependencies
- Tortoise ORM (configured via `register_tortoise` in `main.py`). If you change DB modules, update `modules={"models":["model"]}`.
- pydantic v2 is used (see `requirements.txt`); generated models come from `tortoise.contrib.pydantic`.

When modifying or adding endpoints
- Mirror the existing pattern: async handler -> fetch/operate with Tortoise -> return model or raise HTTPException with appropriate status code.
- Add or update `pydantic_model_creator` lines at top of `main.py` when model fields change. Use `exclude` to prevent exposing `id` or other fields in create/update schemas.

Search targets for quick navigation
- `main.py` — app and routes
- `models.py` — Tortoise model(s)
- `requirements.txt` — pinned deps

If you are uncertain
- Prefer minimal, local changes. Run the dev server and exercise endpoints with curl or an HTTP client.
- Ask for clarification if a requested change touches DB schema or external integrations.

Notes
- This file is intentionally short (~30 lines). Keep future edits focused and concrete. Link to this file from PR descriptions when the change affects app structure or DB models.
