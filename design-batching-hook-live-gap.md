# Batching hook fires on replay but almost never live

`batching-hook-live-gap` · Task · bug, tools, claude · done

## Seen

Report `docs/research/turns/batching-hook-remeasure-2026-11-05` (d05c8d75): the hook fired 13 times in 22 runs; 156 edit follows met 2 edit firings, while a replay of the hook on one recorded edit follow fires. Edit follows rose from 4.2% to 6.9% of context (Webster forks 6.9% to 11.2%), all mid-round, none after a failed gate. The report recommends removing the hook and names the live gap as its one open question.

## Idea

Find why the hook does not fire live on edit follows before deciding to remove it: either fix the live gap and measure again, or remove the hook. Mid-round edit splitting is where the edit-round rule fails, so a fix there is what matters.