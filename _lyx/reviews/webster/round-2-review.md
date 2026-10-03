---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    location: internal/websterengine/cardgate.go:2
    summary: New Go comment lines still join two independent clauses with a semicolon on one line (cardgate.go:2, verifygatereport.go:64, doc.go:401, runlevel.go:724, runlevel_test.go:113, mergestage.go:131, mutation.go:83, pwsh.go:22)
  - id: F2
    severity: LOW
    location: internal/fabricengine/mergestage.go:128
    summary: Wholly new Go comment blocks across the diff are hard-wrapped at a fixed column, breaking mid-phrase and splitting one sentence over several lines, against CONSTRAINTS.md's Comment Line-Break Convention
  - id: F3
    severity: NIT
    location: contracts/specs/webster-spec.md:53
    summary: New markdown lines join two independent clauses with a semicolon on one line, against CLAUDE.md's semantic-line-break rule
---

Review range: no `_lyx/loom/rework/` directory exists, so there is no live rework round; `product.parent` in `_lyx/shed/verify-publish-hardening/status.json` is `main`, and the range is `git diff $(git merge-base main HEAD)..HEAD` (merge base `5805c58fb`, 26 commits, 158 files).

Substrate: `go build ./...` and `go vet ./...` are clean, `go test ./...` passes, and `go test -tags integration` passes over verifytree, websterengine, landingshed, fabricengine, gitrepo, mergeresolve, webstercli, loomshed and shuttleengine.

