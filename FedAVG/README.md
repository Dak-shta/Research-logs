# Communication-Efficient Learning of Deep Networks from Decentralized Data

> **Paper:** Communication-Efficient Learning of Deep Networks from Decentralized Data
> **Authors:** H. Brendan McMahan, Eider Moore, Daniel Ramage, Seth Hampson, Blaise Agüera y Arcas
> **Published:** AISTATS 2017
> **Topic:** Federated Learning, Distributed Optimization, Privacy-Preserving Machine Learning

---

## 📌 Overview

This repository contains my study notes and technical observations from the paper **"Communication-Efficient Learning of Deep Networks from Decentralized Data"**, which introduced the **Federated Averaging (FedAvg)** algorithm.

The paper addresses the problem of training machine learning models when training data is distributed across a large number of devices, while minimizing the amount of communication between devices and the central server.

The key idea is:

> Instead of sending every local gradient update to the server, clients perform multiple steps of local training and the server averages their resulting model parameters.

This significantly reduces the number of communication rounds required for training.

---

## 🎯 Problem

Traditional machine learning generally assumes that training data can be collected in a centralized location:

```text
Users
  ↓
Centralized Dataset
  ↓
Training Server
  ↓
Global Model
```

However, in many real-world applications, data is naturally distributed across users' devices.

Examples include:

* Mobile keyboard data
* User-generated text
* Images stored on devices
* Sensor data
* Personal activity data

Moving all this data to a central server creates problems involving:

* Privacy
* Communication cost
* Network availability
* Storage
* Large numbers of participating devices

The paper therefore investigates:

> **How can deep learning models be trained efficiently when data remains decentralized?**

---

# 🔐 Federated Learning

Federated Learning allows multiple clients to collaboratively train a shared model without directly sending their raw training data to a central server.

```text
             Global Model
                  ↓
        ┌─────────┼─────────┐
        ↓         ↓         ↓
     Client 1  Client 2  Client 3
        ↓         ↓         ↓
     Local      Local      Local
    Training   Training   Training
        ↓         ↓         ↓
      Update    Update    Update
        └─────────┼─────────┘
                  ↓
             Aggregation
                  ↓
            Global Model
```

The data remains on the clients while model parameters or updates are communicated.

---

# 🚀 Federated Averaging (FedAvg)

The central contribution of the paper is the **FederatedAveraging algorithm**.

FedAvg combines:

1. Local Stochastic Gradient Descent (SGD)
2. Multiple local training steps
3. Weighted averaging of client models

### Basic workflow

```text
1. Server initializes global model
              ↓
2. Select a fraction of clients
              ↓
3. Send global model to clients
              ↓
4. Clients train locally
              ↓
5. Clients send updated models
              ↓
6. Server performs weighted averaging
              ↓
7. New global model
              ↓
8. Repeat
```

---

# 🧮 FedAvg Algorithm

Let:

* \(K\) = total number of clients
* \(C\) = fraction of clients selected in a round
* \(n_k\) = number of training examples on client \(k\)
* \(n = \sum_k n_k\)
* \(E\) = number of local epochs
* \(B\) = local minibatch size
* \(w_t\) = global model at round \(t\)

Each selected client starts from the same global model:

$$
w_k \leftarrow w_t
$$

The client then performs local SGD:

$$
w_k \leftarrow w_k-\eta\nabla F_k(w_k)
$$

for multiple local updates.

The server aggregates the resulting models:

$$
\boxed{
w_{t+1}
=
\sum_{k=1}^{K}
\frac{n_k}{n}w_k
}
$$

Therefore, clients with more training data have proportionally greater influence on the global model.

---

# 🔄 FedSGD vs FedAvg

FedSGD performs essentially one local gradient computation before communicating with the server.

FedAvg allows clients to perform **multiple local updates before communication**.

### FedSGD

```text
Client
  ↓
One gradient step
  ↓
Server
  ↓
Aggregation
  ↓
Repeat
```

### FedAvg

```text
Client
  ↓
Multiple local SGD steps
  ↓
Updated local model
  ↓
Server
  ↓
Model averaging
  ↓
Repeat
```

This reduces the amount of communication required.

---

# ⚙️ Important Hyperparameters

## 1. C — Client Fraction

The fraction of clients participating in each communication round.

Higher \(C\):

* More clients contribute
* More computation per round
* Potentially better convergence

Lower \(C\):

* Less communication
* Less computation
* More practical for massively distributed systems

---

