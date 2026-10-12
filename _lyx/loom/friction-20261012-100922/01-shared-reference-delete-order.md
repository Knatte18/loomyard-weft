# Card gate steps naming external test packages as directories

Cards 2 and 3 and the batch gate list `./internal/websterengine_test` and `./internal/planparser_test` as packages to `lyx gate test`.
Those are the external test packages of `internal/websterengine` and `internal/planparser`, not directories, so each step exits 1 with `directory not found` and `[setup failed]` while the real packages pass.
I ran the gates with the real directories only and read the setup failure as a gate-definition flaw, not a card failure.
