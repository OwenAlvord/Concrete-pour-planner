# Project Layout

This project is a single-file web app.

- `index.html` stays at the top level because GitHub Pages serves it from there.
- `AGENTS.md`, `spec.md`, and `prompts.md` also stay at the top level.
- `00_planning/` contains notes and sketches.
- `01_data/` contains data files.
- `02_validation/` contains hand calculations and test cases.
- `03_output/` contains screenshots and exported results.

## File Rules

- Save every new file in the appropriate folder; do not add unrelated files at the top level.
- Keep the app implementation in `index.html`.
- Put supporting project files in their numbered folder: planning, data, validation, or output.
- After each request, add an entry to `prompts.md` describing what was requested, when it was changed, and whether it worked.
