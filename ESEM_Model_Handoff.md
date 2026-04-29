# ESEM Generic Grid Model — Context Handoff Document
**For use when continuing this work in a new Claude session**
**Model version: v6 | File: ESEM_v6.xlsx**
**Last updated: April 2026**

---

## 1. What this model is

A stylised Excel-based policy communication tool built for the DCCEEW NEM Review workstream. It simulates the Australian National Electricity Market (NEM) transition from 2025–2050 (26 years, t=0 to t=25) under ESEM — the Electricity Security and Emissions Mechanism.

ESEM is modelled as a **government market-maker**: it signs Contracts for Difference (CfDs) with generators, holds the contracts until assets are built, then divests offtake agreements to retailers. Its purpose is to solve the missing money problem — bridging the gap between merchant wholesale prices (which may not cover LRMC as VRE penetrates) and investment requirements.

This is a **stylised** model — not a dispatch model. No hourly resolution. Annual averages throughout. Designed for policy communication, not engineering.

---

## 2. File structure — 4 sheets

| Sheet | Purpose |
|---|---|
| **Assumptions** | All parameters. Active value in col C via `=CHOOSE($C$4,...)`. Four scenarios: I=Base, J=High WACC, K=No ESEM, L=WACC disc. |
| **Model** | 26-column time series (cols C–AB, t=0 to t=25). All calculations here. |
| **Building assets** | Derives LRMC for VRE+storage, firming, and coal from first principles. Feeds Model. |
| **Dashboard/Charts** | Outputs (not detailed in this document). |

---

## 3. Scenario structure

| Scenario | Col | ESEM_On | Key differences |
|---|---|---|---|
| Base | I | 1 | Reference case |
| High WACC | J | 1 | Higher financing costs across all technologies |
| No ESEM | K | **0** | Counterfactual — merchant market only |
| WACC disc | L | 1 | ESEM_WACC_Disc = 0.02; lower LRMC for ESEM projects |

Active scenario controlled by `Assumptions!C4` (currently = 1). Change this to switch scenarios.

---

## 4. Known bugs / stale named ranges (need fixing)

These named ranges exist but point to `#REF!` — the cells they referenced were removed:
- `Drought_Gap` → `Assumptions!#REF!` (user removed Drought_Gap for simplicity)
- `Fwd_Alpha` → `Assumptions!#REF!` (removed, replaced by logistic Fwd_k)
- `Other_Gen` → `Assumptions!#REF!` (may have been moved/removed)

**Action needed**: Delete these three stale named ranges via Excel Name Manager, or update them to valid cells.

Also:
- **R64 ESEM_On fix went to wrong row**: The CHOOSE formula was written to R64 col C (which is the section header "7. ESEM DESIGN"), not R65 col C (which is the actual `ESEM_On` parameter row — still hardcoded at 1). The `ESEM_On` named range points to `Assumptions!$C$65`. Fix: delete the formula from R64C and put `=CHOOSE($C$4,I65,J65,K65,L65)` in R65C with scenario values I=1, J=1, K=0, L=1.

- **R23 Model VRE_PrePipe bug**: Col D formula reads `=VRE_Start+VRE_PrePipe*MIN(D2,Lag)+D43+D40` — the `*MIN(D2,Lag)` term makes VRE_PrePipe grow from 3 to 9 GW over the first 3 years. Should be `=VRE_Start+VRE_PrePipe+D43+D40` (constant). This was fixed in earlier sessions but appears to have reverted. Check all columns D onwards.

---

## 5. Assumptions — full parameter list with base values

### Section 1 — Demand & Supply
| Row | Name | Base | Unit | Source |
|---|---|---|---|---|
| R7 | Dem_Start | 180 | TWh | AEMO 2024 |
| R8 | Dem_Growth | 0.025 | %/yr | AEMO ISP 2024 |
| R9 | Peak_Start | 36 | GW | AEMO 2024 |
| R10 | Peak_Growth | 0.02 | %/yr | Own estimate |
| R11 | Other_Firm | 0 | GW | — |
| R12 | VRE_PrePipe | 3 | GW | AEMO 2024 — VRE under construction at t=0 |

### Section 2 — Coal Exit
| Row | Name | Base | Unit | Source |
|---|---|---|---|---|
| R14 | Coal_Start | 22 | GW | AEMO 2024 |
| R15 | Coal_CFe | 0.45 | — | AEMO 2024 |
| R16 | Coal_CFp | 0.75 | — | Own estimate |
| R17 | Coal_SRMC | 80 | $/MWh | AEMO 2024 — reconciles to Building Assets Section E |
| R18 | Coal_Esc | 0.04 | %/yr | Own estimate |
| R19 | Coal_Accel | 0.15 | — | Calibrated — economic exit acceleration |

