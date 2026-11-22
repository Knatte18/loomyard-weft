# Card 5 — parent-ask-clear

**Create:**
- `plan:internal/burlerengine#ClearParentAsk` -> `func ClearParentAsk(files ParentReviewFiles, number int) error`
- `plan:internal/burlerengine_test#TestClearParentAsk` -> `func TestClearParentAsk(t *testing.T)`

**Edit:**
- `internal/burlerengine#ParentAsk`
- `internal/burlerengine/doc.go`

**Uses:**
- `internal/burlerengine#EndParentAsk`
- `internal/burlerengine_test#recordParentAsks`

**Intent:** D3's resume needs a way to retire an ended judge ask so the gate asks again over the same rulings.

`ParentAsk` gains `Cleared bool` (`json:"cleared,omitempty"`): a cleared ask is ended and answers nothing, and the seat that made it asks again.
The record stays tolerant: an older record reads with `Cleared` false.

`ClearParentAsk` sets `Cleared` on the ask numbered `number` under the record's lock and rewrites the record.
Its one caller passes an ask it found present and ended, so it checks no state;
a record that lacks the number is a caller error, returned naming the number, with no test row.
An ask already cleared is no error.

`doc.go`'s parent-review asks section, which says an ask moves only to an end state, states that an ended ask may be cleared:
a cleared ask answers nothing, and the seat that made it asks again.

`TestClearParentAsk` is tier0, over `recordParentAsks`:

- an ask set timeout through `EndParentAsk` is cleared, and a re-read record shows `Cleared` with the state and reason unchanged;
- clearing it again is no error.

**ImpactSummary:** One new tolerant field and one writer under the existing lock; no current reader reads the field until card 6's gate does.
