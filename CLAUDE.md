# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

`Ebb` is **not a software project** — there is no build, no test suite, no
dependencies, and no code to run. It is the configuration and prompt library for
a recurring **morning EBB sweep**: a read-only research workflow that checks the
Electronic Bulletin Boards (EBBs) of interstate natural gas pipelines for
critical notices, maintenance, force majeure, and capacity constraints, then
reports what matters to the user's book.

The "program" is the agent that runs the sweep. This repo holds its prompt, its
target list, and its filter criteria. Edits here are almost always *data* edits
(a URL, a meter, a pipeline row) or *prompt* edits (the scout template).

## Layout

```
README.md                     Human-facing summary of the sweep
ebb/scout-task-template.md    The scout prompt, with {{PIPELINE_NAME}} / {{POSTINGS_URL}} slots
ebb/pipelines.yaml            One row per pipeline: name, operator, postings_url
ebb/my-meters.yaml            The user's book — meters/points/zones the sweep filters against
```

Three files, all hand-maintained. Keep it that way; resist adding tooling,
scripts, or package manifests unless the user asks for them.

## Running the sweep

1. Read `ebb/pipelines.yaml`. Dispatch **one scout per pipeline, in parallel**,
   using `ebb/scout-task-template.md` with `{{PIPELINE_NAME}}` and
   `{{POSTINGS_URL}}` substituted from that row.
2. Scouts are **read-only** — WebSearch and WebFetch only. Never give a scout
   Bash, Edit, Write, or git. A scout that changes state is a bug in the
   dispatch, not a feature.
3. Merge every returned notice into one table sorted by `effective_start_date`.
4. Add a **"⚑ HITS MY BOOK"** section for notices touching anything in
   `ebb/my-meters.yaml`. Skip this section entirely when that file has no
   entries — an empty book means no filter criteria, not "no hits".
5. Close with a 3-line "what I'd re-nom today" call. Without a populated book,
   this is a generic capacity read and should be labeled as such.

The sweep reports; it does not act. Nothing in this workflow nominates,
schedules, or transacts.

## Conventions

### Accuracy rules (non-negotiable)

These exist because the output feeds scheduling decisions:

- **Never fabricate** notice IDs, dates, capacity numbers, or URLs. Unknown
  fields are written `unknown`, not guessed.
- **Prefer the TSP's own postings.** A third-party aggregator or news article is
  never a silent substitute for the pipeline's own critical-notices page. If one
  is used, flag it explicitly in the output.
- **Report blockers plainly.** Login wall, JS portal, 403, Cloudflare, or simply
  nothing posted — say which, and say which URLs were tried. Accuracy over
  completeness.
- Most of these portals actively block automated fetch. That is the expected
  state, not a failure to hide.

### `pipelines.yaml`

- `postings_url` must be a **deep link** to the operator's critical-notices or
  informational-postings page — not a marketing landing page.
- Annotate the confidence of each URL in a comment above it, matching the
  existing style: confirmed-but-bot-blocked, `CANDIDATE (unverified — ...)`, or
  `TODO:` with what was tried and what remains unresolved.
- A blank `postings_url` means the scout has to discover the page itself. That
  is the least reliable mode; filling these in is the main way to improve the
  sweep.

### `my-meters.yaml`

- Grouped by pipeline name (top-level key matching a `pipelines.yaml` `name`) so
  any hit can be attributed back to a pipeline.
- Entries are loose key/value shapes — `point`, `zone`, `path`, `meter`,
  `delivery_point`, with an optional `direction`. Don't impose a stricter schema
  than the leading comment block documents; follow the shapes already present.
- `direction: unknown` is an acceptable, honest value.

### `scout-task-template.md`

The prose below the `---` is the literal prompt sent to each scout. Edit it as a
prompt — precise, imperative, no preamble — and keep the field list in step 3 in
sync with whatever the merged table renders.

## Git workflow

- Work on the branch assigned for the task; never push to another branch without
  being asked.
- Push with `git push -u origin <branch-name>`.
- Commit subjects are short and specific about *what data changed*, e.g.
  `Add Frisco (Acadian) and TETCO zone M3 to my-meters.yaml`. When a change
  carries research context — which URLs were confirmed, which were left
  unresolved and why — put that in the commit body. That body is the only record
  of verification work that produced no diff.
- Don't open a pull request unless the user explicitly asks.