## 2. E — Number of Local Epochs

Controls how much local training each client performs before communicating.

Higher \(E\):

* More local computation
* Fewer communication rounds

However, excessively large \(E\) can cause **client drift**, especially when data is non-IID.

---

## 3. B — Minibatch Size

Controls the number of local examples used per SGD update.

Smaller \(B\):

* More local updates
* More computation
* Potentially fewer communication rounds

The paper also considers:

$$
B=\infty
$$

which means the entire local dataset is treated as one batch.

---

# 📊 Key Experimental Findings

The authors evaluate FedAvg on several tasks, including:

* MNIST
* Shakespeare
* CIFAR-10
* Large-scale language modeling

The experiments compare FedAvg against FedSGD and investigate IID and non-IID data distributions.

---

## 🧪 MNIST

The authors construct both:

### IID setting

Each client receives a roughly representative distribution of digits.

### Non-IID setting

The dataset is sorted by digit and divided into shards, with each client receiving only a small number of shards.

As a result, individual clients may see only a few digit classes.

This represents a more realistic federated scenario where:

> **Different users have different data distributions.**

---

# 📈 Major Result

One of the most important findings is that increasing local computation can dramatically reduce communication rounds.

For example, on the MNIST CNN experiment:

| Setting            | IID Rounds | Communication Efficiency |
| ------------------ | ---------: | -----------------------: |
| FedSGD             |        626 |                 Baseline |
| FedAvg, E=5, B=∞   |        179 |                    ~3.5× |
| FedAvg, E=1, B=10  |         65 |                    ~9.6× |
| FedAvg, E=20, B=10 |         18 |                   ~34.8× |

The important takeaway is not simply that FedAvg is "faster."

It is that:

> **More computation can be performed locally to reduce expensive communication between clients and the server.**

---

# ⚠️ The Non-IID Challenge

Federated datasets are often **non-IID**.

For example:

```text
Client A → mostly cats
Client B → mostly dogs
Client C → mostly birds
```

Each client therefore learns a different local objective.

If clients perform too much local training, their models can move in different directions.

This is known as **client drift**.

```text
                Global Model
                     |
          ┌──────────┼──────────┐
          ↓          ↓          ↓
       Client A   Client B   Client C
          ↓          ↓          ↓
        Local      Local      Local
        Model      Model      Model
          ↘          ↓          ↙
              Aggregation
                   ↓
             Global Model
```

Therefore:

$$
\text{More local computation}
$$

does not always mean

$$
\text{better convergence}
$$

especially for highly non-IID data.

---

# 🖼️ CIFAR-10 Findings

The experiments also show a substantial reduction in communication rounds on CIFAR-10.

FedAvg achieved approximately **85% accuracy in around 2,000 communication rounds**, while standard centralized SGD required approximately **197,500 minibatch updates** for comparable performance.

This demonstrates the central advantage of federated optimization:

$$
\boxed{\text{Reduce communication while maintaining model quality}}
$$

---

# 🌐 Large-Scale Federated Learning

The paper also studies a very large-scale language modeling problem involving:

* More than 500,000 clients
* Millions of words
* LSTM-based language model
* Only a subset of clients participating per round

FedAvg required substantially fewer communication rounds than FedSGD.

The experiment demonstrates that the algorithm can scale to scenarios with extremely large numbers of clients.

---

# 🔒 Privacy Discussion

An important observation in the paper is:

> **Federated Learning does not automatically guarantee privacy.**

Although raw data remains on the client, model updates can potentially contain information about the underlying training data.

For example, if a model uses sparse representations such as bag-of-words, gradients could potentially reveal information about which words appeared in a user's data.

Therefore:

```text
Federated Learning
       ↓
Raw data stays local
       ↓
BUT
       ↓
Model updates may leak information
```

The paper discusses stronger privacy mechanisms such as:

* Secure Multiparty Computation
* Secure Aggregation
* Differential Privacy

---

# 🛡️ Differential Privacy

Differential Privacy provides a mathematical framework for limiting how much information about an individual can be inferred from released information.

A simplified privacy-preserving workflow is:

```text
Local Data
    ↓
Local Training
    ↓
Gradient / Model Update
    ↓
Clip Contribution
    ↓
Add Noise
    ↓
Private Update
    ↓
Server
```

The goal is to make it difficult to determine whether a particular individual's data contributed to the result.

A key privacy parameter is:

$$
\epsilon
$$

Generally:

$$
\boxed{\text{smaller }\epsilon \rightarrow \text{stronger privacy}}
$$