### Section 3 — Existing Dispatchable Firming
| Row | Name | Base | Unit | Source |
|---|---|---|---|---|
| R21 | Firm_Start | 22 | GW | AEMO 2024 |
| R22 | Firm_Decay | 0 | %/yr | Own estimate |
| R23 | Firm_CFe | 0.22 | — | Own estimate (used for LRMC calc) |
| R24 | Firm_CFp | 0.9 | — | Own estimate (peak availability) |
| R25 | Firm_SRMC | 180 | $/MWh | Own estimate |
| R26 | Firm_VC | 120 | $/MWh | Own estimate (fuel + variable O&M) |
| R27 | Firm_LRMC | — | $/MWh | Computed from Building Assets R92 (Year 1) |
| R28 | Firm_Sprd | 60 | $/MWh | Own estimate — private entry threshold above LRMC |
| R29 | Firm_Tight | 3.5 | — | Calibrated — scarcity price exponential coefficient |
| R30 | Firm_Gen_CF | 0.05 | — | AEMO ISP 2024 — baseline gas peaking CF (~5% annual potential) |

### Section 4 — VRE + Storage
| Row | Name | Base | Unit | Source |
|---|---|---|---|---|
| R32 | VRE_Start | 30 | GW | AEMO 2024 |
| R33 | CF_Solar | 0.22 | — | GenCost 2024 |
| R34 | CF_Wind | 0.35 | — | GenCost 2024 |
| R35 | Alpha_Solar | 0.67 | — | Own estimate — solar share of VRE portfolio |
| R36 | Alpha_Wind | =1−Alpha_Solar | — | Computed |
| R37 | VRE_CFe | =CF_Solar×Alpha_Solar+CF_Wind×Alpha_Wind | — | Computed blended CF |
| R38 | ELCC_base | 0.15 | — | AEMO ISP 2024 — falls with VRE penetration |
| R39 | VRE_ELCC | =ELCC_base×(1−VRE_Share0) | — | **Endogenised** — ELCC at current penetration ~10%, falls to ~3% at 80% |
| R40 | VRE_Ratio | 3 | ratio | Own estimate — kW VRE per kW storage |
| R41 | VRE_Sprd | 4 | $/MWh | Calibrated — private VRE entry spread above forward price |
| R42 | VRE_Dep | 0 | %/yr | Own estimate |
| R43 | Direct_Flow | 0.8 | — | Own estimate — VRE fraction going direct to load (bypasses battery) |
| R44 | Battery_RTE | 0.9 | — | GenCost 2024 — round-trip efficiency |
| R45 | Y1_Build | 2 | GW | — |

### Section 4b — Battery Duration Mix (weights must sum to 1)
| Row | Name | Base | Notes |
|---|---|---|---|
| R47 | Batt_w1h | 0.1 | 10% of portfolio is 1h batteries |
| R48 | Batt_w2h | 0.2 | |
| R49 | Batt_w4h | 0.5 | Dominant tier |
| R50 | Batt_w8h | 0.2 | |

### Section 5 — Financing (WACC)
| Row | Name | Base | High WACC | WACC disc |
|---|---|---|---|---|
| R53 | Batt_WACC | 0.08 | 0.11 | 0.06 |
| R54 | VRE_WACC_S | 0.07 | 0.10 | 0.05 |
| R55 | VRE_WACC_W | 0.08 | 0.11 | 0.06 |
| R56 | Gas_WACC | 0.09 | 0.12 | 0.07 |
| R57 | ESEM_WACC_Disc | 0 | 0 | 0.02 | — WACC reduction for ESEM-backed CfD projects |

### Section 6 — Price Formation
| Row | Name | Base | Notes |
|---|---|---|---|
| R59 | Surp_Elas | 4 | WS sensitivity to surplus |
| R60 | Def_Elas | 5 | WS sensitivity to deficit |
| R61 | Fwd_k | 3 | Logistic steepness for forward price weighting |
| R62 | Curtail_Thresh | =VRE_Share0 | **Endogenised** — curtailment starts at current VRE penetration |
| R63 | Curtail_Rate | 0.15 | Curtailment per unit VRE share above threshold |

