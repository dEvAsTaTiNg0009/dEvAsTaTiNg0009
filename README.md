<div align="center">

<img src="https://capsule-render.vercel.app/api?type=rect&color=0:0B1220,100:0F2A4A&height=150&text=Devasya%20Dwivedi&fontSize=44&fontColor=E2E8F0&fontAlignY=44&desc=Systems%20%C2%B7%20Machine%20Learning%20%C2%B7%20Quantitative%20Research&descSize=16&descAlignY=72&descColor=38BDF8" width="100%" />

<a href="#cascade">
  <img src="https://readme-typing-svg.demolab.com?font=JetBrains+Mono&weight=500&size=16&pause=1200&color=38BDF8&center=true&vCenter=true&width=640&lines=Storage+engines+in+modern+C%2B%2B;Products+running+in+production;Forecasts+that+never+see+the+future" />
</a>

<br/>

<a href="https://linkedin.com/in/devasya-dwivedi"><img src="https://img.shields.io/badge/LinkedIn-Connect-38BDF8?style=flat-square&labelColor=1E293B&logo=linkedin&logoColor=white" /></a>
<a href="mailto:devasya3005@gmail.com"><img src="https://img.shields.io/badge/Email-Reach%20out-2DD4BF?style=flat-square&labelColor=1E293B&logo=gmail&logoColor=white" /></a>
<a href="https://github.com/dEvAsTaTiNg0009?tab=repositories"><img src="https://img.shields.io/badge/Projects-View%20all-818CF8?style=flat-square&labelColor=1E293B&logo=github&logoColor=white" /></a>

<sub>
<a href="#cascade">CASCADE</a> &nbsp;·&nbsp;
<a href="#bitcoin-anomaly-detection">Bitcoin Anomaly Detection</a> &nbsp;·&nbsp;
<a href="#pravesh">Pravesh</a> &nbsp;·&nbsp;
<a href="#forecasting-research">Forecasting</a> &nbsp;·&nbsp;
<a href="#toolkit">Toolkit</a> &nbsp;·&nbsp;
<a href="#contact">Contact</a>
</sub>

</div>

---

