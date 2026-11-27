```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens, tracing the diff's changes to the record's decisions and back.

**Changes → decisions**

| Change | Traces to |
|---|---|
| Heddle `Passage`, `FileEntries`, the file targets' `Engine`, the `Standing` seam, the `cmd/lym` reader closure | D1 (card 1) |
| Codex refusal names a passage by slug instead of position | forced by card 1 |
| Manifest record, `MissingItems`, the launch copy, the sweep and strand lookup skipping `manifests` | D1 (card 2) |
| `ListedSkills` and the `Plugin`/`Hook` fields | D1 (cards 2, 3) |
| Claude manifest; `askUserQuestionTool` constant | D1 (card 3); the constant is forced by it |
| Codex manifest | D1 (card 4) |
| `resolveUserSetup` logs nothing for an unrun hook | D10 (card 4) |
| `LaunchSeams`, `NewWithSeams`, `registeredConstructors`, provider-seam background sentence | D10, D1 (card 5); the refactor is forced |
| `shuttleparity` compare and seat catalog; `seatengine.Specs`, `FixReviewSeatTable`, `Engine.SeatTables` | D1 (card 6) |
| Fork deny gated on fork support | Added entry, 2026-11-17, fork deny |
| `ReferenceEngine`/`ComparedEngine` constants | Added entry, 2026-11-17, two engines as constants; `Pattern-Departure` trailer on commit 6 |
| Producer table and literal scan | D1 (card 7) |
| Tree-sitter allowlist row | forced by card 7; `Pattern-Departure` and `Loosened-Check` trailers |
| `lym shuttle manifest`, `Report`, `LookupSeat`, `docs/overview.md` | D10 (card 8) |
| `seatengine.Prompts` allowlist row removed | forced by card 8 |
| No allowlist row for `Engine.SeatTables` | `Pattern-Departure: PATTERN-test-only-code` |
| `PATTERN.md` line and `PATTERN-engine-parity.md` | declared PATTERN change (card 9) |

No change is untraced.

**Decisions → changes**

| Decision | Built by |
|---|---|
| D1 | cards 1–7; one gap, finding 1 |
| D10 | cards 4, 5, 8 |
| D11 | no diff, by design (board only, as the two 2026-11-17 parent entries say) |
| Arbiter 11-16, assist-trial risk | nothing to build |
| Arbiter 11-16, round-3 findings | scan skips `Prompt`-less literals; bouncer renderers are used by the producer table |
| Parent 11-17, D11 on `codexengine` | board only |
| Parent 11-17, card 10 drops | no board-writing card or hunk |
| Parent 11-17, every seat compared | no `Unmet` skip in `TestSeats` or `TestProducers`; one seat variant is absent, finding 1 |
| Parent 11-17, fork deny | `Compare` and `TestCompare`'s two fork-deny rows |
| Arbiter 11-17, plan round 3 | `ModelChoice{Engine}` alone, `TestMain`, the constants; the interim allowlist rows net to nothing |
| Parent 11-17, engine constants | `parity.go` and the commit trailer |

**Findings**

1. **MEDIUM, class scope.** `internal/shuttleparity/tables.go`, `burlerSeatTables`: the unassisted burler reviewer chair is never built, so it is neither compared nor printable.
   - Both `SeatTables` calls (solo and fanned) run with `opts.Assist` set.
   - `withAssistant` then replaces the chair's stencil with `assistedReviewerStencil` (`internal/burlerengine/assist.go:152-161`, reached from `probeTable`, `review.go:154-157`).
   - The plain `burler-template-reviewer` seat (`review.go:80`), which a round with no assistant starts, is therefore in no catalog table, solo or fanned.
   - D1 and its acceptance criterion require every seat in `seatengine`'s tables, and the "Every seat is compared" entry names fanned burler review. A Codex rendering of the unassisted reviewer that loses its edit directive, or fails to render, passes `TestSeats_CodexManifestHoldsClaudes`.
   - Plan card 6 words the calls as carrying `Assist`, so the fix is an added call with no `Assist`, under distinct role prefixes.

Outside my lens: the catalog's burler seat names come from the stand-in prefixes (`solo-burler-review`, `fanned-burler-fix`, and so on; `review.go:29`, `profile.go:83`). `lym shuttle manifest` would therefore refuse the strand role a real round's burler seat carries.

Tool calls:
1. `grep` for `ReviewChairRole`, `ledRole` and the burler `RolePrefix` sites: my context had the table builders but not how the seat role names are composed.