### Section 7 — ESEM Design
| Row | Name | Base | Notes |
|---|---|---|---|
| R65 | ESEM_On | 1 | **BUG**: still hardcoded; needs CHOOSE formula (see Section 4 above) |
| R66 | ESEM_Start | 2 | Year index when ESEM begins contracting |
| R67 | VRE_ShareT | 1.0 | Target VRE share of demand (100%) |
| R68 | RE_Cov | 1.0 | ESEM coverage of VRE shortfall |
| R69 | Firm_Cov | 0.9 | ESEM coverage of firming shortfall |
| R70 | Lag | 3 | Years from contracting to online |
| R71 | Recycle_Lag | 7 | Years ESEM holds before divesting |
| R72 | Contract_Years | 20 | Total contract life |
| R73 | Divest_Rate | =1/(Contract_Years−Recycle_Lag) | **Computed** — ~7.7%/yr at base |
| R74 | ESEM_Build_Cap | 6 | Max GW ESEM can contract per year |
| R75 | Markup_Base | 0.01 | Base procurement markup above LRMC (flat, all contracts) |
| R76 | Markup_Rate | 0.02 | Supply curve steepness — additional markup per GW contracted (VRE only) |

### Section 8 — Contract Economics
| Row | Name | Base | Notes |
|---|---|---|---|
| R78 | Admin_Cost | −37 | $m/yr — negative = net revenue to ESEM in base |
| R79 | VRE_Share0 | =VRE_Start×VRE_CFe×8.76/Dem_Start | **Computed** — ~32% in base; used for target ramp and Curtail_Thresh |

### Section 9 — Computed / Endogenised (no colour)
| Row | Name | Value | Notes |
|---|---|---|---|
| R83 | Coal_Heat_Rate | 8.547 GJ/MWh | GenCost 2025-26 Table B.9 black coal (efficiency 42.12%) |
| R84 | Coal_FOM | $64.85/kW/yr | GenCost 2025-26 Table B.9 black coal |
| R85 | Coal_VOM | $4.68/MWh | GenCost 2025-26 Table B.9 black coal |
| R86 | Coal_Price | $8.81/GJ | Back-calculated: (80−4.68)/8.547. GenCost uses $3/GJ (pre-2022); $8.81 reflects post-2022 AUS domestic pricing |
| R87 | Coal_Degrade | 0.015/yr | Own estimate — reliability degradation from ageing + cycling |
| R88 | Batt_OM_1h | $7/kW/yr | Own estimate, NREL ATB 2024 range (routine only; excludes augmentation) |
| R89 | Batt_OM_2h | $8/kW/yr | |
| R90 | Batt_OM_4h | $10/kW/yr | |
| R91 | Batt_OM_8h | $13/kW/yr | |
| R92 | Solar_OM | $12/kW/yr | GenCost 2025-26 Table B.9 |
| R93 | Wind_OM | $29/kW/yr | GenCost 2025-26 Table B.9 |
| R94 | OCGT_OM_Small | $17/kW/yr | GenCost 2025-26 Table B.9 |
| R95 | OCGT_OM_Large | $26/kW/yr | GenCost 2025-26 Table B.9 |

---

## 6. Model sheet — row-by-row

26 columns: col C = t=0 (Year 1) through col AB = t=25 (Year 26).

### Demand (R5–R11)
- **R8** Base energy demand: `=Dem_Start×(1+Dem_Growth)^t`
- **R9** Effective demand: `=R8×(1+demand_shock)` (shock reserved at 0)
- **R10** Base peak: `=Peak_Start×(1+Peak_Growth)^t`
- **R11** Effective peak: `=R10`

### Coal (R12–R16)
- **R13** Coal SRMC: reads from Building Assets Section E derived SRMC (escalating). At t=0 anchored to Coal_SRMC=$80.
- **R14** Coal capacity: `=MAX(0, prev×(1−Coal_Accel×MAX(0,(SRMC−WS)/SRMC)))` — economic exit when WS < SRMC.
- **R15** Coal generation: `=capacity×Coal_CFe_t×8.76` using time-varying CFe from Building Assets R110 (degrades at 1.5%/yr).
- **R16** Coal firming availability: `=capacity×Coal_CFp_t` using time-varying CFp from BA R111.

### Legacy Firming (R17–R20)
- **R18** Legacy firm capacity: `=Firm_Start×(1−Firm_Decay)^t`
- **R19** Legacy firm peak: `=R18×Firm_CFp`
- **R20** Legacy firm generation: `=R18×Firm_CFe×8.76`

