# Ebb

Config for the morning EBB (Electronic Bulletin Board) maintenance sweep across
interstate natural gas pipelines.

## Layout

- `ebb/scout-task-template.md` — the scout prompt. One scout is dispatched per
  pipeline, in parallel, read-only (WebSearch / WebFetch only). Substitute
  `{{PIPELINE_NAME}}` and `{{POSTINGS_URL}}` before dispatch.
- `ebb/pipelines.yaml` — the config table: pipeline names, operators, and the
  postings URL each scout starts from. **Fill in `postings_url` for each.**
- `ebb/my-meters.yaml` — the book the sweep filters against for the
  "HITS MY BOOK" section. **Fill in `meters`.**

## Running the sweep

1. Dispatch one scout per pipeline in parallel using the template + each row's
   `postings_url`.
2. Merge every returned notice into one table sorted by effective start date.
3. Add a "⚑ HITS MY BOOK" section for notices touching anything in
   `my-meters.yaml` (skipped when that file is empty).
4. Close with a 3-line "what I'd re-nom today" call.
