# Card 10 — adopt-docs-and-pattern

**Edit:**
- `PATTERN.md`
- `pattern/PATTERN-dobby-runs.md`
- `internal/adoptcli/doc.go`

**Intent:**
The declared `PATTERN-dobby-runs` change, and the adopt module's package doc over cards 4 to 8.

`PATTERN.md`'s `PATTERN-dobby-runs` line becomes:

```markdown
- `PATTERN-dobby-runs` — Starting or holding a board task's run: only its parent's dobby, started with its orch bar `lym adopt --no-orch`; front tasks only via `fronttask`; an adopt pause lifts only by hand. — [background](pattern/PATTERN-dobby-runs.md)
```

`pattern/PATTERN-dobby-runs.md`:
- its opening sentence follows the new line;
- the "Dobby and orch pair" rule names the adopt `beam` step's dobby-only start under `--no-orch`, through `orchcli.BeamUp` with `SkipOrch`;
- the "Adopt hold" section becomes "Adopt pause".
  On a takeover the `pause` step pauses each adopted run, and each unstarted released task of the subject bar a front task, with a `pauserecord.AdoptCaller` entry.
  Only `lym tender resume` or `lym batten cancel` lifts it, never the adopt check, the dobby's owner lift or a hibernate wake.
  A legacy `adopt-hold` entry reads the same.

`internal/adoptcli/doc.go` is rewritten over what cards 4 to 8 changed:
- the caller and the detached worker;
- the record and `adopt.lock` with resume, `--discard` and the subject match;
- D3's step order with the owner write before the deploy;
- the adopt pause in place of the hold and `holds.json`;
- `--no-orch` and its orch-session default;
- the green end that only removes the fix task and resets the try count;
- the `paused_by_adopt` run state.
The doc no longer says a green check lifts holds, brings up the dobby and orch, or starts runs.
The scratch directory's file list drops `holds.json` and names `adopt.json`, `adopt.lock` and `adopt.out` directly under `fabricengine.AdoptScratchDir`.

No test: the line and the docs are prose, which the code review reads (the record's Review criterion).

**ImpactSummary:** one PATTERN overview line, its background and one package doc; no code changes, and the line's rule text stays under the entry cap.
