# Card 1 — machine-address-parser

**Create:**
- `internal/agentname#MachineAddress`
- `internal/agentname#ParseMachineAddress`
- `internal/agentname#IsMachineAddress`
- `internal/agentname#TestParseMachineAddress`

**Edit:**
- `cmd/lym#testOnlyCodeAllowed`

**Extends:**
- `internal/agentname/agentname.go`

**Uses:**
- `internal/agentname#MachineAddressSeparator`
- `internal/agentname#FormatMachineAddress`
- `internal/agentname#Parse`
- `internal/agentname#Name`

**Intent:**
D1: `internal/agentname` gains the one parser of the cross-machine address `<machine>::<name>`, the form `FormatMachineAddress` already writes.

`MachineAddress` holds `Machine string` and `Name Name`, and its `String` is not added: callers format with `FormatMachineAddress(addr.Machine, addr.Name.String())`.
`IsMachineAddress` reports whether name holds `MachineAddressSeparator`; it is the routing test creel and the relay use, and it judges nothing else.

`ParseMachineAddress` checks, in this order:
1. The address splits at its first `MachineAddressSeparator` into a non-empty machine part and a non-empty name part; otherwise it refuses with `agent address %q is invalid: want <machine>::<name>`.
2. The machine part equals one of ownNames (this machine's current and former names, `machinename.Names`).
   It refuses with `agent address %q names this machine (%s); way forward: name the agent without the machine, as %s`, the bare name part as the way forward.
   This check runs before checkMachine, so a former name that fails the machine name rule still gets this refusal.
3. checkMachine(machine) returns an error: it refuses wrapping that error.
4. `Parse(name part)` fails: it refuses wrapping the parse error, with the way forward `name a strand of that machine, such as <machine>::<shortname>:orch`, so a program name such as `ly:SLACK` is refused.
The parser does not judge the role.

The machine rule is passed in as checkMachine, and every caller passes `machinename.CheckName`.
This keeps the package on the standard library alone, which `TestAgentNameInvariant_StdlibOnly` holds (see `## Shared Decisions` in the overview).
The package comment in `internal/agentname/agentname.go` (the package has no `doc.go`), in its paragraph on the cross-machine form, gains one sentence: `ParseMachineAddress` parses it and `IsMachineAddress` tells it apart.
No main reaches either function until card 3 routes posts through them, so `cmd/lym#testOnlyCodeAllowed` gains `internal/agentname.ParseMachineAddress` and `internal/agentname.IsMachineAddress`, each with the reason `no caller until card 3 routes a post to another machine through it; that card removes this entry`.

`TestParseMachineAddress` is a tier 0 table test in `internal/agentname/agentname_test.go`, each row an address, ownNames, a checker and the want (machine and name, or an error substring).
Rows: `osl-1046::ly:orch` and `neve::ly:beam-x:orch` parse; a machine the checker refuses is refused with the checker's text; a name part `ly:SLACK` and one `ly` are refused; ownNames `neve` with `neve::ly:orch` is refused naming `ly:orch`; a former name `Old_Box` that a real `machinename`-style checker would refuse gets the own-name refusal, with a checker that fails the test if called; `ly:orch` (no separator), `::ly:orch` and `neve::` are refused.
`IsMachineAddress` rows: `neve::ly:orch` true, `ly:orch` false, `ly::orch` true (the routing test is the separator alone, and the parser then refuses it).

**ImpactSummary:** Additive: one type and two functions in `internal/agentname`, which stays on the standard library; no caller exists yet and no existing name's parse changes.

**Commit:** 1: agentname parses the cross-machine address
