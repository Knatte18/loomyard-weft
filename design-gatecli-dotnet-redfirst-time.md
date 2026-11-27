# TestGateRedFirst_Dotnet sets the floor of the gatecli tier1 time at 31 to 47 s

`gatecli-dotnet-redfirst-time` · Note · enhancement, gate

## Seen

The landed split of the long gatecli tier1 scenario tests made `TestGateTest_Scopes` and `TestGateTest_Scenario` parallel; their longest rows now take 2.9 to 7.3 s.
The package time did not fall: `TestGateRedFirst_Dotnet` took 31.0 and 38.9 s before and 47.5 and 44.4 s after (the after runs ran at a load of 31 and 46),
and `TestGateTest_DotnetFixture` takes about 21 s. The figures are in the landing commit's Measurements section.

## Idea

Find where `TestGateRedFirst_Dotnet` spends its time (dotnet restore and build, likely) and cut it, for example by sharing one built .NET fixture across its rows or with `TestGateTest_DotnetFixture`.