although stronger privacy can reduce model utility.

---

# 🔑 Key Insights

### 1. Communication is the main bottleneck

In federated systems, computation can often be performed locally, but communication is expensive.

Therefore:

$$
\boxed{\text{More local computation} \rightarrow \text{less communication}}
$$

---

### 2. Local training is extremely important

FedAvg demonstrates that clients can perform multiple SGD updates before communicating.

This is the main difference from FedSGD.

---

### 3. Non-IID data is a major challenge

Different clients may have very different data distributions.

Too much local training can cause client models to diverge.

---

### 4. Model averaging can work surprisingly well

Although averaging independently trained neural networks is not generally guaranteed to work, FedAvg starts every client from the same global model.

This makes parameter averaging much more effective.

---

### 5. Federated Learning ≠ Complete Privacy

Keeping raw data local reduces centralized data exposure, but model updates themselves can leak information.

Additional privacy mechanisms may therefore be necessary.

---

# 💡 My Research Observations

### Observation 1 — Communication vs Computation

FedAvg changes the optimization trade-off:

$$
\text{Communication} \leftrightarrow \text{Local Computation}
$$

Instead of communicating after every gradient step, clients perform more computation locally.

This is particularly valuable for mobile and edge devices.

---

### Observation 2 — Data Heterogeneity

The effectiveness of FedAvg depends strongly on the distribution of client data.

This suggests an important research question:

> **How can federated optimization remain stable when client data distributions are highly heterogeneous?**

---

### Observation 3 — Privacy is a separate problem

The paper helped establish an important distinction:

$$
\boxed{
\text{Decentralized Data}
\neq
\text{Guaranteed Privacy}
}
$$

This creates a connection between Federated Learning and privacy research.

---

### Observation 4 — Connection to Modern AI

The ideas introduced in this paper form the foundation for many later areas of research involving:

* Federated optimization
* Privacy-preserving ML
* Secure aggregation
* Differential privacy
* Personalized federated learning
* Communication-efficient distributed learning

---

# 🔗 Connection to My Research Interests

This paper is particularly relevant to research in **Privacy in Machine Learning**.

The conceptual pipeline is:

```text
Federated Learning
       ↓
Model Updates
       ↓
Potential Information Leakage
       ↓
Privacy Attacks
       ↓
Membership Inference
       ↓
Model Inversion / Memorization
       ↓
Defenses
       ↓
Differential Privacy
       +
Secure Aggregation
```

This provides a foundation for understanding privacy risks in modern distributed ML systems.

---

# 📚 Related Papers / Topics to Study

### Foundational

* Federated Averaging — McMahan et al.
* Differential Privacy
* Secure Aggregation
* Distributed SGD

### Privacy Attacks

* Membership Inference Attacks
* Model Inversion
* Gradient Leakage
* Data Reconstruction from Gradients

### Further Federated Learning

* Personalized Federated Learning
* Federated Optimization
* FedProx
* FedNova
* SCAFFOLD

---

# 🧠 What I Learned

After studying this paper, I understood:

* Why Federated Learning is needed
* How FedAvg works
* How FedAvg differs from FedSGD
* Why communication is the major bottleneck
* The roles of \(C\), \(E\), and \(B\)
* Why non-IID data makes federated optimization difficult
* What client drift means
* Why model updates can still leak information
* Why Federated Learning alone does not guarantee privacy
* How Differential Privacy and Secure Aggregation can strengthen privacy

---

# ⭐ Final Takeaway

The core contribution of the paper can be summarized as:

$$
\boxed{
\text{Local Training}
+
\text{Model Averaging}
=
\text{Communication-Efficient Federated Learning}
}
$$

FedAvg showed that instead of communicating after every local optimization step, distributed clients can perform multiple local updates and periodically average their models.

This simple idea dramatically reduces communication requirements while maintaining strong model performance, making federated learning much more practical for decentralized and large-scale machine learning.

---

## 🏷️ Topics

`Federated Learning` `FedAvg` `Distributed Learning` `SGD` `Deep Learning` `Privacy-Preserving ML` `Differential Privacy` `Secure Aggregation` `Non-IID Data` `Communication Efficiency` `Machine Learning Research`

---

## 📖 Paper

**McMahan, H. B., Moore, E., Ramage, D., Hampson, S., & Arcas, B. A. (2017).**
*Communication-Efficient Learning of Deep Networks from Decentralized Data.*

AISTATS 2017.
