# Pro-Cure
### AI-powered procurement anomaly detection for South African public institutions

*In memory of Babita Deokaran — Chief Financial Officer, Tembisa Hospital.*
*Assassinated 23 August 2021 after exposing R2 billion in procurement corruption.*
*She found it by hand. Pro-Cure finds it automatically.*

---

## The problem

On 23 August 2021, Babita Deokaran was shot nine times outside her Johannesburg home. She was the key witness in a corruption investigation into the Gauteng Department of Health — having manually traced how R2 billion in public health funds was systematically looted through fraudulent procurement.

The corruption was not hidden. It was hidden in plain sight — in thousands of contracts just below regulatory thresholds, awarded repeatedly to the same connected suppliers, justified with vague single-source deviations, and never flagged because no system existed to look for the patterns.

**Pro-Cure is that system.**

---

## Scope — generic first, then sector-specific

Pro-Cure is built on the **full National Treasury OCDS dataset** — all 33,689 tenders across every government department (2017–2026). This is a deliberate design decision: training on the complete dataset produces more robust models and enables cross-sector comparison.

A health sector lens is applied as a downstream filter — the same models that detect corruption in SAPS procurement, DPWI contracts, or municipal awards will detect it in Tembisa Hospital too. One system, every department.

| Scope | Records | Notes |
|---|---|---|
| Generic (all departments) | 33,689 tenders · 9,090 awards | Full OCDS dataset — model training |
| Health sector subset | 1,376 tenders · 858 with award values | Downstream filter for Tembisa validation |
| Future: SAPS, DPWI, municipalities | Same pipeline, different filter | No retraining required |

---

## What exists — and what's missing

