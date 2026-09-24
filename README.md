# BERT Incident Classifier

Local incident group/category classification for Excel files.

Version 1 freezes `google-bert/bert-base-uncased` as a text encoder and trains a separate scikit-learn logistic regression classifier on your labeled incidents. Daily inference loads only local artifacts, so it can classify new Excel files without retraining or contacting Hugging Face.

## Input Files

Training Excel columns:

```text
incident_number, short_description, group, category
```

Daily Excel columns:

```text
incident_number, short_description
```

## Setup

Run these commands in PowerShell from this folder:

```powershell
uv sync
```

## Train

The first training run downloads `google-bert/bert-base-uncased` from Hugging Face and saves the model, tokenizer, classifier, evaluation report, and confusion matrix locally.

```powershell
uv run incident-classifier train `
  --training-excel .\training_incidents.xlsx `
  --artifacts-dir .\artifacts `
  --confidence-threshold 0.65
```

Training writes:

```text
artifacts\bert-base-uncased\
artifacts\incident_classifier.joblib
artifacts\evaluation_report.txt
artifacts\confusion_matrix.csv
```

The train command uses an 80/20 split. It stratifies by combined `group/category` label when each label has enough rows for both train and test sets; otherwise it falls back to a random 80/20 split and says so in the report.

## Classify Daily Incidents

Daily classification loads the saved artifacts from disk with local model loading enabled.

```powershell
uv run incident-classifier classify-daily `
  --daily-excel .\daily_incidents.xlsx `
  --artifacts-dir .\artifacts `
  --output-excel .\classified_incidents.xlsx
```

Output columns:

```text
incident_number, predicted_group, predicted_category, confidence, manual_review
```

Rows below the confidence threshold are marked `manual_review = TRUE`.

## Checks

```powershell
uv run pytest
```
