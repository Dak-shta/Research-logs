# RepoCoder: Repository-Level Code Completion Through Iterative Retrieval and Generation

**Authors:** Fengji Zhang et al.
**Year:** 2023
**Venue:** EMNLP 2023
**Paper:** https://arxiv.org/abs/2303.12570

---

## 1. Problem

Traditional code completion mainly uses context from the current file. However, useful information for completing code can exist across different files in a repository.

**Research question:**
Can repository-level code retrieval improve code completion?

---

## 2. Key Idea

RepoCoder combines **code retrieval + language models** in an iterative process.

```text
Unfinished Code
      ↓
Retrieve relevant code
      ↓
Generate completion
      ↓
Use prediction for better retrieval
      ↓
Generate improved completion
```

The key idea is that the **generated code is used to improve the next retrieval step**.

---

## 3. RepoEval Benchmark

The authors introduce **RepoEval**, a benchmark for repository-level code completion.

It evaluates:

* **Line completion**
* **API invocation completion**
* **Function body completion**

They also use both text-based metrics and execution-based evaluation.

---

## 4. Experimental Setup

### Baselines

* **In-File:** Uses only the current file context.
* **Oracle:** Uses information closer to the intended completion.
* **Vanilla RAG:** Retrieves repository code once before generation.

### Models

Experiments use GPT-3.5-Turbo and different CodeGen models.

The paper also tests RepoCoder with a dense retriever (**UniXcoder**) to check whether the approach generalizes beyond the main sparse retriever.

---

## 5. Main Findings

* RepoCoder consistently improves over **In-File completion**.
* Iterative retrieval performs better than vanilla RAG.
* Useful context can come from different parts of the repository.
* RepoCoder works with both sparse and dense retrieval.
* More iterations do **not always** improve performance.

### Retrieval Analysis

Useful retrieved snippets commonly come from:

1. Imported files
2. Current file
3. Current directory
4. Similar imports
5. Similar filenames

This shows that relevant information is often distributed across the repository.

---

## 6. Failure Analysis

The authors found that later iterations can fail because:

* retrieved code can be misleading
* similar APIs may have different parameter usage
* generated predictions can contain noise and hurt the next retrieval
* Exact Match can mark functionally correct code as incorrect

This suggests that **retrieval quality and evaluation method are important**.

---

## 7. Limitations

| Limitation           | Observation                                                        |
| -------------------- | ------------------------------------------------------------------ |
| Low code duplication | Less useful code to retrieve                                       |
| Iteration control    | Optimal number of iterations is unclear                            |
| Latency              | Multiple retrieval-generation steps increase cost                  |
| Limited experiments  | More retrieval methods, prompts and newer models could be explored |

---

## 8. My Understanding

RepoCoder can be viewed as:

> **Repository-level RAG with an iterative retrieval-generation feedback loop.**

The most interesting idea for me is that **generation is used to improve retrieval**, rather than treating retrieval as a one-time step.

A possible future direction would be **adaptive retrieval**, where the system decides when to stop iterating and verifies whether retrieved code is actually relevant before using it.

---

## 9. Connection to AI Software Engineering Agents

RepoCoder is relevant to AI software engineering agents because agents also need to retrieve useful repository context for:

* debugging
* code repair
* testing
* refactoring
* code generation

This makes RepoCoder a useful foundation for understanding **repository-level code intelligence and coding agents**.
