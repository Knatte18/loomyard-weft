```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens, covering what the diff's changes force outside the lines it touches.

**Per change, the set I found**

1. **New measurement-entry rule in item 3 of `darn-fragment-make-change-gated.md`.**
   - Covered:
     - the fragment's header comment (lines 4–5);
     - `stencils.go`'s `DarnFragmentMakeChangeGated` doc;
     - `spec.go`'s `DarnTable` doc;
     - the gated row of `TestDarnSpec`;
     - `markers_test.go:69`, which needs no change because the rule adds no marker.
   - Not updated, and none needed:
     - `wiring_darn_test.go:303` keeps its own copy of `gatedNoGateStep`, and the diff leaves that text unchanged;
     - `docs/overview.md:1080` does not describe the writer's Step 4 gate rules;
     - `internal/darnengine/doc.go` does not describe them either;
     - no `PATTERN.md` or `pattern/` entry describes the writer's in-session gate calls.
   - Left wrong: the Step 5 tooling-names rule in `darn-template-write.md:78` (finding F1).
2. **The same rule in item 3 of `darn-fragment-make-change.md`.**
   - Covered: the header comment (lines 2–3), `stencils.go`'s `DarnFragmentMakeChange` doc, `spec.go`, and the ungated first-spawn row of `TestDarnSpec`.
   - Left wrong: the item 2 subagent gate hand-off in the same fragment (F2), and the Step 5 rule (F1).
3. **The `TestDarnSpec` assertions on `measurementEntryRule`.** Nothing else reads the constant.
4. **Commit messages.** There are no `Pattern-Departure:`, `Loosened-Check:` or `Root-Cause:` trailers, and none is needed. The directive declares no PATTERN changes, so nothing is owed there.

**Findings**

- **F1 — MEDIUM, design.**
  - Location: `contracts/stencils/darn/darn-fragment-make-change-gated.md:45` and `contracts/stencils/darn/darn-fragment-make-change.md:27`.
  - Summary: with no parent, the writer must "name them in the change description", meaning the planned `lym gate test --tier …` calls. Step 5 of the same rendered prompt forbids the change description to name lym, a gate or a tier.
  - Evidence: `contracts/stencils/darn/darn-template-write.md:78`, which bans "LoomYard, lym, the board, … a gate or a tier".
  - Why it is new: the older no-parent clause in the ungated fragment, line 17, names skipped tests, which are repository objects, not tooling calls.
  - Effect: on every hub the rendered prompt now gives the writer two rules that contradict each other.
- **F2 — MEDIUM, design.**
  - Location: `contracts/stencils/darn/darn-fragment-make-change.md:12`, item 2, which the diff leaves unchanged.
  - Summary: in the ungated list, item 2 still has the writer give each subagent "the gate rule from {{template "engine-standing-instructions"}} … for the subagent to run on its own package".
  - The subagent sees none of the session (line 12), so it never sees the new parent-first rule. A writer that hands a measurement entry's per-package work to subagents passes on unbounded gate calls.
  - The gate the rule set up can be bypassed through delegation, which is the loop shape of the original incident.
  - Line 13's "keep in this session" list does not cover measurement calls either.
  - The gated list needs no such fix: its item 2 already tells subagents to run no `lym gate test` (gated line 23).
- **F3 — LOW, design.**
  - Location: `contracts/stencils/darn/darn-fragment-make-change.md:23`.
  - Summary: "falls outside these rules" leaves it unclear whether the per-commit `lym gate test` at line 14 still applies to a measurement entry's own commits, such as committed result files or a code change.
  - Read literally, the parent-first rule covers "any gate call for it", so the per-commit test run after a code edit could also wait on the parent.
  - The stencil does not separate the commit gate run from the measurement calls.

Outside my lens: in the no-parent case the rule skips the deliverable but neither stops nor writes a stop marker, which is unlike the template's other no-parent paths at `darn-template-write.md:60-63` and `103`. I leave that to the handler and the correctness lens.

**Tool calls**

1. grep across the repository for docs and code naming the darn gate rules, because the manifest did not cover readers outside the files in the diff.
2. Read `internal/darnengine/doc.go`, the gate references in `wiring_darn_test.go` and `markers_test.go`, and the darn mentions in `PATTERN.md` and `pattern/`, because these readers were not in the manifest.
3. grep the specs and docs for the cold-fixer and measuring-call wording, to find any spec that describes the rule.
4. Listed `contracts/specs` and grepped it for darn writer gate text, to look for a spec of Step 4.
5. grep `docs` for darn writer gate descriptions outside `research`, to look for an overview doc that describes Step 4.
6. Read the darn rows of `docs/overview.md` and `docs/dotnet-hub.md`, to check whether the overview describes the writer's gate rules.
