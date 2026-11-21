# One-repository sandbox trial 1: repository created and hub cloned

Adds the report `docs/research/single-repo/sandbox-1-setup-2026-11-21/report.md`, the first of three trials of the one-repository hub layout from 57015a52.

The trial created the private repository `Knatte18/lym-sandbox-onerepo` and seeded `main` with a Go module of one package and one test.
It then cloned the repository with `lym fabric clone --single-repo --shortname sbone` into `/home/knatte/Code/lym-sandbox-onerepo-LYMHUB`.

Every check passed:

- GitHub's branch list shows `main` alone, and the tag list is empty.
- `git ls-remote` shows the board at `refs/lym/heads/main` and the prime's records branch at `refs/lym/heads/main-weft`, both at the local tips.
- The records store carries the layout's fetch and push refspecs, `--no-tags` and `lym.primeRecordsBranch main-weft`; the code store keeps git's defaults and holds no records ref.
- `lym board list`, `lym fabric status`, `pairs`, `list` and `shortname` answer `ok: true` in the hub, and the one pair reads in sync with a healthy junction.

No lym bug was found, so no board note was filed.
The report records three friction points.
A task session cannot run `lym fabric clone`, so the orch ran it.
The clone's summary names its first pushes rather than origin's final tips.
The sandbox board has no labels yet.

## Design decisions

- The seed went in through GitHub's contents API rather than a local checkout, so no seed directory was left on the machine.
- The task named no shortname; the hub uses `sbone`.

## Merge danger

**Door:** two-way
**Blast radius:** docs
