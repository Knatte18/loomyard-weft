# Card 8 — reed-session-row-prime-first

**Edit:**
- `internal/reedengine#statusFormatSessions`
- `internal/reedengine#sessionButtonFormat`
- `internal/reedengine#Engine.pinGeometryOptionsLocked`
- `internal/reedengine#TestPinGeometryOptionsLocked`
- `internal/reedengine#TestAttachArgv_PreflightOnAKnownGoodSession`
- `internal/reedengine#TestBar_LineTwoListsSessionsInIDOrder`
- `internal/reedengine/doc.go`

**Delete:**
- `internal/reedengine#currentSessionButtonFormat`

**Create:**
- `plan:internal/reedengine#primeSessionOption` -> `const primeSessionOption = "@lyx_prime_session"`
- `plan:internal/reedengine#Engine.primeGeometry` -> `func (e *Engine) primeGeometry() bool`
- `plan:internal/reedengine#TestStatusFormats_SessionLoopsTakeOneFormat` -> `func TestStatusFormats_SessionLoopsTakeOneFormat(t *testing.T)`

**Uses:**
- `internal/reedengine#reedSessionOption`
- `internal/reedengine#newTestEngine`
- `internal/reedengine#installFakeTmux`
- `internal/reedengine#newBarFixture`
- `internal/reedengine#Engine.pinBootOptionsLocked`

**Intent:** D3: tmux 3.4's `#{S:}` takes one format, so today's two-format loop draws every session twice with a literal comma (#738), and 3.4's `S` loop has no sort, so the prime's session can lead the row only from a loop of its own (#740, #748).

- `primeSessionOption` is a session user option marking the prime's session; like the bar's other options it is display only, and no Go decision reads it.
  `bar.go`'s file header comment, which lists the options the bar reads and which of them a Go decision reads, gains `@lyx_prime_session` among the display-only ones and describes line 2's prime-first order.
- `primeGeometry` reports a hub geometry with an empty `NameSlug`: `DiscoverSignalPath` non-empty (a standalone geometry leaves it empty, per `standalonegeom.ReedGeometry`) and `NameSlug` empty.
  A pair's session and a standalone session are never marked.
- `pinGeometryOptionsLocked`, when `primeGeometry` holds, issues `set-option -t <target> @lyx_prime_session 1` after the status pins, idempotently and non-fatal like them (a failure logs a Warn).
  It runs at boot and in the attach pre-flight, so a boot marks the session it spawns, and a prime session already up when the new binary arrived is marked at its next attach pre-flight.
  Bound: until a `lyx reed attach` to the prime whose pre-flight reaches the geometry pins, such a session draws in the second loop; an attach whose pre-flight degrades to the bare argv leaves it unmarked.
- `sessionButtonFormat` becomes one button body for a session range that picks, by `#{==:#{session_name},#{client_session}}`, the lit style with `list=focus` and the full name, or the unlit style with the 12-character cut; `currentSessionButtonFormat` is deleted, its only reader being `statusFormatSessions` in this card.
- `statusFormatSessions` becomes `listPrologueFormat` plus two single-format `#{S:}` loops, then the follow button and the time, which stay where they are:
  the first draws `sessionButtonFormat` only for a session matching both `reedSessionFilter` and the prime mark, and the second for every other session matching `reedSessionFilter`, in the `S` loop's own order, which is session name on tmux 3.4 and session id on newer tmux.
  With no marked session the first loop draws nothing.
  The constant's comment and `internal/reedengine/doc.go`'s geometry-pin paragraph describe the new row and the order.
- `TestStatusFormats_SessionLoopsTakeOneFormat` scans `statusFormatButtons` and `statusFormatSessions` for every `#{S:` loop and fails when a loop's body has a comma outside its nested `#{…}` and `#[…]` groups, which is the second, current-item format tmux 3.4 does not take.
  It also fails when any non-test `.go` file of `internal/reedengine` other than `bar.go` contains `#{S:`, so a session loop declared elsewhere cannot escape the scan.
  Rendering on this host's tmux 3.6 cannot catch such a loop, so this untagged check pins it.
- `TestPinGeometryOptionsLocked` gains a subtest "PrimeSessionIsMarked" with rows prime (`DiscoverSignalPath` set, `NameSlug` empty), pair (`NameSlug` set) and standalone (both empty): only the prime row issues the mark's `set-option`, with the strand window target and value `1`; the `AllOptionsIssued` list is unchanged, since `newTestEngine` is a pair geometry.
  A further row "a boot marks the prime" drives `pinBootOptionsLocked` on a prime engine and asserts the same mark, which is the boot half of the criterion.
  A further row "a failing mark is non-fatal" scripts the mark's `set-option` to fail on a prime engine and asserts a `logcapture` Warn naming the option and that the calls after it (window-size and the hooks) are still issued.
  The test's `//testtiming:keep` reason gains the prime mark among the pins it names.
- `TestAttachArgv_PreflightOnAKnownGoodSession` gains a prime-geometry run beside its existing pair run: on the prime engine it expects `wantSetOptionCalls` plus one and the mark among the `set-option` calls, which is the attach half of "a prime session created by an older binary is marked after an attach"; the pair run's count stays as it is.
- `TestBar_LineTwoListsSessionsInIDOrder`, in its `tmux && linux` file, renders line 2 three times:
  with no session marked, each session's range appears exactly once (`strings.Count` of its `range=session|` start is 1), the client's session lit with its full name, and no comma before the follow button, so the first loop draws nothing;
  with the mark set on the fixture's second session `a-very-long-session-name`, standing for a prime the client is not on, that session's unlit button comes first, then the client's lit one, each range once and no comma;
  with the mark moved to the client's own session, that session's lit button with its full name comes first, then the other's unlit one, each range once.
  Its comment names the prime-first order; the follow-button half keeps its assertions against the last session button.

**ImpactSummary:** Changes the global `status-format[1]` string and adds one session option pin for the prime; no Go decision reads either, and the windows line, follow button and time are unchanged.
