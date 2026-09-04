# SWE-bench — Can Language Models Solve Real Software Engineering Issues?

> **Paper:** SWE-bench: Can Language Models Resolve Real-World GitHub Issues?
> **Authors:** Carlos E. Jimenez et al.
> **Venue:** ICLR 2024
> **Paper:** https://arxiv.org/abs/2310.06770
> **Project:** https://www.swebench.com/

---

## 1. Overview

SWE-bench is a benchmark designed to evaluate whether language models can solve **real-world software engineering problems**, rather than only small, isolated programming tasks.

The benchmark contains **2,294 task instances** collected from real GitHub issues and corresponding pull requests across **12 popular Python repositories**.

For each task, the model receives:

* A snapshot of the repository
* A GitHub issue describing the problem
* Relevant repository context

The model must then generate a **patch** that modifies the codebase to resolve the issue.

The generated patch is applied to the repository and evaluated using the repository's **real tests**.

### Basic workflow

```text
GitHub Issue
     ↓
Repository Snapshot
     ↓
Language Model
     ↓
Generated Patch
     ↓
Apply Patch
     ↓
Run Tests
     ↓
Resolved / Not Resolved
```

---

## 2. Why SWE-bench?

Traditional coding benchmarks such as **HumanEval** mostly evaluate small, self-contained programming problems.

For example:

```text
Write a function that reverses a string.
```

Real software engineering is much more complicated.

A real GitHub issue may require a developer to:

* Understand an unfamiliar repository
* Locate the relevant files
* Understand interactions between functions and classes
* Modify multiple files
* Add or modify tests
* Debug unexpected behavior
* Verify that the change does not break existing functionality

SWE-bench attempts to capture this complexity.

### Traditional coding benchmark

```text
Problem
   ↓
Function
   ↓
Answer
```

### SWE-bench

```text
Real Issue
    ↓
Large Repository
    ↓
Repository Understanding
    ↓
Code Localization
    ↓
Reasoning
    ↓
Code Modification
    ↓
Patch
    ↓
Tests
```

---

# 3. How SWE-bench is Constructed

The benchmark uses a three-stage process.

## Stage 1 — Repository Selection & PR Scraping

The authors collect pull requests from **12 popular open-source Python repositories**.

Approximately **90,000 PRs** were initially collected.

Popular repositories were selected because they generally have:

* Better maintenance
* Clear contribution guidelines
* Better test coverage

---

## Stage 2 — Attribute-Based Filtering

Candidate task instances are selected when a merged pull request:

1. Resolves a GitHub issue
2. Modifies repository test files

The second condition is useful because changes to tests suggest that the contributor added or modified tests related to the issue.

```text
~90,000 PRs
      ↓
Merged PRs
      ↓
Resolves an issue
      +
Changes tests
      ↓
Candidate tasks
```

---

## Stage 3 — Execution-Based Filtering

The candidate pull requests are then evaluated automatically.

The authors apply the tests associated with the PR and compare test results before and after the actual code changes.

A task is retained when at least one relevant test changes from:

```text
FAIL → PASS
```

This helps ensure that the benchmark contains issues that are actually associated with a verifiable software change.

---

# 4. What Does a SWE-bench Task Look Like?

A task contains three important components:

```text
┌───────────────────────┐
│     Issue Description │
└───────────┬───────────┘
            ↓
┌───────────────────────┐
│   Repository Snapshot │
└───────────┬───────────┘
            ↓
       Language Model
            ↓
┌───────────────────────┐
│    Generated Patch    │
└───────────┬───────────┘
            ↓
       Real Test Suite
            ↓
      Pass / Fail
```

The model is not simply asked to generate code from scratch.

It has to **modify an existing codebase**.

---

# 5. Main Results

The initial results were surprisingly low.

Using BM25 retrieval:

> **Claude 2 resolved only 1.96% of SWE-bench issues.**

This demonstrates how difficult real-world software engineering is for language models.

Even powerful models struggled with tasks requiring:

* Repository understanding
* Code localization
* Multi-file reasoning
* Context management
* Correct patch generation
* Testing and debugging

---

# 6. Retrieval Matters

One of the most important findings of the paper is that **retrieval and context selection strongly affect model performance**.

The researchers compared normal retrieval with an **oracle retrieval** setting.

### BM25 retrieval

The system automatically retrieves potentially relevant files.

```text
Issue
 ↓
BM25
 ↓
Potentially relevant files
 ↓
LLM
```

### Oracle retrieval

The model is given files known to be involved in the actual human solution.

```text
Issue
 ↓
Known relevant files
 ↓
LLM
```

Claude 2 improved from approximately:

```text
BM25      → 1.96%
Oracle    → 4.8%
```

This suggests that **finding the correct code is a major bottleneck**.

---

# 7. More Context Can Actually Hurt

A particularly interesting result is that increasing the amount of retrieved context does not necessarily improve performance.

Instead:

```text
More context
     ↓
More irrelevant code
     ↓
Harder code localization
     ↓
Model distraction
     ↓
Lower performance
```

The authors observed that Claude 2's performance decreased as total input context became larger.

This suggests that an effective coding agent should not simply provide the model with as much repository code as possible.

It should provide **focused and relevant context**.

---

# 8. Oracle-Collapsed Retrieval

The authors further tested a setting where they provided the correct files but removed most irrelevant code.

Only the lines modified by the original pull request, with approximately ±15 lines of surrounding context, were retained.

Performance improved.

For example:

| Model         | Oracle-collapsed |
| ------------- | ---------------: |
| Claude 3 Opus |            9.39% |
| Claude 2      |            5.93% |
| GPT-4         |            3.40% |
| ChatGPT-3.5   |            1.09% |

### Key insight

> **The quality of context matters more than simply increasing context size.**

---

# 9. Context Distribution Shift

The paper also identifies an interesting problem with fine-tuned models.

SWE-Llama 7B and 13B were fine-tuned using **oracle-style retrieval contexts**.

During training, the model generally received files that were expected to be modified.

However, BM25 retrieval may provide:

```text
Relevant File
Irrelevant File
Irrelevant File
Relevant File
```

This creates a **context distribution shift**.

The model was trained to expect:

```text
Provided file → likely needs editing
```

but during BM25 evaluation:

```text
Provided file → may or may not need editing
```

This can significantly affect performance.

### Research lesson

Retrieval strategy and model training should ideally be aligned.

---

# 10. Patch Generation vs Whole-File Generation

The researchers also investigated whether models should generate:

### Option A — A patch

```diff
- old code
+ new code
```

or

### Option B — The entire modified file

Although models are more commonly trained on complete source files, **whole-file generation performed worse**.

For Claude 2:

```text
Patch generation       → 4.8%
Whole-file generation  → 2.2%
```

This suggests that patches are a more effective representation for software changes.

---

# 11. Models Tend to Under-Edit

Another interesting finding is that model-generated patches tend to be smaller than human patches.

Average patch size:

```text
Human patch → 74.5 lines
Model patch → 30.1 lines
```

Models also tend to modify fewer locations.

This can be beneficial because smaller changes reduce unintended modifications.

However, it can also cause **under-fixing**.

For example:

```text
Human solution:

Fix bug
+ update helper function
+ modify validation
+ update tests
```

while the model might only:

```text
Change one line
```

The patch may look reasonable but still fail the complete requirements.

---

# 12. Limitations

The authors identify several limitations.

### 1. Python only

The original SWE-bench tasks focus on Python repositories.

Future versions could expand to:

* Java
* JavaScript / TypeScript
* C++
* Go
* Rust
* Other domains

### 2. Simple baseline approaches

The experiments primarily establish baseline approaches.

The authors encourage future work using:

* Agent-based systems
* Tool-augmented language models
* Interactive repository exploration
* More sophisticated retrieval

### 3. Tests are not enough

Passing tests does not guarantee that generated code is:

* Comprehensive
* Efficient
* Readable
* Maintainable

Therefore:

```text
Tests Pass
     ≠
Perfect Software
```

---

# 13. Future Direction: Coding Agents

One of the most important future directions suggested by the paper is moving from:

```text
Issue → LLM → Patch
```

toward interactive agents:

```text
Issue
 ↓
Search repository
 ↓
Read files
 ↓
Reason
 ↓
Edit code
 ↓
Run tests
 ↓
Observe failure
 ↓
Debug
 ↓
Edit again
 ↓
Run tests
 ↓
Final Patch
```

This transforms the LLM from a **one-shot code generator** into an **interactive software engineering agent**.

---

# 14. My Key Takeaways

### Takeaway 1 — Software engineering ≠ code completion

Real software engineering requires repository understanding, code localization, modification, testing, and debugging.

### Takeaway 2 — Retrieval is a critical bottleneck

Even strong LLMs struggle when they cannot identify the correct code.

### Takeaway 3 — More context isn't always better

Irrelevant context can distract the model.

### Takeaway 4 — Context selection matters

Oracle-collapsed retrieval demonstrates the value of providing highly focused context.

### Takeaway 5 — Retrieval and model training should be aligned

Fine-tuned models can suffer when the retrieval distribution changes.

### Takeaway 6 — Patches are preferable to whole-file generation

They are more compact and performed better in the experiments.

### Takeaway 7 — Testing alone is insufficient

A patch can pass tests while still being inefficient, incomplete, or difficult to maintain.

### Takeaway 8 — The future is interactive

Agent-based and tool-augmented systems are natural next steps beyond one-shot LLM code generation.

---

# 15. Connection to My Research Project

This paper is particularly relevant to my work on **AI Software Engineering Agents and repository-level code retrieval**.

The SWE-bench results suggest that an effective coding agent needs a strong retrieval and context-selection system.

A potential architecture is:

```text
                  GitHub Issue
                       ↓
               Repository Index
                       ↓
              ┌────────┴────────┐
              ↓                 ↓
             BM25           Embeddings
              ↓                 ↓
              └────────┬────────┘
                       ↓
                    Ranking
                       ↓
              Context Filtering
                       ↓
                Relevant Code
                       ↓
                     LLM
                       ↓
                Generate Patch
                       ↓
                  Apply Patch
                       ↓
                  Run Tests
                       ↓
                ┌──────┴──────┐
                ↓             ↓
              PASS           FAIL
                ↓             ↓
             Finish       Debug/Retry
```

This motivates further experimentation with:

* BM25 retrieval
* Embedding-based retrieval
* Hybrid retrieval
* Code chunking
* Reranking
* Context compression
* Agentic repository exploration
* Test-driven verification

---

# 16. Questions Raised by the Paper

Some research questions I would like to investigate further:

1. **Can hybrid BM25 + embedding retrieval outperform either method individually?**

2. **How should code repositories be chunked for retrieval?**

3. **Does retrieving functions/classes work better than retrieving entire files?**

4. **Can reranking reduce irrelevant context and improve LLM performance?**

5. **How much context is actually optimal for repository-level tasks?**

6. **Can an agent iteratively improve retrieval after observing test failures?**

7. **Can tool use help models locate code more reliably?**

8. **Can generated patches be evaluated beyond simply passing tests?**

---

# 17. Overall Reflection

SWE-bench changed the way I think about evaluating coding LLMs.

A model performing well on small coding problems does not necessarily mean it can perform real software engineering.

The difficult part is often not writing the code itself, but understanding **where the change belongs, what other code it interacts with, what context is relevant, and whether the resulting change actually solves the problem**.

The most important lesson I take from this paper is:

> **A strong coding agent needs more than a strong language model. It needs effective retrieval, focused context, repository understanding, tool interaction, and reliable verification.**

---



---

## Status

**Paper read:** ✅
**Core concepts understood:** ✅
**Retrieval insights studied:** ✅
**Relevance to AI coding agents:** High

---
