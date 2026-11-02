# Card 4 — webster-file-headers

**Prosa:**
- `internal/websterengine/render.go#`
- `internal/websterengine/strand.go#`
- `internal/websterengine/recoverbatch.go#`

**Uses:**
- `internal/gatefixer#RenderPrompt`
- `internal/gatefixer#StrandLive`
- `internal/gatefixer#TurnEndedAfter`
- `internal/websterengine#RecoverSpawnOrAttach`

**Intent:**
The file headers of three `internal/websterengine` files still name what card 3 deletes; each drops those names and states the post-change ownership.
The headers are file-level comments above `package`, so no member target of card 3 reaches them, and `redundant-file-target` keeps their file glyphs off card 3.

- `render.go`'s header drops `webster-prefix-verify-fix-cold.md` from its asset list (line 1) and `RenderVerifyFixColdPrompt` from its caller sentence (line 4) and its two friction and pattern lists (lines 32 and 35).
  It says `fix-verify` renders the cold verify-gate fixer's prompt through `gatefixer.RenderPrompt`, the shared frame first and `webster-body-verify-fix` as the body, and that the file composes only the in-session fixer fork's prompt, `webster-body-verify-fix` plus `webster-prefix-verify-fix-fork` (line 25).
- `strand.go`'s header (line 1) drops `StrandLive` and `TurnEndedAfter` from what the file implements, and says webster's waits read a strand's liveness and turn end through `gatefixer`'s `StrandLive` and `TurnEndedAfter`.
- `recoverbatch.go`'s header (line 1) describes `RecoverAwait`'s wait as webster's `Classify` and `PollUntilTerminal` over `gatefixer`'s `TurnEndedAfter` and `StrandLive`, in place of naming all four as webster's own classification machinery.
