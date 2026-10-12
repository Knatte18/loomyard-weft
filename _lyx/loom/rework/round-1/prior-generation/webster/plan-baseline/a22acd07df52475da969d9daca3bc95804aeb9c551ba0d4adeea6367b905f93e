# Card 4 — alignment-record-readers

**Create:**
- `plan:internal/alignment#ParseAttemptDir` -> `func ParseAttemptDir(files map[string][]byte) (Attempt, *Verdict, error)`
- `plan:internal/parentreview#RoundNumber` -> `func RoundNumber(dirName string) (int, bool)`
- `plan:internal/parentreview#ParseRoundDir` -> `func ParseRoundDir(number int, files map[string][]byte) (Round, error)`
- `plan:internal/parentreview#TestParseRoundDir` -> `func TestParseRoundDir(t *testing.T)`

**Edit:**
- `internal/alignment#TestAttempts`

**Intent:**
D6 counts parent-review wakes and the superseding alignment approve, D9 counts parent-review rounds, and D12 reports alignment outcomes per point and verdict.
Both formats get the tolerant parse-from-bytes function D5 places in the owning package, taking a directory's files keyed by file name so the file names stay private to their owner.

`alignment.ParseAttemptDir` (in `records.go`) reads one attempt directory: `attempt.json` decoded with `json.Unmarshal`, then the verdict file the attempt's `Token` names (`verdict-<token>.yaml`), decoded by `yaml.v3` without `KnownFields`.
It returns a nil `*Verdict` when that file is absent, runs none of `ReadVerdict`'s checks (verdict word, reason, question, superseded, report file), leaves `Attempt.Dir` empty and ignores every other name.
A missing or undecodable `attempt.json` and an undecodable verdict file are each an error naming the file.
`ReadAttempt`, `ListAttempts` and `ReadVerdict` are unchanged.

`parentreview.RoundNumber` (in `store.go`) parses a round directory name, `round-<N>` with `N` a positive decimal, the inverse of `Store.roundDir`'s naming; anything else is false.
`parentreview.ParseRoundDir` returns the `Round` for one round directory's files: `Number` set, `Dir` empty, `Request`, `Verdict` and `Alignment` non-nil only when `request.json`, `verdict.json` and `alignment.json` are among the files, and `Delivery` decoded from `delivery.json` when present.
Each file decodes with `json.Unmarshal`, so an unknown key reads; a file that does not decode is an error naming it.
`Store.load` and `readHeldJSON` are unchanged.

Tests:
- `TestAttempts` gains a subtest over `ParseAttemptDir` with rows: an attempt at `pr-gate` with `applied: approve` and an `approve` verdict listing two superseded slugs and carrying an unknown `extra:` key reads all of them; an attempt with no verdict file returns a nil verdict; an `escalate` verdict with no question reads, though `ReadVerdict` refuses it; an `attempt.json` of `{` is an error naming `attempt.json`.
- `TestParseRoundDir` (new, in `store_test.go`) is table-driven: a round with `request.json`, a `delivery.json` carrying `delivered_at`, `pings: 2` and an unknown key, an `alignment.json` in state `fallback` and a `verdict.json` of kind `reject` reads every field; a round holding only `request.json` has nil `Verdict` and `Alignment` and a zero `Delivery`; a `verdict.json` of `{` is an error naming `verdict.json`.
  Its rows for `RoundNumber`: `round-1` and `round-12` parse, `round-0`, `round-`, `round-1a` and `brief.md` do not.

**ImpactSummary:** One alignment test gains a subtest; every production change is a new member, so no live reader in `alignment` or `parentreview` changes what it accepts.

**Commit:** `4: tolerant alignment and parent-review readers`
