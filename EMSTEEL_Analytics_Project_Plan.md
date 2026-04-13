# EMSTEEL Data Analytics & Modelling Initiatives
## Project Plan — v1.0

**Author:** Data & Analytics
**Base methodology:** DAEWOO Steel Proof-of-Concept (steel_full_analysis.ipynb)
**Strategic alignment:** UAE Net Zero 2050 · UAE National Energy Strategy 2050 · Abu Dhabi Industrial Strategy · EMSTEEL GHG Commitment (–40% by 2030)

---

## 0. The Strategic Frame

### Why This Project Plan Exists

EMSTEEL publishes a GHG intensity of 0.67 tCO₂e/tonne — the lowest since 2019 and the headline number behind the 40% reduction commitment by 2030. That number is real. But it is computed once a year, at company level, from aggregated data.

What it cannot currently answer:

- Which shift, which furnace, which product type is driving that number up or down?
- Is the improvement happening in DRI (Scope 1) or in electricity efficiency (Scope 2)?
- What does one bad heat cost in CO₂ — and could it have been predicted before it happened?
- When EMSTEEL claims green steel, can it certify the specific batch that went into a European bridge?

This project plan builds the analytical infrastructure to answer those questions. It does not add new targets. It provides the measurement layer that makes the existing targets provable, actionable, and monetisable.

---

## 1. Strategic Alignment

### 1.1 UAE National Strategies

| Strategy | Relevant Commitment | EMSTEEL Connection |
|---|---|---|
| **UAE Net Zero 2050** | Economy-wide carbon neutrality by 2050 | EMSTEEL as flagship industrial decarboniser |
| **UAE National Energy Strategy 2050** | 44% clean energy in the mix; 40% reduction in carbon footprint of energy generation | Scope 2 reduction through energy efficiency directly reduces EMSTEEL's grid-linked emissions |
| **Abu Dhabi Industrial Strategy (ADIS)** | Double industrial GDP; position Abu Dhabi as global advanced manufacturing hub | Green steel premium pricing enables export revenue growth |
| **Make it in the Emirates** | Localise supply chains; add value to raw materials | DRI-to-steel as the value chain anchor — analytics proves its efficiency |
| **UAE Green Agenda 2030** | 30% improvement in resource efficiency | Production/energy intensity = the exact resource efficiency metric this agenda tracks |

### 1.2 EMSTEEL's Own Published Commitments

| Commitment | Current Status | Analytics Role |
|---|---|---|
| 40% GHG reduction by 2030 | 0.67 tCO₂e/tonne baseline | Provide monthly, shift-level intensity tracking — not annual |
| 2% energy intensity improvement per year | Reported annually | Provide weekly tonne/kWh trend — catch drift before year-end |
| First steel-maker to capture CO₂ from DRI | Operational | Quantify capture efficiency at heat level |
| Green steel certification for export | In development | Certify specific production batches against interval-level Scope 1+2 data |

---

## 2. The Intensity Metric That Connects Everything

EMSTEEL's core efficiency measure is:

```
Intensity = Production (tonnes) / Energy Consumption (kWh)
```

Higher = more efficient. Lower = energy was consumed without producing steel.

This single ratio is the north star for every initiative below. It connects:

- **Energy analytics** (denominator management — this notebook)
- **Production analytics** (numerator management — heat optimisation)
- **GHG analytics** (intensity × emission factor = carbon cost per tonne)
- **Commercial analytics** (intensity × steel price = revenue per kWh)

Every initiative in this plan either **increases the numerator**, **decreases the denominator**, or **measures which one moved**.

---

## 3. Emission Scope Architecture

```
SCOPE 1 — Direct (EMSTEEL owns the source)
├── DRI Plant        ~70–75%  CO₂ from reducing iron ore pellets with CO/H₂ gas
├── SMP / EAF        ~20%     Carbon oxidation in arc furnace + electrode consumption
└── Rolling Mill     ~10%     Gas-fired reheating furnaces

SCOPE 2 — Indirect (purchased electricity from ADDC grid)
└── All kWh consumed × UAE grid emission factor
    ← This is what the DAEWOO proof-of-concept analyses

SCOPE 3 — Value chain (upstream suppliers + downstream customers)
├── Upstream: natural gas, iron ore pellets, scrap, electrodes, refractories
└── Downstream: steel fabrication, construction (Scope 3 Category 11)
```

