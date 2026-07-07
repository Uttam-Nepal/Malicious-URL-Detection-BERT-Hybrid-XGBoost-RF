# Malicious URL Detection using a Hybrid BERT–XGBoost–Random Forest Framework

Master's dissertation implementation: a binary (Safe/Unsafe) malicious URL classifier combining
fine-tuned BERT contextual embeddings, handcrafted lexical URL features, and a novel
XGBoost-leaf-encoding + Random Forest ensemble.

## Repository Structure

```
.
├── Malicious_URL_Detection_BERT_Hybrid_XGBoost_RF.ipynb   # Main notebook (run top to bottom)
├── data/
│   └── malicious_phish.csv                                 # Kaggle Malicious URL Dataset (651,191 rows)
├── results/
│   ├── figures/    # All generated figures (300 DPI, PNG)
│   ├── tables/     # All CSV result exports (metrics, tuning sweeps, error analysis)
│   └── models/     # Saved model artifacts (BERT model/tokenizer, embeddings, joblib models)
└── report/
    └── Dissertation_Report.docx   # Full write-up: methodology, results, and interpretation per section
```

## Pipeline Overview

1. **Data Cleaning** — remove missing values and duplicate URLs (651,191 → 641,119 rows)
2. **Binary Labeling** — Benign → Safe (0); Phishing/Malware/Defacement → Unsafe (1)
3. **Balanced Sampling** — 25,000 Safe + 25,000 Unsafe (seed=42), 80/20 stratified train/test split
4. **BERT Fine-Tuning** — `bert-base-uncased`, 2 epochs, LR 2e-5, batch size 16, weight decay 0.01, max_length 128
5. **CLS Embedding Extraction** — 768-dimensional embeddings for all URLs
6. **Classical Classifiers** — SVM, XGBoost, Random Forest on BERT embeddings alone
7. **Handcrafted URL Features** — 20 lexical/structural features (length, entropy, character counts, keywords, etc.)
8. **Hybrid Features** — 768 + 20 = 788-dimensional combined feature vector, evaluated with SVM/XGBoost/RF
9. **Proposed Model** — Hybrid features → XGBoost → leaf-node indices → concatenate with hybrid features → final Random Forest
10. **Hyperparameter Tuning** — sensitivity sweeps for XGBoost (n_estimators, max_depth) and Random Forest (n_estimators)
11. **Final Optimized Model** — retrained with tuned hyperparameters (XGBoost: n_estimators=150, max_depth=10; RF: n_estimators=200)
12. **Master Comparison** — all 9 model configurations ranked by F1-score
13. **Error Analysis** — misclassification patterns, false positive/negative characterization, feature importance

## Results Summary

The **tuned Proposed Hybrid BERT–XGBoost–Random Forest model** achieved the best performance
among all configurations tested:

| Metric    | Value  |
| --------- | ------ |
| Accuracy  | 0.9801 |
| Precision | 0.9806 |
| Recall    | 0.9796 |
| F1-Score  | 0.9801 |
| ROC-AUC   | 0.9953 |

See `report/Dissertation_Report.docx` for the complete results table, per-phase methodology,
and discussion.

## Requirements

- Google Colab (GPU runtime recommended: T4/L4/A100) or a local environment with an NVIDIA GPU
- Python 3.11+
- Key packages: `torch`, `transformers`, `datasets`, `accelerate`, `scikit-learn`, `xgboost`, `pandas`, `numpy`, `matplotlib`, `seaborn`, `joblib` (all installed within the notebook itself)

## How to Run

1. Open `Malicious_URL_Detection_BERT_Hybrid_XGBoost_RF.ipynb` in Google Colab
2. Ensure `data/malicious_phish.csv` is accessible at that relative path (either by cloning this repo into your Colab environment, or uploading the CSV and adjusting `DATA_PATH` in Section 4)
3. Run all cells sequentially, top to bottom
4. Outputs (figures, tables, model artifacts) will populate under `results/`

## Dataset

Kaggle Malicious URL Dataset — 651,191 URLs across four classes (benign, phishing, malware, defacement).

## Reproducibility

All random processes (sampling, splitting, model initialization) use a fixed seed (42) to ensure
reproducibility of every reported result.
