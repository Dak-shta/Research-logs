# RepoCoder: Repository-Level Code Completion Through Iterative Retrieval and Generation

> **Research Note / Paper Analysis**

**Paper:** RepoCoder: Repository-Level Code Completion Through Iterative Retrieval and Generation
**Authors:** Fengji Zhang, Bei Chen, Yue Zhang, Jacky Keung, Jin Liu, Daoguang Zan, Yi Mao, Jian-Guang Lou, Weizhu Chen
**Year:** 2023
**Venue:** EMNLP 2023
**Pages:** 2471–2484
**Paper:** https://arxiv.org/abs/2303.12570
**Official Publication:** https://aclanthology.org/2023.emnlp-main.151/

---

## 1. Research Problem

### What problem does RepoCoder address?

Traditional code completion systems primarily rely on the **unfinished content of the current file**. However, useful information required to complete a piece of code is often distributed across multiple files in a repository.

For example, related code may exist in:

* imported modules
* utility files
* neighboring files
* files using similar APIs
* similarly named files

The paper studies **repository-level code completion**, where the model is expected to use broader repository context rather than relying only on the target file.

### Why is repository-level context important?

Modern software repositories are highly modular. Files frequently share:

* APIs
* utility functions
* configurations
* programming patterns
* dependencies
* implementation conventions

Therefore, repository-level information can provide context that is unavailable to an in-file language model.

---

## 2. Core Research Question

The central question addressed by the paper is:

> **Can retrieval of relevant code from the repository improve language-model-based code completion?**

The authors further investigate:

> **Can iterative retrieval and generation improve upon standard Retrieval-Augmented Generation (RAG)?**

---

## 3. Key Contribution

The paper proposes **RepoCoder**, an iterative retrieval-generation framework for repository-level code completion.

The main idea is to combine:

1. A **code retrieval model**
2. A **pre-trained code language model**
3. An **iterative retrieval-generation process**

Instead of retrieving repository context only once, RepoCoder uses the model's generated completion to construct a better retrieval query for the next iteration.

### Core intuition

```text
Unfinished Code
      ↓
Retrieve relevant repository snippets
      ↓
Generate completion
      ↓
Use generated completion as improved retrieval context
      ↓
Retrieve again
      ↓
Generate improved completion
      ↓
Final Prediction
```

This creates a feedback loop between **retrieval and generation**.

---

# 4. How RepoCoder Works

Let:

* `X` = unfinished code
* `R` = retrieval model
* `M` = code language model
* `C_repo` = repository code snippets
* `Ŷ` = predicted completion

### In-File Completion

The baseline language model performs:

```text
Ŷ = M(X)
```

The model only receives the unfinished code.

---

### Vanilla Repository-Level RAG

The repository is first divided into retrievable code snippets:

```text
C_repo = {c₁, c₂, ..., cₙ}
```

The unfinished code `X` is used as a query:

```text
X
 ↓
Retriever
 ↓
Relevant Code Snippets
 ↓
LLM
 ↓
Completion
```

However, the paper identifies a problem:

> The unfinished code alone may not contain enough information to retrieve the code that is actually relevant to the intended completion.

This creates a **retrieval–target gap**.

---

### RepoCoder

RepoCoder addresses this gap iteratively.

```text
Iteration 1

Unfinished Code
      ↓
Retriever
      ↓
Retrieved Context
      ↓
LLM
      ↓
Predicted Completion
```

The prediction is then incorporated into the next retrieval query:

```text
Iteration 2

Unfinished Code + Previous Prediction
              ↓
          Retriever
              ↓
      Better Retrieved Context
              ↓
             LLM
              ↓
       Improved Prediction
```

The process can continue for additional iterations.

### Key insight

> **Generation is not only the final step; the generated code can also improve subsequent retrieval.**

This is the central idea that distinguishes RepoCoder from vanilla RAG.

---

# 5. Benchmark: RepoEval

The paper introduces **RepoEval**, a benchmark designed for repository-level code completion.

It uses real-world, open-source GitHub repositories and evaluates code completion at different levels of granularity.

### Completion Tasks

| Task                          | Description                                       |
| ----------------------------- | ------------------------------------------------- |
| **Line Completion**           | Predict the continuation of a line/code segment   |
| **API Invocation Completion** | Complete API/function calls                       |
| **Function Body Completion**  | Generate a larger missing function implementation |

The benchmark was designed to evaluate whether models can effectively utilize **repository-level context** rather than only local file context.

