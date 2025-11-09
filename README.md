Instructions on how to set up the environment: 

1. conda create -n llm-eval python=3.10 -y
2. conda activate llm-eval
3. pip install -r requirements.txt

After installation, verify everything works:
- python -c "import torch, transformers, datasets; print('✅ Environment ready')"
