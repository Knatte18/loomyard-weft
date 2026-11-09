# lym machine name shows and sets the machine name and sweeps owners across a rename

`machine-name-verb` · Task · feature, config, slack · draft

Decision record: [Sketch](decision-machine-name-verb.md)

Depends on: [`machine-name-file`](design-machine-name-file.md)

Filed from the Sketch `machine-name-config`.

Takes: machine-name-file's machinename.FilePath, ReadFile, Names, CheckName and FromHostname; internal/state's WriteJSON; hubregistry.Live; fabricengine.Bolt Pull and HasUnpushed; boardengine.ReadOwners
Delivers: the module machine with lym machine name showing and setting the name, its collision check across every swept hub, the file write that keeps names, and the per-hub owner sweep with its recovery output; boardengine.OwnerRequest's replace-set field selecting owners after the write's own pull
Exports: machinecli.Command, the replace-set field of boardengine.OwnerRequest

The part changes these packages:

- internal/boardengine (about 130 changed lines)
- internal/machinecli (about 800 changed lines)
- cmd/lym (about 10 changed lines)