### VRE + Storage (R21–R24)
- **R22** VRE+Storage LRMC (curtailment-adjusted): reads Building Assets R78, divided by `(1−prior_year_R28)`. Standard LRMC for private investment signal.
- **R23** VRE online: `=VRE_Start+VRE_PrePipe+ESEM_stock+private_stock` — **NOTE: BUG in col D+ still has MIN(t,Lag) multiplier on VRE_PrePipe. Should be VRE_PrePipe constant.**
- **R24** VRE generation: `=R23×VRE_CFe×8760×(1−curtailment)`

### Price Formation (R25–R32)
- **R26** Energy balance: `=Coal_gen+VRE_gen+(legacy+ESEM_firm+private_firm)×Firm_Gen_CF×8760+Other_Gen−demand`
  - Firm generation included at 5% baseline CF per AEMO ISP 2024
  - Positive = surplus, negative = deficit
- **R27** Wholesale spot price: `=LRMC×EXP(elasticity×|balance/demand|)`. Surplus → WS < LRMC. Deficit → WS > LRMC. Anchored to LRMC not coal SRMC.
- **R28** Curtailment fraction: `=MAX(0,(VRE_share−Curtail_Thresh)×Curtail_Rate)`. Curtail_Thresh = VRE_Share0 (endogenised).
- **R29** Forward price: `=α×LRMC+(1−α)×WS` where `α=1/(1+EXP(−Fwd_k×balance/demand))`. Logistic weighting — surplus shifts toward LRMC, deficit shifts toward WS.
- **R30** Firming balance: `=coal_peak+legacy_peak+ESEM_firm_peak+VRE×ELCC+Other_Firm−peak_demand`
- **R31** Firming deficit fraction: `=MAX(0,−R30/peak_demand)`
- **R32** Scarcity price: `=Firm_SRMC×EXP(Firm_Tight×deficit_fraction)`

### Market Entry (R33–R41)
- **R34** VRE target share: linear ramp from VRE_Share0 (32%) to VRE_ShareT (100%) over 25 years.
- **R35** VRE target GW: `=target_share×demand/(VRE_CFe×8760×MAX(1−curtailment,0.01))` — **grossed up for curtailment** so target accounts for losses.
- **R36** Current VRE share: `=VRE_gen/demand`
- **R37** Private VRE build: `=MAX(0,(forward_price−LRMC)/VRE_Sprd)` — supply curve entry.
- **R38** ESEM VRE contracted: `=IF(ESEM_On×(t≥ESEM_Start), MAX(0,(target_GW−online_GW)×RE_Cov), 0)` — capped at ESEM_Build_Cap.
- **R39** Firming shortfall: `=MAX(0,peak_demand−(coal_peak+legacy_peak+VRE×ELCC+Other_Firm))`
- **R40** Private VRE cumulative stock: accumulator with VRE_Dep depreciation.
- **R41** ESEM firming contracted: `=IF(ESEM_On, MAX(0,shortfall×Firm_Cov), 0)`

### Cumulative Stocks (R42–R44)
- **R43** ESEM VRE stock: `=prev×(1−VRE_Dep)+prev_year_R38` (Lag year delayed)
- **R44** ESEM firming stock: `=prev+R41`

### ESEM Cost Recovery (R45–R53)
This section tracks what ESEM locked in at contracting, portfolio-weighted.

- **R46** VRE settled stock (GW): `=prev×(1−Divest_Rate)+cohort_from_Lag_years_ago`
- **R47** VRE strike accumulator ($/MWh·GW): `=prev×(1−Divest_Rate)+cohort_GW×ESEM_LRMC×(1+Markup_Base+Markup_Rate×cohort_GW)`. Uses **ESEM LRMC from Building Assets R81** (WACC-discounted), not standard LRMC.
- **R48** VRE blended strike: `=R47/R46`. Falls back to ESEM_LRMC when no stock.
- **R49** Private firm build: `=MAX(0,(scarcity−Firm_LRMC_t)/Firm_Sprd)` — uses time-varying Firm_LRMC from BA R92.
- **R50** Private firm cumulative stock: accumulator with Firm_Decay.
- **R51** Firm settled stock (GW): same as R46 for firm.
- **R52** Firm strike accumulator: `=prev×(1−Divest_Rate)+cohort_GW×Firm_LRMC×(1+Markup_Base)`. Flat markup only (no supply curve — thin market).
- **R53** Firm blended strike: `=R52/R51`.