---

# 6. Experimental Setup

## Baselines

The paper primarily compares RepoCoder against:

### 1. In-File Completion

The model only receives the unfinished code from the current file.

```text
Current File Context
        ↓
       LLM
        ↓
   Completion
```

### 2. Oracle

The Oracle method provides retrieval using information closer to the intended completion, allowing the authors to estimate the potential benefit of effective retrieval.

This provides an upper-reference point for understanding the retrieval quality gap.

### 3. Vanilla RAG

Repository code is retrieved using the original unfinished code as the query, followed by generation.

RepoCoder extends this approach through **iterative retrieval and generation**.

---

# 7. Models and Retrieval

The experiments evaluate RepoCoder using different language models, including:

* GPT-3.5-Turbo
* CodeGen-Mono-6B
* CodeGen-Mono-2B
* CodeGen-Mono-350M

The main experiments use a **sparse similarity-based retriever** because of its effectiveness and computational efficiency.

The paper additionally evaluates RepoCoder using **UniXcoder**, a dense code retrieval model.

### Dense vs. Sparse Retrieval

The dense-retriever experiments show that RepoCoder achieves comparable performance with dense retrieval, suggesting that the framework is not fundamentally dependent on a particular retrieval architecture.

> **Research implication:** RepoCoder is better viewed as a general retrieval-generation framework rather than a method tied to one specific retriever.

---

# 8. Evaluation Metrics

## 8.1 Exact Match (EM)

A prediction receives a score of `1` if it exactly matches the ground-truth completion.

```text
EM = 1 → exact match
EM = 0 → otherwise
```

This metric is strict but easy to interpret.

---

## 8.2 Edit Similarity (ES)

Edit Similarity provides a more fine-grained comparison based on Levenshtein distance:

```text
ES = 1 − Lev(Y, Ŷ) / max(|Y|, |Ŷ|)
```

where:

* `Y` = ground-truth completion
* `Ŷ` = predicted completion
* `Lev` = Levenshtein edit distance

Unlike EM, ES gives partial credit when the prediction is similar but not identical.

---

## 8.3 Execution-Based Evaluation

The paper also emphasizes execution-based evaluation for code completion.

This is important because:

> **Code that is textually different from the reference implementation may still be functionally correct.**

The authors therefore argue that unit tests can provide a more meaningful measure of code correctness than exact string matching alone.

---

# 9. Main Experimental Findings

The experiments show that RepoCoder substantially improves over the **In-File completion paradigm** across the evaluated settings.

The paper reports improvements of more than **10% over the In-File baseline across all settings** and consistent improvements over vanilla retrieval-augmented code completion.

### General pattern

```text
In-File
   ↓
Vanilla RAG
   ↓
RepoCoder
```

RepoCoder demonstrates that:

> **Retrieval provides useful repository context, and iterative retrieval-generation can further improve the usefulness of that context.**

---

# 10. Why Iteration Helps

The key difference can be summarized as:

### Vanilla RAG

```text
Unfinished Code
      ↓
   Retrieval
      ↓
   Generation
```

### RepoCoder

```text
Unfinished Code
      ↓
   Retrieval
      ↓
   Generation
      ↓
Generated Code
      ↓
   Retrieval
      ↓
   Generation
      ↓
   ...
```

The generated completion provides additional semantic information that may be useful for retrieval.

Therefore:

> **Generation helps retrieval, and improved retrieval helps subsequent generation.**

This creates a retrieval-generation feedback loop.

---

# 11. Analysis: Where Does Useful Retrieved Code Come From?

The authors investigate the source locations of successfully retrieved code.

They categorize retrieved snippets into five locations:

| Category              | Meaning                                                |
| --------------------- | ------------------------------------------------------ |
| **Imported**          | Code from a file imported by the target file           |
| **Current File**      | Code from another portion of the target file           |
| **Current Directory** | Code from another file in the same directory           |
| **Similar Import**    | Code from a file sharing an API import with the target |
| **Similar Name**      | Code from a file with overlapping filename tokens      |

### Finding

A substantial portion of useful retrieved context comes from:

* Similar Import
* Similar Name
* Current Directory

This demonstrates that relevant information is distributed across the repository.

### Important observation

The authors also perform an ablation where retrieval is restricted to these manually defined locations.

Performance decreases.

This suggests that:

> **Relevant code cannot always be identified using simple handcrafted file-location rules.**

