# mex 0.8.3 — Faster graphs, more reliable grounding

MEX 0.8.3 makes graph refresh incremental, preserves more knowledge links across
clones and code changes, and expands route and dependency coverage. It also adds
scaffold export and Markdown timelines.

## Faster graph maintenance and reads

- Refresh reuses unchanged file extractions and rewrites only changed graph
  rows. TypeScript and JavaScript dependents are re-extracted when needed;
  broad compiler or configuration changes fall back to full extraction.
  Resolution still runs over the whole corpus.
- A refresh with nothing to publish leaves `graph.db` untouched. JSON output
  reports the refresh mode, fallback reason, and work performed. The extraction
  cache adds to local database size.
- Targeted reads reuse a validated publication audit, and CLI code splitting
  avoids loading the compiler and terminal UI for lightweight commands.

## Grounding that survives everyday work

- TypeScript node IDs no longer depend on the checkout directory. Type signatures
  use canonical ordering so incremental and full extraction agree.
- `mex impact` reads knowledge links from the committed scaffold, so they work
  on fresh clones. Grounding records take priority over transitive callers when
  the output budget is tight; `graph get` keeps node metadata when source does
  not fit. Inline anchors alone are not returned as impact grounding records.
- `mex check` reads groundings at both the root and under `mex:`. Writers
  consolidate compatible root entries under `mex.grounds_to`; conflicting
  entries stay visible for review.
- Source edits no longer disable all grounding checks. Unchanged files can use
  the prior snapshot; edited tree-sitter files can be re-extracted locally.
  Edited TypeScript/JavaScript groundings report `GROUNDING_UNVERIFIED` until
  refresh rather than appearing clean. Other freshness failures still block
  checks that cannot be trusted.
- Small-function renames can reconcile from callers and callees, with an
  explicit `GROUNDING_MOVED_BY_NEIGHBORS` notice. Equally good candidates remain
  ambiguous, and inline anchors can use committed fingerprints after rebuilds.

## More project coverage and useful outputs

- New bounded **FastAPI, Flask, and NestJS route resolvers** join Express and
  Next.js App Router. Static paths and unambiguous same-file handlers are
  supported; this is not general runtime routing analysis.
- Dependency checking reads Python `pyproject.toml`, including common optional
  and grouped dependencies. Architectural labels produce fewer package warnings.
- Graph commands and doctor report recognized source files with no registered
  extractor, and status distinguishes uninspected fields from real zero counts.
- `mex export` bundles scaffold Markdown into one document; use `--out <path>`
  for a file. `mex timeline --format md` produces a Markdown table for reports.
- Heartbeat explains when missing `last_updated` fields leave staleness checks
  inactive, accepts zero-day thresholds, and deduplicates symlinked files.
- Official Inbox and Relay skills are discoverable through standalone skill
  installers and Claude Code marketplace metadata. Standalone installation is
  an alternative to MEX-managed skills; it still requires the CLI and a project.

## Upgrade an existing project

**Keep the old graph for the first refresh.** TypeScript signature fixes can
change node IDs, and the old index lets refresh retain aliases for those IDs.

```bash
npm install -g mex-agent@0.8.3
mex graph refresh
mex sync
```

The first refresh performs a full extraction with `typescript-5.9-v5`.
Review the scaffold diff and any ambiguous or missing groundings before
committing through Git. If the scaffold changed, run `mex wiki rebuild-index`
to update its local index. A fresh clone or rebuild without the old graph uses
committed fingerprints and may require manual re-grounding. For an incompatible
or damaged index, follow the explicit recovery action from `mex graph status`.

For integrations managed by MEX:

```bash
mex skills sync --dry-run
mex skills sync
```

Review conflicts and start a fresh agent session. Do not install standalone
skills over a MEX-managed integration. Completed 0.8.0–0.8.2 setups do not need
setup again solely to upgrade. New projects can start with
`npx mex-agent@0.8.3 setup`; terminal setup remains `setup --cli`.

Node.js 22.5 or newer with SQLite FTS5 remains required. Graph schema stays v4
with an internal extraction cache; canonical Wiki and Relay artifact versions
are unchanged. Public API additions include the optional
`HeartbeatResult.filesWithoutLastUpdated` field and three grounding issue codes;
exhaustive consumers should review [the compatibility guide](COMPATIBILITY.md#additive-api-changes-in-083).

## Contributors

Thanks to everyone who contributed code, tests, documentation, and integration
work since 0.8.2:

- @theyashasvipandey — incremental refresh and grounding, identity, and path fixes ([#247](https://github.com/mex-memory/mex/pull/247), [#246](https://github.com/mex-memory/mex/pull/246), [#244](https://github.com/mex-memory/mex/pull/244), [#243](https://github.com/mex-memory/mex/pull/243), [#242](https://github.com/mex-memory/mex/pull/242), [#241](https://github.com/mex-memory/mex/pull/241), [#200](https://github.com/mex-memory/mex/pull/200)).
- @abhinav-phi — Flask/NestJS routes, Python dependencies, claim filtering, coverage, export, timeline, heartbeat, CLI smoke tests, and changelog history ([#177](https://github.com/mex-memory/mex/pull/177), [#102](https://github.com/mex-memory/mex/pull/102), [#185](https://github.com/mex-memory/mex/pull/185), [#186](https://github.com/mex-memory/mex/pull/186), [#175](https://github.com/mex-memory/mex/pull/175), [#183](https://github.com/mex-memory/mex/pull/183), [#182](https://github.com/mex-memory/mex/pull/182), [#181](https://github.com/mex-memory/mex/pull/181), [#178](https://github.com/mex-memory/mex/pull/178), [#184](https://github.com/mex-memory/mex/pull/184)).
- @sidsri14 — FastAPI routes ([#113](https://github.com/mex-memory/mex/pull/113)).
- @Kaustubh1235 — targeted-read performance ([#196](https://github.com/mex-memory/mex/pull/196)).
- @chiliec — impact output priority and honest status fields ([#238](https://github.com/mex-memory/mex/pull/238), [#215](https://github.com/mex-memory/mex/pull/215)).
- @iveteamorim — graph-get output budgeting ([#239](https://github.com/mex-memory/mex/pull/239)).
- @architdhamija — skill discoverability ([#245](https://github.com/mex-memory/mex/pull/245)).
- @theDakshJaitly — skill discovery validation and installer separation, export FIFO safety tests, historical compatibility corrections, README upkeep, and release integration ([skills fixes](https://github.com/mex-memory/mex/commit/e6fa172), [export tests](https://github.com/mex-memory/mex/commit/a0c1a92), [changelog corrections](https://github.com/mex-memory/mex/commit/81d83d2), [README update](https://github.com/mex-memory/mex/commit/5b6e02c)).

**Full changelog:** [v0.8.2…v0.8.3](https://github.com/mex-memory/mex/compare/v0.8.2...v0.8.3).
