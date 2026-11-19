# Arbiter

A project in development for comparing time series forecasting models reproducibly. It evaluates models under the same conditions and produces rankings by horizon, supporting the decision of whether to replace one model with another.

## Structure

```text
arbiter/
├── .github/
│   └── workflows/
├── README.md
├── pyproject.toml
├── requirements.txt
├── .gitignore
├── src/
│   └── arbiter/
│       ├── ingestion/
│       ├── validation/
│       ├── features/
│       ├── models/
│       ├── backtest/
│       ├── metrics/
│       ├── reporting/
│       └── config.py
├── tests/
├── docs/
└── data/
```