**The 40% reduction target is primarily a Scope 1 challenge** (DRI gas mix → green hydrogen transition).
**This analytics programme addresses Scope 2** and provides the measurement layer for Scope 1 at heat level.
**Scope 3 upstream** becomes quantifiable once supplier tonnage data is integrated.

---

## 4. Initiative Architecture

```
Initiative 1 — Energy-Production Intelligence System   (Operational efficiency)
Initiative 2 — GHG Accounting & Intensity Platform     (Decarbonisation measurement)
Initiative 3 — Predictive Heat Analytics               (Optimisation & anomaly prevention)
Initiative 4 — Green Steel Certification Analytics     (Commercial & ESG monetisation)
Initiative 5 — Decarbonisation Scenario Modelling      (Strategic & investment decisions)
```

Each initiative is described below with: goal, data requirements, analytical questions, deliverables, and UAE strategy alignment.

---

## 5. Initiative 1 — Energy-Production Intelligence System

### 5.1 Goal
Convert EMSTEEL's electricity bill from a monthly invoice into a real-time operational KPI. Track tonne/kWh at shift, furnace, and product level. Make inefficiency visible before it becomes a cost.

### 5.2 Builds Directly On
The DAEWOO proof-of-concept (steel_full_analysis.ipynb) — all Phase 2–7 analyses replicated on internal data.

### 5.3 Data Requirements

| Data Source | System | Resolution | Join Key |
|---|---|---|---|
| Interval energy consumption | ADDC smart meter | 15-min | Timestamp |
| Reactive power & power factor | Power quality analysers | 15-min | Timestamp |
| Production tonnage | ERP / MES | Per heat (~60–90 min) | Heat number + timestamp |
| Furnace operating state | SCADA | Real-time / 1-min | Timestamp → aggregate to 15-min |
| Product type | MES | Per heat | Heat number |
| Shift schedule | HR / Operations | Per shift | Date + shift code |

### 5.4 Analytical Questions

**Replicated from DAEWOO (Phase 2–3), now with intensity:**
- Q1: What is the tonne/kWh signature of a typical production day?
- Q2: How does intensity differ between weekday shifts and weekend operations?
- Q3: Which hour × day combinations have the lowest intensity? (hot spots of waste)
- Q4: Do the top 10% most energy-intensive intervals correspond to the top 10% most productive?
- Q5: What is tonne/kWh by Load Type — and what is the intensity of Light Load (zero)

**New questions enabled by production data:**
- Q6: Which EAF unit has the lowest average tonne/kWh? By how much?
- Q7: Which product type (rebar / wire rod / structural) is most energy-intensive per tonne?
- Q8: Does intensity improve over successive heats in a campaign (furnace warming effect)?
- Q9: What is the energy cost of a tap-to-tap time that runs 15 minutes over target?
- Q10: How does scrap mix quality correlate with heat energy consumption?

### 5.5 Load Type Engineering (No Pre-Label in EMSTEEL Data)

```python
# Option A: Map from SCADA furnace state (preferred)
scada_to_load = {
    'idle':          'Light_Load',
    'charging':      'Light_Load',
    'heating':       'Medium_Load',
    'refining':      'Medium_Load',
    'full_power':    'Maximum_Load',
    'tapping':       'Medium_Load',
}

# Option B: Derive from kWh percentile (fallback)
q33 = df['Usage_kWh'].quantile(0.33)
q67 = df['Usage_kWh'].quantile(0.67)
df['Load_Type'] = pd.cut(df['Usage_kWh'],
    bins=[-np.inf, q33, q67, np.inf],
    labels=['Light_Load', 'Medium_Load', 'Maximum_Load'])
```

### 5.6 Deliverables
- Weekly tonne/kWh report by furnace and product
- Monthly intensity trend vs 2% improvement target
- Anomaly alerts: heats consuming >15% above planned energy target
- Dashboard: real-time intensity gauge per EAF unit

