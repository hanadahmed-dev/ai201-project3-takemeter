# TakeMeter — Planning

## Community

I chose NBA discussion communities, especially Reddit-style basketball discussion where fans react to games, players, trades, playoff performances, and league narratives. NBA discourse is active, opinion-heavy, and varied in quality: some comments provide real basketball reasoning, some are bold unsupported claims, and others are emotional reactions or memes. These distinctions matter because fans often debate whether a comment is meaningful analysis or just noise.

---

## Labels

### Label 1: analysis

**Definition:**
A post or comment that makes a specific basketball argument supported by evidence, reasoning, statistics, tactical observation, historical comparison, or clear explanation.

**Clear Example 1:**
"Jokic controls the game even when he only scores 18 because Denver runs offense through his passing. When teams double him, he consistently creates open threes for cutters and shooters."

**Clear Example 2:**
"Minnesota's defense works because Gobert protects the rim while McDaniels pressures the ball handler. That combination limits drives without constantly giving up corner threes."

**Uncertain Example:**
"LeBron is overrated because his playoff record against top-seeded teams is below .500."

**Decision:**
This is uncertain because it uses a statistic but frames it as a provocative claim. I would label it as `hot_take` unless the post gives more context explaining why that stat is meaningful.

---

### Label 2: hot_take

**Definition:**
A bold, confident, or provocative basketball opinion stated with little or no supporting evidence. The claim may be true or false, but the post asserts more than it explains.

**Clear Example 1:**
"Anthony Edwards is already better than prime Dwyane Wade."

**Clear Example 2:**
"The Thunder are getting swept next round. They are not built for playoff basketball."

**Uncertain Example:**
"The Celtics are frauds because they fold whenever the game gets physical."

**Decision:**
This is uncertain because it references a possible pattern, but it does not give specific evidence. I would label it as `hot_take` unless the post includes examples, stats, or specific game situations.

---

### Label 3: reaction

**Definition:**
An immediate emotional response, celebration, frustration, joke, meme, or one-liner with little to no basketball argument.

**Clear Example 1:**
"REFS SOLD 😭😭😭"

**Clear Example 2:**
"NO WAY HE HIT THAT SHOT!!!"

**Uncertain Example:**
"That was the worst late-game coaching I have ever seen."

**Decision:**
This is uncertain because it could become analysis if the post explains the coaching mistake. As written, it is mostly emotional and would be labeled `reaction`.

---

## Label Boundaries and Edge Cases

The hardest edge case is distinguishing `analysis` from `hot_take`. Some comments include one statistic or one basketball phrase but are still mostly provocative opinions. My decision rule is: if the evidence would still support the claim after removing emotional or exaggerated language, label it `analysis`. If the evidence is vague, cherry-picked, or only decorative, label it `hot_take`.

Another edge case is distinguishing `hot_take` from `reaction`. If the post makes a claim about a player, team, coach, or future outcome, label it `hot_take`. If the post is mainly expressing emotion in the moment without a real claim, label it `reaction`.

---

## Mutual Exclusivity Check

Each post should receive exactly one label based on its primary purpose.

* `analysis` is for reasoned basketball arguments.
* `hot_take` is for unsupported or under-supported claims.
* `reaction` is for emotional, meme-like, or immediate responses.

If a post contains multiple elements, I will choose the label based on the dominant function of the post. For example, a post that starts emotionally but then explains specific defensive adjustments would be labeled `analysis`. A post that includes one statistic but mainly uses it to make a provocative claim would be labeled `hot_take`.

---

## Data Collection Plan

I will collect at least 200 NBA-related posts or comments from public basketball discussion spaces such as Reddit-style NBA threads, game discussions, trade discussions, player debates, and playoff reaction threads.

The dataset will contain the following columns:

* `text`: the post or comment text
* `label`: one of `analysis`, `hot_take`, or `reaction`

I will manually label each example using the taxonomy above.

Target label distribution:

* `analysis`: around 30–40%
* `hot_take`: around 30–40%
* `reaction`: around 20–30%

I will avoid allowing one label to dominate the dataset because a heavily imbalanced dataset could make the model learn to predict the majority class too often.

---

## Dataset Split Plan

After collecting and labeling at least 200 examples, I will split the dataset into:

* Training set: 70%
* Validation set: 15%
* Test set: 15%

The test set will be held out until final evaluation. Both the fine-tuned DistilBERT model and the Groq zero-shot baseline will be evaluated on the same test examples.

---

## Model Plan

I plan to fine-tune `distilbert-base-uncased` for a three-class text classification task. This model is small enough to train quickly in Google Colab but strong enough to learn patterns in short online comments.

The label map will be:

```python
label2id = {
    "analysis": 0,
    "hot_take": 1,
    "reaction": 2
}

id2label = {
    0: "analysis",
    1: "hot_take",
    2: "reaction"
}
```

Initial training settings:

* Model: `distilbert-base-uncased`
* Task: sequence classification
* Epochs: 3
* Batch size: 8 or 16
* Learning rate: 2e-5

