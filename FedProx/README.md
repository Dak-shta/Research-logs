# FedProx: Federated Optimization in Heterogeneous Networks

> **Paper:** *Federated Optimization in Heterogeneous Networks*
> **Authors:** Tian Li, Anit Kumar Sahu, Maziar Zaheer, Maziar Sanjabi, Ameet Talwalkar, Virginia Smith
> **Venue:** MLSys 2020
> **Paper:** FedProx
> **Research Area:** Federated Learning, Distributed Optimization, Privacy-Preserving Machine Learning

---

## 1. Overview

Federated Learning (FL) enables multiple devices or organizations to collaboratively train a machine learning model without directly sharing their local training data.

A major challenge in real-world federated learning is **heterogeneity**.

Different devices may:

* Have different amounts and distributions of data.
* Have different computational capabilities.
* Have different network conditions.
* Perform different amounts of local computation.
* Produce local models that move in different directions.

The paper introduces **FedProx**, a generalized federated optimization framework designed to address both:

1. **Statistical heterogeneity** — differences in local data distributions.
2. **Systems heterogeneity** — differences in device capabilities and available computation.

The key idea is to combine:

* **Variable amounts of local work**, and
* A **proximal regularization term** that prevents local models from drifting too far from the global model.

---

# 2. Motivation

Traditional FedAvg assumes that participating clients perform a fixed amount of local computation.

For example:

```text
Server
   |
   v
Global Model
   |
   +--------+--------+--------+
   |        |        |        |
 Client A Client B Client C Client D
   |        |        |        |
  20       20       20       20
 epochs   epochs   epochs   epochs
   |        |        |        |
   +--------+--------+--------+
            |
            v
        Aggregation
```

This assumption is unrealistic in real federated networks.

A slow device may only be able to perform:

```text
Client A → 20 epochs
Client B → 15 epochs
Client C → 7 epochs
Client D → 3 epochs
```

Additionally, clients may have very different local datasets:

```text
Client A → mostly cats
Client B → mostly dogs
Client C → mostly cars
Client D → mostly airplanes
```

This causes **client drift**, where local models move toward different local objectives.

FedProx was proposed to make federated optimization more robust to these conditions.

---

# 3. Federated Learning Objective

The global optimization problem is:

$$
\min_w f(w)=\sum_{k=1}^{N}p_kF_k(w)
$$

where:

* \(w\) = global model parameters
* \(N\) = number of devices
* \(F_k(w)\) = local objective of device \(k\)
* \(p_k\) = weight associated with device \(k\)

The local objective is:

$$
F_k(w)=E_{x_k\sim D_k}[f_k(w;x_k)]
$$

where:

* \(D_k\) = local data distribution of client \(k\)
* \(f_k(w;x_k)\) = loss for an individual local example

Typically:

$$
p_k=\frac{n_k}{n}
$$

where \(n_k\) is the number of samples on device \(k\), and \(n\) is the total number of samples.

---

# 4. Two Types of Heterogeneity

## 4.1 Statistical Heterogeneity

Statistical heterogeneity occurs when clients have different data distributions:

$$
D_1\neq D_2\neq \cdots \neq D_N
$$

Consequently:

$$
F_1(w)\neq F_2(w)\neq \cdots \neq F_N(w)
$$

and their gradients may point in different directions.

This can cause **client drift**.

### Example

```text
Client 1 → {cats, cats, cats}
Client 2 → {dogs, dogs, dogs}
Client 3 → {cars, cars, cars}
```

Each client learns a different local representation.

---

## 4.2 Systems Heterogeneity

Systems heterogeneity occurs because devices differ in:

* CPU
* Memory
* Battery
* Network speed
* Storage
* Availability
* Computational capacity

Therefore, requiring every device to perform exactly the same amount of computation can lead to **stragglers**.

---

# 5. FedAvg

FedProx generalizes the popular FedAvg approach.

In FedAvg, the server:

1. Selects a subset of clients.
2. Sends the global model.
3. Each client performs a fixed amount of local SGD.
4. Clients send their updated models.
5. The server aggregates the updates.

Simplified workflow:

```text
              Server
                 |
          Global Model wt
                 |
       +---------+---------+
       |         |         |
       v         v         v
    Client 1  Client 2  Client 3
       |         |         |
    Local SGD Local SGD Local SGD
       |         |         |
       +---------+---------+
                 |
                 v
             Aggregation
                 |
                 v
              wt + 1
```

However, FedAvg can struggle with:

* Non-IID data
* Large numbers of local updates
* Stragglers
* Different device capabilities

---

# 6. FedProx

FedProx modifies the local optimization problem.