### ESEM Offtake & Balancing (R54–R65)
- **R55** VRE GW divested: `=IF(t≥Recycle_Lag, Divest_Rate×settled_stock, 0)`
- **R56** VRE offtake revenue: `=(forward−blended_strike)×GW_divested×VRE_CFe×8760`. Positive = ESEM makes money. Negative = contracted above market.
- **R57–R58** Same for firm, using CFp not CFe.
- **R59** Admin cost: `=Admin_Cost` (constant, −$37m/yr in base).
- **R60** Net annual ESEM P&L: `=admin+VRE_offtake+firm_offtake` (when ESEM active).
- **R61** Cumulative ESEM P&L ($bn).
- **R62** ESEM levy: `=−P&L/demand` ($/MWh). Negative P&L → positive levy on consumers.
- **R64** Consumer price: `=WS+levy`.
- **R65** Merchant viability: `=WS/LRMC`. < 1 means missing money problem active.

### Private Cost Accumulators (R67–R71)
- **R68** Private VRE cost accumulator: `=prev×(1−VRE_Dep)+new_GW×current_LRMC`. Mirrors strike accumulator — tracks cost at build year, not current year.
- **R69** Private VRE blended LRMC: `=R68/R40`.
- **R70** Private firm cost accumulator: same using Firm_LRMC_t at build year.
- **R71** Private firm blended LRMC: `=R70/R50`.

### Total System Cost (R73–R86)
Resource costs only — excludes ESEM P&L (transfer, not real resource cost). Excludes transmission (not modelled).

| Row | Component | Formula logic |
|---|---|---|
| R76 | Coal fixed O&M ($m/yr) | coal_capacity × Coal_FOM / 1000. Tracks retirement. |
| R77 | Coal fuel + VOM ($m/yr) | coal_generation × derived_SRMC / 1000 |
| R78 | VRE ESEM fleet ($m/yr) | settled_GW × blended_strike × VRE_CFe × 8760 / 1000 |
| R79 | VRE private fleet ($m/yr) | private_GW × blended_LRMC × VRE_CFe × 8760 / 1000 |
| R80 | Firm ESEM fleet ($m/yr) | firm_settled × blended_firm_strike × Firm_CFe × 8760 / 1000 |
| R81 | Firm private fleet ($m/yr) | private_firm × blended_firm_LRMC × Firm_CFe × 8760 / 1000 |
| R82 | Gas baseline fuel ($m/yr) | total_firm_GW × Firm_Gen_CF × 8760 × Firm_VC / 1000 |
| R84 | ★ Total system cost ($m/yr) | SUM(R76:R82) |
| R85 | ★ Levelised cost ($/MWh) | Total × 1000 / demand |
| R86 | Cumulative ($bn) | Running sum |

---

## 7. Building Assets sheet — section summary

### Section A — Parameters (R5–R46)
All named range references mirroring Assumptions. Read-only — edit via Assumptions. Includes O&M sub-section (R42–R46) added in this session.

### Section B — GenCost Capex Data (R47–R54)
Raw capex trajectories, 26 columns. Source: GenCost 2025-26, Current Policies scenario.
- R47: 2h battery ($1,028→$716/kW)
- R48: 4h battery ($1,508→$1,036/kW)
- R49: 8h battery ($2,408→$1,640/kW)
- R50: Solar PV ($1,381→$660/kW)
- R51: Wind onshore ($3,108→$2,136/kW)
- R53: Small OCGT ($2,824→$1,801/kW)
- R54: Large OCGT ($1,694→$1,081/kW)

**NOTE**: 1h battery row appears to be missing from current B section — check that R46 (before the new O&M block) was the 1h row and hasn't been displaced.

### Section C — VRE+Storage LRMC (R56–R81)
All costs per kW of **battery storage** installed. 26 columns.

**C1 — Blended capex:**
- R60: Blended battery capex = `Σ(weight_i × capex_i)`
- R61: Solar × ratio = `Alpha_Solar × VRE_Ratio × solar_capex`
- R62: Wind × ratio = `Alpha_Wind × VRE_Ratio × wind_capex`

**C2 — All-in annual cost:**
- R65: Battery capital annuity: `PMT(Batt_WACC, 20yr, R60)`
- R66: Battery O&M: `Batt_w1h×Batt_OM_1h+...` (named params, no magic numbers)
- R67: Battery replacement annuity: `PMT(Batt_WACC, 20yr, Repl_Frac×R60×(1+WACC)^(−Repl_Year))`
- R68: Solar annuity × ratio: `PMT(VRE_WACC_S, 30yr, R61)`
- R69: Solar O&M × ratio: `Solar_OM×Alpha_Solar×VRE_Ratio`
- R70: Wind annuity × ratio: `PMT(VRE_WACC_W, 25yr, R62)`
- R71: Wind O&M × ratio: `Wind_OM×Alpha_Wind×VRE_Ratio`
- R72: **TOTAL all-in annual cost**