$\color{#38BDF8}{\textsf{01 / FLAGSHIP / STORAGE SYSTEMS}}$

## [CASCADE](https://github.com/dEvAsTaTiNg0009/CASCADE)

**Cache-Sensitive Adaptive Storage Architecture for Dynamic and Efficient LSM-Tree Design**

<img src="https://img.shields.io/badge/C%2B%2B-17-38BDF8?style=flat-square&labelColor=1E293B&logo=cplusplus&logoColor=white" /> <img src="https://img.shields.io/badge/benchmark-YCSB%20A%E2%80%93F-38BDF8?style=flat-square&labelColor=1E293B" /> <img src="https://img.shields.io/badge/sanitizers-ASan%20%7C%20TSan-38BDF8?style=flat-square&labelColor=1E293B" /> <img src="https://img.shields.io/badge/license-MIT-38BDF8?style=flat-square&labelColor=1E293B" />

CASCADE is a key-value storage engine in modern C++17. Most LSM engines tune the memtable, the compaction policy and the Bloom filters as separate problems. In CASCADE they share live workload signals and adapt together. The result is millions of operations per second, sub-microsecond tail latency, and write amplification that *drops* as the workload gets more skewed.

<table>
<tr>
<td align="center" width="25%">

$\color{#38BDF8}{\Large\textbf{5.5M}}$

<sub>ops/s · read-only<br/>YCSB-C</sub>

</td>
<td align="center" width="25%">

$\color{#38BDF8}{\Large\textbf{3.05M}}$

<sub>ops/s · 50/50 read-update<br/>YCSB-A</sub>

</td>
<td align="center" width="25%">

$\color{#38BDF8}{\Large\mathbf{\leq 0.4\,\mu s}}$

<sub>P99 latency<br/>YCSB A–D</sub>

</td>
<td align="center" width="25%">

$\color{#38BDF8}{\Large\mathbf{3.11\times \rightarrow 1.13\times}}$

<sub>write amplification<br/>uniform → skewed load</sub>

</td>
</tr>
</table>

<table>
<tr>
<td width="33%" valign="top">

$\color{#38BDF8}{\textsf{① MEMTABLE}}$

**Concurrent CSB⁺-Tree**

Nodes aligned to 64-byte cache lines, optimistic lock coupling for concurrent writers, and epoch-based memory reclamation.

</td>
<td width="33%" valign="top">

$\color{#38BDF8}{\textsf{② COMPACTION}}$

**Adaptive Hybrid (AHLC)**

Picks the compaction policy at runtime from write velocity, access skew and level saturation. A hysteresis state machine keeps it from thrashing between policies.

</td>
<td width="33%" valign="top">

$\color{#38BDF8}{\textsf{③ FILTERS}}$

**Dual-Trigger Blocked Bloom**

Moves filter memory where it pays off. It reacts both to compaction events and to observed access frequency.

</td>
</tr>
</table>

<details>
<summary><b>Architecture</b> — how the three subsystems talk to each other</summary>
<br/>

```mermaid
flowchart LR
    W([Writes]):::io --> WAL[WAL · group commit]:::store
    W --> M["CSB+ Tree MemTable<br/>64B-aligned · OLC · EBMM"]:::core
    M -- flush --> S[("SSTables<br/>fence pointers · LRU block cache")]:::store
    S <--> A["AHLC Compaction<br/>write velocity · access skew · hysteresis"]:::core
    A -- budget events --> B["Dual-Trigger Bloom Layer<br/>structural + frequency"]:::core
    R([Reads]):::io --> M
    R --> B --> S

    classDef io fill:#0B1220,stroke:#38BDF8,stroke-width:1.5px,color:#E2E8F0
    classDef core fill:#0F2A4A,stroke:#38BDF8,stroke-width:1.5px,color:#E2E8F0
    classDef store fill:#1E293B,stroke:#2DD4BF,stroke-width:1.5px,color:#E2E8F0
```

</details>

<details>
<summary><b>YCSB A–F benchmark results</b></summary>
<br/>

| Workload | Mix | Throughput | WAF | Bloom FPR | P99 | P99.9 |
|---|---|---|---|---|---|---|
| A | 50% read / 50% update | **3,052K ops/s** | 1.13× | 0.016% | 0.4 µs | 1.6 µs |
| B | 95% read / 5% update | **4,651K ops/s** | 1.24× | 0.052% | 0.2 µs | 1.6 µs |
| C | 100% read | **5,528K ops/s** | 1.33× | 0.040% | 0.2 µs | 1.6 µs |
| D | 95% read / 5% insert | **4,328K ops/s** | 1.24× | 0.037% | 0.2 µs | 0.8 µs |
| E | 95% scan / 5% insert | **827K ops/s** | 1.09× | 0.000% | — | — |
| F | 50% read / 50% RMW | **2,282K ops/s** | 4.21× | 0.364% | 0.4 µs | 1.6 µs |

<sub>Measured with the in-repo YCSB harness using a scrambled Zipfian generator. Read and space amplification figures are in the repo.</sub>

</details>

<details>
<summary><b>Adaptivity under skew</b> — Zipfian θ sweep on Workload A</summary>
<br/>

As access patterns concentrate, AHLC shifts policy and write amplification falls by almost two-thirds.

| θ | Distribution | Throughput | Write amp | Policy switches |
|---|---|---|---|---|
| 0.00 | Uniform | 2,363.5K ops/s | 3.11× | 2 |
| 0.80 | Moderate skew | 2,572.7K ops/s | 2.09× | 1 |
| 0.90 | High skew | 3,023.1K ops/s | 1.13× | 1 |
| 0.99 | Extreme skew | **3,157.1K ops/s** | **1.13×** | 1 |

</details>

<details>
<summary><b>Engineering</b> — durability, measurement and correctness</summary>
<br/>

- **Durability:** write-ahead log with group-commit batching
- **Measurement:** lock-free HDR latency histograms and full read/write/space amplification accounting
- **Correctness:** AddressSanitizer, UBSan and ThreadSanitizer build targets, plus a test suite covering every stage
- **Analysis:** a built-in ablation runner to isolate each subsystem's contribution

</details>

---

$\color{#818CF8}{\textsf{02 / FEATURED / MACHINE LEARNING}}$

## [Bitcoin Anomaly Detection](https://github.com/dEvAsTaTiNg0009/bitcoin-anomaly-detection)
<!-- TODO: update the link above after renaming the repo -->

<img src="https://img.shields.io/badge/Python-3-818CF8?style=flat-square&labelColor=1E293B&logo=python&logoColor=white" /> <img src="https://img.shields.io/badge/domain-time%20series-818CF8?style=flat-square&labelColor=1E293B" /> <img src="https://img.shields.io/badge/task-anomaly%20detection-818CF8?style=flat-square&labelColor=1E293B" />

Finds the moments when Bitcoin stops behaving like itself. The system flags statistically unusual activity in Bitcoin time-series data and separates real anomalies from ordinary volatility.
<!-- TODO: one line naming the data source (market / on-chain), the method, and one headline result -->

<details>
<summary><b>Approach and results</b></summary>
<br/>

<!-- TODO: fill in -->
| | |
|---|---|
| **Data** | — |
| **Method** | — |
| **Evaluation** | — |
| **Result** | — |

</details>

---

$\color{#2DD4BF}{\textsf{03 / IN PRODUCTION / DEPLOYED PRODUCT}}$

## [Pravesh](https://github.com/dEvAsTaTiNg0009/Pravesh)

**Smart campus access, live and in daily use**

<img src="https://img.shields.io/badge/status-live%20in%20production-2DD4BF?style=flat-square&labelColor=1E293B" /> <img src="https://img.shields.io/badge/users-2%2C500%2B-2DD4BF?style=flat-square&labelColor=1E293B" /> <img src="https://img.shields.io/badge/usage-daily-2DD4BF?style=flat-square&labelColor=1E293B" />

Pravesh is a shipped product, not a prototype. It runs campus entry and exit through QR-based identity verification: every person is verified at the gate, and every movement is logged with a timestamp. It serves a user base of more than 2,500 people and handles real traffic every day.

<table>
<tr>
<td align="center" width="33%">

$\color{#2DD4BF}{\Large\textbf{2,500+}}$

<sub>users on the platform</sub>

</td>
<td align="center" width="33%">

$\color{#2DD4BF}{\Large\mathbf{\sim 500}}$

<sub>verified entries every day</sub>

</td>
<td align="center" width="33%">

$\color{#2DD4BF}{\Large\textbf{Live}}$

<sub>deployed and running on campus</sub>

</td>
</tr>
</table>

<details>
<summary><b>What it does</b></summary>
<br/>

- **Verify:** QR-based identity check at every entry and exit point
- **Record:** a time-stamped, auditable log of campus movement
- **Scale:** built for the daily load of a full campus population, not a demo

</details>

---

$\color{#818CF8}{\textsf{04 / RESEARCH / PROBABILISTIC FORECASTING}}$

## Forecasting Research

Both projects follow one rule: **a prediction may only use information that existed before the event.** Many sports models report inflated accuracy because the future leaks into training. These models are built to measure the signal that actually exists.

<table>
<tr>
<td width="50%" valign="top">

### [Stochastic Cricket Prediction](https://github.com/dEvAsTaTiNg0009/Stochastic-Cricket-Prediction)

A strictly causal IPL forecasting framework trained on 18 seasons of ball-by-ball data. It is audited by automated tests that actively try to leak future information into the model.

<img src="https://img.shields.io/badge/blind%20matches-638-818CF8?style=flat-square&labelColor=1E293B" />
<img src="https://img.shields.io/badge/leakage%20tests-10%2F10-818CF8?style=flat-square&labelColor=1E293B" />
<img src="https://img.shields.io/badge/2026%20holdout-SHA--256%20frozen-818CF8?style=flat-square&labelColor=1E293B" />

</td>
<td width="50%" valign="top">

### [Football Prediction](https://github.com/dEvAsTaTiNg0009/football-predictor)

Applies the same discipline to football: strictly time-ordered validation, no lookahead, and calibrated match-outcome probabilities instead of bare picks.

<img src="https://img.shields.io/badge/validation-temporal-818CF8?style=flat-square&labelColor=1E293B" />
<img src="https://img.shields.io/badge/output-calibrated%20probabilities-818CF8?style=flat-square&labelColor=1E293B" />

</td>
</tr>
</table>

<details>
<summary><b>Cricket — methodology and evaluation</b></summary>
<br/>

**Pipeline:** frozen historical state → causal feature engine (7 interpretable feature families, 69 features) → regularized base models → chronological out-of-fold predictions → Elastic Net meta-learner → isotonic calibration.

**Ensemble:** Bayesian Bradley-Terry, MLP, Gradient Boosting and Logistic Regression were kept. XGBoost, LightGBM and ExtraTrees were pruned by the meta-learner as redundant.

| Evaluation (2016–2025, pre-XI mode) | Value |
|---|---|
| Blind held-out matches | 638 |
| Accuracy | 51.7% (ELO baseline 49.5%) |
| Log loss / Brier | 0.7245 / 0.2616 |

<sub>Before the toss, T20 outcomes sit close to a coin flip. The project quantifies how much real signal exists, with no leakage inflating the answer.</sub>

</details>

---

## Toolkit

<p align="center">
  <img src="https://skillicons.dev/icons?i=cpp,c,py,java,dart,linux&theme=dark" /><br/>
  <img src="https://skillicons.dev/icons?i=pytorch,sklearn,opencv,mysql,nodejs,react,flutter,firebase,git,figma&theme=dark" />
</p>

<details>
<summary><b>Full stack</b></summary>
<br/>

| | |
|---|---|
| **Languages** | C · C++ · Python · Java · Dart |
| **ML & Stats** | PyTorch · scikit-learn · XGBoost · LightGBM · pandas · NumPy |
| **Systems** | Linux · concurrency · sanitizers (ASan / TSan) · MySQL |
| **Application** | Node.js · React · Flutter · Firebase |
| **Vision** | OpenCV |
| **Workflow** | Git · Jupyter · Figma |

</details>

<p align="center">
  <img src="https://img.shields.io/badge/Probability-1E293B?style=flat-square" />
  <img src="https://img.shields.io/badge/Statistics-1E293B?style=flat-square" />
  <img src="https://img.shields.io/badge/Optimization-1E293B?style=flat-square" />
  <img src="https://img.shields.io/badge/Econometrics-1E293B?style=flat-square" />
  <img src="https://img.shields.io/badge/Time%20Series-1E293B?style=flat-square" />
  <img src="https://img.shields.io/badge/Statistical%20Learning-1E293B?style=flat-square" />
  <img src="https://img.shields.io/badge/Quantitative%20Finance-1E293B?style=flat-square" />
</p>

---

## Contact

Always up for a conversation about storage systems, forecasting, or quantitative research.

<a href="https://linkedin.com/in/devasya-dwivedi"><img src="https://img.shields.io/badge/LinkedIn-Devasya%20Dwivedi-38BDF8?style=for-the-badge&labelColor=1E293B&logo=linkedin&logoColor=white" /></a>
<a href="mailto:devasya3005@gmail.com"><img src="https://img.shields.io/badge/Email-devasya3005%40gmail.com-2DD4BF?style=for-the-badge&labelColor=1E293B&logo=gmail&logoColor=white" /></a>

<img src="https://capsule-render.vercel.app/api?type=rect&color=0:0F2A4A,100:0B1220&height=60&text=Build.%20Test.%20Question.%20Repeat.&fontSize=16&fontColor=94A3B8&fontAlignY=55" width="100%" />