Instead of minimizing only:

$$
F_k(w)
$$

FedProx minimizes:

$$
\boxed{
h_k(w;w_t)
=
F_k(w)
+
\frac{\mu}{2}\|w-w_t\|^2
}
$$

where:

* \(w_t\) = current global model
* \(w\) = local model
* \(\mu\) = proximal coefficient

---

# 7. Intuition Behind the Proximal Term

The additional term:

$$
\frac{\mu}{2}\|w-w_t\|^2
$$

penalizes a local model for moving too far from the global model.

Think of it as a **rubber band**:

```text
                Local objective
                     ↓
Global Model ● ---------------------> Local Model
             \______________________/
                    penalty
```

Without the proximal term:

```text
Global Model
     |
     +-----------------------------> Local Model
                    large drift
```

With the proximal term:

```text
Global Model
     |
     +-----------> Local Model
             restricted drift
```

Therefore, the proximal term helps control **client drift**.

---

# 8. Role of μ

The parameter \(\mu\) controls the strength of the proximal regularization.

### \(\mu=0\)

$$
h_k(w;w_t)=F_k(w)
$$

The proximal term disappears.

FedProx therefore becomes FedAvg-like when using SGD with fixed local work.

$$
\boxed{
FedAvg \approx FedProx(\mu=0)
}
$$

### Small \(\mu\)

The local model has more freedom to move.

### Large \(\mu\)

The local model is strongly encouraged to remain close to the global model.

Therefore:

$$
\boxed{
\mu\uparrow
\Rightarrow
\text{less local drift}
}
$$

but excessively large \(\mu\) can also slow learning.

---

# 9. Inexact Local Solutions

FedProx does not require every client to solve its local optimization problem exactly.

The paper introduces a \(\gamma\)-inexact solution.

For:

$$
h(w;w_0)
=
F(w)+\frac{\mu}{2}\|w-w_0\|^2
$$

a solution \(w^*\) is considered \(\gamma\)-inexact if:

$$
\boxed{
\|\nabla h(w^*;w_0)\|
\leq
\gamma
\|\nabla h(w_0;w_0)\|
}
$$

where:

$$
0\leq\gamma\leq1
$$

Smaller \(\gamma\) means a more accurate local solution.

---

# 10. Variable Local Work

In real federated systems, different devices may perform different amounts of computation.

Therefore:

$$
\gamma_1^t,\gamma_2^t,\ldots,\gamma_K^t
$$

can vary across:

* clients
* communication rounds

This allows FedProx to model systems heterogeneity.

Example:

```text
Round t

Client A → 20 epochs → high local accuracy
Client B → 10 epochs
Client C → 5 epochs
Client D → 2 epochs → partial solution
```

FedProx can incorporate these partial solutions instead of simply discarding them.

---

# 11. FedAvg vs FedProx

| Feature                   | FedAvg                               | FedProx                             |
| ------------------------- | ------------------------------------ | ----------------------------------- |
| Local objective           | \(F_k(w)\)                           | \(F_k(w)+\frac{\mu}{2}\|w-w_t\|^2\) |
| Fixed local work          | Typically                            | Not required                        |
| Variable local work       | Limited                              | Supported                           |
| Partial solutions         | Generally dropped in straggler setup | Can be incorporated                 |
| Client drift control      | No explicit mechanism                | Proximal term                       |
| Statistical heterogeneity | Can cause instability                | Better controlled                   |
| Systems heterogeneity     | Sensitive to stragglers              | Better tolerated                    |
| Local solver              | SGD                                  | Generalized local solver            |
| \(\mu=0\)                 | —                                    | FedAvg-like                         |

---

# 12. Convergence Analysis

The paper provides convergence guarantees under heterogeneous conditions.

A central result has the form:

$$
\boxed{
E[f(w^{t+1})]
\leq
f(w^t)
-
\rho^t
\|\nabla f(w^t)\|^2
}
$$

If:

$$
\rho^t>0
$$

then the expected global objective decreases.

In other words:

$$
\boxed{
\text{Expected global loss decreases over training rounds}
}
$$

The analysis accounts for:

* Partial local solutions
* Stochastic device participation
* Non-IID data
* Non-convex objectives
* Systems heterogeneity

---

# 13. Bounded Dissimilarity

A key theoretical concept is **device dissimilarity**.

The paper studies how different local gradients are from the global gradient.

One useful empirical measurement is:

$$
\boxed{
E_k
\left[
\|\nabla F_k(w)-\nabla f(w)\|^2
\right]
}
$$

Interpretation:

### Low gradient variance

