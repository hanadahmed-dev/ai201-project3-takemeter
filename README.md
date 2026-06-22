# Project 3: TakeMeter – Classifying NBA Reddit Comments

## Overview

This project builds a text classification system for NBA discussion posts collected from r/nba. The goal is to classify comments into one of three categories:

* **analysis** – Reasoned explanations, evidence-based arguments, historical comparisons, or discussion supported by logic.
* **hot_take** – Strong opinions, predictions, speculation, or controversial claims with little supporting evidence.
* **reaction** – Emotional responses, jokes, observations, sarcasm, or immediate reactions to events.

The project compares two approaches:

1. A zero-shot LLM baseline using Groq.
2. A fine-tuned DistilBERT model trained on a custom labeled dataset.

---

# Dataset

The dataset contains **220 manually labeled NBA discussion comments** collected from Reddit discussions.

### Label Distribution

| Label    | Count |
| -------- | ----: |
| analysis |   122 |
| hot_take |    57 |
| reaction |    41 |
| Total    |   220 |

Examples:

### Analysis

> "The Bucks do not control most of their future draft picks, which makes a traditional rebuild difficult."

### Hot Take

> "Pairing Giannis with Tatum would be absolutely terrifying."

### Reaction

> "I just want the trade to happen already."

---

# Model

### Fine-Tuned Model

Model:

```text
distilbert-base-uncased
```

Training settings:

| Parameter     | Value |
| ------------- | ----- |
| Epochs        | 3     |
| Learning Rate | 2e-5  |
| Batch Size    | 16    |
| Weight Decay  | 0.01  |
| Warmup Steps  | 50    |

Dataset split:

| Split      | Size |
| ---------- | ---: |
| Train      |  154 |
| Validation |   33 |
| Test       |   33 |

---

# Baseline Model

The baseline used a zero-shot prompt with Groq.

The prompt instructed the model to classify NBA discussion comments into:

* analysis
* hot_take
* reaction

The model was required to return only the label name.

---

# Evaluation Results

## Overall Accuracy

| Model                   | Accuracy |
| ----------------------- | -------: |
| Zero-Shot Groq Baseline |   72.73% |
| Fine-Tuned DistilBERT   |   57.58% |

The fine-tuned model underperformed the baseline by approximately **15 percentage points**.

---

## Baseline Per-Class Metrics

| Label    | Precision | Recall |   F1 |
| -------- | --------: | -----: | ---: |
| analysis |      0.94 |   0.79 | 0.86 |
| hot_take |      1.00 |   0.38 | 0.55 |
| reaction |      0.43 |   1.00 | 0.60 |

---

## Fine-Tuned Model Per-Class Metrics

| Label    | Precision | Recall |   F1 |
| -------- | --------: | -----: | ---: |
| analysis |      0.58 |   1.00 | 0.73 |
| hot_take |      0.00 |   0.00 | 0.00 |
| reaction |      0.00 |   0.00 | 0.00 |

---

# Confusion Matrix

Fine-Tuned Model

| True Label | Predicted Analysis | Predicted Hot Take | Predicted Reaction |
| ---------- | -----------------: | -----------------: | -----------------: |
| analysis   |                 19 |                  0 |                  0 |
| hot_take   |                  8 |                  0 |                  0 |
| reaction   |                  6 |                  0 |                  0 |

### Interpretation

The model predicted **analysis for every test example**.

While it successfully classified all analysis examples, it completely failed to learn the distinctions between hot_take and reaction.

---

# Error Analysis

## Error #1

### Text

> "Chicago Bulls: the last 30 years."

### True Label

reaction

### Predicted

analysis

### Analysis

This comment is extremely short and relies on sarcasm. Human readers understand it as a humorous reaction, but the model likely interpreted it as a factual statement about the Bulls organization.

---

## Error #2

### Text

> "I think this gets done today."

### True Label

hot_take

### Predicted

analysis

### Analysis

This comment is a prediction without evidence, which matches the hot_take definition. The model appears to associate trade discussions with analytical language and therefore misclassified it.

---

## Error #3

### Text

> "I wonder if even Giannis knows who the third team is."

### True Label

reaction

### Predicted

analysis

### Analysis

This comment expresses amusement and uncertainty rather than analysis. The model focused on the trade-related topic rather than the rhetorical nature of the statement.

---

# Sample Classifications

| Text                                                               | Prediction           | Confidence |
| ------------------------------------------------------------------ | -------------------- | ---------: |
| "Just stay in Milwaukee and go trade for someone like Zach LaVine."    | analysis             |       0.40 |
| "Chicago Bulls: the last 30 years."                                   | analysis  |       0.42 |
| "I think this gets done today."                                | analysis      |       0.37 |
| "TThe Bucks are playing the Heat like a fiddle. "       | analysis             |       0.40 |
| "The Bucks do not control most of their future draft picks."    | analysis             |       0.81 |

### Example Discussion

The statement:

> "The Bucks do not control most of their future draft picks."

was correctly classified as analysis because it provides a factual observation and supports a broader argument about rebuilding strategy.

---

# Reflection

The model captured basketball-related discussion topics more effectively than the intended communication styles.

Rather than learning the difference between analysis, hot_take, and reaction, the model learned that most comments belonged to the majority class, analysis. As a result, it predicted analysis for every test example.

The zero-shot baseline significantly outperformed the fine-tuned model because it was able to reason about tone, sarcasm, and intent without relying on a relatively small training dataset.

The project demonstrated that fine-tuning does not automatically improve performance. Dataset quality, class balance, and dataset size are all critical factors.

---

# Spec Reflection

## How the Specification Helped

The project specification provided a clear process for collecting data, creating labels, establishing a baseline, fine-tuning a model, and performing error analysis. The structured workflow made it easy to compare traditional fine-tuning against a modern LLM baseline.

## How My Implementation Diverged

My final dataset was not perfectly balanced across labels. Analysis examples were significantly more common than reaction examples. This likely contributed to the model collapsing toward the majority class during training. A more balanced dataset may have produced stronger fine-tuning results.

---

# AI Usage

## Dataset Development

I used ChatGPT to help rewrite and standardize Reddit comments collected from r/nba discussions. I reviewed each example manually before adding it to the final dataset.

## Label Review

I used ChatGPT to help identify potentially inconsistent labels and duplicate examples. Several labels were manually corrected after reviewing the AI suggestions.

## Error Analysis

After evaluation, I used ChatGPT to identify common patterns among misclassified examples. The AI suggested that sarcasm, short comments, and predictions were common failure cases. I verified these patterns manually before including them in this report.

---

# Repository Contents

```text
README.md
planning.md
updated_dataset_deduped.csv
evaluation_results.json
confusion_matrix.png
```
# Demo Link
https://drive.google.com/file/d/1z2VrgQ0W0WqLmzTIpzIKpXcLSmqMUeLB/view?usp=drive_link