### 5.7 UAE Strategy Alignment
- **ADIS**: proves industrial productivity improvement with data
- **UAE Energy Strategy 2050**: quantifies Scope 2 reduction per year
- **Make it in the Emirates**: demonstrates that local steel production is becoming more efficient

---

## 6. Initiative 2 — GHG Accounting & Intensity Platform

### 6.1 Goal
Disaggregate EMSTEEL's annual 0.67 tCO₂e/tonne figure into monthly, shift-level, and heat-level components. Enable the company to prove — not just report — its decarbonisation trajectory.

### 6.2 The Three-Scope Calculation

```python
# Emission factors (adjust to EMSTEEL's actual engineering values)
EF_DRI  = 0.45   # tCO2 / tonne DRI produced (natural gas shaft furnace)
EF_EAF  = 0.08   # tCO2 / tonne liquid steel (electrode + carbon)
EF_RM   = 0.03   # tCO2 / tonne rolled product (reheating furnace)
EF_GRID = 0.45   # tCO2 / MWh — UAE ADDC grid emission factor

# Scope 1 — at heat level
df['scope1_tCO2'] = (
    df['dri_tonnes']    * EF_DRI   +   # 70-75% of total
    df['steel_tonnes']  * EF_EAF   +   # ~20%
    df['rolled_tonnes'] * EF_RM        # ~10%
)

# Scope 2 — from electricity interval data (this is what DAEWOO proxied)
df['scope2_tCO2'] = df['Usage_kWh'] / 1000 * EF_GRID

# GHG intensity — the published KPI
df['ghg_intensity'] = (
    (df['scope1_tCO2'] + df['scope2_tCO2']) / df['steel_tonnes']
)
# Target: reduce from 0.67 tCO2e/tonne → 0.40 tCO2e/tonne by 2030
```

### 6.3 Analytical Questions
- What % of the 0.67 tCO₂e/tonne is Scope 1 vs Scope 2 — and how does that split change month by month?
- Is the annual improvement in intensity coming from Scope 1 (process change) or Scope 2 (energy efficiency)?
- Which specific heats have the highest CO₂/tonne — and what operational variables predict them?
- What would a 10% improvement in DRI process efficiency do to the headline GHG intensity?
- What is EMSTEEL's current Scope 2 emission per tonne, and how does it compare to European EAF producers?

### 6.4 Data Requirements
All of Initiative 1, plus:
- DRI plant gas consumption by shift (Nm³/tonne DRI)
- EAF carbon injection volumes per heat
- Rolling mill gas consumption (GJ/tonne)
- ADDC published grid emission factor (annual update)

### 6.5 Deliverables
- Monthly Scope 1/2/3 split dashboard
- Annual GHG intensity trend vs 0.67 baseline
- Heat-level CO₂ attribution report
- Regulatory-ready GHG inventory (CDP, UAE reporting frameworks)

### 6.6 UAE Strategy Alignment
- **UAE Net Zero 2050**: provides the measurement infrastructure that national reporting requires
- **UAE Green Agenda 2030**: directly tracks the 30% resource efficiency improvement
- **Abu Dhabi Industrial Strategy**: positions EMSTEEL as a data-driven decarboniser — not just a reporter

---

## 7. Initiative 3 — Predictive Heat Analytics

### 7.1 Goal
Shift from describing what happened (Phases 2–6 in this notebook) to predicting what will happen. Identify which heats will overrun their energy budget before they start, enabling pre-emptive operational intervention.

### 7.2 Why Prediction Is Now Justified
The DAEWOO proof-of-concept established:
- Energy patterns are consistent and shift-driven (Phase 2)
- Power factor collapses cluster at predictable times (Phase 6 Type 2)
- Certain heats systematically overconsume vs their baseline (Phase 6 Type 3)

These patterns are learnable. Once the descriptive layer is solid, predictive models have a validated foundation.

### 7.3 Modelling Initiatives