I may adjust these after looking at validation performance.

---

## Baseline Plan

I will compare the fine-tuned DistilBERT model against a zero-shot Groq baseline using `llama-3.3-70b-versatile`.

The Groq prompt will ask the model to classify each comment into exactly one of the three labels: `analysis`, `hot_take`, or `reaction`. The prompt will include the label definitions but no training examples from the dataset. I will run the baseline on the same held-out test set used for the fine-tuned model.

---

## Evaluation Plan

I will evaluate both models using:

* Overall accuracy
* Per-class precision, recall, and F1
* Confusion matrix
* At least three incorrectly classified examples with written analysis

I will look especially closely at confusion between `analysis` and `hot_take`, because that is the most subjective boundary in the taxonomy. I will also check whether the model overpredicts `reaction` for short comments or overpredicts `hot_take` for strong opinions that actually include reasoning.

---

## Difficult Labeling Examples

### Difficult Example 1

Text:
"LeBron is overrated because his playoff record against top seeds is below .500."

Possible labels:

* `analysis`
* `hot_take`

Decision:
Label as `hot_take` because the comment uses one stat to support a provocative claim but does not explain context, era, roster strength, opponent quality, or why that stat should define the player.

---

### Difficult Example 2

Text:
"That was the worst late-game coaching I have ever seen."

Possible labels:

* `hot_take`
* `reaction`

Decision:
Label as `reaction` because it is an immediate emotional response. If the comment explained the specific rotation, timeout, or play call mistake, it would become `analysis`.

---

### Difficult Example 3

Text:
"The Celtics cannot win close games because their offense turns into isolation ball late."

Possible labels:

* `analysis`
* `hot_take`

Decision:
Label as `analysis` if the surrounding comment explains late-game possessions or gives examples. If it appears alone with no support, label as `hot_take`.

---
## Community Review Process

Before finalizing my label taxonomy, I reviewed approximately 30–40 NBA discussion comments from public basketball discussion threads. During this review, I observed three common categories of discourse: detailed basketball analysis, unsupported hot takes, and immediate emotional reactions. These patterns appeared frequently enough that the labels analysis, hot_take, and reaction covered the vast majority of comments without requiring an additional category.

---
## Expected Challenges

The biggest challenge will be maintaining consistent boundaries between labels. Online NBA comments often mix emotion, claims, and reasoning in the same post. I will label based on the primary function of the comment rather than individual words.

Another challenge is class balance. Reaction comments are easy to find, but too many of them could make the model overconfident on short emotional text. I will deliberately collect enough analysis and hot take examples so the dataset remains balanced.

---
## Evaluation Metrics

I will evaluate both the fine-tuned DistilBERT model and the Groq zero-shot baseline using multiple metrics.

### Accuracy

Accuracy measures the percentage of comments that are classified correctly overall. This provides a simple high-level measure of performance.

### Precision

Precision measures how often a predicted label is correct. This is important because I want the classifier to avoid incorrectly labeling emotional reactions as analysis.

### Recall

Recall measures how many examples from each true class are successfully identified. This is important because I want the model to find analysis comments rather than missing them.

### F1 Score

F1 balances precision and recall. Because some labels may appear more frequently than others, F1 provides a better measure than accuracy alone.

### Confusion Matrix

A confusion matrix will help identify which labels are most frequently confused. I expect the largest source of confusion to be between analysis and hot_take.

I will report:

* Overall accuracy
* Per-class precision
* Per-class recall
* Per-class F1
* Confusion matrix

---

## Definition of Success

This classifier is intended to distinguish meaningful basketball discussion from unsupported opinions and emotional reactions.

I will consider the project successful if:

* Overall test accuracy is at least 75%.
* The analysis label achieves an F1 score of at least 0.70.
* The fine-tuned DistilBERT model outperforms the Groq zero-shot baseline on overall accuracy.
* The confusion matrix shows that most mistakes occur between analysis and hot_take rather than random misclassifications.

For a real deployment, I would consider the classifier useful if it consistently identifies analysis comments while avoiding excessive false positives.

---

## AI Tool Plan

### Label Stress-Testing

Before beginning annotation, I will use ChatGPT to generate comments that sit on the boundary between analysis and hot_take, as well as between hot_take and reaction.

If I find comments that cannot be classified consistently using my current definitions, I will refine the label definitions and decision rules before collecting the full dataset.

---

### Annotation Assistance

I may use ChatGPT to generate an initial label suggestion for some comments. However, every example will be manually reviewed before being added to the dataset.

If AI-generated label suggestions are used, I will track which examples were pre-labeled and disclose this process in the README.

The final label assignment will always be determined by manual review.

---

### Failure Analysis

After evaluating the model, I will provide examples of incorrect predictions to ChatGPT and ask it to identify common patterns.

Potential patterns include:

* Confusion between analysis and hot_take.
* Difficulty with short comments.
* Difficulty with sarcasm.
* Difficulty with comments containing both emotion and reasoning.

Any patterns suggested by the AI will be verified manually by reviewing the original examples before including them in the final report.
