---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: handler
    location: internal/hubgeom/beammodes.go#readBeamMode
    summary: A beam whose worktree is absent on the rendering machine gets its parent's values on its mode line, shown as inherited, though the beam's runs on its owner machine read its own beam.yaml; the committed README then disagrees with what a run does and overwrites the owner's correct line.
    fix: "In readBeamMode, stat fabricengine.WorktreePath(prime, slug) before building the chain; when it is fs.ErrNotExist, return an error naming the beam and saying its worktree is not on this machine, so BeamModes leaves the beam out of Beams and its sections show no mode line. Drop the now-dead not-exist branch before lymcwd.NamedWorktree. Update the BeamModes doc comment (replace the sentence that the beam is left out of its chain so its parent's value shows), the BeamModes paragraph in internal/hubgeom/doc.go, and the Open risks bullet in _lym/landing/summary.md. Add to TestBeamModes a second slug in the beams argument with no worktree, and assert it is absent from the Beams map."
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/boardengine/beammodes.go#modeHeader
    summary: The Beam PR link follows the landing record, which keeps a pull request after GitHub closes it (Finalize halts without dropping it) or merges it before the beam's Finalize runs, while the entry asks for the beam's open pull request and the README gives no hint that the link may be spent.
    fix: "Add one legend line to modeHeader saying that Beam PR links the pull request the beam's landing record names, and that the link stays until the beam lands, even after GitHub closes or merges it. Say the same in the RenderToDisk paragraph of internal/boardengine/doc.go."
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/hubgeom/beammodes.go#primeOfBoard
    summary: primeOfBoard calls dotgit.ListWorktrees directly, which answers ErrRefStorage for a records repository whose refs are not files, so every render on such a hub drops all mode lines with a warning; fabricengine.List already falls back to porcelain for that case.
    fix: "In primeOfBoard, list through fabricengine.List(boardDir) and take the entry whose Main is true as recordsMain, as hubgeom.ParentOf already does, instead of dotgit.ListWorktrees; drop the dotgit import from beammodes.go."
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/dobbyengine/config.go:58
    summary: The Config doc comment still says the per-task keys are read through LookupPullRequest, LookupLandNotice and LookupAutonomy alone, though the diff added the exported LookupWithSource as a reader and updated the same list in doc.go.
    fix: "Change the line to: They are read through LookupPullRequest, LookupLandNotice, LookupAutonomy and LookupWithSource alone."
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/boardengine/beammodes_test.go#TestRenderToDisk_ShowsTheInstalledBeamModes
    summary: No row pins sourceText naming a beam's handle for a value inherited from another beam, or the code-span slug for a worktree with no board entry, which is how the added decision's inherited marking reads for nested beams.
    fix: "Add a row whose beam-b PullRequest is SettingValue{Value: \"merge\", SetBy: \"beam-a\"} and expect the line \"Autonomy **off** · PR setting _merge_ (P1) · Parent branch `beam-a`\"; and a row whose beam-b PullRequest has SetBy \"side-pair\", a slug on no board entry, expecting \"PR setting _merge_ (`side-pair`)\" in that line."
  - id: F6
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/boardengine/beammodes_test.go#TestRenderToDisk_ShowsTheInstalledBeamModes
    summary: No test reads project-<slug>.md or project-root.md, so the mode line fileHeader adds to each project file is unchecked; the mutation report's survivor at internal/boardengine/render.go:423 confirms it.
    fix: "In the test's first row, also read project-beam-a.md and project-root.md from dir and assert each holds its project's mode line as a whole line, the same lines the README row expects for beam-a and the root."
  - id: F7
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/boardengine/beammodes_test.go#TestRenderToDisk_ShowsTheInstalledBeamModes
    summary: No test pins that readBeamModes hands the reader only open beams, leaving out a done beam and a non-beam project, or that a reader error renders the README with no hub-wide header and no mode line.
    fix: "Add a done beam project and a non-beam project to the fixture tasks and keep asserting gotBeams equals [beam-a beam-b]; add a row whose reader returns an error and assert the README holds no line starting with \"Hub-wide default:\" and no line starting with \"Autonomy \"."
---

