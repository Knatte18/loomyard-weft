---
round: 1
focus:
  - "internal/orchengine/reinterpret.go FlagRemoteControlDefault: settle whether keys.Unmarshal into map[string]yaml.Node handles an empty file, a null document and a hub file with a commented-out or explicit remote_control, and whether the contract in configreg's Reinterpretation doc (rewrite, flag, ordering, once per hub) is met with a nil rewrote list and unchanged bytes."
  - "internal/configreg/configreg.go orch row and internal/configreg/doc.go '# Reinterpretations': settle whether the diff follows the registry's reinterpretation rules (ID uniqueness, module-level fields, PATTERN-config-layers meaning-change duty) and whether any docs or specs the rules name for this kind of change land in the same diff."
  - "Decision record Fix bullet 4, 'no other launch path turns remote control on by its own default': check internal/orchcli/start.go and remotecontrol.go (launchRemoteControlName, respawn, beam orch, resume) and internal/orchengine/config.go for any path that reads a Go-side default or a second template copy of remote_control, and settle whether the diff leaves one."
  - "Template and doc wording: settle whether internal/orchengine/template.yaml and the doc.go remote-control paragraph state the new default consistently, and whether the doc.go sentence naming FlagRemoteControlDefault describes what the function does."
  - "internal/orchengine/config_test.go TestLoadConfig_RemoteControl and reinterpret_test.go TestFlagRemoteControlDefault: settle whether each expected value comes from a literal rather than the code under test, and whether any assertion existing at the review base was widened or removed."
  - "Commit message of 69d2fae06: settle whether the Root-Cause trailer names the defect the diff removes and whether the body's claims (once per hub, the notice text) match the diff."
---

Seed focus from reading the decision record, the six-file diff and the orch launch path.
The reach and mutation reports show the new function covered and both mutants killed, so the focus is on edge inputs, the registry contract and the launch-path sweep the record asks for.
