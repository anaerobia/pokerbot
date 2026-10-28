# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this is

A heads-up No-Limit Texas Hold'em engine, a ladder of bots, and a teaching guide
(`docs/01-05`). Pure Python standard library — no dependencies to install.
Run everything from the repository root.

```bash
python -m pokerbot play --bot equity                       # human vs bot
python -m pokerbot match equity rule --hands 2000 --seed 1 # bot vs bot
python -m pokerbot tournament --hands 500                  # round robin
python -m pokerbot equity AhKh --board "Qh Jh 2d"          # equity calculator
python -m pokerbot.examples.kuhn_cfr                       # CFR demo
python -m unittest discover -s pokerbot/tests -t .         # tests
```

## Architecture

- `engine.py` owns cards/chips; calls `Bot.get_action(GameView)` and sanitises the result (illegal → check/fold, raise amounts clamped). `RaiseAction(amount)` is "raise **to**". Button = small blind, acts first preflop and last postflop. Stacks reset to 200 each hand; profit tracked as bankroll. `run_match` alternates the button and reports bb/100 with a standard error.
- `evaluator.py`: `evaluate(5-7 cards)` → comparable tuple `(category, tiebreaks...)`; works on rank/suit counts, not subset enumeration.
- `equity.py`: Monte Carlo/exact equity, Chen score, range filters (`chen_range`, `board_range`), `count_outs`.
- `bots/`: `random`, `station`, `rule` (Chen + hand tiers), `equity` (MC equity vs modelled range + pot odds + `OpponentModel`), `human`. `make_bot(spec, seed)` also loads `path/file.py:ClassName` and passes `seed` only if the bot accepts it.
- CLI seeds the deck with `--seed` and bots with `--seed`/`--seed + 1`, so runs are reproducible.
- Tests use stdlib `unittest` and must run from the repo root (they load example bots by relative path).
- Numbers quoted in `README.md` and `docs/` come from real runs with the seeds shown; re-run and update them if bot logic changes.
