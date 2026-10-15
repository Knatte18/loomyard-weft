# Card 8 — creelengine-identity

**Create:**
- `internal/creelengine/identity.go`
- `internal/creelengine/peer_linux.go`
- `internal/creelengine/peer_other.go`
- `internal/creelengine/identity_test.go`
- `plan:internal/creelengine#Claim` -> `type Claim struct`

**Edit:**
- `//cmd/lyx/testdata/test-budget.yaml`

**Intent:** Resolve who is calling (D4), so privilege and holdings rest on what creel sees rather than on what a caller says.
`Claim` is what a client sends with each request: its `$LYX_STRAND_NAME`, a program-sender name set only through the Go client API (the CLI has no sender flag), and for the envelope its terminal, parent process name and working directory.
`peer_linux.go` reads the connecting peer's pid with `SO_PEERCRED` (`golang.org/x/sys/unix`); `peer_other.go` reports none.
An unexported resolver takes the peer pid, the claim, this hub's shortname, the strand list from `Delivery.Strands` with its error, a parent-pid function (told; the CLI passes `proc.ParentPID`) and the GOOS.
On Linux it walks the peer's ancestry against the live strands' `PanePID`s: a peer inside a strand pane is that strand, whether or not `LYX_STRAND_NAME` is set, and a `LYX_STRAND_NAME` naming a different strand is refused.
A peer in no strand pane whose `LYX_STRAND_NAME` names a strand of this hub is refused, since every live strand's pane process is known.
A program-sender claim (`agentname.ParseProgram`) under another shortname is refused; under this shortname it is accepted from a peer in no strand pane, accepted from inside an orch strand's pane, and refused from inside any other strand's pane, so a claim never raises a pane's privilege.
A program claim of the reserved name `EXTERNAL` is refused on every platform, since D4 admits any upper-case name but it, so EXTERNAL always carries the envelope creel saw.
The walk starts at the peer pid itself, so a peer that is a pane's own process is inside that pane.
When the strand list fails, or a live strand reports no pane pid, a peer in no known strand pane is refused naming what could not be read, and is never resolved as EXTERNAL or as a program.
Anything else is `<shortname>:EXTERNAL` with the claim's envelope.
Off Linux the sender is `LYX_STRAND_NAME` when it parses as an agent name of this shortname, else a program claim of this shortname, else EXTERNAL.
Bound: off Linux identity is the environment alone, so an agent that unsets `LYX_STRAND_NAME` passes as EXTERNAL and any process speaking the protocol may claim a program name; on Linux a process detached from its pane's ancestry (`setsid`, a double fork, `tmux run-shell`) passes as EXTERNAL or claims a program name; the trust boundary is the OS user, since the socket sits in the 0700 lyxtemp root.
`TestResolveSender` table rows, over a scripted ancestry: on Linux a strand-pane process with `LYX_STRAND_NAME` unset resolves to its strand; a disagreeing `LYX_STRAND_NAME` refused; a peer outside every pane claiming a strand name refused; a program claim from a non-orch pane refused, from an orch pane accepted, from outside every pane accepted; a program claim under another shortname refused; a program claim of `EXTERNAL` refused on Linux and off it; a peer whose own pid is a pane pid resolves to that strand; a peer in no known pane while a live strand's pane pid is 0 refused naming it, and while the strand list errs refused naming the error; a plain outside peer resolves EXTERNAL with its envelope; off Linux the environment name, the program claim and EXTERNAL.
`cmd/lyx/testdata/test-budget.yaml` raises `internal/creelengine` by one.

**ImpactSummary:** New identity files in creelengine and one budget row raise; no existing caller.
