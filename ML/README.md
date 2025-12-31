# Machine Learning Practices

## Overview
This guide consolidates practices for developing, deploying, and maintaining production-grade Machine Learning systems. It covers the ML lifecycle, focusing on data hygiene, robust model governance, and modern Generative AI techniques.

We aren't just building models; we're building data products. The focus here is on reliability, scalability, and—most importantly—business impact. If a model is 1% more accurate but 10x more expensive to serve, it’s a failure.

## Table of Contents
1. [Foundational Best Practices](#foundational-best-practices)
2. [Advanced ML & Production Systems](#advanced-ml--production-systems)
3. [Generative AI & LLM Practices](#generative-ai--llm-practices)
4. [Industry Leader Strategies](#industry-leader-strategies)
5. [Common Pitfalls to Avoid](#common-pitfalls-to-avoid)
6. [Resources](#resources)

---

## Foundational Best Practices

### 1. Data Hygiene and Preparation
* **Data Quality Assessment (DQA):** Start with EDA, but don’t pretend it’s sufficient—pair it with *repeatable* checks for missingness, outliers, label weirdness, duplicates, and schema drift.

* The **Data First** Mindset: In a company, we say "Garbage In, Garbage Out." Don't just do EDA; implement Data Contract checks. If the upstream pipeline changes a unit from "cm" to "m," your model should fail loudly via an alert, not silently with bad predictions.

* **Privacy by Design**: This is missing from most guides. You must scrub PII (Personally Identifiable Information) before data hits your training bucket. GDPR/CCPA isn't optional.

* **Splitting:** Use non-overlapping **Train/Validation/Test** splits that match reality:
  * **Time-series:** time-based splits (no shuffling). If you use random splits on time-series data, you’re "predicting the past with the future," which is a recipe for fake high performance.
  * **User/item leakage risks:** group-based splits (e.g., by user_id).
  * **Classification:** use **Stratified Sampling** to preserve class ratios, but it’s not a free pass if leakage exists.

* **Versioning:** Treat data like code *and* like a product. Version datasets (DVC/LakeFS/Delta snapshots), record lineage (sources + transforms), and make “what exactly trained this model?” answerable in minutes.


### 2. Feature Engineering and Management
* **Feature Creation:** Features should encode domain signal *and* be stable under real production messiness. If you can’t compute it reliably online/offline, it’s probably not a feature—it’s a future incident.
* **Scaling:** Standardize/normalize when it actually matters (linear models, distance-based methods, some neural nets). For trees/boosting, scaling is often noise; I usually skip it unless there’s a clear reason.
* **Categorical Encoding:** 
  * Low-cardinality: one-hot is fine.
  * High-cardinality: embeddings or hashing are commonly robust; target encoding can be strong but is dangerously leak-prone—do it with proper cross-validation folds and strong leakage safeguards.
* **Feature Stores (Fundamental MLOps):** A feature store can be great when you have many consumers and repeated features. But I’d avoid forcing it early—teams often succeed with shared libraries + strong contracts first, then adopt Feast/Tecton once reuse and online/offline consistency become painful enough.


### 3. Model Training and Evaluation
* **Baseline First:** Always ship a baseline (simple model, rules, or even “do nothing”). Baselines clarify ROI and prevent overengineering.
* **Hyperparameter Tuning:** Tune systematically, but don’t worship any single method. Bayesian optimization is often sample-efficient, random search is shockingly competitive, and grid search is fine for tiny spaces or debugging.
* **Bias-Variance Trade-off:** Learning curves are genuinely useful. I also like slice-based evaluation (by region/device/user cohort) because production failures are usually *localized*, not global.
* **Metric Alignment:** Pick metrics that reflect the decision and costs:
  * **Classification:** ROC-AUC is good for ranking, PR-AUC is often better for rare positives; F1 is threshold-dependent (so choose and justify the threshold).
  * **Regression:** MAE/RMSE are fine, but *business loss* is better if you can define it (asymmetric costs, thresholds, penalties).
* **Error Analysis:** Do this early and often. I’m biased toward building a small “error review” workflow (top misses, confusing slices, annotation notes) because it tends to uncover data and labeling issues faster than model tweaks.


### 4. MLOps and Deployment Fundamentals
* **Experiment Tracking:** Use a dedicated system (MLflow, W&B) to log all hyperparameters, metrics, and associated artifacts for every run.
* **Model Registry:** Maintain a single source of truth for all models, tracking their version, stage (staging/production), and lineage.
* **API Design:** Models must be served behind a low-latency, resilient API endpoint (e.g., using TensorFlow Serving or Triton).
* **Model Card:** Create a document detailing the model's performance, intended use, limitations, and fairness metrics before deployment.

* **Experiment Tracking:** Use a dedicated system (MLflow, W&B) to log all hyperparameters, metrics, and associated artifacts for every run, but also log *data snapshot IDs*, feature definitions, and evaluation slices. Otherwise “reproducible experiments” is just a vibe.
* **Model Registry:** Maintain a single source of truth for all models, tracking their version, stage (staging/production), and lineage. Use a registry as the system of record for versions, artifacts, metadata, owners, and approval status. In practice, the biggest win is operational clarity during incidents.
* **API Design:** Serve behind a resilient interface with clear SLOs (latency/availability) and strict input contracts. TensorFlow Serving/Triton/TorchServe are fine—choose based on your stack, not trends.
* **Model Card:** Before deployment, create a honest and useful document detailing the model's performance, intended use, limitations, fairness metrics, key slices, data provenance, monitoring plan, and “when not to use this model.”

---

## Advanced ML & Production Systems

### 1. Advanced MLOps Lifecycle
* **CI/CD for ML (CI/CD/CT):** Implement CI for code/tests, CD for packaging/deploy, and *optionally* CT for automated retraining. I’m opinionated here: automated retraining should be gated by quality checks and rollout controls—blind retraining is a frequent source of regressions.
* **Retraining Triggers:** Drift can be a signal, but not always a reason to retrain. Prefer triggers like “fresh labeled data + performance drop on a trusted online metric” or “data distribution shift + confirmed impact.”

* **Reproducibility:** Use containers (Docker) and dependency management to guarantee that the deployed environment matches the training environment exactly, but true reproducibility also needs deterministic seeds (when possible), recorded hardware/software versions, and stable data snapshots.

### 2. System Robustness and Skew
* **Training-Serving Skew:** Rigorously test and monitor for differences in feature distribution between the data used for training and the data seen in production inference. Treat skew as inevitable. Add contract tests, replay tests (offline data through online feature code), and canary/shadow comparisons to catch drift and transform mismatches.
* **Batch vs Online Inference:** Batch is underrated and often cheaper/safer. Online inference is for genuinely interactive or time-sensitive use cases—otherwise batch + caching is usually the sane choice.
* **Distributed Training:** Implement frameworks (PyTorch DDP, TensorFlow Distribution Strategies) for large models that require multi-GPU or multi-node training when needed, but don’t scale training just because you can. Scaling without strong evaluation discipline just produces faster confusion.

### 3. Monitoring and Observability
* **Drift Detection:** Monitor alert on three types of drift:
  * **Data Drift:** input feature distribution changes (but be careful—drift alerts can be noisy).
  * **Label/Outcome Drift:** if you have delayed labels, track them.
  * **Concept Drift:**  The relationship between input features and the target variable has changed (e.g., consumer behavior shifts), harder to detect directly; usually inferred from performance decay or calibrated metrics.
* **Model Explainability (XAI):** SHAP/LIME can help for debugging and stakeholder trust, but don’t oversell them. For high-stakes decisions, I prefer simple, stable explanations plus rigorous policy and audit trails.
* **Shadow Mode:** Strong practice:  Run the new model passively in production alongside the old model, log deltas, comparing outputs before routing live traffic, then ramp with guarded rollouts (canary → small % → staged).

---

## Generative AI & LLM Practices

### 1. LLM Deployment Patterns
* **RAG (Retrieval-Augmented Generation):** Often the default *when you actually need factual grounding*. But it’s not magic: retrieval quality and chunking strategy usually matter more than the LLM choice. Vector DBs (Pinecone/Chroma/FAISS/etc.) are implementation details; the real “standard” is retrieval + ranking + grounding + citations.
* **Fine-Tuning (FT):** Use techniques like **LoRA/PEFT** when you need style, format, or domain behavior shifts. Don’t fine-tune to “memorize” facts that change—RAG is typically better for that.
* **Tool Use / Function Calling:** In many real products, the safest pattern is “LLM as router + tool caller,” with deterministic tools (search, calculators, DB queries) doing the truth work.
* **Evaluation for GenAI:** Offline accuracy metrics are not enough. Build eval suites: golden sets, adversarial prompts, regression tests, and human review for high-impact flows.


### 2. Safety and Security
* **Guardrails:** Use layered controls: input validation, policy checks, output constraints (schemas), and post-generation filters. Guardrails frameworks help, but you still need product-specific rules and monitoring.
* **Prompt Injection:** Assume you will be attacked. Isolate instructions, treat retrieved content as untrusted, apply allowlists for tools, and log/alert suspicious patterns.
* **Latencies & Cost:** LLM inference is expensive and can be slow. Practical knobs: smaller models for routing, speculative decoding where available, caching, batching, prompt compression, retrieval-first short-circuiting, and strict timeouts/fallbacks.
* **Privacy & Data Handling:** Be conservative: redact PII where possible, minimize retention, and clearly separate training data from user data unless you have explicit policy + consent.

---

## Industry Leader Strategies
### Google (TFX & Standardization)
* **TFX (TensorFlow Extended):** Strong emphasis on pipeline rigor (data validation, training, evaluation), and repeatability.
* **Vertex AI:** Centralized managed tooling is convenient, but I’d still treat it as “platform glue,” not a replacement for good engineering and governance.

### Meta (PyTorch & Speed)
* **PyTorch Ecosystem:** Fast iteration from research to production is a real advantage.
* **Serving:** TorchServe exists, but plenty of teams use Triton/custom stacks; what matters is operational maturity (rollouts, observability, cost control).
* **Experimentation Rigor:** Large-scale A/B testing culture is a genuine differentiator; offline wins that don’t move online metrics are treated as non-wins.

### Amazon (SageMaker & Cloud Native)
* **SageMaker End-to-End:** Managed services can accelerate delivery, especially with standardized pipelines and monitoring.
* **Cost Optimization:** Strong emphasis on cost/perf tradeoffs (spot, autoscaling, right-sizing, batching). In practice, cost is a first-class metric, not an afterthought.

### OpenAI (Frontier Models)
* **API-First Design:** Clear interfaces and developer UX matter a lot for adoption.
* **Scaling Laws Mindset:** Systematic scaling and evaluation discipline are key to frontier performance.
* **Safety Alignment:** Red-teaming, safety mitigations, and iterative post-deployment improvements are not optional at this scale.

---
## Common Pitfalls to Avoid
### Data & Model Pitfalls
* **Data Leakage:** Still the #1 killer. Leakage is often subtle (time, user overlap, label proxies, post-event features). Assume it exists until proven otherwise.
* **Ignoring Imbalance:** Raw accuracy is frequently misleading. Use PR-AUC, calibrated thresholds, and cost-aware evaluation.
* **Overfitting the Validation Set:** Repeated tuning on a single val set turns it into a soft test set. Use multiple folds, maintain a locked final test set, and prefer online validation when possible.
* **Label Quality Neglect:** Bad labels silently cap performance. I’m biased toward investing early in label audits and annotator guidelines.

### MLOps & Production Pitfalls
* **Offline Metrics Lie:** Offline metrics are necessary but not sufficient. If you can’t connect offline evaluation to online outcomes, you’re flying blind.
* **Silent Degradation:** Monitoring that only checks uptime is not monitoring. Track data health, latency, error rates, and performance proxies with alerts and owner on-call.
* **No Rollback Plan:** Rollback should be boring and fast. If rollback is a “big event,” you will hesitate during incidents and lose time.
* **No Capacity Planning:** Latency and cost regressions happen at scale. Load test, set budgets, and enforce performance gates before ramping traffic.

---

## Resources

### Core MLOps & Frameworks
* **MLflow:** Experiment tracking and Model Registry.
* **Weights & Biases (W&B):** Advanced visualization and hyperparameter search.
* **TensorFlow/PyTorch:** Primary deep learning frameworks.
* **Scikit-learn:** Core classical ML algorithms.

### Industry Reading
* **Google's "Rules of Machine Learning":** The foundational 43 rules for production ML.
* ***Designing Machine Learning Systems*** by Chip Huyen.
* ***The MLOps Book*** by Mark Treveil et al.

### Conferences
* **NeurIPS, ICML, ICLR:** Core academic research.
* **KDD, AAAI:** Applied ML and Data Mining.
* **MLOps World:** Focus on production and engineering practices.