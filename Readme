# Compound Hot–Dry–Windy (CHDW) Events across the Contiguous United States

County-level thresholds, optimal paths and event catalogs for daily compound hot–dry–windy (CHDW) events in all 3108 counties of the contiguous United States, 1980–2025, identified with the Optimal Path Threshold (OPT) method.

This repository accompanies:

> Zhao, B., & Horvat, C. *When is a compound extreme really compound? Co-occurrence and change in hot–dry–windy events across the United States.* (in review)

The OPT method is described in:

> Zhao, B., Horvat, C., & Gao, H. (2025). An optimal path threshold method for rigorously identifying extreme climate events. *Environmental Research Letters*, 20(2), 024048. 

---

## Contents

```
data/
├── full_1980-2025/          # thresholds calibrated on the full record (primary analysis)
│   ├── thresholds.parquet
│   ├── path.parquet
│   └── events.parquet
├── historical_1980-1999/    # thresholds calibrated on the historical window
│   ├── thresholds.parquet
│   ├── path.parquet
│   └── events.parquet
└── recent_2006-2025/        # thresholds calibrated on the recent window
    ├── thresholds.parquet
    ├── path.parquet
    └── events.parquet
```

The three folders have identical file structures. They differ only in the reference (calibration) period on which the OPT thresholds are estimated:

| Folder | Reference period | Days in reference period | Role in the paper |
|---|---|---|---|
| `full_1980-2025` | 1980-01-01 – 2025-12-31 | 16,802 | Primary analysis |
| `historical_1980-1999` | 1980-01-01 – 1999-12-31 | 7,305 | Reference-period sensitivity |
| `recent_2006-2025` | 2006-01-01 – 2025-12-31 | 7,305 | Reference-period sensitivity |

---

## Input data and variables

- **Meteorology:** gridMET daily surface meteorology (~4 km; Abatzoglou, 2013).
- **Spatial unit:** each county is represented by the gridMET cell containing its centroid (3108 counties).
- **Variables** (all three enter as upper tails):

| Symbol | Variable | Unit | Definition |
|---|---|---|---|
| T | Daily maximum 2-m air temperature | °C | gridMET `tmmx` |
| D | Relative-humidity deficit | % | D = 100 − RH_mean, with RH_mean from T_max, T_min, RH_max and RH_min following FAO-56 (Allen et al., 1998) |
| W | Daily 2-m wind speed | m s⁻¹ | gridMET 10-m wind speed converted to 2 m with the FAO-56 log profile (factor 0.748) |

A **CHDW day** is a day on which T, D and W all reach or exceed their county-specific thresholds.

Four severities are provided:

| `severity` label | Target probability p_t | Approx. event days per year |
|---|---|---|
| `10permil` | 0.010 | 3.65 |
| `5permil` | 0.005 | 1.83 (primary level in the paper) |
| `1permil` | 0.001 | 0.37 |
| `0.5permil` | 0.0005 | 0.18 |

---

## File descriptions

### `thresholds.parquet`: one row per county × severity

3108 counties × 4 severities = 12,432 rows.

| Column | Type | Description |
|---|---|---|
| `FIPS` | string | 5-digit county FIPS code |
| `severity` | string | `10permil`, `5permil`, `1permil`, `0.5permil` |
| `p_target` | float | Target joint exceedance probability p_t |
| `p_achieved` | float | Joint exceedance probability at the selected path node (reference period) |
| `n_base_events` | int | Number of CHDW days in the reference period (= `p_achieved` × `n_base_days`) |
| `n_base_days` | int | Number of days in the reference period |
| `path_step` | int | Index of the selected node in `path.parquet` |
| `a_T`, `a_D`, `a_W` | float | Threshold percentile (non-exceedance probability) of each variable, 0.50–0.99 |
| `thr_T` | float | Temperature threshold (°C) |
| `thr_D` | float | Relative-humidity-deficit threshold (%) |
| `thr_W` | float | 2-m wind-speed threshold (m s⁻¹) |
| `q_T`, `q_D`, `q_W` | float | Single-variable exceedance probability, q_i = 1 − a_i |
| `r_T`, `r_D`, `r_W` | float | Single-variable rarity, r_i = −ln q_i |
| `R` | float | Joint rarity, R = −ln p_achieved |
| `lnC` | float | ln of the co-occurrence amplification, ln C = r_T + r_D + r_W − R |
| `C` | float | Co-occurrence amplification, C = p_achieved / (q_T q_D q_W); C > 1 means the three exceedances coincide more often than under independence, C < 1 less often |
| `phi_T`, `phi_D`, `phi_W` | float | Share of the total single-variable rarity carried by each variable, φ_i = r_i / (r_T + r_D + r_W) |
| `frechet_ok` | bool | `p_achieved` lies within the Fréchet bounds, max(0, q_T + q_D + q_W − 2) ≤ p ≤ min(q_T, q_D, q_W) |
| `r_le_R_ok` | bool | Each single-variable rarity does not exceed the joint rarity (r_i ≤ R) |

The rarity budget R = r_T + r_D + r_W − ln C holds exactly for every row.

### `path.parquet`: the full OPT path for each county

3108 counties × 148 steps = 459,984 rows. The path runs from node (0, 0, 0) to node (49, 49, 49), and each step increases exactly one index by one.

| Column | Type | Description |
|---|---|---|
| `FIPS` | string | 5-digit county FIPS code |
| `step` | int | Position along the path, 0 (mildest) to 147 (most extreme) |
| `i`, `j`, `k` | int | Grid indices of the T, D and W thresholds (0–49) |
| `a_T`, `a_D`, `a_W` | float | Corresponding percentiles, a = 0.50 + 0.01 × index |
| `p` | float | Empirical joint exceedance probability at this node in the reference period |

`p` is non-increasing along the path. The path can be used to read thresholds at any severity not listed in `thresholds.parquet`, and to inspect how OPT distributes rarity among the three variables.

### `events.parquet`: catalog of CHDW days

One row per county × event day × severity.

| Column | Type | Description |
|---|---|---|
| `FIPS` | string | 5-digit county FIPS code |
| `date` | int | Event date as `YYYYMMDD` |
| `severity` | string | Severity whose thresholds the day meets |
