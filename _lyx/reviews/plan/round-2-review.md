---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    location: _lyx/plan/02-logger-config-module.md
    summary: Card 2 claims a float fails the int unmarshal, but yaml.v3 v3.0.1 truncates 1.5 to 1 and 14.0 to 14 without error, so the stated mechanism does not carry D5's non-integer rejection
  - id: F2
    severity: LOW
    location: _lyx/plan/05-exit-sweep.md
    summary: Card 5's logger.yaml fixtures name only trace_retention_count, so a count-only fixture fails load on the missing-key check and the count-0 case can pass for the wrong reason
  - id: F3
    severity: LOW
    location: _lyx/plan/02-logger-config-module.md
    summary: Card 2's per-key rejection tests neither require the other key present nor assert the error names the offending key, so a missing-key error satisfies them vacuously
---

### [MEDIUM] Float values pass card 2's unmarshal

**Location:** `_lyx/plan/02-logger-config-module.md`, `config.go` bullet ("unmarshals both keys into `int` fields … A non-integer value (a float, a string) fails the unmarshal and returns that error") and the `config_test.go` list.

**Issue:** D5 and the acceptance criterion "`logger.yaml` rejects zero, negative, and non-integer values for both keys" need a float rejected.
The pinned `gopkg.in/yaml.v3 v3.0.1` decodes a `!!float` into an `int` field with `out.SetInt(int64(resolved))` and no error (`decode.go`, the `reflect.Int` case's `float64` branch).
`yamlengine.Resolve` re-emits the scalar unchanged, so `LoadOrTemplate` hands the float through.
A probe run against this repo's own `yamlengine.Resolve` plus `yaml.Unmarshal` into an `int` field confirmed it:
`1.5` → `1`, `14.0` → `14`, `1e3` → `1000`, all with a nil error; only a string (`abc`, `"7"`) or a bool errors.
So the card's mechanism rejects strings but accepts floats, and the test list's generic "non-integer values" can be met by a string-only fixture, leaving `trace_retention_days: 1.5` loading as 1 day.
If the implementer does write a float fixture, the card's own mechanism fails it and the implementer must invent the rejection unguided.

**Fix:** State a mechanism that rejects a float: decode each key into a `yaml.Node` field and require a scalar whose `ShortTag()` is `!!int` before decoding it into an `int`, returning an error naming the key otherwise.
Name the float cases in the test list: `1.5` and `14.0` for each key each return an error naming the key, alongside a string value.

### [LOW] Card 5's logger.yaml fixtures do not state both keys

**Location:** `_lyx/plan/05-exit-sweep.md`, the `main_integration_test.go` cases and the `exitsweep_test.go` valid/invalid cases.

**Issue:** `configengine.load` runs `yamlengine.MissingKeys(template, fileBytes)` before resolving and errors on any absent template key.
The first integration case specifies a `logger.yaml` "setting `trace_retention_count: 2`" and the second "`trace_retention_count: 0`", without saying the file also carries `trace_retention_days`.
A count-only fixture makes the first case's load fail on the missing key, so the exit hook warns and sweeps under the 200-trace default, and the "exactly two seeded traces survive" assertion fails for a reason unrelated to retention.
The second case passes with a count-only fixture, but through the missing-key error rather than the zero-value rejection it is meant to exercise; its "Warn line naming `logger.yaml`" assertion cannot tell the two apart.
The `exitsweep_test.go` case "whose anchor holds a valid `logger.yaml`" has the same gap.

**Fix:** State that every `logger.yaml` fixture carries both keys: the valid ones set `trace_retention_days` to a value that keeps the seeded traces inside the age bound, and the invalid ones change exactly one key from a valid file.
Have the count-0 integration case assert that its Warn line names `trace_retention_count`.

### [LOW] Card 2's rejection tests can pass on a missing-key error

**Location:** `_lyx/plan/02-logger-config-module.md`, `config_test.go` bullet "Zero, negative, and non-integer values for each key each return an error".

**Issue:** The case asserts only that an error is returned.
A fixture carrying just the key under test fails `configengine.load`'s missing-key check first, so every rejection case passes whether or not `Load` validates the value at all.
The card's own `Load` bullet already says the zero/negative error names the key, but no test holds it to that.

**Fix:** State that each rejection fixture is a valid file with exactly one key changed, and that each case asserts the error names that key.

## Focus-directive verdicts

- Card 2's float rejection: the stated mechanism does not reject `1.5` or `14.0` for either key, and the test list does not force a float fixture, so together they do not carry D5's non-integer rejection (F1).
- Card 5's `logger.yaml` fixtures: none is specified to carry both template keys; the count-2 case would fail and the count-0 case would pass for the wrong reason (F2).
  The same gap sits in card 2's own rejection tests (F3).
