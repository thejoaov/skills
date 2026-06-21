---
name: llm-ml-analysis
description: Use when analyzing the results of a machine learning classification experiment and generating a comparative report. Covers constructing a structured prompt from holdout and cross-validation metrics across multiple models, calling Gemini or Ollama, and saving the output as a Markdown report. Trigger when the user asks how to summarize ML experiment results, generate an analysis report from model metrics, or compare classifiers automatically.
---

# LLM-Generated ML Analysis Report

Pattern for using an LLM to generate a comparative analysis report from machine learning classification results. The LLM receives structured metrics and produces a Markdown report with tables, interpretation, and recommendations — not code, just the analytical text.

## When to use

- After training multiple classifiers and wanting a first-pass comparative analysis
- Generating a results section draft for an academic paper or internal report
- Comparing holdout metrics vs cross-validation metrics to detect overfitting

## Pattern

### 1. Collect metrics from each model

After training, gather for each model:
- Hyperparameters selected (from GridSearchCV or equivalent)
- Holdout accuracy, F1-score (weighted), precision, recall
- Cross-validation (k=10) mean ± std for the same metrics
- Top-N feature importances
- Full classification report (per-class precision/recall/F1)

### 2. Build the prompt

```python
def build_analysis_prompt(models: list[dict], dataset_info: dict) -> str:
    """
    models: list of dicts with keys:
        name, params, holdout (accuracy/f1/precision/recall/report), 
        cv (accuracy/f1/precision/recall as {mean, std}), feature_importance
    dataset_info: {n_samples, split, balancing, cv_strategy}
    """
    sections = []
    for m in models:
        cv = m["cv"]
        sections.append(f"""
--- {m['name'].upper()} ---
Params: {m['params']}
Holdout  Acc={m['holdout']['accuracy']:.4f}  F1={m['holdout']['f1']:.4f}  P={m['holdout']['precision']:.4f}  R={m['holdout']['recall']:.4f}
CV k=10  Acc={cv['accuracy']['mean']:.4f}±{cv['accuracy']['std']:.4f}  F1={cv['f1']['mean']:.4f}±{cv['f1']['std']:.4f}

Top 5 features:
{m['feature_importance'].head(5).to_string(index=False)}

Classification report:
{m['holdout']['report']}
""")

    return f"""You are a machine learning expert analyzing a comparative study.

Dataset: {dataset_info['n_samples']} samples
Split: {dataset_info['split']}
Balancing: {dataset_info['balancing']}
CV strategy: {dataset_info['cv_strategy']}

=== RESULTS ===
{''.join(sections)}

=== TASK ===
Write a report in Markdown with:
1. A comparison table (holdout and CV metrics for all models)
2. Which model performed best and why
3. Overfitting analysis (holdout vs CV gap)
4. Feature importance analysis (similarities and differences across models)
5. Recommendations for improving generalization
6. A brief conclusion

Output plain Markdown only. Do not wrap in code fences. Start directly with the content."""
```

### 3. Call the LLM

```python
import json
import urllib.request
import os

def call_gemini(prompt: str, model: str = "gemini-2.0-flash-lite") -> str:
    url = f"https://generativelanguage.googleapis.com/v1beta/models/{model}:generateContent?key={os.environ['GEMINI_API_KEY']}"
    payload = {
        "contents": [{"parts": [{"text": prompt}]}],
        "generationConfig": {"temperature": 0.7, "maxOutputTokens": 4096},
    }
    req = urllib.request.Request(
        url,
        data=json.dumps(payload).encode(),
        headers={"Content-Type": "application/json"},
    )
    with urllib.request.urlopen(req) as r:
        result = json.loads(r.read())
    return result["candidates"][0]["content"]["parts"][0]["text"]

def call_ollama(prompt: str, model: str = "llama3.1", base_url: str = "http://localhost:11434") -> str:
    payload = {"model": model, "prompt": prompt, "stream": False,
               "options": {"temperature": 0.7, "num_predict": 4096}}
    req = urllib.request.Request(
        f"{base_url}/api/generate",
        data=json.dumps(payload).encode(),
        headers={"Content-Type": "application/json"},
    )
    with urllib.request.urlopen(req) as r:
        return json.loads(r.read())["response"]
```

### 4. Save as timestamped Markdown

```python
from datetime import datetime

def save_report(llm_response: str, models: list[dict], data_file: str, env_vars: str) -> str:
    timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
    
    # Build a summary table as a header before the LLM analysis
    rows = [f"| {m['name']} | {m['holdout']['accuracy']*100:.2f}% | {m['holdout']['f1']*100:.2f}% | "
            f"{m['cv']['accuracy']['mean']*100:.2f}%±{m['cv']['accuracy']['std']*100:.2f}% |"
            for m in models]
    table = (
        "| Model | Accuracy (holdout) | F1 (holdout) | Accuracy (CV k=10) |\n"
        "|---|---|---|---|\n" +
        "\n".join(rows)
    )

    content = f"""# ML Experiment Report

**File:** `{data_file}`  
**Run:** {timestamp}

## Summary

{table}

---

{llm_response}
"""
    output_path = f"output/result-{timestamp}.md"
    os.makedirs("output", exist_ok=True)
    with open(output_path, "w", encoding="utf-8") as f:
        f.write(content)
    return output_path
```

## Environment config

```ini
# .env
GEMINI_API_KEY=your-key-here
GEMINI_MODEL=gemini-2.0-flash-lite

# To use local Ollama instead:
OLLAMA_ENABLE=true
OLLAMA_MODEL=llama3.1
OLLAMA_BASE_URL=http://localhost:11434
```

## Multi-scenario runs

Run the same pipeline under different population configurations using scenario `.env` files:

```bash
python run.py --env .env             # default scenario
python run.py --env .env.high-perf   # high-performing cohort
python run.py --env .env.low-income  # lower socioeconomic scenario
```

Each run produces a timestamped output file, preserving all results for comparison.

## Notes

- Use `gemini-2.0-flash-lite` for cost efficiency — the analysis task is well within its capabilities.
- The LLM analysis is a first draft, not a final result. Edit the generated report before including it in a paper or presentation.
- Always include the raw metrics table (generated in Python, not by the LLM) at the top of the report. This ensures the numbers are exact, even if the LLM's prose interpretation is approximate.
- Asking the LLM to output plain Markdown (not wrapped in fences) requires explicit instruction and defensive stripping — models occasionally wrap the entire response despite being told not to.
