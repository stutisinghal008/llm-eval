## Re-Scoring Language Model Benchmarks with Explicit Confidence Targets
Veer Kejriwal · Stuti Singhal · Raima Nawal · Shubhangi Asthana
Georgia Institute of Technology

### Abstract
Large language models (LLMs) often produce confident but incorrect answers, yet mainstream benchmarks such as MMLU and GPQA score models using binary accuracy, treating confident mistakes and abstentions identically. This masks calibration failures, rewards guessing, and distorts model comparisons. We introduce our own confidence-aware scoring framework that applies directly to existing benchmarks without modifying model weights or requiring new datasets. Models are prompted to answer only when their confidence exceeds a threshold t, and incorrect answers above this threshold incur a penalty proportional to both t and the model’s estimated confidence. Using self-consistency vote fractions as a model-agnostic confidence signal, we evaluate six models across multiple thresholds. Our results show that thresholding does not make models more selective; overconfidence remains widespread, especially on GPQA, and penalty-adjusted scores sharply separate calibrated models from those that answer aggressively. The framework exposes reliability gaps that binary accuracy conceals and provides a more informative basis for assessing trustworthiness in reasoning tasks.

### Overview
This repository contains our full pipeline for evaluating large language models under confidence-aware scoring. Instead of relying on binary accuracy which gives equal credit to confident mistakes and genuine uncertainty, we evaluate models only when they self-report confidence above a threshold t, and penalize them proportionally when confidently wrong. This approach reveals reliability failures, calibration gaps, and misleading leaderboard rankings that binary grading masks.

### Motivation
Modern LLMs still hallucinate, especially under pressure to answer everything. Standard benchmarks such as MMLU and GPQA grade each question as correct/incorrect with no penalty, which rewards guessing and inflates performance. Our goal is to introduce a lightweight, dataset agnostic scoring rule that rewards calibrated abstention and exposes overconfident errors directly on existing benchmarks.

### Methodology

  <img src="assets/fig_1.png" width="300" alt="Confidence-Aware Pipeline">

1. Confidence-Aware Prompting. 
  Each model receives the instruction:
  “Answer only if you are more than t confident. Mistakes incur t/(1−t) points; correct = +1; IDK = 0.”
  This enforces calibrated answering without modifying model weights.
2. Model-Agnostic Confidence via Self-Consistency
  For each question, we run multiple independent generations (typically 6–20) and compute confidence as the vote fraction of the majority answer.
  This works uniformly for API models and open-weight models
3. Thresholded Answering
  A model answers only when p > t. Otherwise it abstains.
  We evaluate at four thresholds:
  t ∈ {0.25, 0.50, 0.75, 0.90}
4. Confidence-Proportional Scoring
  If p > t, answers are scored as:
  • +1 for correct
  • − p·t / (1−t) for incorrect
  • 0 for abstentions
  This amplifies penalties for unjustified certainty.

### Benchmarks
We evaluate on two major multiple-choice benchmarks:
MMLU – broad 57-domain knowledge benchmark
GPQA – graduate-level reasoning benchmark (40-question evaluation split used here)
To ensure reproducibility, both datasets are converted to a shared template used by every model.

### Models Evaluated
We evaluate six instruction-tuned LLMs across both datasets:
API Models
1. GPT-4o-mini
2. Claude-Haiku
3. Gemini 2.5 Flash-Lite
Open-Weight Models
1. Llama-3.2-3B
2. Mistral-7B-Instruct
3. Qwen2.5-1.5B-Instruct
All models use the same prompt template and confidence estimation pipeline.

### Key Metrics Reported
Each model–threshold pair reports:
1. Accuracy@t – accuracy on answered questions
2. Coverage – fraction of questions answered
3. Penalty-Adjusted Mean Score – overall performance
4. Overconfidence Rate – incorrect answers with p > t
These metrics expose how often models answer with unjustified confidence, not just how often they are correct.

### Key Findings
Our confidence-aware evaluation reveals several patterns that do not appear under standard accuracy grading.


1. Thresholding does not make models more selective

Increasing the confidence threshold t does not meaningfully change accuracy, and most models continue answering even when uncertain. This shows that models rarely recognize uncertainty under self-consistency sampling.

2. Penalty Mean exposes calibration failures that raw accuracy hides

Although accuracy changes only slightly across thresholds, penalty-adjusted scores sharply separate calibrated models from overconfident ones. Several models on GPQA accumulate large negative penalties because they answer confidently and incorrectly.

3. Coverage differences reveal whether a model knows when not to answer

Under confidence-aware scoring, stronger models reduce coverage more selectively. GPT on MMLU answers fewer questions at high t and avoids penalties, while many models on GPQA maintain almost full coverage regardless of difficulty.

4. Calibration depends strongly on the dataset

A model that behaves sensibly on MMLU may still show strong overconfidence on GPQA. This demonstrates that calibration does not transfer consistently across datasets and that reliability must be evaluated task by task.

Overall, these findings show that correctness alone is insufficient for understanding model behavior. Confidence-aware evaluation reveals reliability gaps that remain invisible under standard accuracy metrics.

### Conclusion
This project demonstrates that adding a simple confidence requirement on top of standard benchmarks provides a clearer picture of model reliability. By evaluating both correctness and confidence, we expose behaviors that raw accuracy overlooks, including overconfidence, poor selectivity, and inconsistent calibration across datasets.
Across six models and two benchmarks, we find that higher confidence thresholds do not automatically produce more careful answering. Many models continue to answer aggressively, accumulating large penalties for confidently wrong outputs. At the same time, well-calibrated models benefit from answering selectively, showing improved penalty-adjusted performance even when accuracy remains similar.
Taken together, these results suggest that future evaluations should treat confidence and abstention as essential components of model assessment rather than optional diagnostics. Confidence-aware scoring moves model evaluation closer to the expectations placed on models in real-world decision-making settings, where knowing when not to answer is as important as answering correctly.