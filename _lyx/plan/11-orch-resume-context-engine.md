# Card 11 — orch-resume-context-engine

**Create:**
- `plan:internal/orchengine#ResumeMark` -> `type ResumeMark struct`
- `plan:internal/orchengine#WriteResumeMark` -> `func WriteResumeMark(p Paths, mark ResumeMark) error`
- `plan:internal/orchengine#ReadResumeMark` -> `func ReadResumeMark(p Paths) (ResumeMark, bool, error)`
- `plan:internal/orchengine#ClearResumeMark` -> `func ClearResumeMark(p Paths) error`
- `plan:internal/orchengine#ResumeContext` -> `func ResumeContext(p Paths, stencilsDir string, now time.Time) (string, error)`
- `plan:internal/orchengine#Watcher.hookDelivered` -> `func (w *Watcher) hookDelivered(st State) (ResumeMark, bool)`
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

**Uses:**
- `internal/fsx#AtomicWriteBytes`

**Intent:** The engine half of the resume pointer's hook delivery (D11): the pointer the hook renders, the delivery mark, and the watcher's skip of a pointer the hook already delivered.
`Paths` gains `ResumeMarkPath`, the delivery mark file beside the cycle request; `ResumeMark` holds `At time.Time` and `Text string`, and `WriteResumeMark`, `ReadResumeMark` and `ClearResumeMark` are its JSON codec and removal in the shape of the cycle request's, an absent mark reading as none.
`ResumeContext` renders the pointer from the persisted state: phase `resuming` with a `PendingResume` set returns that text verbatim; phase `compacting` renders the resume prompt naming `State.LastHandoff`; any other phase renders the reload prompt naming the role alone.
In every phase but `resuming` it renders the role file through `RenderRoleFile` first, so the file the pointer names is current when the session reads it; a render failure returns the error and writes no mark.
After a successful render it writes the mark with `now` and the text, then returns the text.
`renderFile` writes through `fsx.AtomicWriteBytes`, a temporary file and a rename, so the verb and the watcher rendering the role file at the same moment never leave a reader a partial file; `RenderRoleFile`'s comment says so, and `seam_enforcement_test.go` admits `internal/fsx`.
`typeReloadStep`'s pointer step calls `hookDelivered` first: a mark dated at or after `State.CompactionBaseline` whose text equals `PendingResume` means the hook delivered the pointer, so the step types nothing, logs `orch: pointer delivered by the session-start hook` at Info with the mark's time, and confirms the step as typed, so the phase ends at the first turn end after the boundary and times out as today; no mark, an older mark or a mark with other text types the pointer as today, logging what was read.
`toIdle` clears the mark when the reload ends, so a mark never carries into the next compaction.
Bound: a mark the watcher has not read, or whose text differs, makes the pointer arrive twice, once by hook and once typed, which costs the session one redundant read and loses nothing.
`resumecontext_test.go` covers `ResumeContext` over seeded stencils: the verbatim `PendingResume` in `resuming`, the resume prompt in `compacting`, the reload prompt in `idle`, `handoff-requested` and `clearing`, the role file rendered before the pointer in every phase but `resuming`, the mark written with the time and text after a success, and no mark and an error on a render failure (a missing stencil).
`state_test.go` covers the mark codec and its absent reading.
`watcher_test.go` adds the draft case: after a compaction boundary, a mark holding the reload prompt and a probe held with a draft reason, then idle, types the color and plugins steps and skips the pointer, ending the phase at the next turn end; and the fallback rows: no mark, a mark older than the boundary and a mark with other text each type the pointer after the plugins step, and the mark is gone once the reload ends.
`doc.go`'s reload section and `.lyx/orch/` layout state the hook delivery, the mark and the skip rule.

**ImpactSummary:** The role render becomes atomic for every caller, the reload's pointer step gains one read of a new mark file, and the new functions have no callers until the next card's verb.
