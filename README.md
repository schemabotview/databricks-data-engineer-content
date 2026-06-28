# databricks-data-engineer-content

Content repo for the **Databricks Certified Data Engineer Associate** concept,
consumed at runtime by the [`graphl-ux`](https://github.com/schemabotview/graphl-ux)
app. This repo holds **content only** — notebooks, narration, and the wiring
manifest. There is **nothing to build or run** here.

The app fetches this repo over raw GitHub
(`https://raw.githubusercontent.com/schemabotview/databricks-data-engineer-content/main/…`):
it reads `manifest.json`, then each module's notebook, splits the notebook at every
`## ` heading into pages, and renders the scene named per section.

## Layout

```
manifest.json   # wires each module: notebook ref + per-section overlay (scene / spine / role / highlight / focus)
notebooks/      # the teaching .ipynb (prose + code source of truth) — 01..10
tts/            # per-module narration scripts (plain spoken prose)
audio/          # generated .wav narration
scenes/         # reserved/empty — the real scene lives in graphl-ux (src/scenes/databricks-data-engineer.ts)
scripts/        # colab_generate_audio.ipynb — tts/*.tts -> audio/*.wav on a Colab GPU
```

## The contract (shared with the sibling content repos)

1. **The notebook is the single source of truth** for a module's prose and code.
   `manifest.json` only *wires* — it never duplicates notebook content.
2. The app matches a section to its manifest overlay by **normalized heading text**
   (lower-cased, backticks stripped). Edit a notebook `## ` heading → mirror it in
   the manifest `heading`.
3. A section's diagram images are stripped by the app — the **scene** replaces them.
4. **The scene lives in `graphl-ux`** (`src/scenes/databricks-data-engineer.ts`),
   authored with the engine's pattern helpers. Here it's referenced by id only
   (`databricks-data-engineer`).

A content change is live once pushed to `main` — no app rebuild needed.

## Status

- **Modules 01–09 wired** in `manifest.json` (161 sections), all riding the one dense
  `databricks-data-engineer` scene with per-section `highlight`/`focus`. Module 10
  (practice exam) is parked.
- **Per-section narration authored** — `tts/<NN>-<SS>-<slug>.tts`, one script per
  teaching page (133 total), each referenced from its section's `audio` field. **Intros
  and outros are silent** (no `.tts`): `What's covered`, every `Section N self-check`,
  `Summary`, and `Course recap` (28 silent sections).
- **`.wav`s not generated yet** — `audio/` is empty. Run `scripts/colab_generate_audio.ipynb`
  over `tts/*.tts` to produce `audio/<stem>.wav`; until then a narrated section's clip
  404s in the app (it just doesn't play).
- Source curriculum: `~/Projects/databricks-data-engineer`.
