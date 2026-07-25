# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

Ebb is a **configuration and prompt repository**, not an application. It holds
the inputs for a recurring "morning EBB sweep" — a read-only scan of interstate
natural gas pipelines' Electronic Bulletin Boards (EBBs) for critical notices,
maintenance, force majeure, and capacity constraints.

There is no source code, no build, no dependencies, no test suite, and no CI.
Everything here is either YAML config or a prompt template. The "program" is the
sweep procedure in `README.md`, executed by an agent (you) reading these files.

Domain shorthand used throughout: **EBB** = Electronic Bulletin Board (a
pipeline's public postings site), **TSP** = Transportation Service Provider (the
pipeline operator), **OFO** = Operational Flow Order, **nom** = nomination
(a shipper's scheduled gas volume).

## Layout

```
README.md                    Human-facing overview + the 4-step sweep procedure
ebb/pipelines.yaml           The config table: one entry per monitored pipeline
ebb/my-meters.yaml           The book to filter notices against ("HITS MY BOOK")
ebb/scout-task-template.md   The per-pipeline scout prompt (has {{placeholders}})
```

## The sweep workflow

When asked to run the morning sweep:

1. Read `ebb/pipelines.yaml`. For each entry, fill `{{PIPELINE_NAME}}` and
   `{{POSTINGS_URL}}` into `ebb/scout-task-template.md` (use the body **below**
   the `---` separator; everything above it is documentation about the template,
   not part of the prompt).
2. Dispatch one scout per pipeline, **in parallel**. Scouts are read-only:
   WebSearch and WebFetch only, no Bash / Edit / Write / git.
3. Merge every returned notice into one table sorted by `effective_start_date`.
4. Add a `⚑ HITS MY BOOK` section for notices touching any entry in
   `ebb/my-meters.yaml`. Skip this section entirely if that file is empty.
5. Close with a 3-line "what I'd re-nom today" call. If `my-meters.yaml` is
   empty this is a generic capacity read, not a book-specific one.

Entries with a blank `postings_url` still get a scout — the scout must discover
the page itself. Flag that in the output; discovered URLs are less trustworthy
than configured ones.

## Conventions

**Accuracy over completeness.** This config feeds scheduling decisions on real
gas. Never fabricate a notice ID, date, capacity number, or URL. Unknown fields
are written literally as `unknown`. A scout that hits a login wall, JS portal,
or 403 must say so and name the URLs it tried rather than substituting a
third-party or news source for the TSP's own postings.

**`postings_url` must be the TSP's own deep link** to its critical-notices /
informational-postings page — not a marketing landing page, not an aggregator.
Most of these endpoints are bot-blocked (Cloudflare, JS portals, `.asp` query
strings); that is expected and is not a reason to swap in a different source.

**Comment the verification status of every URL.** The `#` comment above each
`postings_url` is load-bearing — it records how much to trust the value. The
established vocabulary:

- confirmed-but-blocked, e.g. `# Enbridge LINK / infopost critical-notices list (bot-blocked to automated fetch):`
- unverified, e.g. `# CANDIDATE (unverified — portal 403'd automated fetch; confirm in a browser):`
- unresolved, written as a `# TODO:` explaining what was tried and what is
  still unknown, with `postings_url: ""`

Never silently upgrade a CANDIDATE to a plain entry, and never delete a TODO
without recording what resolved it.

**`my-meters.yaml` is intentionally loose.** It is a mapping of pipeline name to
a list of entries; each entry uses whatever keys fit that point (`point`,
`zone`, `meter`, `path`, `delivery_point`, `direction`). Do not impose a rigid
schema — the file's own header comment shows the example shape. Preserve the
header comments when editing; they document the empty-file fallback behavior.

**Keep `README.md` in sync.** It describes the same layout and procedure. A
change to the sweep steps or to what a file is for belongs in both places.

## Verifying changes

There is nothing to build or test. After editing either YAML file, confirm it
still parses:

```bash
python3 -c "import yaml; yaml.safe_load(open('ebb/pipelines.yaml')); yaml.safe_load(open('ebb/my-meters.yaml')); print('OK')"
```

Beyond that, verification is manual: a `postings_url` is only "confirmed" when
someone has actually loaded it and seen the TSP's notices list.

## Git

Commits are single-purpose and imperative, naming the pipelines or points
touched — e.g. `Add Eastern Gas Transmission (EGTS/BHE GT&S) to pipelines
config`. When a value is uncertain, the commit body records *why* (what was
tried, what is still unconfirmed), matching the `#` comment left in the file.
Do not push to a branch other than the one you were assigned.