**Model 1 — Heat Energy Prediction**
```
Target:      kWh consumed in next heat
Features:    scrap mix grade, scrap weight, previous heat tap temperature,
             electrode length, time since last heat, ambient temperature
Model type:  Gradient Boosting (XGBoost) — handles non-linear interactions
Business use: Flag heats predicted to exceed target by >10% → operator alert
```

**Model 2 — Power Factor Collapse Prediction**
```
Target:      Will PF drop below 0.85 in the next 30 minutes?
Features:    Current PF trend, reactive power trajectory, load state,
             hour of day, furnace phase (charging/melting/refining)
Model type:  Binary classifier (Logistic Regression or Random Forest)
Business use: Pre-emptive reactive compensation switching
```

**Model 3 — Production Intensity Anomaly Detection**
```
Target:      Is this heat's tonne/kWh abnormally low?
Features:    All heat parameters + historical baseline per product type
Model type:  Isolation Forest or statistical control chart (CUSUM)
Business use: Real-time alert when a heat is 2σ below intensity baseline
```

### 7.4 Modelling Scope Boundary
- **In scope**: supervised prediction on structured operational data
- **Out of scope**: image recognition (camera-based scrap grading), NLP on maintenance logs — these are Phase 3 initiatives
- **No black-box models for safety-critical decisions** — all models require explainability layer (SHAP values) before operational deployment

### 7.5 Data Requirements
All of Initiatives 1–2, plus:
- Historical heat logs (minimum 2 years for model training)
- Scrap mix composition per heat (grade, weight, chemistry if available)
- Electrode consumption per heat
- Tap temperature per heat
- Maintenance event logs (to exclude breakdown heats from training data)

### 7.6 Deliverables
- Heat energy prediction model (accuracy target: within ±8% of actual kWh)
- PF collapse early warning system (30-min horizon)
- Intensity anomaly dashboard with SHAP explanation per flagged heat
- Model performance monitoring report (monthly drift detection)

### 7.7 UAE Strategy Alignment
- **UAE AI Strategy 2031**: industrial AI application in a strategic sector
- **UAE National Energy Strategy 2050**: AI-driven energy optimisation as an enabler of the 40% carbon reduction target
- **ADIS**: positions Abu Dhabi as a hub for advanced manufacturing with embedded intelligence

---

## 8. Initiative 4 — Green Steel Certification Analytics

### 8.1 Goal
Enable EMSTEEL to certify specific production batches as "green steel" — defined by a verifiable CO₂e/tonne threshold per heat — to access the European green steel premium market.

### 8.2 The Commercial Case
European steelmakers and automotive OEMs are paying a premium for certified low-carbon steel. The EU Carbon Border Adjustment Mechanism (CBAM), effective 2026, will impose a carbon cost on imported steel proportional to its embedded emissions. EMSTEEL's DRI-EAF route already has a structural advantage over blast furnace steel. The analytics layer converts that structural advantage into a certifiable, batch-level claim.

### 8.3 What Certification Requires
```
For every coil / billet exported:
├── Scope 1 CO₂ attributed to that specific heat (tCO₂/tonne)
├── Scope 2 CO₂ from electricity for that heat (kWh × grid EF / tonnes)
├── Traceability: heat number → interval energy data → emission calculation
└── Third-party verifiable: methodology must follow ISO 14064 or ResponsibleSteel
```

### 8.4 Analytical Infrastructure Needed

```python
# Heat-level CO2 certificate
def generate_heat_certificate(heat_id):
    heat = df_heats[df_heats['heat_id'] == heat_id].iloc[0]

    scope1 = heat['dri_tonnes'] * EF_DRI + heat['steel_tonnes'] * EF_EAF
    scope2 = heat['kwh_consumed'] / 1000 * EF_GRID
    intensity = (scope1 + scope2) / heat['steel_tonnes']

    return {
        'heat_id':         heat_id,
        'steel_tonnes':    heat['steel_tonnes'],
        'product_type':    heat['product_type'],
        'scope1_tCO2':     round(scope1, 4),
        'scope2_tCO2':     round(scope2, 4),
        'total_tCO2':      round(scope1 + scope2, 4),
        'intensity_tCO2_per_tonne': round(intensity, 4),
        'certification_threshold':  0.40,  # tCO2e/tonne — green steel threshold
        'is_green_steel':  intensity <= 0.40,
        'timestamp':       datetime.now().isoformat()
    }
```

