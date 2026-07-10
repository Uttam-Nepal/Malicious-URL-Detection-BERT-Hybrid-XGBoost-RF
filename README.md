# DeepTrace: A Hybrid Transformer-Ensemble Framework for Malicious URL Detection

**Beyond Lexical Features: Leveraging Transformer Embeddings and Tree-Based Leaf Encoding for URL Threat Classification**

Master's dissertation implementation: a binary (Safe/Unsafe) malicious URL classifier combining
fine-tuned BERT contextual embeddings, handcrafted lexical URL features, and a novel
XGBoost-leaf-encoding + Random Forest ensemble — **DeepTrace**.

## Repository Structure

```
.
├── DeepTrace_Malicious_URL_Detection.ipynb   # Main notebook (run top to bottom)
├── data/
│   └── malicious_phish.csv                    # Kaggle Malicious URL Dataset (651,191 rows)
├── Result/
│   ├── figures/    # All generated figures (300 DPI, PNG)
│   ├── tables/     # All CSV result exports (metrics, tuning sweeps, error analysis)
│   └── models/     # Saved model artifacts (BERT model/tokenizer, embeddings, joblib models)
├── Report/
│   └── Dissertation_Report.pdf   # Full write-up: methodology, results, and interpretation
└── A notebook snapshot/          # Archived earlier 9-model exploratory notebook (reference only)
```

## Pipeline Overview

1. **Data Cleaning** — remove missing values and duplicate URLs (651,191 → 641,119 rows)
2. **Binary Labeling** — Benign → Safe (0); Phishing/Malware/Defacement → Unsafe (1)
3. **Balanced Sampling** — 25,000 Safe + 25,000 Unsafe (seed=42), 80/20 stratified train/test split
4. **BERT Fine-Tuning** — `bert-base-uncased`, 2 epochs, LR 2e-5, batch size 16, weight decay 0.01, max_length 128
5. **CLS Embedding Extraction** — 768-dimensional embeddings for all URLs
6. **Classical Classifiers** — XGBoost and Random Forest on BERT embeddings alone
7. **Handcrafted URL Features** — 20 lexical/structural features (length, entropy, character counts, keywords, etc.)
8. **Hybrid Features** — 768 + 20 = 788-dimensional combined feature vector, evaluated with XGBoost and Random Forest
9. **DeepTrace (Proposed Model)** — Hybrid features → XGBoost → leaf-node indices → concatenate with hybrid features → final Random Forest
10. **Hyperparameter Tuning** — sensitivity sweeps for XGBoost (n_estimators, max_depth) and Random Forest (n_estimators)
11. **Final Tuned DeepTrace** — retrained with tuned hyperparameters (XGBoost: n_estimators=150, max_depth=10; RF: n_estimators=200)
12. **Master Comparison** — 6 final model configurations ranked by F1-score
13. **Error Analysis** — misclassification patterns, false positive/negative characterization, feature importance

**Note:** Support Vector Machine (SVM) classifiers were evaluated during earlier exploratory
research (see `A notebook snapshot/`) but are excluded from this final notebook and paper, since
DeepTrace's architecture does not use SVM and the final comparison focuses on the six models
directly relevant to the proposed pipeline's development.

## Results Summary

**DeepTrace (Final Tuned Model)** achieved the best performance among all six final configurations:

| Rank | Model                           | Accuracy   | Precision  | Recall     | F1-Score   |
| ---- | ------------------------------- | ---------- | ---------- | ---------- | ---------- |
| 1    | **DeepTrace (Proposed, Tuned)** | **0.9801** | **0.9806** | **0.9796** | **0.9801** |
| 2    | Hybrid + XGBoost                | 0.9800     | 0.9808     | 0.9792     | 0.9800     |
| 3    | BERT + XGBoost                  | 0.9797     | 0.9802     | 0.9792     | 0.9797     |
| 4    | Hybrid + Random Forest          | 0.9793     | 0.9798     | 0.9788     | 0.9793     |
| 5    | BERT + Random Forest            | 0.9789     | 0.9788     | 0.9790     | 0.9789     |
| 6    | BERT (standalone)               | 0.9777     | 0.9778     | 0.9777     | 0.9777     |

DeepTrace also achieved a ROC-AUC of 0.9953 and made 199 total misclassifications out of
10,000 test URLs — the fewest of any model evaluated, and 24 fewer than the standalone BERT
baseline.

Feature importance analysis on the final tuned Random Forest revealed that BERT embeddings
account for 97.46% of the model's decision-making importance, with XGBoost leaf-index features
contributing 2.53%, and the 20 handcrafted lexical features contributing only 0.02%.

See `Report/Dissertation_Report.pdf` for the complete results, per-phase methodology, and discussion.

## Requirements

- Google Colab (GPU runtime recommended: T4/L4/A100) or a local environment with an NVIDIA GPU
- Python 3.11+
- Key packages: `torch`, `transformers`, `datasets`, `accelerate`, `scikit-learn`, `xgboost`, `pandas`, `numpy`, `matplotlib`, `seaborn`, `joblib` (all installed within the notebook itself)

## How to Run

1. Open `DeepTrace_Malicious_URL_Detection.ipynb` in Google Colab
2. Upload `data/malicious_phish.csv` when prompted in Section 4 (or adjust `DATA_PATH` if using a repo-relative path)
3. Run all cells sequentially, top to bottom
4. Outputs (figures, tables, model artifacts) will populate as the notebook runs

## Dataset

Kaggle Malicious URL Dataset — 651,191 URLs across four original classes (benign, phishing,
malware, defacement), collapsed into binary Safe/Unsafe for this study.

**Source:** https://www.kaggle.com/datasets/marryjanety/phishing-url-dataset-url-and-label

## Reproducibility

All random processes (sampling, splitting, model initialization) use a fixed seed (42) to ensure
reproducibility of every reported result.

## Related Application

A separate web application prototype (FastAPI backend + simple frontend) using the final tuned
DeepTrace model for real-time phishing URL detection was developed as a companion project. It is
not part of this research repository.
