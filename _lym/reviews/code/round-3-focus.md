---
round: 3
focus:
  - "internal/beamcli/cli.go startCmd: does one exported constant NoPrimeClaimFlag now name the hidden flag in both the cmd.Flags().BoolVar and cmd.Flags().MarkHidden calls, with no remaining \"no-prime-claim\" literal in beamcli production code?"
  - "internal/machinecli/start.go runBeamStarts: does the by-slug child's argument build from \"--\"+beamcli.NoPrimeClaimFlag, with the old noPrimeClaimFlag constant gone, and does the new beamcli import respect machinecli's declared dependency list and create no import cycle?"
  - "internal/beamcli/start_test.go and internal/machinecli/start_test.go: does the beamcli flag test look the flag up through NoPrimeClaimFlag, and do the wantBeamStarts rows still pin the child's literal command line?"
---

Round 3 checks the one fix of the round 2 finding on the repeated flag-name literal.