### 8.5 Deliverables
- Heat-level CO₂ certificate generator
- Green steel eligibility dashboard (what % of production qualifies this month)
- CBAM exposure calculator (import carbon cost if not certified)
- Audit trail: energy interval → heat → certificate → export documentation

### 8.6 UAE Strategy Alignment
- **Make it in the Emirates**: green steel certification opens premium export markets
- **UAE Net Zero 2050**: EMSTEEL as proof-point for industrial decarbonisation at export scale
- **Abu Dhabi Industrial Strategy**: value-added industrial exports vs commodity steel

---

## 9. Initiative 5 — Decarbonisation Scenario Modelling

### 9.1 Goal
Quantify the GHG intensity impact of strategic technology decisions — primarily the transition from natural gas DRI to hydrogen-based DRI (H₂-DRI) — and identify the optimal sequencing of initiatives to meet the 40% reduction target by 2030.

### 9.2 The Scenario Framework

```
BASELINE (2024):     0.67 tCO₂e/tonne
TARGET (2030):       0.40 tCO₂e/tonne  (–40%)

Lever 1 — Scope 2 reduction (this analytics programme)
  Energy efficiency: +2% tonne/kWh improvement per year
  Reactive power compensation: reduce PF penalties
  Load shifting: move Light Load intervals off peak-rate hours
  → Estimated contribution to target: 8–12% of the 40%

Lever 2 — Scope 1 DRI process improvement
  Increase H₂ fraction in reduction gas (partial substitution)
  → Each 10% increase in H₂ fraction reduces DRI CO₂ by ~6%
  → Estimated contribution: 15–20% of the 40%

Lever 3 — Scope 1 full H₂-DRI transition
  Replace natural gas entirely with green hydrogen
  → Reduces DRI CO₂ from ~0.45 to ~0.02 tCO₂/tonne DRI
  → Estimated contribution: 30–35% of the 40% (but requires green H₂ supply)

Lever 4 — Scope 2 grid decarbonisation (external)
  As ADDC's grid becomes cleaner (solar, nuclear), Scope 2 drops automatically
  → Not controllable by EMSTEEL, but must be tracked and credited
```

### 9.3 Scenario Model Structure

```python
def scenario_intensity(year,
                       h2_fraction,          # 0.0 = full gas, 1.0 = full H2
                       energy_efficiency_gain,  # cumulative % vs baseline
                       grid_ef):             # tCO2/MWh in that year

    scope1_dri = EF_DRI * (1 - h2_fraction * 0.96)  # H2 eliminates 96% of DRI CO2
    scope1_eaf = EF_EAF  # unchanged
    scope1_rm  = EF_RM   # unchanged

    scope1 = (scope1_dri * dri_ratio + scope1_eaf + scope1_rm)
    scope2 = (baseline_kwh_per_tonne * (1 - energy_efficiency_gain)) * grid_ef / 1000

    return scope1 + scope2

# Run scenarios
scenarios = {
    'BAU':              scenario_intensity(2030, h2_fraction=0.0, energy_efficiency_gain=0.0,  grid_ef=0.45),
    'Efficiency only':  scenario_intensity(2030, h2_fraction=0.0, energy_efficiency_gain=0.14, grid_ef=0.40),
    'Partial H2':       scenario_intensity(2030, h2_fraction=0.3, energy_efficiency_gain=0.14, grid_ef=0.38),
    'Full H2 + grid':   scenario_intensity(2030, h2_fraction=1.0, energy_efficiency_gain=0.14, grid_ef=0.30),
}
```

### 9.4 Deliverables
- Annual scenario model: which combination of levers hits 0.40 tCO₂e/tonne by 2030
- Investment prioritisation matrix: AED cost vs tCO₂ reduction per initiative
- Sensitivity analysis: what if H₂ supply is delayed? What if grid EF doesn't improve?
- Board-level summary: "to hit our 2030 target, we need X from Scope 1 and Y from Scope 2"

