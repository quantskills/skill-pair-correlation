# skill-pair-correlation

Two-symbol relationship skill: given two tickers, it computes in one call the return correlation (full window + recent 20/60d), the hedge beta (A on B), and the latest log-spread z-score for pairs trading / hedging / correlation checks, routing each leg independently across A-share / HK / US.

> This first paragraph doubles as the platform summary (suggested Topics: `quant` `pairs-trading` `correlation` `hedge-ratio` `mean-reversion` `a-share` `hk` `us`).

> 中文说明见 [`README.md`](./README.md)（中文优先）。

## What it does

Each leg's daily closes are fetched independently from `panda_data` (A-share) /
`tqx_data` (HK / US), then **inner-joined on shared trading dates before any
statistic** — so a mismatched calendar or a halt day can't silently misalign the two
return series (the classic pairs bug). Returns correlation, hedge beta, and a
ready-to-read spread z-score as a JSON string.

Full spec — parameters, output schema, formulas, when-NOT-to-use — is in [`SKILL.md`](./SKILL.md).

## Layout

```
skill-pair-correlation/
├── SKILL.md                  # required: skill spec
├── README.md                 # Chinese-first readme (first paragraph = summary)
├── README.en.md              # this file (English)
├── LICENSE                   # GPLv3
├── quantskills.yaml          # upstream manifest (license: GPL-3.0-only)
├── agents/                   # runtime adapters (cursor / hermes / openclaw)
│   ├── cursor-rule.mdc
│   ├── openai.yaml
│   └── portable-loader.md
├── scripts/
│   └── pair_correlation.py   # executable: async def run(...) -> str + CLI
└── references/
    └── example_output.md     # example output
```

## Run it

**As a platform skill** — entry point `async def run(...) -> str` in
`scripts/pair_correlation.py` (Panda QuantFlow skill contract).

**Standalone CLI:**

```bash
python scripts/pair_correlation.py 0700.HK 9988.HK --lookback-days 250
python scripts/pair_correlation.py 600519.SH 000858.SZ --zscore-window 60
```

Dependencies: `pandas`, `numpy` + `panda_data` / `tqx_data`. Missing data libs → a
structured `Error: …` string, never a raise.

## Status

Listed → Runnable → Verified. Local checks passing: `py_compile`, frontmatter lint,
`run()` signature ↔ parameters match, and a synthetic-data smoke test (alignment,
correlation bounds, degenerate-leg guard, empty-symbol rejection).

## Disclaimer

- **Research & educational use only.** Output is informational research, **not investment advice**, and makes **no promise of returns** or profitability.
- **Data sources:** A-share via `panda_data`; HK / US via `tqx_data` (platform-provided). Output quality depends entirely on upstream data availability and correctness.
- **Assumptions & limitations:** the two legs are inner-joined on shared trading dates first; correlation and hedge beta use daily simple returns; spread = `ln(Pa) − beta·ln(Pb)`. Correlation/beta are historical and unstable out-of-sample; a degenerate leg (0 variance) returns `null`. Cross-market pairs shrink the aligned sample due to calendar differences.
- **Risk boundary:** do not use as the sole basis for real-money pair/hedge decisions; validate independently and understand market risk.

## License

GPL-3.0-only. See [`LICENSE`](./LICENSE).
