# Production MVP Checklist

Everything URBAn needs to be a finished production UI, ordered by value delivered to the civil
engineers who'll actually drive it — not by what fits in any one deadline. The line below marks
where we realistically expect to be for TraTrac's civil engineering congress presentation
(~1 week out from 2026-09-09); everything above it is targeted for that date, everything below
stays open afterward as real, valued roadmap — not dropped.

Sibling checklists: `TraTrac/docs/PRODUCTION_MVP.md`, `FloCo/docs/PRODUCTION_MVP.md`.

URBAn is the civil-engineer-facing front door to TraTrac (and, eventually, FloCo) — the point of
this repo is that an engineer never has to touch a terminal, a TOML file, or hand-authored JSON.
See [`config_editor_spec.md`](config_editor_spec.md) for the config-editor screen's field-level
spec, sourced from TraTrac's `application/config.py` schema.

---

- [ ] Config-editor screen: video/detector/calibration/ego-motion form, pre-filled and validated live via `tratrac --check --json`
- [ ] Run launcher with live progress
- [ ] Minimal results screen — play the finished overlay video

**— 🏛️ realistic congress-day line — everything above is targeted for this window; everything below is real production-MVP work that stays open after —**

- [ ] Visual calibration tool — click image↔world correspondence points on an anchor frame instead of hand-authoring `calibration.json`
- [ ] Visual exclusion-zone drawing tool on anchor frames instead of hand-authoring `zones.json`
- [ ] Post-process screen exposing `tratrac-postprocess`'s smoother tuning, exclusion, and calibration in one place
- [ ] Results dashboard: vehicle counts, speed ranges, `validate_trj` compliance — not just the video
- [ ] FloCo integration: surface flow/turning-movement counts in the same results view
- [ ] Multi-run / project workspace: manage configs and results across many videos and sites
- [ ] Shareable report export (PDF/HTML) bundling video + stats + counts for a client deliverable
- [ ] Packaged, installable build (not just `tauri dev`) so an engineer can run it without a dev environment
