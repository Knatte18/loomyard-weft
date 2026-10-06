# Reflection report

All four notes describe one problem and were merged into one issue:

- https://github.com/Knatte18/loomyard/issues/401

The issue covers a batch order that ran a delete card before a card it depends on, which neither Plan-Bouncer nor Batchifier caught.
It also covers the `webster` refusal whose `way forward` (`recover-batch`) cannot succeed on an unchanged tree.
