# Auto-Prompting: Theory, Experiments & Iterations

> **Project:** Farmer.CHAT Stitching Prompt Optimization  
> **Goal:** Optimize prompts that convert structured agricultural facts into natural, conversational responses for farmers in Bihar, India  
> **Last Updated:** January 2025

---

## Table of Contents

1. [Introduction & Problem Statement](#1-introduction--problem-statement)
2. [Theoretical Background](#2-theoretical-background)
3. [DSPy Framework Overview](#3-dspy-framework-overview)
4. [Experiment Timeline & Iterations](#4-experiment-timeline--iterations)
5. [Optimizer Techniques Deep Dive](#5-optimizer-techniques-deep-dive)
6. [Key Findings & Lessons Learned](#6-key-findings--lessons-learned)
7. [Code Artifacts](#7-code-artifacts)
8. [Future Directions](#8-future-directions)
9. [References & Resources](#9-references--resources)

---

## 1. Introduction & Problem Statement

### 1.1 Context

Farmer.CHAT is an agricultural advisory system designed to help smallholder farmers in Bihar, India. The system uses a **stitching prompt** to convert structured agricultural facts (from various data sources) into natural, conversational responses.

### 1.2 The Stitching Pipeline

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         FARMER.CHAT PIPELINE                                 │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  Farmer Query ──→ Query Router ──→ MCP Executor ──→ Response Compiler       │
│       │                                │                    │                │
│       │                                ▼                    ▼                │
│       │                         [Parallel API Calls]   [Raw Facts]          │
│       │                         - Weather API                │               │
│       │                         - Soil API                   │               │
│       │                         - Pest API                   ▼               │
│       │                         - Crop Intel         ┌──────────────┐       │
│       │                                              │  STITCHING   │       │
│       └─────────────────────────────────────────────→│    PROMPT    │       │
│                                                      │ (This is what│       │
│                                                      │ we optimize) │       │
│                                                      └──────┬───────┘       │
│                                                             │               │
│                                                             ▼               │
│                                                    Conversational Response   │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 1.3 Problem Statement

**Challenge:** The stitching prompt needs to:
- Convert structured JSON facts into natural language
- Match the query language (English/Hindi/Hinglish)
- Be warm, supportive, and culturally appropriate
- Provide actionable, low-cost advice
- Reference local Bihar context

**Question:** Can we use auto-prompting techniques (DSPy) to systematically improve this prompt beyond manual engineering?

### 1.4 Success Metrics

We evaluate responses on 6 dimensions (1-5 scale):

| Dimension | Weight | Description |
|-----------|--------|-------------|
| Content Quality | 30% | Specific, actionable, region-appropriate |
| Communication Style | 25% | Warm, simple, culturally appropriate |
| Practical Advice | 25% | Low-cost, accessible, considers constraints |
| Safety & Credibility | 10% | Safe chemical advice, references experts |
| Conversation Flow | 5% | Builds context, offers follow-up |
| Response Format | 5% | Natural conversation, 150-300 words |

**Target:** Overall score ≥ 4.0/5.0 (80%)

---

## 2. Theoretical Background

### 2.1 What is Auto-Prompting?

Auto-prompting refers to techniques that automatically discover, optimize, or refine prompts for language models. Instead of manually iterating on prompt text, we use systematic approaches to find effective prompts.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    PROMPT OPTIMIZATION SPECTRUM                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  Manual ◄─────────────────────────────────────────────────────────► Auto    │
│                                                                              │
│  [Hand-written]  [Template-based]  [Few-shot tuning]  [Full optimization]   │
│       │                │                  │                   │              │
│       ▼                ▼                  ▼                   ▼              │
│   "You are a      Signature +        LabeledFewShot      MIPRO/OPRO         │
│    helpful..."    Input/Output       BootstrapFewShot    Genetic algorithms │
│                   field specs                                                │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 2.2 Why Auto-Prompting?

| Manual Prompt Engineering | Auto-Prompting |
|---------------------------|----------------|
| Relies on intuition | Data-driven optimization |
| Hard to systematically improve | Measurable iterations |
| May miss non-obvious patterns | Can discover surprising strategies |
| Time-intensive iteration | Automated search |
| Works well when expert knows domain | Works well with clear metrics |

### 2.3 Key Concepts

#### 2.3.1 Prompt Components

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         PROMPT ANATOMY                                       │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────┐        │
│  │ SYSTEM INSTRUCTIONS (Signature Docstring)                       │        │
│  │ "You are Farmer.CHAT, a knowledgeable agricultural advisor..." │        │
│  └─────────────────────────────────────────────────────────────────┘        │
│                              │                                               │
│                              ▼                                               │
│  ┌─────────────────────────────────────────────────────────────────┐        │
│  │ FEW-SHOT DEMONSTRATIONS (Demos)                                 │        │
│  │ Example 1: Q: "How to manage rice pests?" → R: "To manage..."  │        │
│  │ Example 2: Q: "गेहूं की सिंचाई..." → R: "गेहूं की सिंचाई के..." │        │
│  └─────────────────────────────────────────────────────────────────┘        │
│                              │                                               │
│                              ▼                                               │
│  ┌─────────────────────────────────────────────────────────────────┐        │
│  │ CURRENT INPUT                                                   │        │
│  │ farmer_question: "What fertilizer for mustard?"                │        │
│  │ agricultural_facts: [{...}, {...}]                             │        │
│  │ crop_context: "Crop: mustard, Bihar"                           │        │
│  └─────────────────────────────────────────────────────────────────┘        │
│                              │                                               │
│                              ▼                                               │
│  ┌─────────────────────────────────────────────────────────────────┐        │
│  │ OUTPUT                                                          │        │
│  │ response: "Mustard ki fasal ke liye..."                        │        │
│  └─────────────────────────────────────────────────────────────────┘        │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

#### 2.3.2 Optimization Targets

What can be optimized?

1. **Instructions** - The system prompt text itself
2. **Demos** - Which examples to include as few-shot
3. **Demo Selection** - Which demos work best for which inputs
4. **Demo Generation** - Creating synthetic demos that teach desired behavior
5. **Reasoning Strategy** - Chain of Thought, Tree of Thought, etc.

#### 2.3.3 Supervised vs. Self-Supervised Optimization

| Approach | Data Required | Method |
|----------|---------------|--------|
| **Supervised** | Golden answers (human-validated) | Learn from correct examples |
| **Self-Supervised** | Metric function only | Generate & filter by metric |
| **Hybrid** | Both | Bootstrap from golden, filter by metric |

---

## 3. DSPy Framework Overview

### 3.1 What is DSPy?

DSPy (Declarative Self-improving Python) is a framework for programming with language models through:
- **Signatures**: Define input/output structure
- **Modules**: Wrap signatures with reasoning strategies
- **Optimizers**: Automatically improve prompts based on metrics

### 3.2 Core Components

#### 3.2.1 Signatures

```python
class FactStitching(dspy.Signature):
    """Synthesize agricultural facts into conversational response."""  # Instructions
    
    farmer_question: str = dspy.InputField(desc="The farmer's question")
    agricultural_facts: str = dspy.InputField(desc="JSON array of facts")
    crop_context: str = dspy.InputField(desc="Crop, season, location")
    
    response: str = dspy.OutputField(desc="Natural conversational response")
```

The docstring becomes the system instructions. Field descriptions guide the model.

#### 3.2.2 Modules

```python
# Basic prediction
predictor = dspy.Predict(FactStitching)

# With Chain of Thought reasoning
cot_predictor = dspy.ChainOfThought(FactStitching)

# Custom module
class FarmerChatStitcher(dspy.Module):
    def __init__(self):
        self.generate_response = dspy.ChainOfThought(FactStitching)
    
    def forward(self, farmer_question, agricultural_facts, crop_context):
        return self.generate_response(
            farmer_question=farmer_question,
            agricultural_facts=agricultural_facts,
            crop_context=crop_context
        )
```

#### 3.2.3 Chain of Thought (CoT)

CoT adds a `reasoning` field before the output, encouraging step-by-step thinking:

```
Input: farmer_question, agricultural_facts, crop_context
    ↓
Reasoning: "Farmer is asking in English about rice pests. Need to synthesize 
           facts about brown planthopper, mention neem oil, keep advice 
           practical and low-cost..."
    ↓
Output: response
```

### 3.3 DSPy Optimizers

#### 3.3.1 Optimizer Comparison Table

| Optimizer | What it Optimizes | Data Required | Best For |
|-----------|-------------------|---------------|----------|
| **LabeledFewShot** | Demo selection | Golden answers | When you have quality labels |
| **BootstrapFewShot** | Demo generation | Metric function | Self-supervised learning |
| **BootstrapFewShotWithRandomSearch** | Demos + exploration | Metric function | Finding diverse solutions |
| **MIPRO** | Instructions + demos | Both | Full prompt optimization |
| **MIPROv2** | Advanced MIPRO | Both | State-of-the-art results |

#### 3.3.2 LabeledFewShot

Uses provided golden answers directly as demonstrations.

```python
from dspy.teleprompt import LabeledFewShot

optimizer = LabeledFewShot(k=5)  # Use 5 demos
optimized_module = optimizer.compile(
    student=FarmerChatStitcher(),
    trainset=train_set  # Must have 'response' field with golden answers
)
```

**Pros:**
- Simple, fast
- Uses known-good examples

**Cons:**
- Quality depends entirely on demo selection
- No filtering or adaptation

#### 3.3.3 BootstrapFewShot

Generates demos by running the model and keeping those that pass the metric.

```python
from dspy.teleprompt import BootstrapFewShot

optimizer = BootstrapFewShot(
    metric=my_metric_function,  # Returns True/False
    max_bootstrapped_demos=8,
    max_labeled_demos=4,
    max_rounds=3
)
optimized_module = optimizer.compile(
    student=FarmerChatStitcher(),
    trainset=train_set
)
```

**How it works:**
1. Run model on training examples
2. Evaluate each output with metric
3. Keep outputs that pass (metric returns True)
4. Use passing examples as demos

**Pros:**
- Self-supervised (no golden answers needed)
- Finds examples that demonstrably work

**Cons:**
- Depends heavily on metric quality
- May find "gaming" solutions

#### 3.3.4 MIPRO (Mixed-Initiative Prompt Optimization)

Optimizes both instructions and demos using Bayesian optimization.

```python
from dspy.teleprompt import MIPRO

optimizer = MIPRO(
    metric=my_metric,
    num_candidates=10,
    init_temperature=1.0
)
optimized_module = optimizer.compile(
    student=FarmerChatStitcher(),
    trainset=train_set,
    num_trials=20
)
```

**How it works:**
1. Generate candidate instructions via LLM
2. Generate candidate demos
3. Search over combinations
4. Use Bayesian optimization to find best

**Pros:**
- Can discover novel instructions
- Optimizes full prompt

**Cons:**
- Expensive (many LLM calls)
- May overfit to metric

### 3.4 Metric Functions

DSPy metrics must return **boolean** for optimizers (pass/fail):

```python
def conversationality_metric(example, prediction, trace=None) -> bool:
    response = prediction.response
    question = example.farmer_question
    
    score = evaluator.get_overall_score(question, response)
    
    # Binary threshold - True if "good enough"
    return score >= 4.0
```

**Critical:** The metric evaluates **model-generated responses**, not golden answers.

---

## 4. Experiment Timeline & Iterations

### 4.1 Iteration 0: Baseline Establishment

**Date:** Initial setup

**Setup:**
- Model: GPT-4o → GPT-4.1 → GPT-5.1
- Base prompt: Hand-written FactStitching signature
- Evaluation: 6-dimension ConversationalityEvaluator

**Baseline Results:**
```
┌─────────────────────────────────────────┬────────────┐
│ Configuration                           │ Avg Score  │
├─────────────────────────────────────────┼────────────┤
│ Base FactStitching (no optimization)    │   4.00     │
└─────────────────────────────────────────┴────────────┘
```

**Key Observation:** Base prompt already performs well (80%). This is important—optimization has limited headroom.

---

### 4.2 Iteration 1: BootstrapFewShot Attempt

**Date:** First optimization attempt

**Hypothesis:** BootstrapFewShot can find demos that improve conversationality.

**Configuration:**
```python
bootstrap_optimizer = BootstrapFewShot(
    metric=conversationality_metric,
    max_bootstrapped_demos=35,
    max_labeled_demos=5,
    max_rounds=3
)
```

**Results:**
```
┌─────────────────────────────────────────┬────────────┐
│ Configuration                           │ Avg Score  │
├─────────────────────────────────────────┼────────────┤
│ Base (unoptimized)                      │   4.73     │
│ BootstrapFewShot                        │   4.63     │
└─────────────────────────────────────────┴────────────┘
```

**Problem:** Optimized version performed WORSE than base.

**Analysis:**
1. Metric was returning continuous scores (0-1), not boolean
2. Parameters were hardcoded, not matching actual data size
3. No visibility into what optimization changed

---

### 4.3 Iteration 2: LabeledFewShot with Golden Answers

**Date:** Pivoting to supervised approach

**Hypothesis:** Since we have ~10,000 human-validated golden answers, use them directly instead of bootstrapping.

**Data:** "RLHF SFT DATA with Facts Information.xlsx"
- 8,480 valid rows with questions, facts, and golden answers
- Sampled 97 examples (67 train, 30 validation)

**Configuration:**
```python
from dspy.teleprompt import LabeledFewShot

labeled_optimizer = LabeledFewShot(k=5)
optimized_stitcher = labeled_optimizer.compile(
    student=FarmerChatStitcher(),
    trainset=train_set  # Contains golden 'response' field
)
```

**Results:**
```
┌─────────────────────────────────────────┬────────────┐
│ Configuration                           │ Avg Score  │
├─────────────────────────────────────────┼────────────┤
│ Base (unoptimized)                      │   4.00     │
│ LabeledFewShot (5 golden demos)         │   4.04     │
└─────────────────────────────────────────┴────────────┘
```

**Problem:** Marginal improvement, but new issue discovered—**language mismatch**.

---

### 4.4 Iteration 3: Language Mismatch Discovery

**Date:** Debugging unexpected behavior

**Issue Discovered:** Model responding in Hindi to English queries (30% mismatch rate).

**Language Consistency Check:**
```
[1] Query: ENGLISH → Response: ENGLISH ✓
[2] Query: HINDI   → Response: HINDI   ✓
[3] Query: ENGLISH → Response: ENGLISH ✓
[4] Query: HINDI   → Response: HINDI   ✓
[5] Query: ENGLISH → Response: HINDI   ✗ MISMATCH
[6] Query: ENGLISH → Response: ENGLISH ✓
[7] Query: ENGLISH → Response: ENGLISH ✓
[8] Query: ENGLISH → Response: ENGLISH ✓
[9] Query: ENGLISH → Response: HINDI   ✗ MISMATCH
[10] Query: ENGLISH → Response: HINDI  ✗ MISMATCH

SUMMARY: 3/10 language mismatches (30%)
```

**Root Cause Analysis:**

1. **Training data language imbalance:**
   - Training set had more Hindi golden answers
   - LabeledFewShot randomly selected demos
   - Selected demos were predominantly Hindi

2. **Demo influence on output:**
   - Few-shot demos heavily influence model behavior
   - Hindi demos → model mimics Hindi style → Hindi output

**CoT Visualization showed model reasoning correctly:**
```
"Farmer is asking in English; respond in English. Need to synthesize facts..."
```

But demos overrode this intention.

---

### 4.5 Iteration 4: Pattern Extraction Approach

**Date:** New strategy

**Hypothesis:** Instead of using raw golden answers as demos:
1. Extract PATTERNS from 100 golden answers
2. Generate SYNTHETIC demos that demonstrate patterns
3. Control language distribution in demos

**Pattern Extraction Pipeline:**
```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    PATTERN EXTRACTION PIPELINE                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  100 Golden Answers                                                          │
│         │                                                                    │
│         ▼                                                                    │
│  ┌─────────────────────────────────────────────────────────────────┐        │
│  │ BATCH ANALYSIS (5 batches × 20 examples)                        │        │
│  │ GPT-4o extracts:                                                │        │
│  │ - Greeting patterns                                             │        │
│  │ - Structure patterns                                            │        │
│  │ - Language rules                                                │        │
│  │ - Tone patterns                                                 │        │
│  │ - Fact integration techniques                                   │        │
│  │ - Bihar-specific elements                                       │        │
│  │ - Anti-patterns (what to avoid)                                 │        │
│  └─────────────────────────────────────────────────────────────────┘        │
│         │                                                                    │
│         ▼                                                                    │
│  ┌─────────────────────────────────────────────────────────────────┐        │
│  │ PATTERN CONSOLIDATION                                           │        │
│  │ Merge patterns across batches into:                             │        │
│  │ - Top 5 rules                                                   │        │
│  │ - Critical language rule                                        │        │
│  │ - Key anti-patterns                                             │        │
│  └─────────────────────────────────────────────────────────────────┘        │
│         │                                                                    │
│         ▼                                                                    │
│  ┌─────────────────────────────────────────────────────────────────┐        │
│  │ SYNTHETIC DEMO GENERATION                                       │        │
│  │ Generate 8 compact demos:                                       │        │
│  │ - 3 English queries → English responses                         │        │
│  │ - 3 Hindi queries → Hindi responses                             │        │
│  │ - 2 Hinglish queries → Hinglish responses                       │        │
│  │ - Each 80-120 words (compact)                                   │        │
│  └─────────────────────────────────────────────────────────────────┘        │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

**Extracted Patterns:**
```json
{
  "top_5_rules": [
    "Responses match the query language exactly",
    "Supportive and informative tone with actionable steps",
    "Facts integrated naturally within explanations",
    "Encouragement to consult local experts",
    "Avoidance of overly technical jargon"
  ],
  "critical_language_rule": "Response language MUST match query language"
}
```

**Synthetic Demos Generated:**

| # | Language | Crop | Topic | Words |
|---|----------|------|-------|-------|
| 1 | English | Rice | Pest management | 65 |
| 2 | Hindi | Wheat | Irrigation | 48 |
| 3 | English | Maize | Soil health | 117 |
| 4 | Hindi | Potato | Disease management | 104 |
| 5 | Hinglish | Vegetables | Organic farming | 63 |
| 6 | English | Pulses | Seed selection | 75 |
| 7 | Hindi | Sugarcane | Weather protection | 113 |
| 8 | Hinglish | Mustard | Fertilizer | 69 |

---

### 4.6 Iteration 5: Pattern-Enhanced Signature (First Attempt)

**Date:** Integration attempt

**Approach:** Replace original signature docstring with pattern rules + inject synthetic demos.

**Configuration:**
```python
class PatternEnhancedFactStitching(dspy.Signature):
    """Synthesize agricultural facts into conversational response for Bihar farmers.
    
    CRITICAL RULES (extracted from 100 golden answers):
    1. Response language MUST match query language exactly
    2. Use supportive, warm tone with actionable steps
    3. Integrate facts naturally - don't list them mechanically
    4. Reference local resources (KVK, कृषि विज्ञान केंद्र)
    5. Keep responses 100-150 words
    """
    # ... fields ...
```

**Results:**
```
┌─────────────────────────────────────────┬────────────┐
│ Configuration                           │ Avg Score  │
├─────────────────────────────────────────┼────────────┤
│ Base (unoptimized)                      │   4.17     │
│ Pattern-Enhanced (8 demos)              │   3.89     │
└─────────────────────────────────────────┴────────────┘
```

**Problem:** Score DECREASED by 6.8%!

**Root Cause:** We REPLACED the original detailed prompt instead of ENHANCING it. The original signature had important instructions that were lost.

---

### 4.7 Iteration 6: Merged Prompt (Correct Approach)

**Date:** Current iteration

**Fix:** Merge original SYSTEM_PROMPT + pattern additions, don't replace.

**Original Prompt Structure:**
```
SYSTEM_PROMPT (detailed, ~400 words)
├── YOUR ROLE (3 points)
├── RESPONSE GUIDELINES
│   ├── 1. Content Quality (4 sub-points)
│   ├── 2. Communication Style (4 sub-points)
│   ├── 3. Practical Advice (4 sub-points)
│   ├── 4. Safety & Credibility (4 sub-points)
│   └── 5. Conversation Flow (4 sub-points)
├── RESPONSE FORMAT
└── AVOID (5 points)
```

**Merged Prompt Structure:**
```
FULL_SYSTEM_PROMPT
├── [Original SYSTEM_PROMPT - all sections preserved]
└── ADDITIONAL RULES (from pattern analysis)
    ├── Language matching rule
    ├── Local resources reference
    └── Encouragement closing
```

**Configuration:**
```python
FULL_SYSTEM_PROMPT = """[Original 400-word prompt]...

**ADDITIONAL RULES (from pattern analysis):**
- CRITICAL: Response language MUST match query language exactly
- Reference local resources when relevant (KVK, कृषि विज्ञान केंद्र)
- End with encouragement or offer for follow-up
"""

class FactStitchingFull(dspy.Signature):
    __doc__ = FULL_SYSTEM_PROMPT
    # ... fields ...
```

**Results:** (Pending evaluation)

---

## 5. Optimizer Techniques Deep Dive

### 5.1 When to Use Each Optimizer

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    OPTIMIZER DECISION TREE                                   │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  Do you have golden answers (human-validated)?                               │
│         │                                                                    │
│         ├── YES ──→ Are they high quality and diverse?                      │
│         │                │                                                   │
│         │                ├── YES ──→ LabeledFewShot                         │
│         │                │           (Use golden answers as demos)          │
│         │                │                                                   │
│         │                └── NO ───→ BootstrapFewShot                       │
│         │                            (Generate better demos via metric)     │
│         │                                                                   │
│         └── NO ───→ Do you have a reliable metric?                         │
│                          │                                                  │
│                          ├── YES ──→ BootstrapFewShot                       │
│                          │           (Self-supervised demo generation)      │
│                          │                                                  │
│                          └── NO ───→ Manual prompt engineering              │
│                                      (No auto-prompting possible)           │
│                                                                              │
│  Is your base prompt already strong (>80%)?                                 │
│         │                                                                    │
│         ├── YES ──→ Marginal gains likely                                   │
│         │           Consider manual refinement instead                       │
│         │                                                                   │
│         └── NO ───→ Good candidate for optimization                         │
│                     Try MIPRO for instruction + demo optimization           │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 5.2 LabeledFewShot Deep Dive

**How it works:**
1. Takes training examples with golden `response` field
2. Randomly selects `k` examples
3. Injects them as few-shot demos in prompt

**Code:**
```python
from dspy.teleprompt import LabeledFewShot

optimizer = LabeledFewShot(k=5)
optimized = optimizer.compile(
    student=MyModule(),
    trainset=train_examples  # Must have response field
)
```

**Gotchas:**
- Doesn't filter demos by quality—uses them as-is
- Random selection may pick suboptimal examples
- Language/style of demos heavily influences output

**When to use:**
- High-quality, diverse golden answers
- Simple optimization needs
- Quick baseline for few-shot

### 5.3 BootstrapFewShot Deep Dive

**How it works:**
1. Run model on training examples
2. Evaluate outputs with metric function
3. Keep outputs where metric returns True
4. Use kept outputs as demos for future inputs

**Code:**
```python
from dspy.teleprompt import BootstrapFewShot

def my_metric(example, prediction, trace=None) -> bool:
    score = evaluate(prediction.response)
    return score >= 4.0  # Must return boolean!

optimizer = BootstrapFewShot(
    metric=my_metric,
    max_bootstrapped_demos=8,   # Max self-generated demos
    max_labeled_demos=4,        # Max demos from golden answers
    max_rounds=3                # Optimization rounds
)
```

**Gotchas:**
- Metric MUST return boolean, not float
- May "game" metric if metric has loopholes
- Expensive (runs model many times)

**When to use:**
- No golden answers available
- Want to discover what works empirically
- Metric captures desired behavior well

### 5.4 Manual Demo Injection

When DSPy optimizers don't work well, manually inject demos:

```python
class PatternEnhancedStitcher(dspy.Module):
    def __init__(self, demos=None):
        super().__init__()
        self.generate_response = dspy.ChainOfThought(FactStitching)
        
        if demos:
            demo_examples = [
                dspy.Example(**d).with_inputs('farmer_question', 'agricultural_facts', 'crop_context')
                for d in demos
            ]
            self.generate_response.demos = demo_examples
```

**When to use:**
- Need precise control over demos
- Optimizers have compatibility issues (e.g., ChainOfThought)
- Want curated, synthetic demos

---

## 6. Key Findings & Lessons Learned

### 6.1 Finding #1: Strong Base Limits Optimization Gains

**Observation:** Base prompt scored 4.0-4.17/5.0 (80-83%) without optimization.

**Implication:** When base is strong, optimization has limited headroom. Effort may be better spent on other system components.

```
Optimization Potential vs. Base Quality

Potential │
Gain      │ ████████
          │ ████████
          │ ████████ ████
          │ ████████ ████ ████
          │ ████████ ████ ████ ▓▓▓▓
          └─────────────────────────────
            Poor    Medium  Good  Excellent
            Base    Base    Base    Base
            (2.0)   (3.0)   (4.0)   (4.5)
```

### 6.2 Finding #2: Demo Language Influences Output Language

**Observation:** 30% language mismatch when demos were predominantly Hindi.

**Root Cause:** LLMs mimic the style/language of few-shot examples very strongly.

**Solution:** Control demo language distribution to match expected query distribution.

### 6.3 Finding #3: Don't Replace, Enhance

**Observation:** Replacing original prompt with new rules decreased score by 6.8%.

**Root Cause:** Original prompt had valuable instructions developed over time.

**Solution:** Append new rules to original prompt, don't replace.

### 6.4 Finding #4: DSPy + ChainOfThought Demo Attachment Issues

**Observation:** `LabeledFewShot` demos weren't attaching to `ChainOfThought` predictor (showed `demos: 0`).

**Workaround:** Manual demo injection to the predictor's `demos` attribute.

### 6.5 Finding #5: Metric Design is Critical

**Observation:** Conversationality metric didn't penalize language mismatch.

**Implication:** Model optimized for metric, not for actual desired behavior.

**Lesson:** Metric must capture ALL aspects of desired output, including language matching.

### 6.6 Summary: When Auto-Prompting Helps vs. Doesn't

| Helps | Doesn't Help |
|-------|--------------|
| Weak base prompt | Strong base prompt (>80%) |
| Clear, comprehensive metric | Metric missing key aspects |
| Large, diverse demo pool | Imbalanced demo distribution |
| Well-defined task | Nuanced requirements (language matching) |
| Have compute budget for search | Need quick, cheap optimization |

---

## 7. Code Artifacts

### 7.1 Project Structure

```
farmer-chat-optimization/
├── notebooks/
│   ├── dspy_stitching_optimization.ipynb     # Main optimization notebook
│   └── pattern_extraction.ipynb               # Pattern analysis notebook
├── outputs/
│   ├── pattern_extraction_results.json        # Extracted patterns
│   ├── synthetic_demos.json                   # Generated demos
│   └── evaluation_results.json                # Experiment results
├── prompts/
│   ├── base_system_prompt.txt                 # Original production prompt
│   └── enhanced_system_prompt.txt             # Merged prompt with patterns
└── docs/
    └── auto_prompting_documentation.md        # This document
```

### 7.2 Key Code Snippets

#### Base Signature
```python
class FactStitching(dspy.Signature):
    """Synthesize structured agricultural facts into a natural, conversational 
    response for a farmer in Bihar, India.
    
    The response should:
    - Address the farmer's question directly and practically
    - Use warm, supportive language appropriate for smallholder farmers
    - Integrate all relevant facts naturally without listing them mechanically
    - Focus on actionable, low-cost solutions
    - Include Bihar-specific context and local examples
    - Be 150-300 words in length
    - IMPORTANT: Respond in the SAME LANGUAGE as the farmer's question
    """
    
    farmer_question: str = dspy.InputField(
        desc="The farmer's question (may be in English, Hindi, or Hinglish)"
    )
    agricultural_facts: str = dspy.InputField(
        desc="JSON array of structured facts"
    )
    crop_context: str = dspy.InputField(
        desc="Context about crop, season, location"
    )
    response: str = dspy.OutputField(
        desc="Natural response in SAME LANGUAGE as query"
    )
```

#### Enhanced Signature with Full Prompt
```python
FULL_SYSTEM_PROMPT = """You are Farmer.CHAT, a knowledgeable agricultural advisor...
[Full 400-word prompt]
...
**ADDITIONAL RULES (from pattern analysis):**
- CRITICAL: Response language MUST match query language exactly
- Reference local resources when relevant (KVK, कृषि विज्ञान केंद्र)
- End with encouragement or offer for follow-up
"""

class FactStitchingFull(dspy.Signature):
    __doc__ = FULL_SYSTEM_PROMPT
    
    farmer_question: str = dspy.InputField(desc="...")
    agricultural_facts: str = dspy.InputField(desc="...")
    crop_context: str = dspy.InputField(desc="...")
    response: str = dspy.OutputField(desc="...")
```

#### Manual Demo Injection
```python
class PatternEnhancedStitcher(dspy.Module):
    def __init__(self, demos=None):
        super().__init__()
        self.generate_response = dspy.ChainOfThought(FactStitchingFull)
        
        if demos:
            demo_examples = []
            for d in demos:
                ex = dspy.Example(
                    farmer_question=d['farmer_question'],
                    agricultural_facts=d['agricultural_facts'],
                    crop_context=d['crop_context'],
                    response=d['response']
                ).with_inputs('farmer_question', 'agricultural_facts', 'crop_context')
                demo_examples.append(ex)
            self.generate_response.demos = demo_examples
    
    def forward(self, farmer_question, agricultural_facts, crop_context=""):
        return self.generate_response(
            farmer_question=farmer_question,
            agricultural_facts=agricultural_facts,
            crop_context=crop_context
        )
```

#### CoT Visualization
```python
def visualize_cot_reasoning(stitcher, example):
    result = stitcher(
        farmer_question=example.farmer_question,
        agricultural_facts=example.agricultural_facts,
        crop_context=example.crop_context
    )
    
    reasoning = getattr(result, 'reasoning', None)
    print(f"Reasoning: {reasoning}")
    print(f"Response: {result.response}")
    
    return result
```

#### Print Full Optimized Prompt
```python
# One-liner to see the full prompt
print(
    PatternEnhancedFactStitching.__doc__, 
    "\n\nDEMOS:\n", 
    "\n---\n".join([
        f"Q: {d.farmer_question}\nR: {d.response}" 
        for d in pattern_enhanced_stitcher.generate_response.demos
    ])
)
```

---

## 8. Future Directions

### 8.1 Short-Term Improvements

1. **Add language matching to metric:**
   ```python
   def improved_metric(example, prediction, trace=None) -> bool:
       # Check language match
       query_lang = detect_language(example.farmer_question)
       response_lang = detect_language(prediction.response)
       if query_lang != response_lang:
           return False  # Automatic fail
       
       # Then check conversationality
       score = evaluator.get_overall_score(...)
       return score >= 4.0
   ```

2. **Evaluate merged prompt:** Run full evaluation on `FactStitchingFull` + synthetic demos.

3. **A/B test in production:** Compare base vs. pattern-enhanced in real user interactions.

### 8.2 Medium-Term Explorations

1. **MIPROv2 with improved metric:** Let optimizer discover novel instructions.

2. **Dynamic demo selection:** Select demos based on query characteristics (language, topic).

3. **Retrieval-augmented demos:** Use embedding similarity to find most relevant demos per query.

### 8.3 Long-Term Research Questions

1. **Cross-lingual consistency:** How to ensure consistent quality across English/Hindi/Hinglish?

2. **Personalization:** Can prompts be optimized per-user or per-region within Bihar?

3. **Efficiency:** Can we achieve similar quality with smaller models + better prompts?

---

## 9. References & Resources

### 9.1 DSPy Resources

- [DSPy GitHub Repository](https://github.com/stanfordnlp/dspy)
- [DSPy Documentation](https://dspy-docs.vercel.app/)
- [DSPy Paper: "DSPy: Compiling Declarative Language Model Calls into Self-Improving Pipelines"](https://arxiv.org/abs/2310.03714)

### 9.2 Related Papers

- [Chain-of-Thought Prompting](https://arxiv.org/abs/2201.11903)
- [Self-Consistency Improves Chain of Thought](https://arxiv.org/abs/2203.11171)
- [Automatic Prompt Engineer (APE)](https://arxiv.org/abs/2211.01910)
- [OPRO: Optimization by PROmpting](https://arxiv.org/abs/2309.03409)

### 9.3 Project Links

- Farmer.CHAT MCP Server: [HuggingFace Space]
- RLHF Dataset: "RLHF SFT DATA with Facts Information.xlsx"
- Evaluation Framework: ConversationalityEvaluator (6-dimension)

---

## Appendix A: Experiment Log Template

```markdown
### Experiment: [Name]

**Date:** YYYY-MM-DD

**Hypothesis:** [What you expect to happen]

**Configuration:**
- Model: 
- Optimizer:
- Parameters:
- Training data:
- Validation data:

**Results:**
| Metric | Base | Optimized | Delta |
|--------|------|-----------|-------|
| Overall | X.XX | X.XX | +X.XX |
| Content | X | X | +X |
| Style | X | X | +X |

**Observations:**
- [Key observation 1]
- [Key observation 2]

**Next Steps:**
- [What to try next]
```

---

## Appendix B: Glossary

| Term | Definition |
|------|------------|
| **Signature** | DSPy class defining input/output structure and instructions |
| **Module** | DSPy class wrapping signature with reasoning strategy |
| **Demo** | Example input-output pair shown in prompt (few-shot) |
| **Optimizer** | DSPy component that improves prompts automatically |
| **CoT** | Chain of Thought - reasoning before answering |
| **Golden Answer** | Human-validated correct response |
| **Bootstrap** | Generate examples by running model and filtering |
| **Metric** | Function evaluating output quality (returns bool for DSPy) |
| **Stitching** | Converting structured facts into natural language |

---

*Document maintained by: Aakash (AI Engineer, Digital Green)*
*For questions or contributions, contact the Farmer.CHAT team.*
