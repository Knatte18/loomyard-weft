```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. For each thing the diff changes, I looked for everything that calls, reads or asserts it and checked whether the diff left any of it wrong.

**Affected set, per change**

- **`rememberRuns` signature** (the `members` parameter is gone): its only caller is `landNotices` at `internal/dobbyengine/landnotice.go:62`, which the diff updates. Covered.
- **`LandNoticeState.Member` and the `member` JSON key** removed: no Go reader is left (grep for `.Member` in `internal/dobbyengine` is empty).
  - An older state record still loads. `State` decodes through `internal/state`, which keeps keys its type does not know, and no `DisallowUnknownFields` call is involved (`pattern/PATTERN-tolerant-records.md:3-6`).
  - The summary's claim that the key is ignored holds. Covered.
- **`isBeamMember`, `landNoticeTasks`, `daemon.landNoticeBoard`** removed: no remaining references in the package. The board read `Deps.Tasks` is still used by the start step, which the removed test comment confirms, so the seam stays. Covered.
- **Line-decision behaviour** (a member gets a line only when its `land_notice` is on):
  - The package doc (`internal/dobbyengine/doc.go:144-151`), the `landnotice.go` header, the doc comments of `landNotices`, `rememberRuns` and `pullRequestFollowUp`, and the `Deps.PullRequestRecorded` doc comment in `loop.go` are all updated. The `doc.go` lines that the diff leaves alone (a done entry owes none by itself; a run first seen past Land gets no line) still hold.
  - `internal/dobbyengine/template.yaml:6` describes `land_notice` without the exception.
  - `internal/beamsetting/beamsetting.go` and `internal/dobbyengine/config.go` comments are unaffected.
  - `internal/beamcli/cli.go:341` (`beam status` help, "whether it owes a landing notice") is unaffected.
  - Covered.
- **Rule documents**:
  - `pattern/PATTERN-landing.md` has no mention of the exception.
  - `pattern/PATTERN-config-layers.md:119` names only the lookup rule, which the change follows.
  - The directive declares no PATTERN changes, so "Declared PATTERN changes landed" raises nothing.
  - Covered.
- **Orch contract `contracts/orchrole/role-beams.md:55-56`**:
  - The old "whatever its `beam.land_notice` reads" is gone, and the new text says a member gets the notice only when `land_notice` reads on, which is what the entry asks for.
  - The rule to run `lym beam describe &lt;slug&gt;` when the notice asks is kept.
  - No other stencil or contract mentions the landing notice: a grep over `contracts` and `stencils` finds only these lines.
  - Covered.
- **Commit messages and departures**:
  - There are no `Pattern-Departure:` or `Loosened-Check:` trailers.
  - The `Root-Cause:` trailer on `09d908924` names the defective condition in `rememberRuns`, which the diff removes.
  - Covered.
- **`docs/research/loom/lym-state-map-2026-11-15.md:144`** mentions `land_notices` state. It is a dated research snapshot and exempt as a measurement record. Not affected.

**Findings:** none.

**Tool calls**

1. Grep of all `.md` files and of `internal/dobbyengine` for leftover member, landing-notice and `rememberRuns`/`.Member` wording: the manifest's grep did not cover all `.md` files or the Go callers.
2. Grep of `beamsetting`, `dobbyengine` config and template, `beamcli`, `docs`, `specs` and `stencils` for `land_notice` and recorded-pull-request wording: setting docs and CLI help were not in the manifest.
3. Read of `internal/beamcli/cli.go:330-350`, `describe.go` and the `contracts`/`stencils` mentions: needed the context of the one CLI help hit.
4. Read of `pattern/PATTERN-tolerant-records.md` and the `LandNotices` field in `internal/dobbyengine/state.go`: needed to confirm a state record that still holds `member` loads and keeps the key.
