# Card 3 — loom-record-readers

**Create:**
- `internal/loomengine#LoomFrictionDirRel`
- `internal/loomengine#IsLoomFrictionArchive`
- `internal/loomengine#ArbiterRecord`
- `internal/loomengine#ParseArbiterAttemptDir`
- `internal/loomengine#LandingCheckpointSubject`
- `internal/loomengine#LandingCheckpointSlug`

**Edit:**
- `internal/loomcli#parentRecordsCommitter`
- `internal/loomengine#TestLoomFrictionPaths`
- `internal/loomengine#TestArbiterAttempts`
- `internal/loomengine#TestLandingAndApprovalAccessors`

**Intent:**
`internal/loomengine` owns three record formats the report reads from git trees: the friction directory with its archive siblings (D7), the arbiter attempt directory (D8), and the landing-checkpoint commit subject (D10).
Each reader takes bytes or names and never a path on disk, and the report reaches none of them through `internal/loomcli`.

Friction paths (in `config.go`):
`LoomFrictionDirRel` returns `filepath.Join(LoomDurableDirRel(), frictionDirName)`, the anchor-relative form of `LoomFrictionDir`, in the same separator form as its `Rel` siblings; a git-tree reader applies `filepath.ToSlash`.
`IsLoomFrictionArchive` reports whether a directory name under `LoomDurableDirRel()` is a timestamped reflection-archive sibling of the friction directory: `frictionDirName` followed by `-` and at least one more character, the name `LoomFrictionArchivePrefix` composes.
`friction` itself is not an archive.

Arbiter records (in `arbiter.go`):
`ArbiterRecord` holds one attempt directory's parsed content: `Attempt ArbiterAttempt`, `Verdict *ArbiterVerdict` (nil when no `verdict.yaml`) and `Applied bool`.
`ParseArbiterAttemptDir` takes the attempt directory's files keyed by file name, so the file names stay private to this package.
It decodes `attempt.json` with `json.Unmarshal` and `verdict.yaml` with a `yaml.v3` decoder without `KnownFields`, so an unknown key reads in both (D5), and it runs none of `ReadArbiterVerdict`'s validation.
It sets `Applied` when the `applied` mark is among the names, ignores every other name, and returns an error naming the file for an `attempt.json` that is absent or does not decode and for a `verdict.yaml` that does not decode.
`ReadArbiterAttempt`, `ReadArbiterVerdict` and `ArbiterApplied` are unchanged.

Landing checkpoint (in `config.go`):
`LandingCheckpointSubject` returns `loom: run records checkpoint for landing <slug>`, the subject `parentRecordsCommitter` writes today, and `LandingCheckpointSlug` is its inverse: the slug when the subject has that prefix and a non-empty rest, else false.
`parentRecordsCommitter` builds its commit message with `LandingCheckpointSubject(slug)` instead of its inline `fmt.Sprintf`, so the subject keeps one owner and `TestLandingDeps_ConfigNoticeSeamsOverRealHub`'s literal still matches.

Tests:
- `TestLoomFrictionPaths` adds that `LoomFrictionDirRel()` equals `filepath.Join(LoomDurableDirRel(), "friction")` and is the suffix of `LoomFrictionDir(l)` after `l.AnchorPath()`, and rows for `IsLoomFrictionArchive`: `friction-20261012-120446` and `friction-20261012-120446-2` true, `friction`, `friction-` and `plan` false.
- `TestArbiterAttempts` gains a subtest over `ParseArbiterAttemptDir` with rows: an attempt with a `resume` verdict carrying an unknown `bogus:` key and the `applied` mark reads the verdict and `Applied` true; an attempt with no `verdict.yaml` has a nil `Verdict` and `Applied` false; an `attempt.json` with an unknown key reads its `Number`; a missing `attempt.json` and an `attempt.json` of `{` are each an error naming `attempt.json`; a `verdict.yaml` of `verdict: [` is an error naming `verdict.yaml`.
- `TestLandingAndApprovalAccessors` adds the round trip `LandingCheckpointSlug(LandingCheckpointSubject("my-slug"))` returning `my-slug`, true, and rows where `loom: run records checkpoint for landing ` (empty slug) and `batten: Land -> running` return false.

**ImpactSummary:** One production caller changes its message construction without changing the bytes; every other change is new members and test rows in `internal/loomengine`.

**Commit:** `3: loom record readers for the report`
