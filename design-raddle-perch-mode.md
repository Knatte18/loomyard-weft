# Raddle's perch mode: recorded from the recipe's rows, with its outcome checks and refusals

`raddle-perch-mode` · Task · feature, shed, loom, burler, arbiter, board · running · Darn-Write

Decision record: [Sketch](decision-raddle-perch-mode.md)

Filed from the Sketch `motif-raddle-recipe`.

Takes: raddle's auto and interactive modes in the raddle state, the raddle gate, raddle-propose, raddle-approve, Raddle-Write and Raddle-File, internal/raddleparser's checks and `lym loom decision add`
Delivers: the raddle mode `perch`, recorded by Raddle-Write on its first call when the recipe's rows hold a split-perch Bouncer over the proposal, tested with a fixture recipe; in perch mode the gate stores the cold proposal, raddle-approve refuses naming the perch, Raddle-File files with no approval-hash match and a fresh entry moves the last proposal aside; the D5 outcome checks in internal/raddleparser behind a perch option, with the slug-sketch and slug-taken exemption, the spec text and the raddle stencil's perch mode text; `freshRaddleEntry` and its table test corrected for a goto entry; `lym loom decision add` refusing in perch mode from the Raddle-File row on, with its ways forward. A perch-mode project proposal passes the checks here; its filing as the converted entry comes with arras-family-filing
Exports: the `perch` RaddleMode, the perch option of raddleparser's checks, the rows test that decides perch mode

The part changes these packages:

- contracts/specs (about 30 changed lines)
- internal/loomshed (about 120 changed lines)
- internal/loomcli (about 180 changed lines)
- internal/raddleparser (about 110 changed lines)
- internal/loomengine (about 60 changed lines)
- internal/shedrecipe (about 30 changed lines)
