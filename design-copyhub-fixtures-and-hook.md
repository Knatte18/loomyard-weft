# CopyHub in the NewHub fixtures of fabricengine and adoptcli, and the hook without dirname and basename

`copyhub-fixtures-and-hook` · Task · enhancement, fabric, testkit, git · running · Code-Burler-Fix

Decision record: [Sketch](decision-copyhub-fixtures-and-hook.md)

Filed from the Sketch `fabric-test-speed`.

Takes: newRecordsFixture, the adoptcli hubforge.NewHub sites, post-checkout.sh and hook.go's chainedHookPreamble
Delivers: hubforge.CopyHub of the same anchor at each site whose test is not about the clone or wiring, and shell parameter expansion in the hook and chain wrapper, falling back to . when $0 holds no slash
Exports: nothing

The part changes these packages:

- internal/fabricengine (about 25 changed lines)
- internal/adoptcli (about 15 changed lines)