**C3 — DEF and LRMC:**
- R75: VRE blended CF = `Alpha_Solar×CF_Solar + Alpha_Wind×CF_Wind`
- R76: DEF = `(Direct_Flow+(1−Direct_Flow)×RTE) × VRE_Ratio × CF_blended` (Drought_Gap removed by user)
- R78: ★ **Standard VRE+Storage LRMC** = `1000×R72/(8760×R76)` → feeds Model R22
- R80: ESEM total annual cost (R72 with annuities recalculated at WACC−ESEM_WACC_Disc)
- R81: ★ **ESEM LRMC** = `1000×R80/(8760×R76)` → feeds Model R47 strike accumulator

### Section D — Firming LRMC (R82–R92)
- R84: Blended OCGT capex = `AVERAGE(small, large)` — 26 time-varying columns
- R85: Capital annuity at Gas_WACC, 25yr life
- R86: O&M = `AVERAGE(OCGT_OM_Small, OCGT_OM_Large)` — named params, no magic numbers
- R87: Total fixed cost
- R90–R91: Sensitivity table (fixed cost at various CFs)
- R92: ★ **Firm LRMC** = `R87×1000/(8760×Firm_CFe)+Firm_VC` — **time-varying across all 26 columns** — feeds Assumptions Firm_LRMC (Year 1) and Model R49/R71

### Section E — Coal Cost Structure (R94+)
Existing fleet — **no capex annuity** (sunk cost). Time-varying across 26 columns.
- Heat rate: 8.547 GJ/MWh (GenCost B.9 row 15, black coal, 42.12% efficiency)
- Coal commodity price: $8.81/GJ at t=0, escalating at Coal_Esc (4%/yr)
- Derived SRMC: fuel cost + VOM. Reconciliation check vs Coal_SRMC=$80 (should be <$1 diff at t=0)
- Fixed O&M: Coal_FOM × coal_capacity (time-varying as fleet retires)
- Coal_CFe time-varying: `Coal_CFe×(1−Coal_Degrade)^t` — feeds Model R15
- Coal_CFp time-varying: `Coal_CFp×(1−Coal_Degrade)^t` — feeds Model R16

---

## 8. Key economic mechanisms

### Wholesale spot price (R27)
Exponential formation anchored to LRMC:
```
WS = LRMC × EXP(±elasticity × |balance/demand|)
```
In surplus: WS < LRMC. In deficit: WS > LRMC. Anchored to VRE LRMC (not coal SRMC) — appropriate for a high-VRE long-run equilibrium.

### Forward price (R29)
Logistic blend of LRMC and spot:
```
α = 1/(1+EXP(−Fwd_k × balance/demand))
Forward = α × LRMC + (1−α) × WS
```
In surplus → α→1 → Forward tracks LRMC (investment signal dominates).
In deficit → α→0.5 or lower → Forward tracks spot (scarcity dominates).
Avoids floors/kinks. Uses prior-year balance to prevent circularity.

### Strike accumulator (R47/R52)
Portfolio-weighted average of what ESEM locked in at contracting:
```
Accumulator(t) = prev × (1−Divest_Rate) + GW_contracted_Lag_yrs_ago × LRMC_at_contracting × markup
Blended_strike(t) = Accumulator(t) / Settled_stock(t)
```
Key property: divestment alone doesn't change blended strike (both decay at same rate). Only new cohorts at different prices change it.

VRE uses **ESEM LRMC** (WACC-discounted) as contracting price. Firm uses standard Firm_LRMC (flat markup, no supply curve).

### DEF (Delivered Energy Factor)
Converts $/kW-storage/yr to $/MWh:
```
DEF = (Direct_Flow + (1−Direct_Flow)×RTE) × VRE_Ratio × CF_blended
```
- Direct_Flow (0.8): fraction of VRE bypassing battery (no RTE loss)
- (1−Direct_Flow)×RTE: indirect path efficiency (0.2×0.9 = 0.18)
- Effective flow rate: 0.98 (98% of VRE reaches load)
- × VRE_Ratio (3): scales from per-kW-storage to total bundle
- × CF_blended (0.263): converts capacity to average hourly energy
DEF ~0.73 → 6,395 MWh/yr per MW of storage (above 8,760 because 3 MW VRE behind each MW storage).

