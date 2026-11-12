# xcaret-oran-flooding-dataset

This repository holds a labeled three-tier O-RAN anomaly dataset, together with the feature roles and the multiclass detection results computed on it. We collected the data on an over-the-air O-RAN testbed with the OCUDU CU/DU protocol stack, an Ettus X310 software-defined radio as the RU, the Open5GS core network, and the FlexRIC Near-RT RIC. An Amarisoft UE simbox simulated both the benign and the attacker UEs. The dataset covers four flooding attacks at three tiers, namely the O-DU, the O-CU, and the Near-RT RIC. We used this data in the paper listed under Citation and license.

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
| `data/attack_<x>/combined/<ATK>_high_ric_50ms.parquet` | Labeled Near-RT RIC dataset for attack `<ATK>` (50 ms windows) |
| `analysis/feature_roles_multiclass_v1.csv` | Per-feature, per-tier detection and classification roles, and the feature presets (`in_<preset>` columns) |
| `analysis/portability_manifest_v2.csv` | Per-column keep/strip verdict (portable vs. testbed-specific) |
| `results/summary.csv` | Multiclass detection results (F1-score, recall, precision, inference cost, and model size) per tier, preset, and model |

## Names in the files and in the paper

The attack and tier names in the files differ from the names in the paper. The model and preset names are the same.

| Files | Paper | Description |
|---|---|---|
| `A1` | `UDP_DoS` | UDP flood from one attacker UE |
| `A2` | `SYN_DoS` | SYN flood from one attacker UE |
| `C1` | `UDP_DDoS` | UDP flood from three attacker UEs |
| `C2` | `SYN_DDoS` | SYN flood from three attacker UEs |
| `odu` | `du` | O-DU tier, 5 ms windows |
| `ocu` | `cu` | O-CU tier, 5 ms windows |
| `ric` | `ric` | Near-RT RIC tier, 50 ms windows |

## Data collection

We collected five runs of each attack. In each run, benign traffic ran for 120 s before the attack. The attack lasted 300 s, and benign traffic continued for about 120 s afterward. Four benign UEs with different traffic profiles were active in every run.

## Using the data

Each row holds the features of one UE (`rnti`) in one window (`window_start_ms`) of one run (`run`).

- **Rows.** The files also contain the windows right after the attack stopped, in which attack traffic still drains from the higher-layer buffers. Hence, keep only the rows with `is_attack_active | is_clean_normal`, as we did in the paper.
- **Labels.** The binary label is `is_attacker & is_attack_active`. For the multiclass label, the rows with a binary label of 1 take their attack type from `attack`, and all other rows are benign.
- **Metadata.** The columns `run`, `calibration`, `attack`, `intensity`, `rnti`, `window_start_ms`, `time_epoch`, `is_attacker`, `is_attack_active`, and `is_clean_normal` are not features. The same holds for `ue_ip` in the O-CU files and for `ue_index` and `e2_delay_ms` in the Near-RT RIC files.
- **Missing values.** A missing value means that the metric was not available in that window (e.g., for an idle UE). It is not a zero.
- **Splits.** In the paper, we used five-fold leave-one-run-out splits on `run`, with one test run and one validation run per fold.

```python
import pandas as pd

df = pd.read_parquet("data/attack_c1/combined/C1_high_ric_50ms.parquet")
df = df[df.is_attack_active | df.is_clean_normal]
y_binary = df.is_attacker & df.is_attack_active
y_multiclass = df.attack.where(y_binary, "benign")
```

## Feature presets

The `full` preset uses all features of a tier in `feature_roles_multiclass_v1.csv`. The `portable` preset removes the testbed-specific features (`is_strip` is `True`, the same as the `strip` verdict in `portability_manifest_v2.csv`). The other presets keep the portable features whose `in_<preset>` column is `True`.

| Tier | `full` | `portable` | `screened` | `informative` | `cause_effect` | `discriminating` |
|---|---|---|---|---|---|---|
| `odu` | 37 | 29 | 14 | 23 | 14 | 14 |
| `ocu` | 27 | 27 | 24 | 19 | 13 | 11 |
| `ric` | 51 | 43 | 32 | 18 | 10 | 7 |

## Results

`results/summary.csv` has one row per tier, preset, and model. The performance columns give the mean and the standard deviation over the five folds, repeated with three seeds for the stochastic models. The cost columns are the median batch inference time per row (`infer_us_batch`, µs) with its standard deviation (`infer_us_batch_std`), the median online inference time (`infer_us_online`, µs) with its 95th and 99th percentiles (`infer_us_online_p95`, `infer_us_online_p99`), the model size (`model_kb`, kB), and the runtime memory footprint (`model_rss_kb`, kB). The paper reports only the `_low` and `_high` model variants and `LogReg`. Therefore, the rows of the other models (e.g., `XGBoost` and `LSTM`) are not used in the paper.

The data-collection and modeling code is not part of this repository. Contact the authors for access.

## Citation and license

The data and the results are archived on Zenodo at [10.5281/zenodo.24173687](https://doi.org/10.5281/zenodo.24173687).

If you use the data or the results in a publication, please cite the paper:

D. Attanayaka, Y. Rumesh, P. Porambage, and S. Samarakoon, "Intelligent Security Orchestration in Open RAN with Anomaly Detection Model Analysis," in *Proc. IEEE Consumer Communications & Networking Conference (CCNC)*, 2027.

For questions, please contact Dinaj Attanayaka (dinaj.attanayaka@vtt.fi).

The data and the results are under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) (see [LICENSE](LICENSE)).
