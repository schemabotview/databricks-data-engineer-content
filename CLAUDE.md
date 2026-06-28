# CLAUDE.md — databricks-data-engineer-content

A **content repo**, not an app. It holds the **Databricks Data Engineer** concept
that the `graphl-ux` app (sibling repo) loads **at runtime**. No render engine and no
scenes live here — the app fetches this repo's `manifest.json` + notebooks over the
network and renders them. Read alongside the workspace map in `../CLAUDE.md` and the
sibling `apache-spark-content/CLAUDE.md` (the mature reference for this contract).

There is **nothing to build, run, or test** here. The one executable is
`scripts/colab_generate_audio.ipynb`, a Colab tool that turns `tts/` scripts into
`audio/` `.wav`s.

## The core contract (do not break)

1. **The notebook is the single source of truth** for a module's prose and code.
   `manifest.json` only *wires* — it must never duplicate notebook content.
2. The app splits each notebook at every `## ` heading into **sections** (= pages),
   matched to the manifest overlay by **normalized heading text** (lower-cased,
   backticks stripped, whitespace collapsed). A heading edit must be mirrored in the
   manifest `heading`.
3. A section's diagram images (`![]()`) are **stripped** by the app — the **scene**
   replaces them.
4. **The scene lives in the `graphl-ux` app** (`src/scenes/databricks-data-engineer.ts`),
   authored with the engine's pattern helpers. Here you only reference it **by id**
   (`databricks-data-engineer`). The local `scenes/` dir is reserved/empty.

## manifest.json shape

- Top level: `concept`, `design`, `scenes[]` (id/title/status), `presentations[]`
  (one per module).
- Each presentation: `id`, `title`, `notebook` (path), `defaultScene`, `sections[]`.
- Each section overlay: `heading` (matches a notebook `## ` heading), `scene`
  (always `databricks-data-engineer`), `spine` (bool — drives feed-mode linear
  narration), an optional `role` (`"hook"` on each module's first section), an
  optional `highlight` (string[] of scene node ids — those light AMBER, the rest
  dim), and an optional `focus` (a node id the camera frames). Omit `highlight` to
  show the scene full-strength (the module hook).

## The scene (one dense map, app-side)

Every module rides the **single** `databricks-data-engineer` scene — a faithful port
of NodeMap's `databricks-data-engineer.ts`. Four columns left → right:

```
Sources (Batch · Streams) → Connectivity (Lakeflow Connect · Federation)
  → Data Intelligence Platform (Lakeflow Jobs · Medallion · Workspace+Compute
      · Foundation [Delta Lake · Engine · Unity Catalog] · Cloud Storage)
  → Consumers (BI · Apps · Sharing)
```

The manifest frames one subsystem per section via `highlight`/`focus`. The node ids
live in the scene's TS (`graphl-ux/src/scenes/databricks-data-engineer.ts`) — check
there before adding a `highlight`. Key ids: `bronze`/`silver`/`gold`, `delta-lake`
(+ `delta-log`/`acid`/`time-travel`/`optimize-zorder`/`vacuum`/`merge-into`),
`unity-catalog` (+ `uc-metastore`/`uc-namespace`/`uc-grant`/`uc-lineage`/`uc-volumes`),
`lakeflow-connect`/`auto-loader`/`copy-into`/`connectors`, `declarative-pipelines`
(+ `dlt-table`/`streaming-table`/`dlt-expectations`/`apply-changes`/`dlt-live`),
`lakeflow-jobs`, `compute` (+ `all-purpose-cluster`/`job-cluster`/`sql-warehouse`/
`serverless`/`dbr-runtime`), `engine` (+ `spark-engine`/`photon`).

## Narration (TTS)

`.tts` scripts are read aloud by ChatterboxTTS — plain spoken prose only (no
markdown, no raw code/SQL, spell out symbols and acronyms). See the source repo's
`CLAUDE.md` (`~/Projects/databricks-data-engineer/CLAUDE.md`) for the full TTS guide.

Narration is authored **per section** (one `.tts` per teaching page), named
`tts/<NN>-<SS>-<slug>.tts` → `audio/<NN>-<SS>-<slug>.wav`, where `SS` is the section's
page position so the Colab glob (`tts/*.tts`, sorted) stays in reading order. Each
narrated section's `audio` field in the manifest points at the matching `.wav`.

**Intros and outros are silent** (no `.tts`, no `audio`): `What's covered`, every
`Section N self-check`, `Summary`, and `Course recap`. Everything else is narrated —
including `Reference domain` and the `Hands-on` walkthroughs (described conceptually,
no code/SQL read aloud, per the TTS guide). 133 scripts; 28 silent sections.

The `.wav`s are **not generated yet** — `audio/` is empty. Run
`scripts/colab_generate_audio.ipynb` over `tts/*.tts` to produce them, then commit/push.
Until then a narrated section's clip 404s in the app (it doesn't play). To re-derive the
`audio` fields after adding/removing a `.tts`, the silent-set predicate + per-module
count-match logic lives in the wiring approach noted in this repo's history.

## Source of notebooks

Notebooks are copied as-is from `~/Projects/databricks-data-engineer`. The curriculum
targets the **Databricks Certified Data Engineer Associate** exam (version May 4, 2026):
9 thematic notebooks (01–09) mapped to the seven exam sections, plus a practice exam (10).

## Status

- **Modules 01–09 wired** (161 sections), all on the `databricks-data-engineer` scene
  with per-section `highlight`/`focus` and spine flags; each module's first section is
  the `hook` (full map). Module 10 (practice exam) is parked.
- App wiring done in graphl-ux: scene registered (`src/scenes/index.ts`) and concept
  added (`src/content/catalog.ts`, id `databricks-data-engineer`, category `Data`).
- **Per-section `.tts` authored** (133 scripts; intros/self-checks/outros silent) and
  wired to each section's `audio` field.
- Pending: **generate the `.wav`s** (`scripts/colab_generate_audio.ipynb`) and commit
  `audio/`; push this repo to its remote
  (`github.com/schemabotview/databricks-data-engineer-content`).
