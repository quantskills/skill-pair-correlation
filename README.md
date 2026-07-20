# skill-pair-correlation

两标的关系 skill：给两个股票代码，一次算出收益相关系数（全窗口 + 近 20/60 日）、对冲 beta（A 对 B）、以及最新对数价差 z-score，用于配对交易 / 对冲 / 相关性判断，每条腿独立按后缀路由 A股 / 港股 / 美股。Two-symbol relationship (correlation, hedge beta, spread z-score) for pairs trading across and within A-share / HK / US.

> 首段即平台简介（Topics 建议：`quant` `pairs-trading` `correlation` `hedge-ratio` `mean-reversion` `a-share` `hk` `us`）。

## What it does

Each leg's daily closes are fetched independently from `panda_data` (A 股) /
`tqx_data` (港股 / 美股), then **inner-joined on shared trading dates before any
statistic** — so a mismatched calendar or a halt day can't silently misalign the two
return series (the classic pairs bug). Returns correlation, hedge beta, and a
ready-to-read spread z-score as a JSON string.

Full spec — parameters, output schema, formulas, when-NOT-to-use — is in [`SKILL.md`](./SKILL.md).

## Layout

```
skill-pair-correlation/
├── SKILL.md                  # 必须：技能说明
├── README.md                 # 本文件（首段=简介）
├── scripts/
│   └── pair_correlation.py   # 可执行代码：async def run(...) -> str + CLI
└── references/
    └── example_output.md     # 示例输出
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
