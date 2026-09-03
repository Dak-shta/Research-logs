# Research-logs
A research-oriented study of Large Language Models for software engineering, covering repository-level code generation, coding agents, program repair, testing, and evaluation.
# LLM Software Engineering Research

> A research-oriented study of Large Language Models for Software Engineering.

This repository documents my study, analysis, implementation, and experimentation with Large Language Models (LLMs) applied to software engineering.

The goal is to understand how LLMs can be used for repository-level code understanding, code generation, debugging, program repair, refactoring, testing, and autonomous software engineering agents.

---

## Research Motivation

Large Language Models have demonstrated strong capabilities in code generation and understanding. However, real-world software engineering requires more than generating isolated code snippets.

Software engineering tasks often require an AI system to:

- Understand large codebases
- Retrieve relevant repository context
- Reason about existing implementations
- Modify multiple files
- Generate and execute tests
- Diagnose failures
- Iteratively repair code
- Use external tools
- Verify whether a proposed solution actually works

This research log explores the techniques and research directions that enable LLMs to move from simple code generation toward repository-aware software engineering agents.

---

## Research Questions

The research is organized around the following questions:

1. How do LLMs understand and generate code?
2. Why is repository-level context important for software engineering tasks?
3. How can retrieval improve code generation?
4. How can LLMs reason and act through tools?
5. How can coding agents debug and repair their own generated code?
6. How should AI software engineering agents be evaluated?
7. What limitations remain in current LLM-based software engineering systems?

---

## Research Areas

### 1. LLM Foundations

- Transformers
- Attention mechanisms
- Large Language Models
- Context windows
- Prompting
- Embeddings

### 2. Repository-Level Code Intelligence

- Code retrieval
- Repository understanding
- Retrieval-Augmented Generation
- Repository-level code completion
- Code search

### 3. AI Software Engineering Agents

- ReAct
- Tool calling
- Planning
- Agent loops
- Repository navigation
- Code editing
- Debugging
- Test execution

### 4. Software Engineering Applications

- Code generation
- Program repair
- Code refactoring
- Bug detection
- Software testing
- Code verification

### 5. Evaluation

- SWE-bench
- Code generation benchmarks
- Test-based evaluation
- Agent success rate
- Failure analysis

---

# Papers Studied

| Paper / Work | Area | Status |
|---|---|---|
| Attention Is All You Need | Transformers | ✅ Studied |
| RepoCoder | Repository-level code generation | 🔄 In Progress |
| ReAct | LLM reasoning and acting | ⏳ Planned |
| SWE-bench | Software engineering evaluation | ⏳ Planned |
| Code LLM Literature | Code generation | ⏳ Planned |
| Program Repair Literature | Automated program repair | ⏳ Planned |
| LLM Refactoring Literature | Code refactoring | ⏳ Planned |

---

# Research Log

Each paper is documented using a structured research log.

The logs focus on:

- Research problem
- Motivation
- Existing limitations
- Proposed approach
- Methodology
- Architecture
- Experimental setup
- Results
- Limitations
- Personal observations
- Implementation
- Experiments
- Connection to my research direction

---

# Implementations

Where practical, concepts from the literature are implemented as small experimental prototypes.

Current implementation areas include:

- Repository retrieval
- Embedding-based code search
- Retrieval-augmented code generation
- LLM-based coding agents
- Tool calling
- Automated testing
- Iterative debugging

Implementations are intended as simplified reproductions or experiments inspired by the studied literature and are not necessarily exact reproductions of the original research systems.

---

# Experiments

Experiments are maintained separately from the paper study.

The purpose of the experiments is to investigate questions such as:

- Does repository context improve code generation?
- Which retrieval strategies provide useful context?
- How does retrieval quality affect generated code?
- Can an LLM use tools effectively for software engineering?
- Can iterative execution and feedback improve solutions?
- What are the common failure modes of coding agents?

Experiment results will include observations, metrics, and failure analysis where applicable.

---

# Research Direction

The long-term goal of this work is to develop and study an AI Software Engineering Agent capable of interacting with real software repositories.

A simplified research direction is:

```text
LLM
 |
 +-- Repository Understanding
 |
 +-- Code Retrieval
 |
 +-- Reasoning
 |
 +-- Tool Use
 |
 +-- Code Generation
 |
 +-- Testing
 |
 +-- Debugging
 |
 +-- Verification
 |
 v
AI Software Engineering Agent
