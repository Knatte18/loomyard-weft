# Card 2 — board-cli-kept-keys

**Create:**
- `plan:internal/boardcli_test#seedBoardFile` -> `func seedBoardFile(t *testing.T, cwd, body string)`

**Edit:**
- `internal/boardcli_test#stepLookupContract`
- `internal/boardcli/doc.go`

**Intent:**
Pin at the CLI surface that a whole printed `Task` carries its kept keys (D4), which card 1's `Task.MarshalJSON` gives every verb that prints one, with no change to `internal/boardcli`'s production code.

`seedBoardFile`, in `cli_test.go`, writes `body` as the board.json in `fabricengine.BoardDir(filepath.Dir(cwd))`, creating the directory.
`stepLookupContract` gains rows whose setup calls `f.reset(t)`, then `seedBoardFile` with a board holding a task `a` labelled `bug` carrying `"zz_kept":{"n":9007199254740993}`, a note `n` labelled `bug` carrying the same key and a task `b` labelled `bug`.
`stepLookupContract`'s doc comment gains a line saying the step also pins that `get`, `list-full` and the task envelope of `upsert`, `promote` and `merge` print an entry's kept keys with exact digits.
The table gains `assertStdout func(*testing.T, string)`, which the runner calls with the raw stdout after its existing checks, since the runner's own plain `json.Unmarshal` has already rounded the number in the map `assertResult` gets, and a merge or promote cannot be rerun on the same state.
Each new row's `assertStdout` decodes stdout with `UseNumber` so the digits stay exact, reads the printed entry's `zz_kept` and requires `{"n":9007199254740993}`:
- `get` of `a`, from the `task` object;
- `list-full`, from `a`'s element of its task list;
- `upsert` of `a`'s title, from the envelope's `task`;
- `promote` of `n`, from the envelope's `task`;
- `merge` removing `b` and upserting `a`'s title, from the envelope's `task`.
Each row reads the envelope key its verb prints today, as the existing rows of that verb in `cli_test.go` do.

`doc.go` gains one line beside the `get --body` paragraph: `get`, `list-full` and the task envelope of `upsert`, `promote` and `merge` print an entry's kept keys, the board.json keys this binary does not know, while `list` and the README use known fields only.

**ImpactSummary:** Test and doc only: the rows extend one existing step of `TestCLI`, add no top-level test, and change no production code.

**Commit:** 2: boardcli: pin that printed entries carry their kept keys
