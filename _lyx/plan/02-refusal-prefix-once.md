# Card 2 — refusal-prefix-once

**Edit:**
- `internal/creelcli#writeRefusal`
- `internal/creelcli/doc.go`
- `internal/creelcli#TestCreelCLI`

**Uses:**
- `internal/creelcli#runCreelInto`
- `internal/creelcli#failingWriter`
- `internal/creelcli#lockedBuffer`
- `internal/creelcli#newRig`
- `internal/creelcli#rig.startDaemon`
- `internal/creelcli#rig.name`
- `internal/gitkit#Git`
- `internal/testkit/creelfake#LiveStrand`
- `internal/testkit/creelfake#Delivery.SetStrands`

**Intent:**
D3: `writeRefusal`'s stderr fallback prepends `creel: ` only when the trimmed message does not already open with `creel: `.
A message opening with another form, such as `creel needs a hub ...` or `creel read: ...`, still gets the prefix, so every fallback line opens with `creel: ` and carries it once at its head; `creel: creel read: ...` stays as it is.
The JSON envelope's `error` field keeps the message unchanged, and no error text in `internal/creelengine` or `internal/creelcli` is reworded.
Update `writeRefusal`'s doc comment to say the line is the message with `creel: ` prepended unless it already opens with it.

D5: in `internal/creelcli/doc.go`, the line about a refusal whose JSON envelope cannot be written says the stderr line opens with `creel: ` once, whether or not the error already carried the prefix.

Test: add one subtest to `TestCreelCLI`, `a refusal into an output that cannot be written prints the creel: prefix once`, with rows run through `runCreelInto` into `failingWriter{}`, each asserting exit 1, a stderr of exactly one line, that the line opens with `creel: ` and that `strings.Count(line, "creel: ")` is 1:
- an engine refusal: as EXTERNAL, `post <r.name("nobody:driver")> text` on a rig whose daemon knows only `alpha`, refused as an unknown strand (`creel: unknown strand ...`);
- an engine refusal: as EXTERNAL, `inbox`, refused because an EXTERNAL caller holds no mail (`creel: an EXTERNAL caller holds no mail; ...`);
- a creelcli refusal: `inbox` in a plain repository made with `gitkit.Git(t, repo, "init", "-b", "main")`, refused with `creel needs a hub ...`;
- a creelcli refusal: `post text`, one argument without `--reply`, refused with `post takes a receiver and a text ...`.
The two engine rows fail today with `creel: creel: ...`, and the two creelcli rows pin that a message without the prefix still gets it.

**ImpactSummary:** one condition in `writeRefusal`, the sole stderr fallback of every creel refusal; the envelope on stdout and every exit code are unchanged.

**Commit:** 2: creel refusal fallback prints the creel: prefix once
