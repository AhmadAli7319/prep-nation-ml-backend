# Prep Nation — ML Prediction Backend

Prep Nation is a Flutter mobile app that predicts likely board exam topics for BISE Abbottabad, Classes 9–11. This repository contains the **ML/statistical prediction backend** — the frontend and other app components are developed separately by teammates.

## Overview

For each subject and class, the pipeline analyzes past-paper topic appearance patterns and outputs a ranked list of topics students should prioritize, categorized by prediction confidence.

## Methodology

**Feature Engineering**
Five engineered features drive the predictions:
- `Gaps` — differences between the years a topic has appeared
- `Frequency_Last_5` — how often a topic appeared in the last 5 years
- `Appeared_last_year` — whether the topic appeared in the most recent exam
- `rolling_3` — rolling frequency over the last 3 years
- `chapter_weight` — weight derived from BISE's official Table of Specifications

**Modeling Approach**
- Time-based train/test split: trained on data before 2025, tested on 2025
- Models compared: Logistic Regression, Random Forest, XGBoost (via scikit-learn Pipeline objects)
- Logistic Regression consistently outperformed Random Forest and XGBoost on these dataset sizes
- Recall was used as the primary evaluation metric (prioritizing coverage of topics that will actually appear)

**Statistical Ranking (for data-limited subjects)**
For subjects with insufficient historical data (e.g. 9th Computer Science, 9th/10th Pakistan Studies), a statistical score is used instead of a trained model:

```
Score = Appearance_Rate - (Gap / Gap.max()) × 0.3
```

**Prediction Output**
- Topics filtered at ≥0.5 predicted probability
- Capped at the top 25 topics per subject
- Categorized as:
  - **A — Must Prepare** (probability ≥ 0.7)
  - **B — Should Prepare** (probability 0.5–0.7)

## Data Notes

- Excel files in each folder contain topic-level data manually compiled from past paper analysis (year appeared, chapter, frequency) — not full paper text or scanned content.
- Known data issues identified and resolved during development: a data leakage column in 10th-class Math, chapter weight sums not summing to 1.0 in several subjects, an inconsistent chapter-name string requiring exact matching in 11th Physics, and fabricated topic entries removed from 12th Biology.
- 10th-class Computer Science was dropped due to insufficient/incompatible data.

## Repository Structure

```
prep-nation-ml-backend/
├── 9th_modeling/     — Notebooks, topic/feature Excel files, and predictions for Class 9
├── 10th_modeling/    — Notebooks, topic/feature Excel files, and predictions for Class 10
├── 11th_modeling/    — Notebooks, topic/feature Excel files, and predictions for Class 11
└── README.md
```

Each class folder contains, per subject:
- A Jupyter notebook with the modeling/analysis pipeline
- Excel file(s) with compiled topic data and engineered features
- Excel file(s) with final predicted topics
- Supporting visualizations (frequency and prediction graphs)

## Output Format

Predictions are exported to a JSON schema for Flutter integration, structured as:

```json
{
  "class": "...",
  "subject": "...",
  "year": "...",
  "board": "...",
  "total_predicted": 0,
  "predicted_topics": [
    {
      "rank": 1,
      "topic_id": "...",
      "topic": "...",
      "chapter": "...",
      "probability": 0.0,
      "category": "A - Must Prepare",
      "years_appeared": [],
      "total_appearances": 0
    }
  ]
}
```

## Scope

This repository covers the ML prediction backend only. The Flutter mobile app frontend and other components of Prep Nation are maintained in a separate repository by other team members.
