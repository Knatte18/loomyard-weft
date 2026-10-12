# Card 5 — validate-plan-overlay-flag

**Create:**
- `plan:internal/loomcli#readOverlay` -> `func readOverlay(in io.Reader) (map[string]string, error)`
- `plan:internal/loomcli#inputIsTerminal` -> `func inputIsTerminal(in io.Reader) bool`
- `plan:internal/loomcli#errOverlayNoInput` -> `var errOverlayNoInput error`
- `plan:internal/loomcli#errOverlayMalformed` -> `var errOverlayMalformed error`
- `plan:internal/loomcli#overlayRefusal` -> `func overlayRefusal(err error) string`
- `plan:internal/loomcli#TestValidatePlanCmd_Overlay` -> `func TestValidatePlanCmd_Overlay(t *testing.T)`

**Edit:**
- `internal/loomcli#loomCLI.validatePlanCmd`
- `internal/loomcli/doc.go`
- `docs/overview.md`
- `contracts/specs/refusal-spec.md`
- `pattern/PATTERN-gate-self-check-parity.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
Add `lyx loom validate-plan --overlay`: validate the plan with proposed contents of existing plan files read from stdin, in memory, writing nothing (Decision 5, the verb).

`readOverlay`, `inputIsTerminal`, the two sentinels and `overlayRefusal` live in a new file `internal/loomcli/overlay.go`.
`inputIsTerminal` reports whether `in` is an `*os.File` whose mode has `os.ModeCharDevice`, checked before any read so a terminal never blocks the verb.
`readOverlay` reads all of `in` and decodes it with a `json.Decoder` token by token: empty or whitespace-only input is `errOverlayNoInput`; input that is not exactly one JSON object (invalid JSON, an array, a scalar, a second value after the object), a name that occurs twice, or a value that is not a string is `errOverlayMalformed`, naming the offending name where there is one.
`overlayRefusal` words each refusal as `loom: validate-plan --overlay: <cause>; way forward: <step>`:
no input — pipe one JSON object mapping plan file names to their full proposed content on stdin, as `lyx loom validate-plan --help` shows;
malformed — the same, with distinct names and string values;
`planparser.ErrOverlayName` — name only `00-overview.md` or a card file the Card Index lists, and add or remove a card with `lyx loom insert-card` or `lyx loom remove-card`;
`planparser.ErrOverlayFileMissing` — write the card file the Card Index names, then re-run.
Any other error of `NewOverlay` or `Overlay.Parse` is a plan that does not parse, reported through the verb's existing "loom: parse plan at" refusal.

`validatePlanCmd` gains `--overlay`.
It is mutually exclusive with `--rework` and with `--require-approved`, refused before stdin is read with `loom: validate-plan: --overlay and --<flag> are mutually exclusive; way forward: run --overlay alone, which runs the flag-less check set`.
With the flag, the verb refuses a terminal stdin as no input, reads the overlay with `readOverlay` from `cmd.InOrStdin()`, builds it with `planparser.NewOverlay(planDir, files)`, parses with `Overlay.Parse`, and runs `loomshed.ValidatePlan` exactly as the flag-less mode, reporting through the same envelope; a parse error of the overlaid plan is the existing "parse plan" refusal.
It validates through `c.env.PlanIndex`, as the flag-less mode does, so in a hub its type load waits for a gate slot.
It is the flag-less mode's check set and the same functions the Plan-Write gate calls, so `PATTERN-gate-self-check-parity` holds; `{}` reports what the flag-less run reports.
What the flag lets through, and its bound: it runs every check, skips none and writes no file, and it cannot change the tree the glyphs resolve against, so it validates target-list, verify-command and card-text changes, never a proposed code change; it adds, removes or renames no plan file, so a card insert, remove or renumber still goes through `insert-card` and `remove-card`.
The help text documents the flag, the stdin format (one JSON object, keys the file names, values their full content), the accepted names, the in-memory canonicalization, the exclusions, and an example piping a heredoc: `lyx loom validate-plan --overlay <<'EOF'` … `EOF`.
A heredoc into `lyx` is no in-place edit, so the way forward names no denied command (`PATTERN-no-denied-recovery`).

`doc.go` gains the verb's overlay mode in the module doc, and `docs/overview.md`'s line on `validate-plan`'s modes gains `validate-plan --overlay`.
`refusal-spec.md`'s `## loom` section gains, each with the way forward the message quotes: "validate-plan --overlay: no input" (stdin a terminal or empty), "validate-plan --overlay: malformed overlay" (not one JSON object, a duplicate name, a non-string value), "validate-plan --overlay: not a plan file" (a path separator, `..`, a non-`.md` name, `amendments.md`, an orphan `.md` file), "validate-plan --overlay: card file missing" and "validate-plan --overlay: flag combination".
`PATTERN-gate-self-check-parity.md` names `--overlay` among the verb's modes as a self-check mode no gate runs, calling the flag-less mode's function over an overlaid plan.

`TestValidatePlanCmd_Overlay` is a new top-level test, so `internal/loomcli`'s row in `test-budget.yaml` rises by one; it sets the command's input with `SetIn` and snapshots every file under the plan directory before and after each row.
Rows:
`{}` exits as the flag-less run does with the same envelope;
an overlay replacing card 1's target list with one naming `sub#Missing` reports `glyph-not-found`, and the plan files are byte-identical;
an overlay whose card 1 declares `plan:sub#Draft` -> `func Foo() {}` while `sub/a.go` declares `Foo` reports `create-already-exists` for the canonical `sub#Foo`, and the plan files are byte-identical;
with a done batch recorded at the on-disk card's own hash, an overlay of that card reports `done-card-edited`;
with no plan on disk, `{}` reports the existing parse refusal;
and one refusal row each, asserting exit 1, the cause and its way forward: stdin opened from `os.DevNull` (a character device), empty stdin, `[]`, `{"a":"x"} {}`, a duplicate name, a non-string value, `amendments.md` for `ErrOverlayName` (Card 3's `TestOverlay` pins the other refused names onto the same sentinel), an overlay of a Card Index card whose file was removed from disk, `--overlay --rework` and `--overlay --require-approved`.

**ImpactSummary:** one verb gains a flag; the flag-less, `--rework` and `--require-approved` modes and their existing refusal keep their behavior.

**Commit:** 5: validate-plan --overlay checks proposed plan contents in memory