### ESEM WACC discount
ESEM-backed projects face lower financing risk (CfD eliminates merchant revenue uncertainty). Building Assets R80 recomputes the annuities at `(WACC − ESEM_WACC_Disc)` for each technology. In WACC disc scenario (disc=2%): battery annuity falls ~14%, solar ~19%, wind ~14%, resulting in ESEM_LRMC ~10–15% below standard LRMC. The strike accumulator uses ESEM_LRMC — ESEM locks in lower prices than private developers need.

---

## 9. Design decisions and things explicitly rejected

| Decision | What we chose | What we rejected and why |
|---|---|---|
| Coal in energy balance | Coal generation included via Coal_CFe. Firm generation included at 5% baseline CF (Firm_Gen_CF). | Adding firm generation at full CFe — double counts; firm dispatch is endogenous residual. |
| Curtail_Thresh | Endogenised = VRE_Share0. Curtailment starts at current penetration. | Fixed 0.2 — was causing treadmill where target recedes faster than procurement grows. |
| VRE_ELCC | Endogenised = ELCC_base×(1−VRE_Share0). Falls with penetration. | Fixed value ignores declining value of VRE as penetration rises. |
| Forward price construction | Logistic blend of LRMC and WS using prior-year balance. | Fixed Fwd_Alpha (removed from model). Static blends don't respond to market conditions. |
| Drought_Gap | **Removed by user** for simplicity. DEF no longer applies drought haircut. | Previously 5% — deemed excess complexity. |
| ESEM P&L in total system cost | Excluded — it's a transfer payment not a resource cost. | Including would double-count. |
| ESEM LRMC vs private LRMC | ESEM uses WACC-discounted LRMC (BA R81). Private uses standard LRMC (BA R78). | Same LRMC for both — incorrect; CfD de-risking reduces financing cost for ESEM-backed projects. |
| Firm LRMC time-varying | Yes — BA R92 varies across all 26 columns as OCGT capex changes. | Scalar Year 1 only — was giving wrong investment signal for private firm entry in later years. |
| Coal capex in system cost | Excluded — sunk cost. | Including would overstate costs. |
| Transmission costs | Not modelled — documented limitation. | Adding would require REZ cost assumptions; out of scope for stylised model. |

---

## 10. Outstanding items (as of this handoff)

