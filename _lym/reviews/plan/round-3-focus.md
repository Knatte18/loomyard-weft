---
round: 3
focus:
  - "In _lym/plan/07-chase-clock.md, settle whether any path still makes the CREEL program a known mailbox that posts are accepted for and nobody reads. Check both the applyPosted branch and the applyCarried branch for a posted record whose From is a program named ChaserProgram, against checkReceiverLocked in internal/creelengine/pool.go accepting any name in pool.programs, and against carryRecords carrying unread messages recorded before the retention cutoff. Check that a TestJournal_Retention row reaches the carried case."
  - "In _lym/plan/07-chase-clock.md, settle whether the chaser's first tick charges. The card says a tick charges one agedPersistInterval only when its probe and the previous tick's both read the clock running, and that the chaser ticks every agedPersistInterval from the moment it starts. Check that the chaser.run paragraph, the Shared Decision chase-clock-cadence sentence and the TestScheduler_Chases rows (chase 1 after 15 minutes of running time, one interval of running time after the restart) agree on how many ticks each takes."
---

Both entries name a plan file and the question to settle.
Neither pre-states a verdict or caps severity.
