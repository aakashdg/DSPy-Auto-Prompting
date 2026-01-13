# Auto-Prompting with DSPy: Executive Summary

**Project:** Farmer.CHAT Stitching Prompt Optimization

**Date:** January 2026

## 🎯 Project Overview

We aimed to automatically optimize the prompt that converts structured agricultural facts into conversational responses for farmers in Bihar. Instead of manual prompt engineering, we utilized **DSPy**—a framework that programmatically optimizes prompts using metrics and examples.

### The Pipeline

`Farmer Query` + `Agricultural Facts` → **[STITCHING PROMPT]** → `Conversational Response`

> **Goal:** Optimize the [STITCHING PROMPT] automatically.

---

## ⚡ DSPy in 30 Seconds

| Component | What It Does |
| --- | --- |
| **Signature** | Defines prompt structure (instructions + input/output fields) |
| **Module** | Wraps signature with reasoning strategy (e.g., Chain of Thought) |
| **Optimizer** | Automatically finds better prompts/demos based on a metric |

### Key Optimizers Used:

* **LabeledFewShot:** Uses golden answers directly as few-shot examples.
* **BootstrapFewShot:** Generates examples and keeps those that pass the metric.
* **MIPRO:** Searches over both instructions and examples simultaneously.

---

## 📈 Iterations & Results

| Iteration | Approach | Score (out of 5.0) | Outcome |
| --- | --- | --- | --- |
| **0. Baseline** | Hand-written prompt | 4.07 - 4.40 | Strong starting point |
| **1. Bootstrap** | Auto-generate demos | 4.17 → 4.23 | ❌ Worse than base |
| **2. Labeled** | 5 golden answers as demos | 4.04 | ⚠️ Marginal gain |
| **3. Lang Check** | Identify 30% language mismatch | — | 🔍 Root cause found |
| **4. Patterns** | Rule extraction + Synthetic demos | — | ✅ New approach |
| **5. Merged** | Original + Patterns + Synthetic | **4.40** | ✅ Best performance |

---

## 💡 Key Insights

1. **Strong Base = Limited Gains:** Our base prompt already scored 83% (4.17/5.0). DSPy works best when there is significant room for improvement.
2. **Demos Override Instructions:** Even if instructions say "respond in English," Hindi demos caused Hindi outputs. **Demo language distribution matters more than instructions.**
3. **History Contamination:** DSPy's internal structure can pollute current context with past examples, shifting bias toward specific languages (e.g., Hindi) regardless of the query.
4. **Metric is King:** Our metric didn't penalize language mismatch initially. If a requirement isn't in the metric, DSPy won't optimize for it.

---

## 🔍 Critical Discovery: The "Hidden Candidate" Problem

We found that major DSPy optimizers often successfully optimize prompts internally but do not return the result in the expected `.compile()` output.

```python
# ❌ The obvious way often looks empty
optimized_model = optimizer.compile(student, trainset)
# optimized_model.generate_response.demos = [] (Empty!)

# ✅ The REAL optimized model is hidden here:
optimized_model.candidate_programs[1]['program'] # Contains demos + instructions!

```

### Optimizer Behavior Matrix

| Optimizer | Result Object | Where the Optimization Lives |
| --- | --- | --- |
| **BootstrapFewShot** | Original module | Works correctly (attaches demos) |
| **RandomSearch** | Student with metadata | `candidate_programs` list |
| **MIPRO** | Student with metadata | `candidate_programs` list |

---

## 🛠️ Technical Lessons Learned

### 1. ChainOfThought vs. Predict

* **Problem:** Optimizers attach demos to `dspy.Predict`. `dspy.ChainOfThought` wraps Predict, often creating a layer where demos fail to attach properly.
* **Solution:** Use `for_optimization=True` flag to switch to `Predict` during optimization and back to `CoT` for inference.

### 2. Validation Set Size

* **Insight:** Tiny sets (e.g., 2 examples) create "false perfects." MIPRO might select a baseline with 0 demos because it scored 100% on a very easy test.
* **Fix:** Use **≥5-10 validation examples** with varying difficulty.

### 3. Metric Design

* **Avoid Boolean Metrics:** Don't just return `True/False` or `Score >= 4`.
* **Use Continuous Scores:** Return 0.0–1.0 so the optimizer can distinguish between a "good" 0.8 and a "great" 0.95.

---

## 🚀 When to Use DSPy Auto-Prompting

| ✅ USE DSPY | ❌ DON'T BOTHER |
| --- | --- |
| Weak base prompt (<70%) | Strong base prompt (>80%) |
| Continuous metric (0.0-1.0) | Binary metric (pass/fail) |
| Large validation set (10+) | Tiny validation (2-3 examples) |
| Using `dspy.Predict` | Requires `ChainOfThought` |
| Have time for candidate extraction | Need "plug-and-play" solution |

---

## 📋 Recommendation & Template

**Start with BootstrapFewShot** for reliability. Only move to **MIPRO** if using `Predict` and you have a robust extraction script for the `candidate_programs`.

### Final Working Pattern

```python
# 1. Initialize for optimization
student = FarmerChatStitcher(use_cot=False, for_optimization=True)

# 2. Extract best candidate manually
if hasattr(result, 'candidate_programs'):
    for cand in result.candidate_programs:
        if len(cand['program'].generate_response.demos) > 0:
            optimized_model = cand['program']
            break

```

---

**Detailed logs and implementation:** See `auto_prompting_documentation.md`