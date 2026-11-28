# Coordinating a resumed run with every batch already done

This session started with all batches recorded `done`, so it forked nothing and read no digest.
The summary contract asks for each batch's deviations, departures and scope guard result from the digests,
but the prompt gives a resumed Master no way to read the digests of batches an earlier session recorded.
`lym webster status` reports only that each digest exists.
