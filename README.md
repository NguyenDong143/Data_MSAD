# Data Directory

> **Purpose:** Hold all MSAD pipeline data across 3 layers `raw → interim → processed`,
> plus UBCKNN event labels kept separate to prevent label leakage.

## Committed layout

```text
data/
├── raw/                       # Raw CSVs, NEVER edit by hand
│   ├── market/                #   16 market CSVs (15 tickers + VNI.csv benchmark)
│   ├── control/               #   9 control CSVs (8 single tickers + control_universe.csv pooled panel)
│   └── news/                  #   5 raw news CSVs (Baodautu, CafeF, DNSE, Fireant, Vneconomy)
├── interim/                   # Reproducible from raw via pipeline Step 1
│   ├── market/                #   *_clean.csv / *_synthetic_clean.csv (ffilled, tail-trimmed, VNI + sentiment fused)
│   ├── control/               #   *_control_clean.csv
│   ├── news/                  #   news_daily_sentiment.csv
│   └── cleaning_summary.csv   #   Per-ticker raw → cleaned row counts
├── processed/                 # Reproducible from interim via pipeline Step 2
│   ├── market/                #   21 features per ticker (order flow, price, market context, sentiment)
│   ├── control/               #   21 features per control ticker
│   └── feature_engineering_summary.csv
└── labels/                    # UBCKNN + internal aggregated labels (9 CSVs)
    ├── market_events.csv
    ├── market_events_expanded.csv
    ├── events_master_normalized.csv
    ├── event_daily_labels_normalized.csv
    ├── event_windows_normalized.csv
    ├── pseudo_labels.csv
    ├── synthetic_events.csv
    ├── synthetic_extreme_events.csv
    └── news_weak_events.csv
```



## Conventions

- **Raw (**`raw/`**)**: never edit by hand. All transforms go through the `src/data/` pipeline.
The pooled `control_universe.csv` panel must NOT be cleaned/modeled as a single ticker (see `is_pooled_control_panel()`).
- **Intermediate (**`interim/`**,** `processed/`**)**: reproducible by re-running the pipeline from `raw/`.
- **Labels (**`labels/`**)**: UBCKNN + internal aggregated events. Kept separate from features to avoid label leakage.
