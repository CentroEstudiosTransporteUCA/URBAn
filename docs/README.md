# URBAn Documentation

URBAn is the civil-engineer-facing UI for TraTrac (the trajectory-extraction engine) and,
eventually, FloCo (the flow counter). It has no code yet — this `docs/` folder holds design
docs written before implementation started, so the first milestone begins from a spec instead
of a blank slate.

- [`config_editor_spec.md`](config_editor_spec.md) — spec for the run-config editor screen:
  a Tauri client that generates a `run.toml` for TraTrac and launches `tratrac --config`.
  Written from TraTrac's `application/config.py` schema (the `RunConfig` resolver) — that
  schema is the source of truth; this doc is a UI spec on top of it, not a duplicate of it.
  References TraTrac's `src/tratrac/CHECK_COMMAND.md` for the `tratrac --check --json`
  contract this editor validates against.
- [`PRODUCTION_MVP.md`](PRODUCTION_MVP.md) — full production-MVP checklist, ordered by value to
  civil engineers, with the realistic congress-presentation cutoff marked.
