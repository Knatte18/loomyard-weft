# Fixer report — discussion, round 2

## Changes

- **F1 (fixed differently)**: D17's next-wake fallback now compares `origin/prod` against `origin/main` and runs `update-plugins.sh` when `main` holds a commit `prod` lacks; the same wake starts any task the board shows ready and not yet started.
  D16's last paragraph names the same baseline, and the last acceptance row pins it in the fallback sentence.
  The `fix` key offered two baselines.
  The notice-log one was not taken: it reads landed lines against the time of the orch's last `update-plugins.sh` run, and that time is recorded nowhere, the same gap the finding names.
  The taken baseline is the fix key's second option, the installed build's revision against `main`, read from where it is recorded: `tools/deploy/main.go:103-108` pushes the deployed commit to `origin/prod` after the deploy succeeds.
  Both rejected baselines are recorded under Rejected alternatives in the support log.
- **F2 (fixed as proposed)**: the Scope bullet now names an audit of every Warn or Info-or-above log line in a loop or polled path.
  D11's first sentence says the audit covers every Warn line and every Info-or-above line inside a loop or polled path, the sentence about a non-loop failure staying Warn is deleted, and the bound now ends with "such a site is out of this audit".
  D11's heading and the support log's Q7 answer were narrowed to match, since both repeated the broad reach.
- Support log: Round 2 entry added under Review rounds.

## Disputed

None.

## Deferred

None.
