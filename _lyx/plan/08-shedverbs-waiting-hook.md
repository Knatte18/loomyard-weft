# Card 8 — A told waiting note on the generic status verb

**Edit:**
- `internal/shedverbs#Hooks`
- `internal/shedverbs#statusCmd`
- `internal/shedverbs#runStatusWatch`
- `internal/shedverbs#RenderStatusHuman`
- `internal/shedverbs/status_test.go`
- `internal/shedverbs/human_test.go`

**Intent:**
Give the generic `status` body the told hook decision D9 asks for, so loom can report a run waiting on its reviewer without reimplementing `status` (Shed Verb-Set Invariant).

`Hooks` gains `Waiting func() (string, error)`: a module-told one-line note naming what a running run waits on, nil-by-default and skipped when nil.
`statusCmd` calls it only when the status file's state is `running`; a non-empty note adds a `waiting` key to the envelope, and an error is reported verbatim on the error envelope like `StatusExtras`.
`RenderStatusHuman` takes the note and prints `waiting` with the note in its header in place of the plain `running` state when the note is non-empty.
`runStatusWatch` appends ` | waiting <note>` to the polled line while the note is non-empty, so the line changes when the wait starts or ends.
`shedverbs` stays path-free: the hook closes over whatever location the arming module was told.

Tests: the envelope carries `waiting` only for a running state with a non-empty note, the hook's error reaches the error envelope, the human header swaps state for the note, and the watch line includes it.

**ImpactSummary:** One new optional hook and one new envelope key in `shedverbs`; with the hook nil every envelope and rendering is byte-identical to today, and `RenderStatusHuman`'s only caller is `statusCmd`.
