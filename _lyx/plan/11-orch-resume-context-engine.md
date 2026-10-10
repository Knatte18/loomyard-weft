# Card 11 — orch-resume-context-engine

**Create:**
- `internal/orchengine#ResumeMark`
- `internal/orchengine#WriteResumeMark`
- `internal/orchengine#ReadResumeMark`
- `internal/orchengine#ClearResumeMark`
- `internal/orchengine#ResumeContext`
- `internal/orchengine#Watcher.hookDelivered`
- `internal/orchengine/resumecontext_test.go`

**Edit:**
- `internal/orchengine#Paths`
- `internal/orchengine#renderFile`
- `internal/orchengine#RenderRoleFile`
- `internal/orchengine#Watcher.typeReloadStep`
- `internal/orchengine#Watcher.toIdle`
- `internal/orchengine/seam_enforcement_test.go`
- `internal/orchengine/state_test.go`
- `internal/orchengine/watcher_test.go`
- `internal/orchengine/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/fsx#AtomicWriteBytes`

**Intent:** The engine half of the resume pointer's hook delivery (D11): the pointer the hook renders, the delivery mark, and the watcher's skip of a pointer the hook already delivered.
`Paths` gains `ResumeMarkPath`, the delivery mark file beside the cycle request; `ResumeMark` holds `At time.Time` and `Text string`, and `WriteResumeMark`, `ReadResumeMark` and `ClearResumeMark` are its JSON codec and removal in the shape of the cycle request's, an absent mark reading as none.
`ResumeContext` renders the pointer from the persisted state: phase `resuming` with a `PendingResume` set returns that text verbatim; phase `compacting` renders the resume prompt naming `State.LastHandoff`; any other phase renders the reload prompt naming the role alone.
In every phase but `resuming` it renders the role file through `RenderRoleFile` first, so the file the pointer names is current when the session reads it; a render failure returns the error and writes no mark.
After a successful render it writes the mark with `now` and the text, then returns the text.
`renderFile` writes through `fsx.AtomicWriteBytes`, a temporary file and a rename, so the verb and the watcher rendering the role file at the same moment never leave a reader a partial file; `RenderRoleFile`'s comment says so, and `seam_enforcement_test.go` admits `internal/fsx`.
`typeReloadStep`'s pointer step calls `hookDelivered` first: a mark dated at or after `State.CompactionBaseline` whose text equals `PendingResume` means the hook delivered the pointer, so the step types nothing, logs `orch: pointer delivered by the session-start hook` at Info with the mark's time, and ends the phase at once through `toIdle` with no abort reason.
The first turn end after the boundary has already been read when the pointer step is reached, the one that revealed an automatic boundary or completed a requested cycle, or the operator's turn a draft held the color and plugins steps behind, so a skip that waited for a turn end past its own offset would wait for a turn that may never come and end in `resume timed out`.
The stored context reading stays as the phase left it, since every idle trigger re-reads a fresh reading before it fires.
No mark, an older mark or a mark with other text types the pointer as today, and the phase ends at its first turn end, logging what was read.
`toIdle` clears the mark when the reload ends, so a mark never carries into the next compaction.
Bound: the skip lets a file under the orch dir suppress the typed pointer, and a mark written for a render whose output the provider then drops skips a pointer the session never got (D11's Bound, resting on the hook semantics in its Auto-mode assumptions); it is bounded by the mark being written only after a successful render, dated at or after the boundary and equal to `PendingResume`.
A mark the watcher has not read, or whose text differs, makes the pointer arrive twice, once by hook and once typed, which costs the session one redundant read and loses nothing.
`ResumeContext` reads the state through `state.ReadJSON` and returns an error with no mark when the state file is absent, since `LoadState` reads an absent state as phase `idle` and would otherwise render the reload prompt.
`resumecontext_test.go` covers `ResumeContext` over seeded stencils: the verbatim `PendingResume` in `resuming`, the resume prompt in `compacting`, the reload prompt in `idle`, `handoff-requested` and `clearing`, the role file rendered before the pointer in every phase but `resuming`, the mark written with the time and text after a success, no mark and an error on a render failure (a missing stencil), and no mark and an error with no state file.
`state_test.go` covers the mark codec and its absent reading.
`watcher_test.go` adds the draft case: after a compaction boundary, a mark holding the reload prompt and a probe held with a draft reason, then idle, types the color and plugins steps and skips the pointer, ending the phase on the skip's tick with no abort reason and no further turn end; and the fallback rows: no mark, a mark older than the boundary and a mark with other text each type the pointer after the plugins step, and the mark is gone once the reload ends.
`doc.go`'s reload section and `.lyx/orch/` layout state the hook delivery, the mark and the skip rule.
`cmd/lyx/testdata/test-budget.yaml` raises `internal/orchengine` by the top-level tests this card adds, on top of card 9's raise.

**ImpactSummary:** The role render becomes atomic for every caller, the reload's pointer step gains one read of a new mark file, and the new functions have no callers until the next card's verb.