Per-card mechanical check: card 17 carries the plan's only `Delete` group.
Its nine target files are gone, and a whole-tree grep (Go, markdown, YAML, `_lyx` and `.lyx` excluded) finds zero references to every symbol and stencil name card 17 and card 20 retire (`runIntegrationStage`, `FabricBisector`, `FixStarter`, `OpenBisector`, `IntegrationFailure`, `CheckoutDetached`, `RestoreBranch`, `webster-template-integration`, and the rest of card 17's list).
No card carries a `Rename` or `Move` group.

Code spot-checks of `internal/verifytree/verify.go` (the HEAD-moved guard before the record write, the `-z` porcelain parse and its rename/copy source skip) found nothing.

The convention measured against is CONSTRAINTS.md's Comment Line-Break Convention: one sentence per line, a further break at an independent-clause boundary, new and changed comments only, never a line-width finding, untouched lines never rewrapped.
A sentence is judged new when the diff wrote or reworded it and it ends on lines the diff wrote; a sentence edited inside a legacy fixed-column paragraph that runs on into untouched lines is legacy and not flagged, since bringing it into line would rewrap those lines.

### [LOW] Semicolon-joined independent clauses in new Go comments

**Location:** internal/websterengine/cardgate.go:2, internal/websterengine/verifygatereport.go:64, internal/websterengine/doc.go:401, internal/websterengine/runlevel.go:724, internal/websterengine/runlevel_test.go:113, internal/fabricengine/mergestage.go:131, internal/fabricengine/mutation.go:83, internal/shell/pwsh.go:22

**Issue:** Each of these lines carries two independent clauses joined by `; ` on one line, where the convention requires a break after the semicolon.
Focus instances, each judged a defect:
`cardgate.go:2` (`Go derives the command and the fork runs it; Go never runs it, …`), new file;
`verifygatereport.go:64` and its doc.go:401 twin (`The hint claims only that a card touched a failing package; Merriam judges it …`), new lines;
`runlevel.go:724` (`A done whose verify gate did not pass ends stuck; Master's own stuck keeps its own reason.`), a rewritten line;
`runlevel_test.go:113` (`on this helper; a test that needs one calls appendIntegrationVerify …`), where the second clause is reworded and ends on the diff's own line, so splitting it touches no untouched line;
`mergestage.go:131` (`Untracked files are never staged; MergeUntrackedFiles lists them …`), a new block;
`mutation.go:83` (`mid-merge; Target is the warp checkout path. It is recorded …`), a new block that also puts two sentences on one line (see F2).
The sweep found one more: `pwsh.go:22`, where `; Source builds on it.` is a clause the diff appended to a changed line.
Not flagged: `render.go:30` (`RenderVerifyFixPrompt resolve friction.RoleImplementer; RenderMasterPrompt resolves`) and `shedrecipe/entries_simple.go` (`caller leaves it nil; and ShuttleCfg, …`) are token edits inside legacy fixed-column sentences that run on into untouched lines.

**Fix:** Break each line after the semicolon so the second clause starts its own comment line.

### [LOW] Fixed-column wrapping in wholly new Go comment blocks

**Location:** internal/fabricengine/mergestage.go:128

**Issue:** Round 1's sweep covered semicolons only partly and left fixed-column wrapping in place.
These comment blocks are wholly new in the diff, yet break at roughly 100 columns mid-phrase (`in the warp checkout of an` / `in-progress fabric merge`), so a sentence spans lines at arbitrary points:
`internal/fabricengine/mergestage.go:128-138`;
`internal/fabricengine/mutation.go:82-84`;
`internal/fabricengine/mergestagetracked_integration_test.go:3-6, 22-23, 87-88, 102-103`;
`internal/gitrepo/merge.go:202-206`;
`internal/gitrepo/worktree.go:60-61`;
`internal/gitrepo/stagetracked_integration_test.go:3-4, 18-19, 46-47`;
`internal/gitrepo/doc.go:243-244` (the new sentence `Pull stays` / `CLI-bound.`);
`internal/loomcli/driverprompt_test.go:122-124`;
`internal/loomcli/wiring_test.go:283-284`;
`internal/mergeresolve/deps.go:28-29, 31-32`;
`internal/mergeresolve/mergeresolve.go:50-51, 123-124, 133-135`;
`internal/shuttleengine/claudeengine/settings_test.go:471-474`;
`internal/standalonegeom/webstergeom.go:30-31`;
`internal/webstercli/recoverbatch.go:1-2`;
`internal/websterengine/config.go:42-43`;
`internal/websterengine/render.go:322-326`;
`internal/websterengine/runlevel_test.go:2777-2778`;
`internal/websterengine/template_test.go:1209-1210`;
`internal/websterengine/webstergeom_test.go:184-185`.
Not flagged: diff-written lines inside legacy paragraphs whose sentence runs on into untouched lines (`fabricengine/doc.go:1105`, `gitrepo/doc.go:133`, `warpforward_integration_test.go:4-5`, `integration_driverbootstrap_test.go:45-47`, `loomcli/wiring.go:559-560, 672-673`, `loomshed/webster.go:89-90`, `mergeresolve/spec.go:36`, `webstercli/cli.go:112`, `websterengine/doc.go:17-18`, `websterengine/recoverbatch.go:12-13`, `render.go:41-42`, `runlevel.go:1276-1277`, `state.go:239`).

**Fix:** Rejoin each listed block to one sentence per line, with a further break at each independent-clause boundary, touching only the lines the diff wrote.

### [NIT] Semicolon-joined independent clauses in new markdown lines

**Location:** contracts/specs/webster-spec.md:53

**Issue:** CLAUDE.md requires semantic line breaks in every `.md` file per `scribe:prose`, which breaks after a semicolon that joins independent clauses.
These new lines keep both clauses on one line:
`contracts/specs/webster-spec.md:45, 53, 56`;
`CONSTRAINTS.md:473`;
`contracts/specs/loom-plan-spec.md:265`;
`contracts/stencils/webster/webster-body-verify-fix.md:29`;
`contracts/stencils/webster/webster-template-master.md:141, 143`;
`docs/overview.md:342`;
`CLAUDE.md:50`.

**Fix:** Break each line after the semicolon, keeping list-item continuation indentation where the line sits in a list item.
