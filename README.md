# Pro-Cure
### AI-powered procurement anomaly detection for South African public health

*In memory of Babita Deokaran — Chief Financial Officer, Tembisa Hospital.*
*Assassinated 23 August 2021 after exposing R2 billion in procurement corruption.*
*She found it by hand. Pro-Cure finds it automatically.*

---

## The problem

On 23 August 2021, Babita Deokaran was shot nine times outside her Johannesburg home. She was the key witness in a corruption investigation into the Gauteng Department of Health — having manually traced how R2 billion in public health funds was systematically looted through fraudulent procurement.

The corruption was hidden in plain sight — in thousands of contracts just below regulatory thresholds, awarded repeatedly to the same connected suppliers, justified with vague single-source deviations, and never flagged because no system existed to look for the patterns.

**Pro-Cure is that system.**

---

## What exists — and what's missing

The [Open Contracting Partnership](https://www.open-contracting.org/) released **Cardinal** in 2024 — an open-source library calculating 10 generic procurement red flag indicators on any OCDS dataset, tested in Ecuador and the Dominican Republic. It is the closest existing tool to Pro-Cure.

Pro-Cure runs Cardinal first as a baseline, then shows what SA-specific signals catch that Cardinal misses:

| Capability | Cardinal | Pro-Cure |
|---|---|---|
| Generic OCDS red flags | ✅ 10 indicators | ✅ Included as baseline |
| SA procurement threshold awareness (R200k / R500k) | ❌ | ✅ |
| Tembisa-specific contract splitting patterns | ❌ | ✅ DBSCAN temporal clustering |
| Commodity-level price anomaly (health sector) | ❌ | ✅ Isolation Forest per category |
| Supplier relationship network (family clusters) | ❌ | ✅ NetworkX + Louvain |
| NLP on tender justification documents | ❌ | ✅ TF-IDF + transformer classifier |
| Retrospective validation against SIU ground truth | ❌ | ✅ Tembisa reconstruction |

The gap between Cardinal's recall and Pro-Cure's recall on known Tembisa actors **is the core research finding.**

---

## What Pro-Cure does

Pro-Cure applies data mining and NLP to South Africa's public procurement data to automatically detect the exact corruption patterns documented in the Tembisa SIU investigation:

| Corruption pattern | Detection method |
|---|---|
| Contract splitting — awards kept just below R500k threshold | DBSCAN temporal clustering + threshold proximity scoring |
| Inflated prices vs market rates | Isolation Forest per commodity category |
| Awards concentrated in connected supplier families | Herfindahl-Hirschman Index + Association Rule Mining |
| Supplier relationship networks (227 companies, 4 families) | NetworkX graph analysis + Louvain community detection |
| Vague single-source justifications without substance | NLP classifier (TF-IDF + transformer fine-tuning) |
| Near-identical tender specs written for a specific supplier | TF-IDF cosine similarity across tender documents |

All signals are combined into a single **Pro-Cure Risk Index (0–100)** per contract and per supplier.

---

## Data sources

| Source | Coverage | Licence | URL |
|---|---|---|---|
| National Treasury OCDS | Feb 2017 – present · 33,689 tenders · 9,090 awards | PDDL (fully open) | [data.open-contracting.org](https://data.open-contracting.org/en/publication/143) |
| Vulekamali | Deviation (single-source) awards | Open | [vulekamali.gov.za](https://vulekamali.gov.za/datasets) |

---

## Modules applied

Developed as part of BSc Hons Computer Science at the University of Pretoria:

- **COS 783 — Data Mining:** contract splitting (DBSCAN), price anomaly (Isolation Forest), supplier concentration (HHI), association rule mining (Apriori/FP-Growth), network analysis (NetworkX, Louvain)
- **COS 760 — NLP:** tender document classification (TF-IDF + transformer), key phrase extraction (spaCy NER), specification similarity (cosine similarity), formality scoring

---

## Repository structure

```
pro-cure/
│
├── data/
│   ├── raw/                        # Raw OCDS downloads (gitignored)
│   ├── processed/                  # Cleaned master dataset
│   └── health_sector/              # Health-sector filtered subset
│
├── notebooks/
│   ├── 01_data_parsing.ipynb
│   ├── 02_eda.ipynb
│   ├── 03_cardinal_baseline.ipynb  # ← Cardinal run + gap analysis
│   ├── 04_contract_splitting.ipynb
│   ├── 05_price_anomaly.ipynb
│   ├── 06_supplier_network.ipynb
│   ├── 07_nlp_pipeline.ipynb
│   └── 08_risk_index.ipynb         # Cardinal vs Pro-Cure results table
│
├── models/
│   └── checkpoints/
│
├── app/
│   └── app.py                      # Streamlit dashboard
│
├── reports/
│   └── tembisa_reconstruction.md
│
├── results/
│   ├── cardinal_baseline.csv       # Cardinal scores on SA health data
│   ├── procure_risk_scores.csv     # Pro-Cure Risk Index scores
│   └── cardinal_vs_procure.csv     # Head-to-head comparison
│
├── requirements.txt
├── .gitignore
└── README.md
```

---

## Results

*To be updated as experiments complete.*

### Cardinal baseline vs Pro-Cure Risk Index — Tembisa validation

| System | Precision | Recall | F1 | Notes |
|---|---|---|---|---|
| Cardinal (10 generic indicators) | — | — | — | Global baseline |
| Pro-Cure (SA-specific signals) | — | — | — | |
| Pro-Cure + Cardinal (combined) | — | — | — | |

Ground truth: companies named in the [Tembisa SIU Report](https://www.siu.org.za/)

---

## Demo

*Streamlit dashboard link — added after Phase 5 deployment.*

Three views:
1. **National overview** — risk map by department and province
2. **Supplier deep-dive** — full risk profile with Cardinal vs Pro-Cure score breakdown
3. **Red flag feed** — latest high-risk contracts ranked by Pro-Cure Risk Index

---

## Tembisa reconstruction

`reports/tembisa_reconstruction.md` answers: **if Pro-Cure had existed in January 2021, would it have flagged the Maumela, Mazibuko and Govindraju networks before Babita was killed in August?**

The reconstruction runs both Cardinal and Pro-Cure on 2021 data. The difference in what they catch is the value of SA-specific signal engineering.

---

## Acknowledgements

**The journalism that started this.**
Pro-Cure exists because of ***The Shadow State: Why Babita Deokaran Had to Die*** by **Jeff Wicks** (Tafelberg, 2025) — the book that made the human cost of procurement corruption impossible to ignore. Wicks, a News24 investigative journalist and multiple Taco Kuiper Award winner, spent years unravelling the web of corruption that led to Babita's assassination. His work is the reason this project exists. If you haven't read it, buy it.

**The technical foundation.**
Built in alignment with the [Data Science for Social Impact (DSFSI)](https://dsfsi.github.io/) research group, University of Pretoria. Baseline methodology from the [Open Contracting Partnership Cardinal library](https://www.open-contracting.org/2024/06/12/cardinal-an-open-source-library-to-calculate-public-procurement-red-flags/). Ground truth from the [Special Investigating Unit](https://www.siu.org.za/) Tembisa Hospital report.

---

## Citation

```bibtex
@software{procure_2026,
  author  = {Kamogelo Jabulile Motlhale},
  title   = {Pro-Cure: AI-Powered Procurement Anomaly Detection for South African Public Health},
  year    = {2026},
  url     = {https://github.com/jabbzthecoder/pro-cure}
}
```

---

## Licence

Code: [MIT Licence](LICENSE)
Processed data derivatives: [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/)
Source data: [PDDL](https://opendatacommons.org/licenses/pddl/) — National Treasury

---

*"The only thing necessary for the triumph of evil is for good people to do nothing."*
*Babita Deokaran did something. Pro-Cure continues it.*