```text
Client gradients

↗
 ↗
  ↗
   ↗

Global gradient ↗
```

Clients mostly agree.

### High gradient variance

```text
Client 1 → ↗
Client 2 → ←
Client 3 → ↓
Client 4 → ↘
```

Clients disagree strongly.

Therefore:

$$
\boxed{
\text{Higher gradient dissimilarity}
\Rightarrow
\text{greater statistical heterogeneity}
}
$$

and generally:

$$
\boxed{
\text{greater heterogeneity}
\Rightarrow
\text{harder convergence}
}
$$

---

# 14. Experimental Setup

The authors evaluate FedProx on both synthetic and real federated datasets.

## Synthetic Data

They control statistical heterogeneity using parameters:

$$
\alpha,\beta
$$

### \(\alpha\)

Controls how much the local models differ.

$$
\boxed{\alpha\rightarrow\text{model heterogeneity}}
$$

### \(\beta\)

Controls how much local data distributions differ.

$$
\boxed{\beta\rightarrow\text{data heterogeneity}}
$$

They create:

* Synthetic-IID
* Synthetic (0,0)
* Synthetic (0.5,0.5)
* Synthetic (1,1)

with increasing heterogeneity.

---

# 15. Real Federated Datasets

The paper evaluates four real datasets.

| Dataset      | Devices | Samples | Task                      | Model               |
| ------------ | ------: | ------: | ------------------------- | ------------------- |
| MNIST        |   1,000 |  69,035 | Digit classification      | Logistic Regression |
| FEMNIST      |     200 |  18,345 | Character classification  | Logistic Regression |
| Shakespeare  |     143 | 517,106 | Next-character prediction | RNN/LSTM            |
| Sentiment140 |     772 |  40,783 | Sentiment analysis        | LSTM                |

The experiments therefore cover both:

* Convex optimization
* Non-convex optimization

---

# 16. Systems Heterogeneity Experiment

The authors simulate devices with different computational capabilities.

They set:

$$
E=20
$$

as the maximum local work.

Some devices are forced to perform fewer epochs.

They investigate:

* 0% stragglers
* 50% stragglers
* 90% stragglers

### FedAvg

Incomplete clients are dropped.

### FedProx

Partial updates are incorporated.

Result:

$$
\boxed{
\text{FedProx is more stable under systems heterogeneity}
}
$$

In highly heterogeneous environments, FedProx significantly improves convergence compared with FedAvg.

---

# 17. Statistical Heterogeneity Experiment

The authors remove systems heterogeneity by forcing every device to perform the same amount of local work.

They then increase statistical heterogeneity.

The observed trend is:

$$
\boxed{
\text{Statistical heterogeneity}\uparrow
\Rightarrow
\text{FedAvg convergence}\downarrow
}
$$

Adding a suitable proximal term:

$$
\mu>0
$$

helps stabilize convergence.

---

# 18. Effect of μ

The authors compare:

$$
\mu=0
$$

with appropriately tuned:

$$
\mu>0
$$

They observe that a suitable proximal coefficient can:

* Improve stability
* Prevent divergence
* Improve convergence
* Improve testing accuracy in many settings

In highly heterogeneous environments with 90% stragglers, the paper reports an average **22% improvement in absolute testing accuracy** relative to FedAvg across the reported experiments.

---

# 19. Adaptive μ

Selecting the best \(\mu\) analytically can be difficult.

The authors therefore experiment with a simple adaptive heuristic:

```text
If loss increases:
        μ ← μ + 0.1

If loss decreases for 5 consecutive rounds:
        μ ← μ - 0.1
```

The experiment shows that this simple adaptive strategy can work well empirically.

However, the authors identify automatic tuning of \(\mu\) as an area for future research.

---

# 20. Main Findings

The paper's experimental results can be summarized as:

### Finding 1

Increasing statistical heterogeneity makes FedAvg convergence worse.

### Finding 2

The proximal term becomes especially useful under high statistical heterogeneity.

### Finding 3

Allowing partial local work improves robustness to systems heterogeneity.

### Finding 4

A suitable \(\mu\) can stabilize methods that otherwise diverge.

### Finding 5

Gradient dissimilarity correlates with convergence behavior.

### Finding 6

FedProx is not theoretically guaranteed to outperform classical distributed SGD in every setting.

---

# 21. Limitations

The paper itself is careful about its claims.

FedProx does **not** guarantee that:

$$
FedProx > FedAvg
$$

in every situation.

It is also not guaranteed to outperform distributed SGD.

Theoretical convergence conditions are:

$$
\boxed{\text{sufficient, not necessary}}
$$

The optimal \(\mu\) is also difficult to determine automatically.