The diff does what the amended board entry asks for:
the root's and each open beam's autonomy and pull-request setting, read through the per-task lookup from the beam's worktree up,
the parent branch and the recorded pull request, the hub-wide default in the README header,
and the added decision's own-versus-inherited marking, met with bold, italic and a bracketed source plus a legend.
No finding blocks.
The findings below concern a wrong value on a machine that lacks a beam's worktree, the spent pull-request link, one lister choice, one stale doc line and test gaps.

### [MEDIUM] A beam absent on the rendering machine shows its parent's values

**Location:** `internal/hubgeom/beammodes.go#readBeamMode`

**Issue:** `TaskSettingChain` leaves an absent worktree out of the chain, which is right for a run on this machine.
But no run of that beam runs here, since its worktree is on its owner machine.
So the render writes the parent's value marked as inherited from the parent, which is not what the beam's runs read when the beam sets its own value.
The board files are committed and shared, so such a render replaces the owner machine's correct line.
That defeats the entry's aim that the README never disagree with what a run does.
The summary names the risk under Open risks; leaving the beam's line out removes the wrong value instead.

### [LOW] The Beam PR link can name a closed or merged pull request

**Location:** `internal/boardengine/beammodes.go#modeHeader`

**Issue:** `LandingPullRequest` holds no state.
`recordedBeamLanding` halts on a closed pull request without dropping it, and the drop runs only after a completed landing (`internal/landingshed/finalize.go:421`).
Reading GitHub during a render is ruled out by design, which the summary states.
The remaining gap is that the README presents the link as the beam's pull request with nothing telling the reader it may be spent.

### [LOW] primeOfBoard bypasses fabric's worktree lister

**Location:** `internal/hubgeom/beammodes.go#primeOfBoard`

**Issue:** `dotgit.ListWorktrees` refuses a repository whose refs are not files.
`fabricengine.List` falls back to porcelain for that case (`internal/fabricengine/worktreelist.go:35`), and `hubgeom.ParentOf` already lists through it.

### [LOW] Stale list of the per-task readers in dobbyengine's Config doc

**Location:** `internal/dobbyengine/config.go:58`

**Issue:** The line says the three typed lookups are the only readers.
The diff updated the same list in `internal/dobbyengine/doc.go` but left this one.

### [LOW] Inherited-from-a-beam and unknown-slug sources are unpinned

**Location:** `internal/boardengine/beammodes_test.go#TestRenderToDisk_ShowsTheInstalledBeamModes`

**Issue:** The fixtures only inherit from `P0` and the hub-wide default.
`sourceText`'s handle lookup and its code-span fallback (`internal/boardengine/beammodes.go`) run in no test.

### [LOW] The project files' mode line is unchecked

**Location:** `internal/boardengine/beammodes_test.go#TestRenderToDisk_ShowsTheInstalledBeamModes`

**Issue:** The test reads `README.md` alone.
A nested beam has no README section, so its project file is the only place that shows its mode line, and dropping that line passes every test.

### [LOW] The beam filter and the failed-read path are unchecked

**Location:** `internal/boardengine/beammodes_test.go#TestRenderToDisk_ShowsTheInstalledBeamModes`

**Issue:** The fixture holds two open beams only, and the reader never fails.
The filter in `readBeamModes` and its documented fallback, no header and no mode line with a warning, are both untested.

## Rejected

- code-scope's finding 2, that the project-file mode line is unrequested: a nested beam has no README section and shows only in its parent's project file, and the project files are the README's split view, so the line is what lets every beam show its values.
- code-scope's finding 3, that the hub-wide header is unconditional: the header line is true whatever the beams set and carries the legend the added decision asks for. A setting every place takes from the hub-wide default is still left off the mode lines, which is what the entry asks.
- code-tests' finding 5 is folded into F1, whose fix adds the absent-worktree row; the other paths it lists (no origin record, no pull request in the record) are plain zero values the existing happy-path assertions already compare against.
- code-tests' finding 6: `Names` is appended in lockstep with `Anchors` on both branches. The `primeOfBoard` refusal is an internal guard that only logs a warning, not a trust boundary.
- Merge danger: the README and project files are read only by people, and no other binary parses them. The two-way door and the `board` blast radius hold. Differing renders across machines are covered by F1.
