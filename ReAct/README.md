# ReAct: Synergizing Reasoning and Acting in Language Models

## 📌 Paper

**Title:** ReAct: Synergizing Reasoning and Acting in Language Models
**Authors:** Shunyu Yao et al.
**Conference:** ICLR 2023
**Paper:** https://arxiv.org/abs/2210.03629

---

## 🧠 What is ReAct?

ReAct is a framework that combines **reasoning (thinking)** and **acting (using tools/environment)** in an LLM.

Instead of only generating an answer, the model follows an iterative process:

```text
Thought → Action → Observation → Thought → Action → ...
```

The model can reason about what it needs, take an action such as searching a knowledge source, observe the result, and then use that information for its next decision.

---

## 🔍 Problem

LLMs can reason using methods such as **Chain-of-Thought (CoT)**, but they may hallucinate facts.

On the other hand, systems that only perform actions may lack an explicit reasoning process.

ReAct combines both:

> **Reasoning helps decide what action to take, while actions provide external information that improves reasoning.**

---

## ⚙️ How ReAct Works

Example:

```text
Question:
Who founded Company X?

Thought:
I need to find reliable information about Company X.

Action:
search[Company X]

Observation:
Company X was founded by Person Y.

Thought:
The information confirms that Person Y founded the company.

Action:
finish[Person Y]
```

The important part is the continuous interaction between:

* **Thought** → internal reasoning
* **Action** → interaction with an external tool/environment
* **Observation** → feedback from the environment

---

## 🆚 ReAct vs CoT vs Act

| Method    | Main idea                                       |
| --------- | ----------------------------------------------- |
| Standard  | Question → Answer                               |
| CoT       | Question → Reasoning → Answer                   |
| Act       | Question → Action → Observation                 |
| **ReAct** | **Thought → Action → Observation → Thought...** |

### Main trade-off

**CoT:** More flexible reasoning but can hallucinate facts.

**Act:** Can interact with external information but does not explicitly represent reasoning.

**ReAct:** Combines reasoning with external interaction, making the process more grounded and interpretable.

---

## 📊 Experiments

ReAct was evaluated on:

* **HotPotQA** – multi-hop question answering
* **FEVER** – fact verification
* **ALFWorld** – interactive decision making
* **WebShop** – web-based decision making

### Important observations

* ReAct performed better than CoT on **FEVER** (60.9 vs 56.3).
* CoT slightly outperformed ReAct on **HotPotQA** (29.4 vs 27.4).
* ReAct produced fewer hallucinations than CoT.
* Poor or irrelevant searches could cause ReAct to fail.
* ReAct could sometimes get stuck repeating the same actions.

---

## 🎯 Prompting vs Fine-tuning

One important distinction from the paper:

**Prompting** means giving examples inside the prompt without changing model weights.

```text
Few-shot examples
       ↓
     LLM
       ↓
    ReAct
```

**Fine-tuning** means training the model on ReAct trajectories so that it learns the behavior.

```text
3000 ReAct examples
        ↓
   Fine-tuning
        ↓
 ReAct-trained model
```

The paper found that ReAct became much stronger after fine-tuning.

Interestingly, a fine-tuned **PaLM-8B ReAct** model outperformed prompted larger models, while fine-tuned **PaLM-62B ReAct** outperformed prompted PaLM-540B methods.

---

## 💡 Key Research Insights

1. **Reasoning and acting complement each other.**
2. External information can reduce hallucination.
3. Reasoning helps an agent decide which action/tool to use.
4. ReAct creates interpretable decision trajectories.
5. Fine-tuning can make ReAct much more effective than few-shot prompting.
6. Retrieval quality is important — bad searches can derail the reasoning process.
7. ReAct provides a foundation for modern **tool-using LLM agents**.

---

## 🔗 Connection to AI Agents

ReAct is closely related to the architecture of modern AI agents:

```text
User Goal
   ↓
LLM Reasoning
   ↓
Choose Tool
   ↓
Tool Execution
   ↓
Tool Result
   ↓
LLM Reasoning
   ↓
Next Action
   ↺
```

This connects directly to concepts such as:

* Tool calling
* RAG
* Web agents
* Coding agents
* Autonomous agents
* LLM-based software engineering

---

## 📝 My Takeaway

> ReAct showed me that an LLM can be more useful when it does not try to answer everything directly. Instead, it can reason about what information it needs, interact with an external tool, observe the result, and continue reasoning. This idea is especially important for building reliable and tool-using AI agents.

---

## 🚀 Future Directions

The paper suggests several directions:

* More high-quality human ReAct trajectories
* Multi-task ReAct training
* Fine-tuning at larger scale
* Combining ReAct with reinforcement learning
* Incorporating human feedback

---

## 📚 Related Papers

* **Chain-of-Thought Prompting** – Wei et al., 2022
* **WebGPT** – Nakano et al., 2021
* **SayCan** – Ahn et al., 2022
* **Inner Monologue** – Huang et al., 2022
* **SWE-bench** – Jimenez et al., 2024

---

## ⭐ One-Line Summary

> **ReAct = Reason → Act → Observe → Reason again.**
