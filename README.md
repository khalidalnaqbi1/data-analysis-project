# Steel Industry Energy Analysis

Descriptive and diagnostic analysis of energy consumption patterns in an EAF-based steel plant.  
Based on the **DAEWOO Steel Co. Ltd** dataset as a proof-of-concept for EMSTEEL's operational context.

---

## What This Project Does

Breaks open the electricity bill into seven analytical phases:

| Phase | Question |
|---|---|
| 2 — Consumption Patterns | When and how much energy is consumed? |
| 3 — Efficiency Analysis | How well is the plant converting electricity into useful work? |
| 4 — CO₂ as Activity Signal | Are we producing when we are consuming? |
| 5 — Load Type Deep Dive | What characterises each operating state and what does it cost? |
| 6 — Anomaly Identification | What surprises does the data hide? |
| 7 — Synthesis & Business Translation | What did we find, what does it cost, what should we do? |

---

## Dataset

**DAEWOO Steel Co. Ltd** — Gwangyang, South Korea  
Source: [UCI Machine Learning Repository](https://archive.ics.uci.edu/dataset/851/steel+industry+energy+consumption)

- 35,040 rows · 15-minute intervals · 1 calendar year (2018)
- 11 variables: `Usage_kWh`, `Lagging_Current_Reactive_Power_kVarh`, `CO2(tCO2)`, `Load_Type`, and more

Download the dataset and place `Steel_industry_data.csv.zip` in the same folder as the notebook before running.

---

## Setup

```bash
# 1. Clone the repo
git clone https://github.com/khalidalnaqbi1/data-analysis-project.git
cd data-analysis-project

# 2. Install dependencies
pip install -r requirements.txt

# 3. Place the dataset in the project folder
# Download from: https://archive.ics.uci.edu/dataset/851/steel+industry+energy+consumption
# File needed: Steel_industry_data.csv.zip

# 4. Launch Jupyter
jupyter notebook steel_full_analysis.ipynb
```

---

## Project Structure

```
data-analysis-project/
├── steel_full_analysis.ipynb       # Main analysis notebook (Phases 2–7)
├── steel_eda.ipynb                 # Initial EDA and data cleaning
├── Steel_industry_data.csv.zip     # Dataset (download separately if not present)
├── EMSTEEL_Analytics_Project_Plan.md  # Strategic roadmap for real-data application
├── requirements.txt
└── README.md
```

---

## Key Findings (DAEWOO Proxy)

- Energy consumption follows a consistent shift-driven pattern visible purely from interval data
- Power factor drops below the 0.85 penalty threshold in a significant fraction of operating hours
- Light Load intervals (minimal production) account for a material share of annual kWh
- Monday morning cold-start ramp is measurable and annualisable as an AED cost
- CO₂ and kWh diverge in specific intervals — flagging energy consumed without productive output

---

## Strategic Context

This analysis serves as a proof-of-concept for **EMSTEEL** (Emirates Steel Arkan), the largest steel producer in the UAE, aligned with:

- UAE Net Zero 2050 Strategic Initiative
- UAE National Energy Strategy 2050
- Abu Dhabi Industrial Strategy
- EMSTEEL's published commitment to 40% GHG reduction by 2030

See `EMSTEEL_Analytics_Project_Plan.md` for the full initiative roadmap.

---

## Intensity Metric

EMSTEEL measures efficiency as **Production / Energy** (tonne/kWh) — higher is better.  
Adding production tonnage data to this framework enables kWh/tonne by shift, furnace, and product — the analytical layer beneath the annual GHG intensity headline.