### 9.5 UAE Strategy Alignment
- **UAE Net Zero 2050**: provides the bottom-up industrial model that national scenarios need
- **UAE Hydrogen Leadership Roadmap**: EMSTEEL as the anchor demand case for green hydrogen
- **UAE Energy Strategy 2050**: links grid decarbonisation to industrial intensity improvement

---

## 10. Roadmap & Sequencing

```
2025 — FOUNDATION
├── Initiative 1: Integrate ADDC 15-min data + ERP production tonnage
├── Initiative 1: Compute first tonne/kWh baseline (replaces annual KPI)
└── Initiative 2: Build Scope 1/2 accounting at heat level

2026 — INTELLIGENCE
├── Initiative 1: Weekly intensity dashboard live (per furnace, per product)
├── Initiative 2: Monthly GHG intensity report vs 0.67 baseline
├── Initiative 3: Deploy heat energy prediction model (v1)
└── Initiative 4: Pilot green steel certificate on 3 export heats

2027 — OPTIMISATION
├── Initiative 3: PF collapse prediction + automated compensation trigger
├── Initiative 3: Intensity anomaly detection → operator alert system
├── Initiative 4: Green steel certification at scale (all export heats)
└── Initiative 5: First decarbonisation scenario model for board review

2028–2030 — STRATEGIC
├── Initiative 5: Annual scenario update tied to H₂ supply agreements
├── Initiative 4: CBAM compliance reporting (EU Carbon Border Adjustment)
└── All initiatives: integrated into EMSTEEL's ESG reporting framework
```

---

## 11. Data Governance Requirements

Before any initiative goes live, three non-negotiable foundations:

| Requirement | Why It Matters | Owner |
|---|---|---|
| **Single source of truth for production tonnage** | If ERP and MES disagree on heat tonnes, every intensity calculation is wrong | IT / Operations |
| **Timestamp synchronisation across systems** | SCADA, ADDC meter, and ERP must align to the same 15-min clock | IT / SCADA team |
| **Emission factor governance** | EF_DRI, EF_EAF, EF_RM must be engineering-validated and version-controlled | Process Engineering |
| **Data access protocol** | Who can see heat-level energy data — operations, finance, external auditors | Legal / IT Security |

---

## 12. Definition of Success

| Initiative | Success Metric | Target |
|---|---|---|
| 1 — Energy-Production Intelligence | Weekly tonne/kWh report live | Q2 2026 |
| 1 — Energy-Production Intelligence | Identify 3 specific operational changes from intensity data | Q4 2026 |
| 2 — GHG Accounting | Monthly Scope 1/2 intensity vs 0.67 baseline | Q1 2026 |
| 3 — Predictive Heat Analytics | Heat energy model within ±8% of actual | Q3 2026 |
| 3 — Predictive Heat Analytics | PF collapse predicted 30 min in advance | Q1 2027 |
| 4 — Green Steel Certification | First certified export heat | Q2 2026 |
| 4 — Green Steel Certification | Green steel premium revenue > analytics programme cost | Q4 2027 |
| 5 — Scenario Modelling | Board-approved decarbonisation pathway model | Q3 2027 |

---

## 13. The One Sentence Per Initiative

| Initiative | What it does in one sentence |
|---|---|
| 1 — Energy-Production Intelligence | Tells you which hour, shift, and furnace is producing the least steel per kWh consumed |
| 2 — GHG Accounting | Breaks the annual 0.67 tCO₂e/tonne into monthly, heat-level components with Scope 1/2/3 attribution |
| 3 — Predictive Heat Analytics | Flags which heats will overrun their energy budget before they start |
| 4 — Green Steel Certification | Generates a verifiable CO₂ certificate per exported batch to access the European green premium |
| 5 — Decarbonisation Scenario Modelling | Answers how much of the 40% reduction target comes from energy efficiency vs hydrogen vs grid decarbonisation — and by when |

---

*This plan is a direct extension of the DAEWOO proof-of-concept methodology. Every analytical technique demonstrated in steel_full_analysis.ipynb has a direct application in one of the five initiatives above.*

*The DAEWOO dataset proved the approach works. EMSTEEL's internal data makes it actionable.*
