```yaml
lens: bound-breaker
outcome: reported
```

Lens: bound-breaker. I attacked the record's bounds, guarantees and claims about existing code and found one break, plus two narrower ones.

**Claims attacked**
- D1, anchor ends the worker mid-step (`runproc.EndSparing` spares only push and sync): held. `internal/fabriccli/anchor.go:468`, `internal/runproc/end.go:67`.
- D1, no other path ends hub-owner processes: held. `tenderengine` reap ends only orphans, and `orphanOwner` never counts `HubOwner` as one (`internal/tenderengine/processes.go:119`). The `beam stop` and slack paths end one role only.
- D1, a lost-lock caller follows only a record whose start is later than its own: held. In the narrow race where the winning worker started before the caller, the caller exits non-zero naming the re-run, which is bounded.
- D1, a direct `lym adopt worker` call is bounded by the same validation: partly broken (F3).
- D2, `--discard` against a starting worker: held as a mechanism. The acceptance wording is wrong for one interleaving (F2).
- D2, a resume at `owner` or earlier re-runs the reconcile; a third machine that adopted since is then taken over by an explicit operator call: held.
- D2, a resume past `owner` reads no origin; a third machine's later adopt meets a beam step whose `BeamUp` reads the local Bolt: held. D2's own bound and D4's "handled as today and by D11" cover it.
- D4, an unlocked read of the record through `internal/state`'s atomic replace: held.
- D4, an orch watcher or dobby started by an older binary: held. Both re-exec onto a replaced binary (`internal/orchengine/watchloop.go:221`, `internal/dobbyengine/loop.go:63`).
- D5, "only `lym tender resume` or `lym batten cancel` lifts it": held.
  - The dobby's owner lift lifts `OwnerCaller` entries only (`internal/dobbyengine/owned.go` `recheckOwnerPause`).
  - The automatic lift of a wake-written entry (`LiftWakeHold`) takes `hibernate` entries of ended runs only (`internal/cleanupcli/reconcile.go:43-60`).
  - A record holds one live entry per scope (`internal/pauserecord/record.go:69`).
- D5, `already_paused: true` for an adopt-paused slug: held, by the same one-entry-per-scope rule.
- D5, the bound on a check started by an older lym: held.
- D6, a failed deploy leaves no adopted run going on under the old lym: held, since front tasks are bounded in D6 itself.
- D10, `beam start` with origin unreachable only warns and judges the owner on the local Bolt: held. `internal/orchcli/start.go:446-447`, `internal/orchcli/refusal.go:57`.
- D10, the shortcut stops at the first non-zero argv: held (`internal/shortcutcli/shortcut.go:218`).
- D10, a record past `check` with a held-back project takes up no worktree: held. `lym s` in the held-back worktree is refused naming the record's subjects and `--discard`, which is a way forward.
- D11, `archive-owner/` tags stay out of every `archive/` reader: held. `ArchiveTagSlug` cuts the prefix `archive/` (`internal/fabricengine/archive.go:42-44`), `remoteArchiveTags` and the remote-warp cleanup match `refs/tags/archive/&lt;slug&gt;/`, and `selectRuns` reads through `ArchiveTagSlug`.
- D11, the precedent that an owned-elsewhere teardown archives: held. `lym fabric remove` accepts the owned-elsewhere verdict (`internal/fabriccli/remove.go:86`, `:121`).
- D11, the archive is a tag-only push with no branch push: held (`archiveTaskTip` and `tagAndPushArchiveTip`).
- D11, "a machine that is off when the owner moves archives nothing until its dobby runs again": broken (F1).
- D12, teardown and `--drop-local` archives are excluded by prefix: held.

**Findings**

F1 — MEDIUM, design, decision-record.md D11 Bound (line 239)
- **Summary:** An old owner whose dobby is not running when its Bolt learns the new owner never archives its unpushed tips.
- **Counterexample:**
  - The old owner's laptop is powered off, rebooted, or its beam was stopped while the owner moved.
  - Its dobby can never start again under the foreign owner. `lym beam start` refuses on a foreign owner before any dobby starts (`internal/orchcli/refusal.go:57-58`). A dobby that does start reads the owner on its first probe and exits (`internal/dobbyengine/loop.go:180-181`, `:494-500`).
  - `lym s` on that machine runs `adopt --here`, which adopts the subject back rather than archiving it.
- **Effect:** "until its dobby runs again" names an event that does not happen. The tips stay local, and the new owner's `--archives` lists nothing.
- **Fix:** Give the archive an entry point that runs without a live dobby, for example the beam-start refusal path or `lym adopt --here` on a foreign-owned worktree. Alternatively, state the real bound and its way forward.

F2 — LOW, consistency, decision-record.md Acceptance criteria (line 281) against D2 (line 58)
- **Summary:** "A discard that races a starting worker leaves its record standing" holds in only one interleaving.
- **Counterexample:** If the discard holds `adopt.lock` when the worker tries it, D2 says the worker "exits at once and writes nothing". No record stands, and the line 275 criterion (caller exits non-zero naming the re-run) applies instead.
- **Fix:** Name the interleaving: a discard while the worker holds the lock is refused and leaves its record.

F3 — LOW, design, decision-record.md D1 Bound (lines 45-46) and D7 (line 165)
- **Summary:** A direct call of the hidden worker verb is not bounded by D7's orch rule.
- **Counterexample:** `--no-orch` comes from the caller's strand name, and only the D1 caller reads it. A direct `lym adopt worker` from an orch session runs the beam step with the orch start. D1 calls this caller "bounded by the same validation", but the orch-session rule is not part of that validation.
- **Fix:** Have the worker take the orch skip only from the request, and state that a direct call from an orch session must pass `--no-orch`. Alternatively, move the strand read into the shared validation.

**Tool calls**
1. `internal/dobbyengine/loop.go` 175-200 and 480-510, plus `beamUpRefusal` in `internal/orchcli/refusal.go`: the dobby's start-time owner read and beam start's foreign-owner refusal were one hop past the manifest.
2. `internal/pairteardown/originseams.go` 95-190: checked D11's owned-elsewhere teardown precedent, which the manifest did not cover.
3. A grep for `EntryOwnedElsewhere` teardown verdict sites: located where the owned-elsewhere teardown that archives runs (`internal/fabriccli/remove.go`).
