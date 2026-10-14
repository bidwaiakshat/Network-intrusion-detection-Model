# Network Intrusion Detector Using Machine Learning

**Challenge 26: Catch the Attack the Signatures Miss**

An ML-powered Network Intrusion Detection System (NIDS) that classifies network flows as **Normal or Attack** using machine learning. The project explores multiple classifiers, feature selection, hyperparameter optimization, out-of-fold (OOF) predictions, threshold optimization, and stacking ensembles to improve intrusion detection.

The goal is to support security analysts by surfacing suspicious network traffic for investigation rather than automatically blocking connections.

## Project Overview

Traditional signature-based intrusion detection systems depend on known attack patterns. Machine learning can help identify traffic patterns associated with malicious activity, including patterns that may not match a predefined signature.

This project develops and evaluates a binary network intrusion detection pipeline using the **UNSW-NB15 dataset**.

### Objectives

- Classify network traffic as normal or malicious.
- Compare multiple machine learning algorithms using consistent evaluation methods.
- Reduce false positives while maintaining strong attack detection.
- Use cross-validation and OOF predictions for more reliable model and ensemble selection.
- Evaluate the selected models on the official held-out test set.
- Document generalization gaps and limitations instead of relying on accuracy alone.

## Dataset

**Dataset:** [UNSW-NB15 Official Dataset](https://research.unsw.edu.au/projects/unsw-nb15-dataset)

The project uses the official pre-split training and testing CSV files.

| Dataset | Number of rows |
|---|---:|
| Training set | 175,341 |
| Official test set | 82,332 |

**Target variable:** `label`

- `0` → Normal traffic
- `1` → Attack traffic

The pipeline excludes identifier and target-related columns from the model inputs. In particular, `id` is excluded as an identifier and `attack_cat` is excluded to prevent attack-category information from leaking into the binary classifier.

The final binary pipeline uses a selected representation of **19 features**.

> Download the dataset from the official source and place the two CSV files in the location expected by the notebooks. The dataset itself does not need to be committed to this repository.

## Repository Structure

| File | Purpose |
|---|---|
| `V1.ipynb` | Earlier development and experimental work; retained as a record of the initial approach. |
| `V2_withNO_MLP.ipynb` | Binary model development, preprocessing, hyperparameter tuning, and candidate-model evaluation. MLP was removed from the current pipeline because it was less competitive and added training cost. |
| `V3.ipynb` | Reconstructs the selected feature representation and handles the model/artifact handoff reproducibly. |
| `V4.ipynb` | OOF-based evaluation, threshold selection, complementary-error analysis, stacking-ensemble selection, and official-test evaluation. |

**Recommended workflow:** use V2 for model development, V3 for the artifact and feature handoff, and V4 for the ensemble evaluation. V1 is an earlier experimental version, not the final result.

## Machine Learning Workflow

The project follows this general pipeline:

1. **Data preparation:** Load the official UNSW-NB15 splits, inspect the data, remove identifier and leakage-prone columns, and prepare the feature representation.
2. **Feature selection:** Retain a consistent 19-feature representation for the final binary pipeline.
3. **Model development:** Compare Random Forest, XGBoost, LightGBM, CatBoost, and Linear SVM.
4. **Hyperparameter tuning:** Use stratified randomized search on the training data, optimizing F1.
5. **Cross-validation:** Run a separate five-fold stratified evaluation and generate OOF predictions.
6. **Threshold optimization:** Select decision thresholds using training-set OOF predictions and the chosen operating constraints.
7. **Ensemble analysis:** Examine complementary model errors and build a stacking ensemble using OOF evidence.
8. **Final evaluation:** Evaluate the frozen model configuration on the official test set.

The official test set is reserved for final evaluation, not feature selection, hyperparameter tuning, or ensemble selection.

## Models Evaluated

- Random Forest
- XGBoost
- LightGBM
- CatBoost
- Linear Support Vector Classifier (LinearSVC)

The MLP experiment was discontinued in the current pipeline because its performance and computational cost did not justify retaining it.

## Evaluation Metrics

The project uses multiple metrics because accuracy alone can hide important weaknesses in an intrusion detector.

| Metric | What it measures |
|---|---|
| Precision | How many predicted attacks are actually attacks |
| Recall | How many actual attacks are detected |
| F1-score | Balance between precision and recall |
| False Positive Rate (FPR) | Proportion of normal traffic incorrectly flagged as malicious |
| ROC-AUC | Ability to rank attack traffic above normal traffic |
| Confusion matrix | Counts of true positives, true negatives, false positives, and false negatives |

OOF predictions are used to evaluate candidate models and select the ensemble without relying on predictions from models trained on the same samples.

## Current Benchmark Results

The following results are from the documented binary IDS experiment.

| Evaluation | Model | F1-score | Additional result |
|---|---|---:|---|
| Five-fold OOF evaluation | LightGBM | 0.9542 | Strongest individual OOF F1 |
| Official held-out test | CatBoost | 0.9470 | Strongest individual test F1 |
| Official held-out test | Frozen stacking ensemble | 0.9444 | FPR = 0.0740 |

The stacking ensemble used a frozen threshold of `0.681141`.

These results show that the ensemble was competitive, but **it did not outperform individual CatBoost on official-test F1**. Its selection was based on OOF evidence and complementary-error analysis, rather than tuning against the official test labels.

The official test results are a benchmark on UNSW-NB15. They do not establish production readiness or guarantee performance on live network traffic.

## Installation

Use Python with Jupyter Notebook or JupyterLab.

Install the main dependencies:

```bash
pip install numpy pandas scikit-learn catboost xgboost lightgbm joblib matplotlib seaborn jupyter
```

Launch Jupyter:

```bash
jupyter notebook
```

Open the notebook you want to run. Make sure the dataset files are accessible from the paths configured in the notebook.

## Reproducibility Notes

- Use the official UNSW-NB15 training and testing splits.
- Follow the notebook workflow and use the same feature representation when evaluating saved models.
- Keep preprocessing, selected features, model artifacts, and thresholds consistent.
- Do not optimize model selection or thresholds on the official test set.
- OOF predictions should be generated by models that have not trained on the corresponding fold.
- Report validation and official-test results separately.
- Results can vary if the feature representation, split, random seed, or model configuration changes.

## Limitations and Future Work

This is a hackathon-stage research prototype, not a production-ready intrusion prevention system.

Potential next steps include:

- External validation on additional network intrusion datasets.
- Further analysis of false positives and false negatives.
- Explainability using feature importance and SHAP.
- Monitoring for changes in network traffic and model drift.
- Integration with an analyst-facing alert dashboard.
- Further testing under realistic deployment conditions.

## Intended Use

The detector is intended to provide **decision support for security analysts**. Predictions should be treated as signals for further investigation, not definitive proof that traffic is malicious.

## Acknowledgements

This project uses the UNSW-NB15 dataset developed by researchers at UNSW Canberra. Please consult the [official dataset page](https://research.unsw.edu.au/projects/unsw-nb15-dataset) for dataset documentation and citation requirements.