The [Open Contracting Partnership](https://www.open-contracting.org/) released **Cardinal** in 2024 — an open-source library calculating 10 generic procurement red flag indicators on any OCDS dataset. It is the closest existing tool to Pro-Cure.

Pro-Cure runs Cardinal first as a baseline, then shows what SA-specific signals catch that Cardinal misses:

| Capability | Cardinal | Pro-Cure |
|---|---|---|
| Generic OCDS red flags | ✅ 10 indicators | ✅ Included as baseline |
| SA procurement threshold awareness (R200k / R500k) | ❌ | ✅ |
| Contract splitting detection across all departments | ❌ | ✅ DBSCAN temporal clustering |
| Commodity-level price anomaly | ❌ | ✅ Isolation Forest per category |
| Supplier relationship network (cross-department) | ❌ | ✅ NetworkX + Louvain |
| NLP on tender justification documents | ❌ | ✅ TF-IDF + transformer classifier |
| Retrospective validation against SIU ground truth | ❌ | ✅ Tembisa reconstruction |

The gap between Cardinal's recall and Pro-Cure's recall on known Tembisa actors **is the core research finding.**

---

## What Pro-Cure does

Pro-Cure applies data mining and NLP to all South African government procurement data to automatically detect documented corruption patterns:

| Corruption pattern | Detection method | Training scope |
|---|---|---|
| Single-source justifications without substance | NLP classifier (TF-IDF + LR/SVM/RF) | All 33,689 tenders |
| Contract splitting — awards just below R500k threshold | DBSCAN temporal clustering | All departments |
| Inflated prices vs market rates | Isolation Forest per commodity | All departments |
| Awards concentrated in connected supplier families | Herfindahl-Hirschman Index + Apriori | All departments |
| Supplier relationship networks | NetworkX + Louvain community detection | All departments |

All signals are combined into a single **Pro-Cure Risk Index (0–100)** per contract and per supplier.

---

## Data sources

The full OCDS download is a normalised relational star schema — 8 CSV files joined on `ocid`:

| File | Rows | Contents |
|---|---|---|
| `main.csv` | 33,689 | All tenders — buyer, description, method, date |
| `awards.csv` | 9,090 | Completed awards + contract values |
| `awards_suppliers.csv` | 9,090 | Supplier names per award |
| `contracts.csv` | 1,223 | Signed contracts |
| `parties.csv` | 9,090 | Company contact details |
| `tender_tenderers.csv` | 3,388 | Bidder records |

> ⚠️ **Schema note:** `tender_value_amount` in `main.csv` is uniformly zero. Real contract values are in `awards.csv → value_amount`. Pro-Cure uses the correct source; tools relying on `main.csv` values will miss all contract amounts.

| Source | Licence | URL |
|---|---|---|
| National Treasury OCDS (full) | PDDL — fully open | [data.open-contracting.org](https://data.open-contracting.org/en/publication/143) |
| Vulekamali deviations | Open | [vulekamali.gov.za](https://vulekamali.gov.za/datasets) |

---

## Ground truth strategy

Two signals combined into one label set for model training:

1. **Procurement method** — `direct` / `limited` / `selective` awards are legally required to have written justification in SA law. They are the documented mechanism for single-source corruption. **407 positive examples** across all departments.
2. **SIU Tembisa syndicates** — known family networks from the SIU report cross-referenced against supplier names. **11 additional positive examples** (NTSAKO SERVICES, RIRHANDZU MAYIJI ATTORNEYS).

Total ground truth: **418 suspicious** vs **37,405 clean** across 33,689 tenders.

---

## Modules applied

Developed as part of BSc Hons Computer Science at the University of Pretoria:

- **COS 783 — Data Mining:** TF-IDF feature engineering, contract splitting (DBSCAN + threshold proximity), price anomaly (Isolation Forest), supplier concentration (HHI), association rule mining (Apriori/FP-Growth), network analysis (NetworkX, Louvain), classical baselines (LR, SVM, RF)
- **COS 760 — NLP:** tender document classification, text normalisation, key phrase extraction (spaCy NER), specification similarity (cosine similarity), formality scoring

---

## Repository structure

```
pro-cure/
│
├── data/
│   ├── raw/full/                   # OCDS star schema CSVs (gitignored)
│   │   ├── main.csv
│   │   ├── awards.csv
│   │   ├── awards_suppliers.csv
│   │   ├── contracts.csv
│   │   ├── parties.csv
│   │   └── tender_tenderers.csv
│   ├── processed/
│   │   └── master_analytical.csv   # Joined analytical table
│   └── health_sector/
│       └── health_sector_master.csv
│
├── notebooks/
│   ├── 01_data_loading.ipynb       # Schema, joins, health filter
│   ├── 02_nlp_classifier.ipynb     # ← NLP single-source classifier (COS 760)
│   ├── 03_cardinal_baseline.ipynb  # Cardinal run + gap analysis
│   ├── 04_contract_splitting.ipynb # DBSCAN + threshold proximity (COS 783)
│   ├── 05_price_anomaly.ipynb      # Isolation Forest (COS 783)
│   ├── 06_supplier_network.ipynb   # NetworkX + Louvain (COS 783)
│   └── 07_risk_index.ipynb         # Pro-Cure Risk Index + Cardinal comparison
│
├── models/
│   ├── nlp_classifier_lr.joblib    # Logistic Regression NLP model
│   └── nlp_tfidf_vectorizer.joblib # TF-IDF vectorizer
│
├── app/
│   └── app.py                      # Streamlit dashboard
│
├── reports/
│   └── tembisa_reconstruction.md
│
├── results/
│   ├── nlp_scores.csv              # NLP risk scores — all 33,689 tenders
│   ├── nlp_confusion_matrix.png
│   ├── cardinal_baseline.csv
│   ├── procure_risk_scores.csv
│   └── cardinal_vs_procure.csv
│
├── .python-version                 # Pinned to 3.12
├── requirements.txt
├── .gitignore
└── README.md
```

---

## Results

*Updated as notebooks complete.*

### NLP Classifier — all departments (Notebook 02)

| Model | CV F1 | Test Precision | Test Recall | Test F1 |
|---|---|---|---|---|
| Logistic Regression | — | — | — | — |
| Linear SVM | — | — | — | — |
| Random Forest | — | — | — | — |

Ground truth: 418 suspicious (direct/limited/selective + SIU Tembisa suppliers) vs 37,405 clean

### Cardinal baseline vs Pro-Cure Risk Index (Notebook 07)

| System | Precision | Recall | F1 | Notes |
|---|---|---|---|---|
| Cardinal (10 generic indicators) | — | — | — | Global baseline |
| Pro-Cure NLP only | — | — | — | |
| Pro-Cure all signals | — | — | — | |
| Pro-Cure + Cardinal combined | — | — | — | |

Validation: Tembisa SIU report named companies + health sector deviation records

---

## Demo

*Streamlit dashboard link — added after Phase 5 deployment.*

Four views:
1. **National overview** — risk map by department and province, all sectors
2. **Supplier deep-dive** — full risk profile, network connections, contract history
3. **Red flag feed** — latest high-risk contracts ranked by Pro-Cure Risk Index
4. **Tembisa reconstruction** — Cardinal vs Pro-Cure on 2021 Gauteng Health data

---

## Tembisa reconstruction

`reports/tembisa_reconstruction.md` answers: **if Pro-Cure had existed in January 2021, would it have flagged the Maumela, Mazibuko and Govindraju networks before Babita was killed in August?**

The reconstruction runs Cardinal first, then Pro-Cure. The difference in what they catch is the value of SA-specific, NLP-augmented signal engineering over generic indicators.

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
  author  = {[Your Name]},
  title   = {Pro-Cure: AI-Powered Procurement Anomaly Detection for South African Public Institutions},
  year    = {2026},
  url     = {https://github.com/JabbzTheCoder/pro-cure}
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
