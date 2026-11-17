```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge each review proposal the fixer carried out as proposed, for cases where the proposal is wrong or breaks something elsewhere.

**[BLOCKING] F4 ruling as carried out: comparing fork seats ships a parity test that fails by construction.** Changed text: card 6's `TestSeats_CodexManifestHoldsClaudes` bullet ("For every `Seat`, a seat Codex cannot start included…"), card 7's `TestProducers` bullet, and the overview decision "every seat is compared…".
- Card 3 gives a `ForkSubagents` spec a compared `guard webster-fork-deny` item: "a spec with `ForkSubagents` holds `webster-fork-deny` with an empty `Engine`".
- Card 4's Codex `guard` channel lists only `question-deny` and the shellguard rows. `internal/shuttleengine/codexengine` has no webster fork deny at all: a grep for webster or fork deny in its non-test files returns nothing. Claude's deny is `steerWebsterForkDeny` in `claudeengine/settings.go:25-29`.
- Every seat the ruling brings in therefore reports `guard webster-fork-deny` missing, and no card gives Codex that delivery: fanned burler review and fix, Plan-Write, Discussion, Merriam's `Run` and `loomcli#driverSpec`.
- This contradicts the overview's premise that "every compared difference found has a Codex channel" and the D1 auto-mode assumption.
- The ruling's own premise ("Prepare does not refuse… so the manifest exists") holds, but the plan now needs one of two things. Either a card giving Codex a fork-deny delivery, or the item keyed to `claude`, which would be a new exemption the record forbids. As written, the guard test fails on landing.

**[MEDIUM] F11: the proposed root derivation misstates `readCheckout` and adds an unhandled failure.** Changed text: card 3, "the repository root is the root `dotgit.FindRoot` returns for that resolved anchor, as `codexengine`'s `readCheckout` derives it".
- `readCheckout` calls `dotgit.FindRoot(anchor)` on the unresolved anchor (`codexengine/home.go:74`) and only then resolves the root with `EvalSymlinks` (`home.go:78`). So "on the resolved anchor, as readCheckout derives it" is false.
- The two differ when a symlink sits between anchor and root. The `FindRoot` doc says "The returned root is spelled as walked from start".
- `FindRoot` returns `ErrNotRepository` when no `.git` lies above the anchor (`dotgit.go:72-78`). Card 3 does not say what `Claude.Prepare` does then, although Claude launches today need no repository.
- Card 3's own test fixture `engineWithAnchor` (`claudeengine/prepare_test.go:72-82`) makes the anchor a bare `t.TempDir()` with no `.git`. Its "anchor CLAUDE.md source" row therefore has no root to key against, unless the card states that fallback or the fixture gains a `.git`.

**[LOW] F3: the stored `Verdict` reads as a failure in every parity and verb record.** Changed text: card 2, "`WriteManifest` sets the record's `Verdict` from `m.Omitted()`…" and "a plugin context left out at run time fails the verdict the bench report reads".
- Under `UnrunHooks`, card 4 lists every plugin context as `omitted` with the `ErrHookUnrun` reason. Every Codex record that `PrepareEach` writes, which includes `lym shuttle manifest`'s, therefore stores a non-empty `Verdict`.
- Card 8's `ok` output prints "both manifests and the verdict, plugin contexts listed unrun". A reader then sees a passing verb beside records whose stored `Verdict` lists omitted items.
- Card 8 needs to say which verdict it prints, or that a record's `Verdict` is meaningful only for a real launch.

No case found against the proposals carried out for F2, F6, F7, F8, F10 and F12.

Tool calls:
- One grep of `codexengine`, `claudeengine`, `dotgit` and the `engineWithAnchor` fixture. My context did not show whether Codex has a webster fork deny, how `FindRoot` behaves when it finds no repository, or whether Claude's test anchor holds a `.git`.
