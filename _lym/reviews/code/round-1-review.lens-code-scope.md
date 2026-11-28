```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. I traced each change in the diff to the board entry's goal points, which are the decision record's only measuring stick since it has no `Added after Discussion` entries.

**Decisions** (goal points of `## Goal`; the record has no added entries):
- **G1.** Before `--reset` removes anything, check for unpushed records and board commits, uncommitted Bolt changes, and ignored files that hold work. Refuse when any is found.
- **G2.** The refusal names what would be lost.
- **G3.** The refusal gives the way forward: push or commit first, or pass an explicit flag that accepts the loss.

**Changes, each mapped to a decision:**
- **C1. The records store and the Bolt checkout are now read.** This is the `stores` loop in `hubWorkOriginLacks` and the per-store code moved into `storeWorkOriginLacks` (clone.go:1003-1072). Traces to G1.
- **C2. `ignoredWork`, plus the cap and count of named paths** (`ignoredPathsNamed`, clone.go:1039-1053, 1074-1098). Traces to G1 (ignored files) and G2 (naming them).
- **C3. `hubWork` is split into `inUse` and `stores`, and `checkResetTarget` drops `stores` under DiscardWork** (clone.go:948-973). Traces to G3, the flag that accepts the loss.
- **C4. The `DiscardWork` field, `DiscardWorkFlag`, the cobra flag, CLI pass-through, usage string and `cloneResetCommand` suffix** (verbresults.go:98-100; fabric.go:100,115; fabriccli/clone.go:37,56; clone.go:43-44,170-173). Traces to G3.
- **C5. `hubWorkWayForward`**, the "push or commit … or replace … with --discard-work" text, used by `checkResetTarget` and `existingHubError` (clone.go:64-83, 955-956). Traces to G3. `existingHubError`'s use is forced by C3's change of the return type.
- **C6. The refusal of `DiscardWork` without `Reset`** (clone.go:341-343). Forced by C4.
- **C7. The seed refusal now names `--reset --discard-work`** (pushclassified.go:230,256). Forced by C1: plain `--reset` now refuses the unpushed seed it used to name.
- **C8. The `resetHub` dirtinessNA reason and the `opts` parameter threading** (clone.go:898-911, 390, 429, 482). Forced by C3 and C4.
- **C9. The `cloneFlags` struct replacing the positional parameters of `runCloneWithReset`** (fabriccli/clone.go diff lines 9-18, 27-59; fabric.go:107-110). Not requested. Only adding one parameter was forced. See finding S2.
- **C10. Package docs, help text and sandbox scenario F12** (fabriccli/doc.go, fabricengine/doc.go:572,602-608, fabric.go:86-92, SANDBOX-FABRIC-SUITE.md:433-446). Forced by C1–C5.
- **C11. Test edits:**
  - the new rows for records, board, Bolt and ignored files;
  - the `DiscardWork` subtest;
  - the refusal of `--discard-work` alone;
  - `DiscardWork` or a push added to existing tests;
  - the reworded header of `clone_reset_guard_test.go`.

  These cover C1–C6 or are forced by C1.

**Decisions, each mapped to changes:**
- **G1:** C1, C2. Partly built: see finding S1.
- **G2:** C2, C5 (each item is named by store path, plus the ignored-path list).
- **G3:** C3, C4, C5.

**Findings**

S1
- Severity: MEDIUM
- Class: scope
- Location: `internal/fabricengine/clone.go:1074-1098` (`ignoredWork`)
- Summary: G1 asks to refuse over "ignored files that hold work". The diff counts every ignored path except links, `.lym` and `.weft`, so it refuses plain `--reset` over files lym itself writes that hold no work of the operator's. The entry's qualifier "that hold work" is therefore not built.
- Evidence: on this machine's hub, `git ls-files --others --ignored --exclude-standard --directory` lists these ignored entries.
  - The prime lists `CLAUDE.local.md`, `AGENTS.override.md` (regular files written by heddle's render), `.claude/` and `.vscode/`.
  - `_bolt` lists `.board-rendered.json` (`boardengine/render.go:22`), `board.lock` and `board.push.lock` (`fabriclayout/hubpaths.go:16,20`), and `.gitrepo-push.lock` and `board.json.swaplock`.
  - None of these is a link, `.lym` or `.weft`, so each would be named as work.
  - `fabricengine/template.yaml:3`'s `warp_exclude` default makes heddle's files ignored in every hub.
  - The summary's "Every ignored path counts as work" turns the entry's qualifier into its opposite.

S2
- Severity: NIT
- Class: scope
- Location: `internal/fabriccli/clone.go` (`cloneFlags`, `runCloneWithReset` signature); `internal/fabriccli/fabric.go:107-110`
- Summary: Replacing the positional parameters with a `cloneFlags` struct is a refactor no goal point asks for. Adding one `discardWork` parameter was all the flag forced.
- Evidence: diff 001 lines 9-28, 46-59, 107-110. `_lym/landing/summary.md` lists it under "Design decisions" with no trace to the entry.

Tool calls: none.
