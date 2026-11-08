---
round: 2
focus:
  - "internal/landingshed/beamdescribe.go `DraftBeamDescription`: when `deps.DescriptionPath` already exists, does the draft path refuse without touching the file, and does the refusal name the way forward?"
  - "internal/beamcli/describe.go: when the description is recorded and the landing record then cannot be read, does the error say the description is recorded and name `lym beam pr update`, and does a test reach that branch?"
  - "internal/beamcli/cli.go describe verb: is `--draft` bound and passed through to `describe`, and does a test drive the cobra verb so an unbound flag would fail?"
---

The entries come from the round 1 review's locations; the reviewer settles each against the diff as it now stands.
