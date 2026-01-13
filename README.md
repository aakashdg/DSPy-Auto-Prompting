# DSPy Prompt Optimization Framework

> **Repository:** DSPy-based prompt optimization toolkit for agricultural advisory systems  
> **Organization:** Digital Green - Farmer.CHAT  
> **Last Updated:** January 2026

## Table of Contents

1. [Overview](#overview)
2. [Repository Structure](#repository-structure)
3. [Notebooks](#notebooks)
   - [Template Notebook](#1-template-notebook)
   - [Stitching Optimization](#2-stitching-optimization)
   - [Evaluation Prompt Optimization](#3-evaluation-prompt-optimization)
4. [Installation](#installation)
5. [Quick Start](#quick-start)
6. [Methodology](#methodology)
7. [Technical Background](#technical-background)
8. [Use Cases](#use-cases)
9. [Best Practices](#best-practices)
10. [Contributing](#contributing)
11. [References](#references)

---

## Overview

This repository contains a suite of DSPy-based notebooks for systematic prompt optimization in production language model systems. The framework is designed for practitioners who need to move beyond manual prompt engineering to data-driven, empirically validated prompt improvement.

### Core Objectives

1. **Systematic Optimization**: Replace intuition-based prompt engineering with measurable, reproducible optimization
2. **Production Readiness**: Provide tools that work with real-world constraints (API costs, latency, quality requirements)
3. **Knowledge Transfer**: Enable teams to adopt DSPy optimization without deep framework expertise

### Key Features

- **Plug-and-play template** for rapid DSPy experimentation
- **Domain-specific optimizers** for agricultural advisory systems
- **Meta-optimization techniques** for evaluator prompt improvement
- **Comprehensive extraction utilities** for accessing optimized prompts across different optimizer types

---

## Repository Structure

```
dspy-prompt-optimization/
├── notebooks/
│   ├── dspy_prompt_optimization_template.ipynb    # General-purpose DSPy starter
│   ├── dspy_stitching_optimization_gemma.ipynb    # Fact-to-response stitching
│   └── eval_prompt_optimization.ipynb             # Meta-optimization for evaluators
├── docs/
│   └── auto_prompting_documentation.md            # Theoretical background
├── README.md
└── requirements.txt
```

---

## Notebooks

### 1. Template Notebook
**File:** `dspy_prompt_optimization_template.ipynb`

#### Purpose
A general-purpose starting point for teams new to DSPy or requiring rapid experimentation setup. This notebook abstracts away DSPy complexity while maintaining full framework capabilities.

#### Key Components

**Signature Definition**
```python
class TaskSignature(dspy.Signature):
    """[Your task description]"""
    input_field = dspy.InputField(desc="[Description]")
    output_field = dspy.OutputField(desc="[Description]")
```

**Optimizer Comparison**
- BootstrapFewShot
- BootstrapFewShotWithRandomSearch
- MIPRO (Multi-prompt Instruction Proposal Optimizer)

**Prompt Extraction Utilities**
The notebook includes comprehensive methods for extracting optimized prompts:
- Direct signature access
- Demo extraction
- inspect_history method
- JSON serialization
- candidate_programs extraction (for RandomSearch)

#### Use Cases
- Quick DSPy prototyping
- Testing different optimizers on new tasks
- Educational purposes for team onboarding
- Baseline establishment before custom optimization

#### Technical Highlights
- Multi-method prompt extraction with fallbacks
- Optimizer comparison matrix
- Success rate tracking across extraction methods
- Compatible with both gpt-4o and gemma-3n models

---

### 2. Stitching Optimization
**File:** `dspy_stitching_optimization_gemma.ipynb`

#### Purpose
Optimize the conversion of structured agricultural facts (JSON format) into natural, conversational responses appropriate for smallholder farmers in Bihar, India.

#### Problem Context

**Input Structure:**
```json
{
  "farmer_question": "गेहूं की बुवाई कब करें?",
  "agricultural_facts": [
    {"category": "timing", "content": "November-December optimal", "confidence": 0.9},
    {"category": "variety", "content": "HD 2967 recommended for Bihar", "confidence": 0.85}
  ],
  "crop_context": "Crop: Wheat, Region: Bihar"
}
```

**Expected Output:**
Natural Hindi/English/Hinglish response that is:
- Culturally appropriate for Bihar farmers
- Actionable with low-cost solutions
- 150-300 words
- Warm and supportive tone

#### Optimization Strategy

**Phase 1: Baseline Establishment**
- Manual prompt engineering baseline
- 6-dimensional evaluation framework
- Human-validated golden answers (RLHF dataset)

**Phase 2: DSPy Optimization Attempts**
- MIPRO: Limited improvement due to language mismatch issues
- BootstrapFewShot: Demo selection problems
- RandomSearch: Instruction variants tested

**Phase 3: Hybrid Approach**
- Pattern extraction from golden answers
- Manual demo synthesis based on patterns
- Explicit language matching rules
- Combined with DSPy structure for maintainability

#### Evaluation Metrics

| Dimension | Weight | Criteria |
|-----------|--------|----------|
| Content Quality | 30% | Specific, actionable, region-appropriate advice |
| Communication Style | 25% | Warm, simple, culturally resonant |
| Practical Advice | 25% | Low-cost, accessible, constraint-aware |
| Safety & Credibility | 10% | Safe chemical advice, expert references |
| Conversation Flow | 5% | Context-aware, non-repetitive |
| Response Format | 5% | Natural structure, appropriate length |

**Target:** Overall score ≥ 4.0/5.0

#### Key Findings

1. **Language matching is critical**: Standard DSPy optimizers struggled with multilingual requirements
2. **Manual demos outperformed bootstrap**: Golden answer patterns were more effective than model-generated examples
3. **Hybrid approach optimal**: DSPy structure + manual pattern injection
4. **Cultural context cannot be auto-discovered**: Domain expertise required for demo creation

#### Model Configuration
- Generation: Gemma-3n-E4B-it (via Together AI)
- Evaluation: GPT-4o (via OpenAI)
- Separation rationale: Cost optimization and quality requirements

---

### 3. Evaluation Prompt Optimization
**File:** `eval_prompt_optimization.ipynb`

#### Purpose
Meta-optimization of evaluation prompts themselves - improving the prompts that score agricultural advisory responses.

#### The Meta-Challenge

**Standard prompt optimization:**
```
Input → [LLM with prompt] → Output
Goal: Optimize prompt for better outputs
```

**Evaluation prompt optimization:**
```
(Question, Response) → [Evaluator LLM] → Scores
Goal: Optimize evaluator for more accurate scoring
```

#### Methodology

**Phase 1: Baseline Evaluation**
- Run current evaluation prompt on development set
- Calculate agreement metrics with ground truth
- Identify systematic scoring failures

**Phase 2: Failure Analysis**
- Categorize failures (golden underscored, low-quality overscored, parse errors)
- Use LLM to identify failure patterns
- Extract root causes of mis-scoring

**Phase 3: Pattern-Based Improvement**
- Generate improved evaluation criteria based on failure patterns
- Avoid prescriptive fixes (e.g., "score higher")
- Focus on making criteria more specific and measurable

**Phase 4: Empirical Validation**
- Test improved prompt on held-out test set
- Measure discrimination (good vs. bad response separation)
- Check for leniency bias (uniform score inflation)

#### Key Metrics

**Agreement Metrics:**
- Golden mean score: Average score for human-validated 5/5 responses (target: 4.5-5.0)
- Score separation: Difference between golden and low-quality means (target: ≥1.0)
- Calibration: Percentage of golden answers scored ≥4.5 (target: ≥90%)
- Parse success rate: JSON parsing reliability

**Discrimination Test:**
```
Good response score - Bad response score = Separation
```
Higher separation indicates better discrimination of quality levels.

#### Why Not Standard DSPy Optimizers?

Standard optimizers (MIPRO, BootstrapFewShot) are designed for generation tasks. For evaluation optimization:

1. **Different objective**: Not improving outputs, but scoring accuracy
2. **Ground truth exists**: Explicit correct/incorrect scores
3. **Need interpretability**: Must understand why changes help
4. **Avoid leniency bias**: Cannot simply inflate all scores

**Solution:** LLM-as-optimizer approach
- LLM analyzes failures
- Proposes specific fixes with reasoning
- Empirical validation against ground truth
- Iterative refinement based on metrics

#### Technical Implementation

**Failure Analysis Signature:**
```python
class AnalyzeEvaluationFailures(dspy.Signature):
    """Analyze where the current evaluation prompt fails."""
    current_prompt = dspy.InputField(desc="Current evaluation prompt")
    failure_examples = dspy.InputField(desc="Examples with wrong scores")
    
    failure_patterns = dspy.OutputField(desc="Systematic failure patterns")
    root_causes = dspy.OutputField(desc="Why evaluator misses cases")
    priority_fixes = dspy.OutputField(desc="Highest-priority improvements")
```

**Improvement Generation:**
```python
class GenerateImprovedEvaluationCriteria(dspy.Signature):
    """Generate improved evaluation guidelines based on failure analysis."""
    current_prompt = dspy.InputField(desc="Current complete evaluation prompt")
    failure_analysis = dspy.InputField(desc="Systematic scoring failures")
    baseline_metrics = dspy.InputField(desc="Current performance metrics")
    
    improved_guidelines = dspy.OutputField(desc="Enhanced evaluation criteria")
    change_reasoning = dspy.OutputField(desc="Why each change improves accuracy")
    expected_improvements = dspy.OutputField(desc="Predicted metric changes")
```

#### Critical Insights

1. **Avoid prescriptive instructions**: Don't tell LLM "add calibration" - let it discover patterns
2. **Provide concrete failure examples**: Show actual cases where evaluator failed
3. **Test for leniency bias**: Ensure improvements don't just inflate scores uniformly
4. **Preserve structure**: Keep critical sections (JSON format, dimensions) intact
5. **Iterative refinement**: Multiple rounds may be needed for complex evaluators

#### Expected Improvements

Based on empirical testing:
- Golden mean score: 3.8-4.2 → 4.5-4.8
- Score separation: 0.3-0.5 → 0.8-1.2
- Calibration consistency: Lower standard deviation
- Parse success: Maintained or improved

A lot of observed variance based on model.
---

## Installation

### Prerequisites
- Python 3.10+
- Google Colab account (recommended) or local Jupyter environment

### Required Packages

```bash
pip install dspy-ai==3.1.0
pip install openai
pip install together
pip install openpyxl
pip install scikit-learn
pip install pandas
pip install numpy
```

### API Keys Required

**OpenAI:**
```python
import os
os.environ['OPENAI_API_KEY'] = 'your-key-here'
```

**Together AI (for Gemma):**
```python
os.environ['TOGETHER_API_KEY'] = 'your-key-here'
```

For Google Colab, use `userdata`:
```python
from google.colab import userdata
OPENAI_API_KEY = userdata.get('OPENAI_API_KEY')
TOGETHER_API_KEY = userdata.get('TOGETHER_API_KEY')
```

---

## Quick Start

### 1. Template Notebook

```python
# Define your task signature
class YourTask(dspy.Signature):
    """Describe your task here"""
    input_text = dspy.InputField(desc="Your input")
    output_text = dspy.OutputField(desc="Your output")

# Create training examples
trainset = [
    dspy.Example(input_text="example 1", output_text="result 1").with_inputs('input_text'),
    dspy.Example(input_text="example 2", output_text="result 2").with_inputs('input_text'),
]

# Choose optimizer and optimize
optimizer = dspy.BootstrapFewShot(metric=your_metric_function)
optimized_program = optimizer.compile(
    student=dspy.ChainOfThought(YourTask),
    trainset=trainset
)

# Extract optimized prompt
inspect_optimized_program(optimized_program, "Your Task")
```

### 2. Stitching Optimization

```python
# Load RLHF dataset
df = pd.read_excel("RLHF_SFT_DATA.xlsx")

# Prepare examples
trainset = prepare_training_examples(df, sample_size=100)

# Run optimization
optimizer = dspy.MIPRO(metric=conversationality_metric)
optimized_stitcher = optimizer.compile(
    student=FactStitcher(),
    trainset=trainset,
    num_trials=10
)

# Evaluate
results = evaluate_on_test_set(optimized_stitcher, test_examples)
```

### 3. Evaluation Optimization

```python
# Create evaluation examples with ground truth
eval_examples = [
    EvaluationExample(
        question="...",
        response="...",
        is_golden=True,
        expected_score_range=(4.5, 5.0)
    ),
    # ... more examples
]

# Run baseline evaluation
baseline_results = run_baseline_evaluation(eval_examples)

# Analyze failures
failures = identify_failures(baseline_results, threshold=4.5)

# Generate improvements
improvement_result = generate_improvements(
    current_prompt=BASELINE_EVAL_PROMPT,
    failure_analysis=analyze_patterns(failures)
)

# Test improved prompt
improved_results = test_improved_prompt(improvement_result, test_examples)
```

---

## Methodology

### DSPy Optimization Pipeline

```
┌─────────────────────────────────────────────────────────────────────┐
│                    DSPy OPTIMIZATION WORKFLOW                       │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  1. Define Signature                                                │
│     ├─ Input/output fields                                          │
│     ├─ Field descriptions                                           │
│     └─ Task instructions (docstring)                                │
│                           │                                         │
│                           ▼                                         │
│  2. Prepare Training Data                                           │
│     ├─ Golden examples (human-validated)                            │
│     ├─ Synthetic examples (model-generated)                         │
│     └─ Stratified sampling (for diversity)                          │ 
│                           │                                         │
│                           ▼                                         │
│  3. Define Metric Function                                          │
│     ├─ Binary (pass/fail) for DSPy optimizers                       │
│     ├─ Continuous (0-1) for analysis                                │
│     └─ Multi-dimensional (for comprehensive eval)                   │
│                           │                                         │
│                           ▼                                         │
│  4. Select Optimizer                                                │
│     ├─ BootstrapFewShot (demo selection)                            │
│     ├─ MIPRO (instruction + demo)                                   │
│     └─ RandomSearch (instruction variants)                          │
│                           │                                         │
│                           ▼                                         │
│  5. Compile & Optimize                                              │
│     ├─ Generate candidates                                          │
│     ├─ Evaluate on validation set                                   │
│     └─ Select best configuration                                    │
│                           │                                         │
│                           ▼                                         │
│  6. Extract & Validate                                              │
│     ├─ Extract optimized prompt                                     │
│     ├─ Test on held-out set                                         │
│     └─ Compare with baseline                                        │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Evaluation Framework

**Six-Dimensional Assessment:**

Each response is evaluated on:
1. Content Quality (specificity, actionability, regional relevance)
2. Communication Style (tone, simplicity, cultural appropriateness)
3. Practical Advice (cost-effectiveness, accessibility)
4. Safety & Credibility (chemical safety, expert references)
5. Conversation Flow (context awareness, coherence)
6. Response Format (structure, length, naturalness)

**Scoring Scale:**
- 5 (Excellent): Fully meets all criteria
- 4 (Good): Meets most criteria with minor gaps
- 3 (Adequate): Meets some criteria but notable deficiencies
- 2 (Poor): Significant gaps in multiple areas
- 1 (Very Poor): Fails to meet basic criteria

**Aggregation:**
```
Overall Score = Σ(dimension_score × weight) / Σ(weights)
```

---

## Technical Background

### Why DSPy?

Traditional prompt engineering faces several challenges:
1. **Manual iteration is time-intensive**: Each prompt variant requires testing
2. **Non-systematic**: Relies on intuition rather than data
3. **Hard to reproduce**: Results depend on specific engineer's expertise
4. **Doesn't scale**: Managing prompts across models/tasks becomes unwieldy

DSPy addresses these by:
- Treating prompts as learnable parameters
- Providing systematic optimization algorithms
- Enabling reproducible improvement through metrics
- Separating task logic from prompt text

### Optimizer Comparison

| Optimizer | Optimizes | Best For | Limitations |
|-----------|-----------|----------|-------------|
| **BootstrapFewShot** | Demo selection | Tasks with good baseline, need better examples | Requires quality bootstrap examples |
| **BootstrapRandomSearch** | Instructions + demos | Finding novel instruction variants | Computationally expensive |
| **MIPRO** | Instructions + demos | Complex tasks, when baseline is weak | Requires many trials, high API cost |
| **LabeledFewShot** | Demo selection | When you have labeled examples | No instruction optimization |

### When to Use Each Notebook

**Template Notebook:**
- New to DSPy
- Need quick experimentation setup
- Testing optimizer suitability for task
- Educational/onboarding purposes

**Stitching Optimization:**
- Structured-to-natural language conversion
- Multilingual requirements
- Cultural/domain-specific constraints
- Need interpretable few-shot examples

**Evaluation Optimization:**
- Improving scoring accuracy
- Calibrating existing evaluators
- Reducing leniency/severity bias
- Need explainable evaluation criteria

---

## Use Cases

### 1. Customer Support Automation

**Problem:** Convert structured knowledge base into conversational responses

**Approach:**
- Use stitching optimization patterns
- Replace agricultural facts with KB articles
- Adapt evaluation dimensions to support quality

**Expected Outcome:**
- More natural response phrasing
- Better query-response matching
- Consistent brand voice

### 2. Educational Content Generation

**Problem:** Transform curriculum outlines into engaging lesson content

**Approach:**
- Template notebook for baseline
- Custom metric for educational quality
- Iterative refinement with teacher feedback

**Expected Outcome:**
- Age-appropriate language
- Engaging explanations
- Curriculum-aligned content

### 3. Code Documentation

**Problem:** Generate clear documentation from code comments/signatures

**Approach:**
- Signature: code → documentation
- Metric: clarity, completeness, accuracy
- Bootstrap from high-quality examples

**Expected Outcome:**
- Consistent documentation style
- Comprehensive coverage
- Reduced manual writing effort

---

## Best Practices

### Data Preparation

1. **Quality over quantity**: 20-50 high-quality examples often outperform 500 mediocre ones
2. **Stratified sampling**: Ensure diversity across key dimensions (language, topic, complexity)
3. **Validation split**: Always hold out 20-30% for final testing
4. **Label carefully**: For evaluation tasks, ensure ground truth is accurate

### Metric Design

1. **Start simple**: Binary metric first, then add nuance
2. **Fast execution**: Metrics are called repeatedly during optimization
3. **Interpretable**: Should be obvious why a response passes/fails
4. **Aligned with production**: Test metric on actual use cases

### Optimization Strategy

1. **Baseline first**: Establish manual prompt performance before optimization
2. **One variable at a time**: Test instruction-only, then demos, then both
3. **Monitor overfitting**: Check test set performance regularly
4. **Cost management**: Set trial limits, use smaller models for prototyping

### Prompt Extraction

1. **Use multiple methods**: Different optimizers expose prompts differently
2. **Verify completeness**: Check for instructions, demos, and special tokens
3. **Test extracted prompt**: Ensure it produces same results as compiled program
4. **Version control**: Save both compiled programs and extracted text

### Production Deployment

1. **A/B test**: Compare optimized vs. baseline in production
2. **Monitor metrics**: Track the same metrics used in optimization
3. **Gradual rollout**: Start with small percentage of traffic
4. **Regression testing**: Ensure new prompt doesn't break edge cases

---

## Contributing

### Code Style

- Follow PEP 8 for Python code
- Use descriptive variable names
- Add docstrings to functions and classes
- Include type hints where beneficial

### Notebook Structure

Standard sections for all notebooks:
1. Setup and installation
2. Configuration
3. Data loading
4. Baseline establishment
5. Optimization
6. Evaluation
7. Results extraction
8. Comparison and analysis

### Testing

Before submitting:
- Run full notebook from top to bottom
- Verify all cells execute without errors
- Check that results are reproducible
- Update documentation if API changes

### Documentation

- Update README for new notebooks or major changes
- Add inline comments for complex logic
- Provide usage examples for new utilities
- Document any dependencies or external requirements

---

## References

### DSPy Resources

- [DSPy GitHub Repository](https://github.com/stanfordnlp/dspy)
- [DSPy Documentation](https://dspy-docs.vercel.app/)
- [DSPy Paper](https://arxiv.org/abs/2310.03714): "Compiling Declarative Language Model Calls into Self-Improving Pipelines"

### Related Research

- [Chain-of-Thought Prompting](https://arxiv.org/abs/2201.11903)
- [Automatic Prompt Engineer (APE)](https://arxiv.org/abs/2211.01910)
- [OPRO: Optimization by PROmpting](https://arxiv.org/abs/2309.03409)
- [Self-Consistency Improves CoT](https://arxiv.org/abs/2203.11171)

### Project Context

- **Organization**: Digital Green
- **Project**: Farmer.CHAT Agricultural Advisory System
- **Region**: Bihar, India
- **Languages**: English, Hindi, Hinglish
- **Target Users**: Smallholder farmers

---

## License

This repository is maintained by Digital Green for internal use and external collaboration on agricultural advisory systems.

---