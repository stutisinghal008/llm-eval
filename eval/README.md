## Evaluation

## What the Evaluation Code Does
The evaluation code formalizes our entire scoring pipeline. Each model’s output file contains a row per question with the model’s predicted answer and its estimated confidence (from self-consistency vote fractions). For a given threshold t, the model is considered to have “answered” a question only if its confidence exceeds t. Everything else counts as an abstention.
The evaluation functions then compute the four metrics:
1. Accuracy@t — among the questions the model answered, how many were correct.
2. Coverage — how often the model answered at all.
3. Penalty-Adjusted Mean Score — the core reliability metric that combines correct answers (+1), confidently wrong answers (negative penalty), and abstentions (0).
4. Overconfidence Rate — the fraction of all questions where the model was wrong and confident enough to answer.

The functions enclosed in this directory implement these rules directly:
decide(p, t) enforces the decision rule (model answers iff p > t).
penalty_score(...) computes the exact score contribution for each example.
accuracy_at_threshold(...) and coverage(...) summarize correctness and answer rate.
penalty_adjusted_mean(...) aggregates the penalty rule across the dataset.
overconfidence_rate(...) isolates confidently wrong outputs.
Together, these functions form the evaluation core used everywhere in the project.

## Usage
Two evaluation notebooks exist, one for GPQA and one for MMLU. They differ only in dataset paths and output filenames. All metric computation is sharedand each notebook:
Loads the model’s output CSV for that dataset (for all four thresholds).
Applies the evaluation functions at each threshold.
Summarizes performance across the full range of t.
Identifies the best threshold for that model–dataset pair.
Compares that best-threshold performance against:
Binary grading (standard benchmark scoring)
Always-abstain (zero-risk lower bound)
This comparison allows us to see how much reliability is lost or gained once confident mistakes are penalized, and how binary accuracy can misrepresent calibration quality.