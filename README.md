# Pump.fun Breakout Bot — "Surge Rider"

[![make.fun](https://make.fun/badge/u-riidzhsn-pump-fun-bot-sprint-make-bot-and-win-cb203fed.svg)](https://make.fun/bounty/u-riidzhsn-pump-fun-bot-sprint-make-bot-and-win-cb203fed)
[![make.fun · u-riidzhsn-pump-fun-bot-sprint-make-bot-and-win-cb203fed · rank](https://make.fun/badge/u-riidzhsn-pump-fun-bot-sprint-make-bot-and-win-cb203fed/greydi92/u-riidzhsn-pump-fun-bot-sprint-make-bot-and-win-cb203fed/rank.svg)](https://make.fun/bounty/u-riidzhsn-pump-fun-bot-sprint-make-bot-and-win-cb203fed/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=u-riidzhsn-pump-fun-bot-sprint-make-bot-and-win-cb203fed)
[![make.fun · u-riidzhsn-pump-fun-bot-sprint-make-bot-and-win-cb203fed · reward](https://make.fun/badge/u-riidzhsn-pump-fun-bot-sprint-make-bot-and-win-cb203fed/greydi92/u-riidzhsn-pump-fun-bot-sprint-make-bot-and-win-cb203fed/reward.svg)](https://make.fun/bounty/u-riidzhsn-pump-fun-bot-sprint-make-bot-and-win-cb203fed?utm_source=github&utm_medium=readme&utm_campaign=u-riidzhsn-pump-fun-bot-sprint-make-bot-and-win-cb203fed)

Join make.fun with referral code `qgn48q27`: https://make.fun/r/qgn48q27

A deterministic paper-trading bot for the pump.fun bonding-curve replay
([Pump.fun Bot Sprint: Make Bot And Win](https://make.fun/bounty/u-riidzhsn-pump-fun-bot-sprint-make-bot-and-win-cb203fed)).
It trades 10 SOL of paper money on a replay of Solana pump.fun activity,
ranked on net PnL after every fee.

> NOTE for the submitter: the two repo-specific badge lines
> (`.../rank.svg` and `.../reward.svg`) must be added once the GitHub
> repository exists, using its real owner/repo in the URLs.

## Strategy

**Sustained-activity scalping, sized small, exited fast — buys only.**

Offline scans of the kit's dev dataset (every trigger family tested with
faithful execution — 2-slot latency, 1.25% curve fee each way, tx fees, the
1%-of-reserve buy cap, market impact) showed that momentum chasing, whale
following, creator buying, wallet accumulation, dip-buying, early bursts,
big-buy following, and buying early and holding all lose money on the dev
window: latency plus fees eat every burst. The one family that holds up is a
volume spike with a flat price — a burst of trading before the move, not
after it — and the 40-slot sustained version beats the 20-slot burst: the
40-slot window is a clear sweet spot in the grid (30-slot windows lose,
50-slot windows lose much worse), and it is the only trigger positive in
both halves of the dev window. Round 2 added the one entry stat that
separates winners from losers: the window must be at least 65% buys with
real reserve non-falling — sustained buying is accumulation, a busy window
that is mostly sells is distribution, and distribution coins are the ones
that crash after entry (the filter cuts gross losses from −0.386 SOL to
−0.059 while keeping all three big winners).

1. **Watch every curve from its CreateEvent.** Each curve keeps a short tape
   of its trades (slot, side, SOL size, price).
2. **Enter only on sustained buying.** A curve qualifies when it printed
   at least 40 trades in the last 40 slots while its price moved less than
   2%, is younger than 2,000 slots, holds 5–30 SOL of real reserve
   (the 5 SOL floor drops fee-poisoned micro-caps), at least 65% of the
   window's trades were buys, and real reserve did not fall over the window.
3. **One small entry per coin.** 0.05 SOL, no pyramiding, no re-entry.
4. **Exit on rules, never on hope.** Take profit at +20% of cost, stop loss
   at −10%, max hold 200 slots, instant exit if the creator dumps.
5. **Guardrail pacing.** Tiny 0.01 SOL probe round trips on liquid curves
   keep the run on pace for the 20-coin / 0.01-SOL round-trip minimum.
6. **Drawdown guard.** New entries stop if equity falls 20% from its peak;
   everything flattens if equity falls to 7 SOL, keeping max drawdown far
   inside the 50% limit.

Dev grade (kit, `--driver process`): net PnL **+0.138 SOL**, max drawdown
0.2%, 36 round trips over 35 coins, win rate 19.4% — 8.5x the v4 grade
(+0.016 SOL; v3.1 −0.126, v1 momentum −1.17 SOL, v2 whale-follow −2.00 SOL,
kit pilot momentum −9.46 SOL, pilot reversion −3.89 SOL, pilot sniper
−0.47 SOL). The 90% interval is −0.026..+0.345 SOL, and the bot stays
positive (+0.103 SOL) even at 2x fees. Honest caveat: the top 3 coins made
88% of the gains (without them net PnL is −0.04 SOL), and a 20-copy stress
test of rebuilt dev windows has median −0.05 SOL with 18 of 20 losing money
— the dev score leans on this window's own coins. The filter still earns
its place: it cuts the crash tail (worst single loss −0.009 SOL vs v4's
−0.028) while keeping every big runner, and the final window is fresh 6h
data, where a near-breakeven-or-better dev profile with tiny drawdown is
the best risk-adjusted place to take that shot from.

Everything is a pure function of the batches the harness sends: no clock,
no randomness, no network, no environment reads — the two graded executions
produce byte-identical action logs.

## Run

The grader runs `node main.ts` (Node 22.18+). The protocol shim is vendored
at `vendor/makefun-bot.ts` (a byte-identical copy of the kit's
`sdk/ts/makefun-bot.ts`).

## License

MIT — see [LICENSE](LICENSE).
