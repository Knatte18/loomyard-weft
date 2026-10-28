```yaml
lens: affected-set
outcome: reported
```

Task: affected-set lens. For each change a card makes, I listed what reads or asserts it and which cards cover that.

**Affected sets and covering cards**

- **Removed Finalize reroute on a conflict or large merge (D1).**
  - Readers: `finalize.go` (`Call` doc, `mergeParent`, `mergeIntoTask`), `finalize_merge_test.go`, `finalize_test.go` retry test, `finalize_integration_test.go:182-184`, the recipe yaml comments in loom, darn and pawl, `docs/overview.md:568`, the orch role stencil (`Landing halts` and parent-lock bullets), `landingshed/doc.go:41-46`, `loomcli/doc.go:17` and the `landingdeps.go:99,265` comments. Card 1 covers all of these.
  - Also reading it: `landingshed/doc.go:120` and `loomRerouteCapWayForward` (`loomcli/wiring.go:955`) with its tests (`wiring_test.go:1048-1060`). No card covers these.
- **Removed flags, `taskMerger` and overlap note (D2).**
  - Readers: `deps.go:156-163`, `envkit.go:153`, `landingdeps.go:100,266`, `landingdeps_test.go:93,129-134`, `wiring_beam_test.go:100-130`, `subjects.go:247-250`, `subjects_test.go:145`, `darnwiring.go:238-239`, `wiring_darn_test.go:58-67`, the Code-Review rubric at line 118, `landingshed/doc.go:41,46`.
  - `writeBytesAtomic` still has readers in `mergerecord.go` and `fixerattempts.go`, and `reviewInputFile` still has readers in `subjects.go:236-237`, so neither is orphaned.
  - Card 1 covers the whole set.
- **Escalate halt text and prefix (D3).**
  - Readers: `haltReason` and its three callers; tests `fixerrun_test.go:178,305-318`, `finalize_gate_test.go:383,428`, `parentalign_fixer_test.go:184,387,445` (still valid with the field false), `handback_test.go`, `arbiterstop_test.go:32`.
  - Docs: `landingshed/doc.go:108,117`, the landing fixer report spec at line 26, `docs/overview.md:569`, the orch role stencil at line 129, the loom status spec's `hand_back` section, `arbiterstop/doc.go`. Card 2 covers all of these.
  - `landing-template-fixer.md:29` says nothing about where an escalate goes, so it is unaffected.
- **Batten re-step `HandBack` (D3).**
  - Readers: `battencli/wire.go:1065-1068`, `battenshed/doc.go:145`, `TestWire_EverySeamFilled`. Card 2 covers these.
  - Also reading it: the `InnerRunDeps.HandBack` field doc (`battenshed/deps.go:293-294`) and the `binaryRestep` doc (`innerrun_binaryrestep.go:53`). No card covers these.
- **`JudgesFirst` signature (D4).**
  - Readers: `drive.go:111-112,713-714`, `drivewiring.go:133`, `drive_test.go:736`, `drivewiring_test.go:74-95`, `sheddriver/doc.go:38,60,72`, `PATTERN.md`. Card 3 covers the whole set; no other setter exists.
- **`restep_hold` key (D3).**
  - Readers: `spec.go`, `status.go:187`, `loop.go:335`, `loopenvelope.go:46,64`, `shedverbs/doc.go:51-53`; closed key-set tests `loop_helper_test.go:28`, `shedcli/cli_test.go:121-127`, `loomcli/status_test.go:96`; `status_test.go:474,507`, `loop_test.go:243-288`; driver stencil lines 97, 100, 101 and 144; guide lines 67-68; the loom status spec.
  - Card 4 covers the whole set.
  - `sheddriver.parseDriveStop` ignores unknown keys, so no change is forced there.

**Findings**

1. **LOW, consistency** — card 1, Docs.
   - Card 1 covers only part of the D1 doc set: `internal/landingshed/doc.go:120` still says "a loom conflict, a large merge or a spent or over-cap fix at Finalize routes back through the recipe's reroute edge".
   - Card 1's Docs names only the Finalize merge paragraph. Card 2's edit to the HandsBack paragraph is about the escalate and does not say it drops the conflict and large-merge causes.
   - Evidence: `internal/landingshed/doc.go:120`.
2. **LOW, consistency** — card 1, `internal/loomcli#loomRerouteCapWayForward`.
   - The reroute-cap way forward still names "for a parent-branch reroute, once the parent branch has settled". After D1 nothing reroutes on a parent-branch cause, so the clause is dead.
   - Two tests pin the clause, and no card lists the constant or the tests: `TestLoomRerouteCapWayForward_IsRowNeutral` and `TestLoomRerouteCapWayForward`.
   - Evidence: `internal/loomcli/wiring.go:955`, `internal/loomcli/wiring_test.go:1048-1056,1059`.
3. **LOW, consistency** — card 2, batten `HandBack` seam.
   - Card 2 widens what `InnerRunDeps.HandBack` marks to include the escalate halt, but updates only `battenshed/doc.go`.
   - The field doc still says it "reports whether the child's blocked status is a landing halt only the orch's resume lifts". The `binaryRestep` doc still reads "is not a landing halt that deps.HandBack marks for the orch".
   - Missed members: `internal/battenshed#InnerRunDeps` and `internal/battenshed#innerRunProducer.binaryRestep`.
   - Evidence: `internal/battenshed/deps.go:293`, `internal/battenshed/innerrun_binaryrestep.go:53`.

Tool calls:
1. Grep for readers of the reroute reason text, `writeBytesAtomic`, `reviewInputFile` and `JudgesFirst`: the manifest had no full reader search for the reroute strings or helper callers.
2. Grep for escalate wording in the fixer spec, orch stencil, overview, status spec and doc.go files: the manifest lacked the escalate-to-orch docs.
3. Grep for the escalate wording in stencils and for arbiter-first docs: to find the fixer stencil and any arbiter-first wording the manifest did not cover.
4. Grep for the fixer stencil's escalate line and `hand_back` mentions in the docs: to confirm doc readers of the `hand_back` and escalate route.
5. Grep in `specs_test.go`, the overview's arbiter lines and the driver guide: to check for spec-content tests and the guide's `hand_back` lines.