### Bugs to fix first
1. **R64/R65 ESEM_On**: CHOOSE formula went to R64 (section header) not R65 (parameter). ESEM_On named range at R65 still = 1 hardcoded. Fix: `=CHOOSE($C$4,I65,J65,K65,L65)` in R65C.
2. **R23 VRE_PrePipe**: Cols D+ still have `×MIN(t,Lag)` bug. Should be constant `VRE_PrePipe`. Fix all 25 columns.
3. **Stale named ranges**: Delete `Drought_Gap`, `Fwd_Alpha`, `Other_Gen` from Name Manager (all point to #REF!).

### Features planned but not yet built (user said "do it later")
1. **Total system cost section** — already partially built (R73–R86 in Model) but built against a version the user then rolled back from. Needs to be rebuilt once the above bugs are fixed. Components: coal fixed O&M, coal fuel, VRE ESEM fleet cost, VRE private fleet cost, firm ESEM fleet cost, firm private fleet cost, gas baseline fuel. Levelised ($/MWh) and cumulative ($bn) outputs.
2. **Building Assets Section E** (coal cost structure) — also partially built in a rolled-back version. Needs to be rebuilt: heat rate×price=fuel cost, VOM, derived SRMC reconciliation check, fixed O&M row, time-varying CFe/CFp degradation rows.
3. **Model R13** to use time-varying derived coal SRMC from BA Section E (currently uses static Coal_SRMC×(1+Coal_Esc)^t).
4. **Model R15/R16** to use time-varying CFe/CFp from BA Section E degradation rows.

### Other improvements discussed but deferred
- Battery O&M description note: explicitly document that $7–13/kW values are routine O&M only, not NREL ATB's augmentation-inclusive FOM (different concepts — augmentation handled via replacement annuity).
- Dashboard/Charts development.

---

## 11. Sources and citations

| Source | Used for |
|---|---|
| GenCost 2025-26 Consultation Draft (Graham & Hayward) | All capex trajectories, technology O&M, heat rates |
| GenCost 2025-26 Table B.9, black coal row (efficiency 42.12%, FOM $64.85, VOM $4.68) | Coal cost structure |
| AEMO ISP 2024 | Demand, coal capacity, VRE pipeline, gas generation trajectory (~5% annual potential output for peakers), ELCC |
| AEMO 2024 (ESOO / Generation Information) | Starting fleet capacities (coal 22 GW, firm 22 GW, VRE 30 GW) |
| NREL ATB 2024 | Battery O&M (routine component only, $7–13/kW/yr by duration) |
| Own estimate / back-calculation | Coal_Price ($8.81/GJ), Coal_Degrade (1.5%/yr), Firm_VC ($120/MWh), forward price calibration |

---

## 12. Named ranges — complete list

All live ranges (excludes broken #REF! ranges listed in Section 4):

```
Admin_Cost       → Assumptions!$C$78
Alpha_Solar      → Assumptions!$C$35
Alpha_Wind       → Assumptions!$C$36
Batt_OM_1h/2h/4h/8h → Assumptions!$C$88–91
Batt_WACC        → Assumptions!$C$53
Batt_w1h/2h/4h/8h  → Assumptions!$C$47–50
Battery_RTE      → Assumptions!$C$44
CF_Solar/Wind    → Assumptions!$C$33–34
Coal_Accel       → Assumptions!$C$19
Coal_CFe/CFp     → Assumptions!$C$15–16
Coal_Degrade     → Assumptions!$C$87
Coal_Esc         → Assumptions!$C$18
Coal_FOM         → Assumptions!$C$84
Coal_Heat_Rate   → Assumptions!$C$83
Coal_Price       → Assumptions!$C$86
Coal_SRMC        → Assumptions!$C$17
Coal_Start       → Assumptions!$C$14
Coal_VOM         → Assumptions!$C$85
Contract_Years   → Assumptions!$C$72
Curtail_Rate     → Assumptions!$C$63
Curtail_Thresh   → Assumptions!$C$62  [computed = VRE_Share0]
Def_Elas         → Assumptions!$C$60
Dem_Growth       → Assumptions!$C$8
Dem_Start        → Assumptions!$C$7
Direct_Flow      → Assumptions!$C$43
Divest_Rate      → Assumptions!$C$73  [computed]
ELCC_base        → Assumptions!$C$38
ESEM_Build_Cap   → Assumptions!$C$74
ESEM_On          → Assumptions!$C$65  [BUG: hardcoded 1]
ESEM_Start       → Assumptions!$C$66
ESEM_WACC_Disc   → Assumptions!$C$57
Firm_CFe/CFp     → Assumptions!$C$23–24
Firm_Cov         → Assumptions!$C$69
Firm_Decay       → Assumptions!$C$22
Firm_Gen_CF      → Assumptions!$C$30
Firm_LRMC        → Assumptions!$C$27  [= BA!B92 Year 1]
Firm_Sprd        → Assumptions!$C$28
Firm_SRMC        → Assumptions!$C$25
Firm_Start       → Assumptions!$C$21
Firm_Tight       → Assumptions!$C$29
Firm_VC          → Assumptions!$C$26
Fwd_k            → Assumptions!$C$61
Gas_WACC         → Assumptions!$C$56
Lag              → Assumptions!$C$70
Markup_Base/Rate → Assumptions!$C$75–76
OCGT_OM_Small/Large → Assumptions!$C$94–95
Other_Firm       → Assumptions!$C$11
Peak_Growth      → Assumptions!$C$10
Peak_Start       → Assumptions!$C$9
RE_Cov           → Assumptions!$C$68
Recycle_Lag      → Assumptions!$C$71
Repl_Frac        → 'Building assets'!$B$19
Repl_Year        → 'Building assets'!$B$20
Solar_OM         → Assumptions!$C$92
Surp_Elas        → Assumptions!$C$59
VRE_CFe          → Assumptions!$C$37  [computed]
VRE_Dep          → Assumptions!$C$42
VRE_ELCC         → Assumptions!$C$39  [computed = ELCC_base×(1−VRE_Share0)]
VRE_PrePipe      → Assumptions!$C$12
VRE_Ratio        → Assumptions!$C$40
VRE_Share0       → Assumptions!$C$79  [computed = VRE_Start×VRE_CFe×8760/Dem_Start]
VRE_ShareT       → Assumptions!$C$67
VRE_Sprd         → Assumptions!$C$41
VRE_Start        → Assumptions!$C$32
VRE_WACC_S/W     → Assumptions!$C$54–55
Wind_OM          → Assumptions!$C$93
```
