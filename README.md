# xcaret-multiclass-ad-analysis

Supplementary data and results for the paper: "Intelligent Security Orchestration in Open RAN with Anomaly Detection Model Analysis"

## Repo structure

```
data/
├── attack_a1/combined/
├── attack_a2/combined/
├── attack_c1/combined/
└── attack_c2/combined/

analysis/
├── feature_roles_multiclass_v1.csv
└── portability_manifest_v2.csv

results/
└── summary.csv
```

## File descriptions

| File | Description |
|---|---|
| `data/attack_<x>/combined/<ATK>_high_odu_5ms.parquet` | Labeled O-DU dataset for attack `<ATK>` (5 ms windows) |
| `data/attack_<x>/combined/<ATK>_high_ocu_5ms.parquet` | Labeled O-CU dataset for attack `<ATK>` (5 ms windows) |
| `data/attack_<x>/combined/<ATK>_high_ric_50ms.parquet` | Labeled RIC dataset for attack `<ATK>` (50 ms windows) |
| `analysis/feature_roles_multiclass_v1.csv` | Per-feature, per-tier role assignment (detection/classification role) |
| `analysis/portability_manifest_v2.csv` | Per-column keep/strip verdict (testbed-specific vs. portable) |
| `results/summary.csv` | Multiclass detection results (F1/recall/precision, inference cost, model size) per tier/preset/model |
