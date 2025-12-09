## Confidence Aware Inference
This folder contains the complete pipeline for running confidence based inference across multiple language models on the GPQA and MMLU datasets. The goal is to study how different models behave when they are required to answer only above a specified confidence threshold. All models are evaluated using the same prompt structure, the same data format, and the same extraction logic to ensure comparability.

### Dataset Preparation
GPQA is downloaded from the Hugging Face hub and converted into a uniform structure with the columns id, question, choices, and answer. The answer choices are shuffled and the correct answer is reassigned based on the new order. The dataset is then split into a calibration set and a larger evaluation set.
MMLU follows the same structure and is already stored in processed form. Both datasets are saved as CSV files to be used during inference.

### Prompt Format
Each question is converted into a multiple choice prompt. For a threshold t the model is instructed to answer only if it is more than t confident. Incorrect answers incur t divided by one minus t points. Correct answers receive one point. An abstention receives zero. Models are told to respond with a single capital letter for the selected option or to say I do not know if unsure.

### Extracting Predictions
After the model responds, we extract only the final letter or the exact phrase I do not know. For locally loaded models such as Qwen and Llama we generate several samples and take a majority vote. The confidence is computed as the fraction of votes supporting the winning answer. API based models return a single response which is parsed using the same extraction code.

### Inference
Inference is run at four thresholds: 0.25, 0.50, 0.75, and 0.90. For each threshold the model answers every question in the evaluation split. The output includes the id, question, choices, ground truth answer, predicted answer, threshold, and confidence. Results are written to a CSV file for later scoring and calibration analysis.

### Models
Some models are loaded locally. Qwen and Llama are downloaded and run on device using PyTorch and the Transformers library. Other models are accessed through provider APIs. GPT, Gemini, Claude, and Mistral are queried using the same prompt and are processed using the same extraction rules.

### Output
Each model produces a CSV file containing predictions for all thresholds. These files are used by the analysis scripts to compute accuracy, coverage, penalty adjusted scores, and calibration results.


Instructions on how to set up the environment: 

1. conda create -n llm-eval python=3.10 -y
2. conda activate llm-eval
3. pip install -r requirements.txt

After installation, verify everything works:
- python -c "import torch, transformers, datasets; print('Environment ready')"