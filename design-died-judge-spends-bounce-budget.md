# A died bouncer judge session spends Code-Bouncer's bounce budget

`died-judge-spends-bounce-budget` · Note · bug, burler, shed

Imported from [issue #239](https://github.com/Knatte18/loomyard/issues/239).

On a flaky network, Code-Bouncer's judge session twice failed to start: shuttle logged 'provider never became ready; strand torn down', relaunched, and the second run finished at once with no transcript. shedadapters then logged 'bouncer judge run did not complete ... outcome=died', and the row stopped with 'bounce budget exhausted for Code-Bouncer'. The judge never gave a verdict, so no bounce happened, yet the run stalled until a manual `lym loom resume`.

Expected: a judge that dies before giving a verdict is relaunched, or the row halts as an infrastructure failure, without spending the bounce budget meant for real verdicts.