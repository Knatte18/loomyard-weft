# Card 3 — creel-seams

**Create:**
- `plan:internal/creelengine#ErrProgramClaimInStrandPane` -> `var ErrProgramClaimInStrandPane = errors.New("creel: a program claim comes from a strand pane that is no orch's")`
- `plan:internal/creelengine#CodeProgramClaimInStrandPane` -> `const CodeProgramClaimInStrandPane = "program-claim-in-strand-pane"`
- `plan:internal/creelengine#PaneOf` -> `func PaneOf(pid int, strands []Strand, strandsErr error, parentPID func(int) (int, bool)) (*Strand, string)`
- `plan:internal/creelengine#IsOrchStrand` -> `func IsOrchStrand(full string) bool`
- `plan:internal/creelcli#GoClient.ReadWait` -> `func (c *GoClient) ReadWait(timeout time.Duration, handle func([]creelengine.Message) error) error`
- `plan:internal/creelcli#HubStrands` -> `func HubStrands(hubPath string) ([]creelengine.Strand, error)`

**Edit:**
- `internal/creelengine#resolveSender`
- `internal/creelengine#daemon.fail`
- `internal/creelengine#Client.exchange`
- `internal/creelengine#TestResolveSender`
- `internal/creelcli#TestCreelCLI`

**Extends:**
- `internal/creelengine/doc.go`
- `internal/creelcli/doc.go`

**Uses:**
- `internal/creelengine#paneOfPeer`
- `internal/creelengine#isOrchName`
- `internal/creelengine#ErrStrandClaimOutsidePane`
- `internal/creelcli#GoClient`
- `internal/creelcli#newHubDelivery`
- `internal/creelcli#newRig`
- `internal/testkit/creelfake#LiveStrand`

**Intent:** Give the Slack relay and `lym slack` the four creel pieces the record's Takes line names, without changing what creel decides.

- `PaneOf` is the exported pane-ancestry walk: it returns `paneOfPeer(pid, strands, strandsErr, parentPID)` unchanged, so `lym slack`'s pane check (card 10) decides "which live strand's pane is this process in, or what keeps that unknown" exactly as creel's sender resolution does (D6).
  `IsOrchStrand` returns `isOrchName(full)`, so the check tells an orch's pane from another strand's by creel's own rule.
- The refusal of a program claim from a non-orch strand's pane becomes a coded refusal: in `resolveSender`, the branch `!isOrchName(pane.Name)` with a program claim returns a `codedRefusal` with its current message and the sentinel `ErrProgramClaimInStrandPane`; `daemon.fail` sets `Response.Code` to `CodeProgramClaimInStrandPane` for an error matching it, as it does for `ErrStrandClaimOutsidePane`; `Client.exchange` maps that code back to a `codedRefusal` with the sentinel.
  The message text is unchanged, so no assertion of it changes.
  The relay (card 9) tells this one refusal, after which it ends, from every other refusal of its first read, after which it retries (D3); no other caller's behavior changes.
- `GoClient.ReadWait` is `ReadAll` with `Wait: true` and the told timeout: it blocks until the caller has unread mail, hands it to handle, and acknowledges only when handle returns nil; it returns nil only once the daemon confirmed the acknowledgement, and a refused or lost confirmation returns its error, so the relay clears its outbound record only after a confirmed acknowledgement (D5).
  A handle error releases the read unacknowledged and is returned as is.
- `HubStrands` lists the hub's strands the way the daemon does, through `newHubDelivery(hubPath, shuttleengines.New).Strands()`, so the pane check reads the same strand set creel reads; it is a one-line forward over the delivery whose listing `delivery_test.go` already pins.

Tests:
- `TestResolveSender` (tier0) gains no row: its existing row `Linux_ProgramClaimFromANonOrchPaneIsRefused` gains the assertion that the error matches `ErrProgramClaimInStrandPane` under `errors.Is`, beside its message check, and no other row's error matches it.
- `TestCreelCLI` (tier1, in-process daemon of `newRig`) gains three subtests:
  `ReadWait` blocks until a post arrives, hands that message to handle, and a later `ReadAll` finds nothing unread;
  a handle that returns an error leaves the message unread, and `ReadWait` returns that error;
  a program client whose process sits in a live non-orch strand's pane (a `creelfake.LiveStrand` whose `PanePID` is the test's own pid) gets an error from `ReadAll` that matches `ErrProgramClaimInStrandPane`, which reaches the daemon's `fail` coding and the client's mapping end to end; this subtest rests on the daemon's Linux peer walk, so it is generated on Linux only, the loop passing over it before `t.Run` elsewhere (`PATTERN-test-speed`, no test skips by design).

The two `doc.go` files gain a sentence each: creelengine names the exported walk and the coded refusal beside `CodeStrandClaimOutsidePane`, and creelcli names `ReadWait` and `HubStrands` among the Go client's calls.

**ImpactSummary:** Additive exports plus one refusal gaining a code; its message is unchanged, so creel's existing callers and assertions are untouched, and only `lym slack` and the relay consume the new pieces.
