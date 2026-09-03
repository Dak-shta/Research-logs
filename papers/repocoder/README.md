# RepoCoder

**Paper:** RepoCoder: Repository-Level Code Completion Through Iterative Retrieval and Generation  
**Authors:**  
**Year:**  
**Venue:**  
**Paper Link:** http://arxiv.org/pdf/2303.12570 

---

## 1. Problem

What problem does RepoCoder address?
-> For automated code completion tools, it is difficult to utilize the useful information scattered in
different files

Why is repository-level context important for code generation?
-> repository-level code completion, where automated tools are expected to utilize the broader context of a repository rather than
relying solely on in-file information to complete
unfinished code. Code files within a repository
often exhibit interrelated dependencies, including
shared utilities, configurations, and cross-API invocations resulting from modularization.

---

## 2. Key Idea

What is the main idea of RepoCoder?

Explain it in my own words.
-> • They propose RepoCoder, a novel iterative retrieval-generation framework for the
repository-level code completion task.
• They introduce the RepoEval benchmark, enabling the evaluation of repository-level code
completion with varying levels of granularity
and improved evaluation accuracy through the
utilization of unit tests.
• Through rigorous experimentation, we demonstrate that RepoCoder significantly outperforms the In-File code completion paradigm
and enhances the performance of vanilla
retrieval-augmented generation.
---

## 3. How It Works

Briefly explain the proposed approach.
-> The task of code completion using a language
model M can be generally described as Yˆ =
M(X), where Yˆ represents the predicted tokens
and X corresponds to the in-file unfinished code.
By introducing an additional code retrieval model
R, we can transform the code completion pipeline
into a Retrieval-Augmented Generation (RAG) approach. Initially, we establish a retrieval database
by partitioning the code files from the repository into a collection of code snippets Crepo =
{c1, c2, · · · }. 

-> However, using the unfinished code X as the
sole retrieval query introduces a gap between the
retrieval context and the intended completion target, as exemplified in Figure 2. To address this
limitation, we propose RepoCoder, an iterative
retrieval-generation pipeline designed to further enhance the performance of the vanilla RAG method

-> The
newly generated code completion can serve as either the output of RepoCoder or be utilized for the
subsequent retrieval-generation iteration.

```text
Repository
    ↓
Retrieve Relevant Code
    ↓
Provide Context to LLM
    ↓
Generate Code
    ↓
Iterative Retrieval + Generation