Additionally, the theoretical analysis relies on assumptions such as bounded device dissimilarity.

---

# 22. Key Parameters

| Parameter      | Meaning                                          |
| -------------- | ------------------------------------------------ |
| \(w_t\)        | Global model at round \(t\)                      |
| \(F_k(w)\)     | Local objective of client \(k\)                  |
| \(f(w)\)       | Global objective                                 |
| \(p_k\)        | Client aggregation weight                        |
| \(E\)          | Local computation/epochs                         |
| \(\mu\)        | Proximal regularization strength                 |
| \(\gamma_k^t\) | Local solution inexactness                       |
| \(B\)          | Device/statistical dissimilarity measure         |
| \(K\)          | Number of selected clients                       |
| \(\rho^t\)     | Quantity controlling expected objective decrease |

---

# 23. Complete FedProx Workflow

```text
                    Server
                      |
                      | Global model wt
                      ↓
          +-----------+-----------+
          |           |           |
          ↓           ↓           ↓
       Client 1    Client 2    Client K
          |           |           |
          | Different local data  |
          | Different resources  |
          ↓           ↓           ↓
      Local optimization of
      Fk(w) + μ/2 ||w - wt||²
          |           |           |
          |           |           |
       Variable amounts of work
          |           |           |
          +-----------+-----------+
                      |
                      ↓
              Partial/full updates
                      |
                      ↓
                  Aggregation
                      |
                      ↓
                   wt+1
```

---

# 24. FedProx in One Equation

The central idea of the paper can be represented by:

$$
\boxed{
\underbrace{F_k(w)}_{\text{learn from local data}}
+
\underbrace{
\frac{\mu}{2}\|w-w_t\|^2
}_{\text{prevent excessive drift}}
}
$$

while allowing:

$$
\boxed{
\text{different clients}
\rightarrow
\text{different amounts of local work}
}
$$

---

# 25. Personal Research Takeaways

### What I learned

1. Federated learning has both **statistical** and **systems** heterogeneity.
2. FedAvg can suffer from client drift when local data is non-IID.
3. Fixed local epochs are problematic when device capabilities differ.
4. FedProx introduces a proximal term to keep local models closer to the global model.
5. FedProx allows clients to perform variable amounts of local computation.
6. The parameter \(\mu\) controls the trade-off between local adaptation and global consistency.
7. \(\gamma_k^t\) provides a way to model varying local solution accuracy.
8. Gradient dissimilarity provides an empirical indication of statistical heterogeneity.
9. FedProx's benefits become particularly visible in highly heterogeneous environments.
10. The paper does not claim universal superiority; its contribution is a more general and robust optimization framework for heterogeneous federated networks.

---

# 26. Research Questions Inspired by the Paper

The paper also raises several possible research directions:

### Adaptive Proximal Regularization

Can \(\mu\) be automatically determined from the current degree of client heterogeneity?

$$
\mu_t=f(\text{gradient dissimilarity},\text{loss},\text{client drift})
$$

### Client-Specific μ

Instead of:

$$
\mu_1=\mu_2=\cdots=\mu_N
$$

could we use:

$$
\mu_1\neq\mu_2\neq\cdots\neq\mu_N
$$

based on device capability or data heterogeneity?

### Dynamic Local Computation

Can the number of local epochs be dynamically determined using:

* device resources,
* gradient variance,
* model convergence,
* communication cost?

### Heterogeneity-Aware Client Selection

Can the server select clients based on their contribution to reducing global heterogeneity rather than selecting them randomly?

---

# 27. My Overall Understanding

FedProx can be viewed as a generalization of FedAvg designed for realistic federated environments.

The core insight is:

> **Clients should be allowed to behave differently because real federated devices and datasets are heterogeneous, but local updates should be controlled so that excessive client drift does not destabilize global training.**

This is achieved through two mechanisms:

$$
\boxed{
\text{Variable local work}
+
\text{Proximal regularization}
}
$$

Therefore:

$$
\boxed{
FedProx =
\text{FedAvg-like aggregation}
+
\text{partial-work tolerance}
+
\text{proximal regularization}
}
$$

---

## Paper Status

* [x] Problem understanding
* [x] FedAvg background
* [x] Statistical heterogeneity
* [x] Systems heterogeneity
* [x] FedProx objective
* [x] Proximal term
* [x] Inexact local solutions
* [x] Variable local work
* [x] Convergence analysis
* [x] Experimental results
* [x] Limitations
* [x] Research questions

**Overall takeaway:** FedProx provides a practical and theoretically motivated framework for federated learning under heterogeneous data and heterogeneous device capabilities.