This supports RepoCoder's relatively simple repository-wide similarity-based retrieval strategy.

---

# 12. Dense Retriever Experiment

The authors replace their sparse retriever with **UniXcoder**, a dense code retrieval model.

The results remain broadly comparable.

### Interpretation

RepoCoder's effectiveness is not limited to sparse retrieval.

```text
Sparse Retriever ──┐
                   ├──→ RepoCoder
Dense Retriever ───┘
```

This provides evidence for the **robustness and generalizability** of the retrieval-generation framework.

---

# 13. Analysis: Code Duplication

The authors investigate whether RepoCoder's effectiveness is related to the amount of duplicated code in a repository.

This is intuitive because similarity-based retrieval should work better when repositories contain reusable or repeated code patterns.

### Observation

Repositories with higher duplication can provide more similar code examples for retrieval.

For example:

```text
High Duplication
      ↓
More similar snippets
      ↓
Better retrieval
      ↓
More useful context
      ↓
Potentially better completion
```

The `diffusers` repository shows a high duplication ratio and correspondingly strong RepoCoder improvements.

In contrast, repositories such as `rl` and `vizier` have lower duplication ratios and comparatively smaller improvements.

However, the relationship is **not absolute**.

Repositories with similar duplication ratios can still show different performance gains.

### Research takeaway

> **Code duplication influences retrieval effectiveness, but repository structure and code characteristics are also important factors.**

---

# 14. Analysis: Why Additional Iterations Can Fail

One of the most interesting findings is that:

> **More iterations do not always improve performance.**

The authors analyze cases where RepoCoder changes from correct to incorrect predictions across iterations.

### Main failure mode 1: Misleading retrieved code

A retrieved example may look relevant but represent a different API usage.

For example, the same API might use different parameters in different files.

```text
Target:
API(x, y)

Retrieved example:
API(x, y, z)
```

The model may incorrectly assume that the retrieved example is directly applicable.

Therefore:

```text
Relevant-looking retrieval
        ↓
Incorrect context
        ↓
Incorrect prediction
```

---

### Main failure mode 2: Poor retrieval query

RepoCoder constructs the next retrieval query using a fixed amount of generated code.

The generated completion may contain:

* useful initial code
* followed by irrelevant/noisy code

That noise can negatively affect the next retrieval step.

```text
Generated prediction
      ↓
Useful information + noise
      ↓
Retrieval
      ↓
Less relevant snippets
```

This explains why another iteration can sometimes make the result worse.

---

### Main failure mode 3: Exact Match is imperfect

The authors also observe that some predictions marked incorrect by Exact Match are actually functionally correct.

For example:

```python
# Ground truth
result = foo(x, y)

# Prediction
result = foo(y=x, x=y)
```

The strings differ, but the semantic behavior might still be correct depending on the API.

This motivates the use of:

> **Execution-based evaluation and unit tests for code-generation research.**

---

# 15. Limitations

## 15.1 Low Code Duplication

RepoCoder may provide limited gains when a repository contains very little duplicated or similar code.

```text
Low repository similarity
        ↓
Few useful retrieval candidates
        ↓
Weak contextual augmentation
        ↓
Smaller improvement
```

---

## 15.2 Optimal Number of Iterations

There is no universally optimal number of retrieval-generation iterations.

More iterations can sometimes:

* improve predictions
* introduce misleading context
* amplify retrieval errors
* decrease performance

Therefore, an important open problem is:

> **How can a system automatically determine when to stop iterating?**

---

## 15.3 Computational Cost and Latency

Each additional iteration requires another retrieval and generation step.

```text
More iterations
      ↓
Potentially better context
      ↓
Higher computation + latency
```

This is particularly important for real-time IDE code completion.

Possible optimization directions include:

* quantization
* knowledge distillation
* hardware acceleration
* retrieval caching
* repository preprocessing
* adaptive iteration counts

---

## 15.4 Limited Experimental Scope

The paper identifies several areas for future investigation:

### Prompt design

Different prompt templates may produce better results.

### Retrieval methods

More retrieval strategies beyond similarity-based retrieval could be explored.

### Newer code models

RepoCoder could be evaluated with stronger code-generation models.

### Better baselines

Repository-level code completion lacked mature standardized baselines, making systematic comparison difficult.

---

# 16. Critical Research Insights

### Insight 1 — Retrieval quality is as important as generation quality

A strong LLM cannot fully compensate for irrelevant retrieved context.

