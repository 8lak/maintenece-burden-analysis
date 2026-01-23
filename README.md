### Repo: `maintenance-burden-analysis`

```markdown
# C vs. Rust: Quantitative Maintenance Burden Analysis

> **Status:** Completed (Fall 2025)
> **Stack:** Python, Google Cloud Platform (Vertex AI, BigQuery), Gemini 2.0 Flash Lite
> **Focus:** MLOps, Static Analysis, Cloud FinOps, Statistical Research

## TL;DR
A large-scale comparative analysis of **182,746 commits** across 25 years of open-source history. This project quantifies the economic "maintenance burden" difference between C and Rust to inform critical infrastructure migration decisions (specifically regarding `libxml2`).

**Key Result:** Demonstrated that C projects pay a **~15% "Memory Tax"** (maintenance effort spent fixing crashes), whereas functionally equivalent Rust projects spend that time on Feature Development.

---

## 1. The Research Question
**Context:** `libxml2` is the standard XML parser for the internet (Linux, Android, macOS). It is written in C and suffers from memory safety vulnerabilities.
**The Question:** Does migrating to Rust actually reduce maintenance effort, or does it just shift the complexity elsewhere?
**The Hypothesis:** 
*   **C** pays a *Runtime Tax*: High frequency of simple fixes for dangerous bugs (Segfaults).
*   **Rust** pays a *Compile-Time Tax*: High frequency of complex fixes to satisfy the compiler (Borrow Checker), but near-zero memory bugs.

## 2. Methodology: "Apples-to-Apples"
To ensure scientific validity, I analyzed 5 matched pairs of libraries serving identical functions:

| Domain | C Representative | Rust Representative |
|:---|:---|:---|
| **XML Parsing** | `libxml2` | `quick-xml` |
| **HTTP/Networking** | `libcurl` | `hyper` |
| **Cryptography/TLS** | `openssl` | `rustls` |
| **Database** | `sqlite` | `limbo` |
| **System Utils** | `coreutils` (GNU) | `coreutils` (uutils) |

---

## 3. The Engineering Pipeline (MLOps)

I engineered a custom ETL pipeline to classify human intent from code diffs, addressing key challenges in data quality and model reliability.

### A. The "Smart Truncation" Algorithm (The Accordion)
Raw git diffs are noisy and token-expensive. I wrote a custom parser using `unidiff` that:
1.  **Strips Artifacts:** Removes auto-generated files, assets, and lockfiles.
2.  **Compresses Context:** Keeps the "Head" and "Tail" of function changes while truncating the repetitive middle.
3.  **Result:** Reduced token usage by **60-90%**, preserving semantic context while fitting within the LLM context window.

### B. The Classifier & Data Strategy
I fine-tuned **Gemini 2.0 Flash Lite** to classify commits into a custom taxonomy. To achieve high accuracy on rare edge cases, I implemented two advanced strategies:

*   **Synthetic Data Augmentation ("Booster Packs"):**
    Real-world concurrency bugs (deadlocks, race conditions) are rare (<1% of commits). To prevent the model from ignoring them due to class imbalance, I generated high-fidelity synthetic training examples for C and Rust. This "Booster Pack" ensured the model could recognize complex threading violations despite their scarcity in the wild.

*   **Chain-of-Thought Prompting ("Mini-Lessons"):**
    I enforced a strict JSON output schema that required a **"Mini-Lesson"**. Instead of jumping to a label, the model had to generate a 15-word educational explanation (e.g., *"This is a Use-After-Free because the pointer is accessed after `free()`..."*). This forced the model to "show its work," significantly reducing hallucinations on ambiguous commits.

*   **Validation & Quality Control:**
    *   **Human-in-the-Loop (HITL) Curation:** Instead of relying on unsupervised learning, I manually verified and labeled the "Gold Standard" training dataset. This ensured the model learned from high-quality, human-adjudicated examples before scaling to the full 180k commit dataset.
    *   **Consistency Checks:** The unified model architecture (classifying both C and Rust with the same fine-tuned brain) ensures that the definition of "Complexity" and "Security" remains consistent across languages, eliminating observer bias.

### C. Cloud FinOps & Optimization
*   **Initial Architecture:** Online Inference (sending commits one-by-one).
    *   *Result:* Too slow, hit API rate limits, projected cost >$500.
*   **Optimized Architecture:** Migrated to **Vertex AI Batch Prediction**.
    *   *Result:* Processed 182k commits for **$64**.
    *   *Impact:* **92% Cost Reduction** and 36x throughput increase.
---

## 4. Key Findings & Data

### Finding 1: The "Rust Dividend"
*   **C Libraries:** Consistently allocate **10-20%** of maintenance effort to Memory Safety.
*   **Rust Libraries:** Allocate **<2%** to Memory Safety.
*   **The Trade-off:** The data shows a near-perfect correlation: The time Rust saves on memory bugs is reinvested directly into **Feature & Value Add**.

### Finding 2: The "Shift Left" (Commit Complexity Score)
I developed a **Commit Complexity Score (CCS)**:
$$ CCS = (0.5 \cdot CogLoad) + (0.25 \cdot \log(Entropy)) + (0.25 \cdot \log(Churn)) $$

*   **Result:** Rust commits have a higher average CCS than C commits.
*   **Interpretation:** Rust forces developers to handle complexity *upfront* (satisfying the compiler). C allows "simple" (but incorrect) code to merge, pushing the cost to the debugging phase.

---

## 5. Repository Structure

```bash
├── etl_pipeline/
│   ├── 01_extractor.py       # GitPython extractor & "Accordion" truncation
│   ├── 02_batch_prep.py      # JSONL formatting for Vertex AI
│   └── 03_uploader.py        # Cloud Storage interface
├── analysis/
│   ├── taxonomy_finetune.py  # Prompt engineering & HITL validation
│   └── visualization.R       # R code for Violin Plots & Bar Charts
├── docs/
│   └── Research_Report.pdf   # Full findings
└── README.md
```

## 6. Impact
This research provides quantitative evidence supporting the strategic migration of critical infrastructure to memory-safe languages. It moves the debate from "Rust feels safer" to "Rust statistically eliminates the most expensive category of maintenance."
```