```text
Good Retrieval + Good LLM
          ↓
      Strong result

Bad Retrieval + Good LLM
          ↓
   Potentially misleading result
```

---

### Insight 2 — Repository context is non-local

Useful information does not necessarily exist:

* immediately before the target code
* in the same file
* in directly imported files

It can exist elsewhere in the repository.

This is a fundamental motivation for repository-level code intelligence.

---

### Insight 3 — Retrieval and generation should not be treated as independent stages

RepoCoder demonstrates a feedback relationship:

```text
Retrieval
   ↓
Generation
   ↓
Improved query
   ↓
Retrieval
   ↓
Generation
```

The output of generation can improve the next retrieval step.

---

### Insight 4 — More context is not always better

Additional retrieved code can introduce:

* irrelevant information
* conflicting API usage
* misleading examples
* retrieval noise

Therefore:

> **Context quality matters more than simply increasing context quantity.**

---

### Insight 5 — Evaluation of generated code is difficult

Exact string matching does not necessarily measure functional correctness.

A stronger evaluation framework should consider:

```text
Text similarity
      +
Execution correctness
      +
Unit tests
      +
Potentially semantic equivalence
```

---

# 17. My Understanding of RepoCoder

RepoCoder can be viewed as a **repository-level RAG system with an iterative feedback loop**.

The key innovation is not simply retrieving code.

Rather:

> **The model's own generated completion is fed back into the retrieval process to obtain more relevant repository context for the next generation step.**

This addresses the limitation of using only the unfinished code as a retrieval query.

---

# 18. Connection to AI Software Engineering Agents

RepoCoder provides an important foundation for understanding modern AI software engineering agents.

A software engineering agent also needs to answer:

> **What code/context should I retrieve before reasoning about a task?**

The same challenges appear in:

* code search
* bug localization
* repository understanding
* debugging
* program repair
* test generation
* refactoring
* issue resolution

A simplified progression is:

```text
Code Completion
      ↓
Repository-Level Retrieval
      ↓
Repository Understanding
      ↓
Planning + Tool Use
      ↓
Debugging / Testing
      ↓
Software Engineering Agent
```

RepoCoder therefore provides a useful conceptual bridge from **LLM code generation → repository-level RAG → iterative software engineering agents**.

---

# 19. Open Research Questions

RepoCoder raises several questions that are relevant for future research:

1. **How should retrieval queries be constructed for code?**
2. **How can irrelevant or misleading snippets be filtered?**
3. **How can the optimal number of iterations be determined dynamically?**
4. **Can retrieval be guided by program structure such as ASTs, dependency graphs, or call graphs?**
5. **Can execution feedback improve retrieval?**
6. **Can repository-level retrieval scale efficiently to very large repositories?**
7. **How should repository-level code generation be evaluated beyond Exact Match?**
8. **Can iterative retrieval-generation be extended from completion to debugging and autonomous software engineering?**

---

# 20. Final Takeaway

> **RepoCoder demonstrates that code completion can be substantially improved by retrieving relevant repository context and iteratively refining that context through model-generated predictions. Its main contribution is the retrieval-generation feedback loop, while its analysis exposes important challenges around retrieval quality, iteration control, latency, and functional evaluation.**

### One-line summary

**RepoCoder = Repository-level RAG + iterative retrieval-generation feedback for code completion.**

---

## Related Concepts to Study

* Retrieval-Augmented Generation (RAG)
* Code Retrieval
* Code LLMs
* Repository-Level Code Understanding
* Code Search
* Program Synthesis
* Program Repair
* Software Engineering Agents
* ReAct
* SWE-bench
* Repo-level prompting
* Function calling / tool use
* Unit-test-based code evaluation

---

## Research Note

**What I found most interesting:**
The most important insight is that **generation can improve retrieval**. Instead of treating retrieval as a one-time preprocessing step, RepoCoder creates a feedback loop in which the model's prediction becomes additional evidence for finding relevant repository context.

**Potential weakness I would investigate:**
The iterative process can amplify retrieval errors. A misleading retrieved snippet can produce a poor prediction, which then becomes the query for the next retrieval step. This suggests that **retrieval verification, reranking, or confidence-aware stopping** could be valuable extensions.

**Potential extension:**
A future repository-level coding agent could combine RepoCoder-style iterative retrieval with **dependency-aware retrieval, test execution, and tool feedback**, allowing retrieval to be guided not only by textual similarity but also by actual program behavior.
