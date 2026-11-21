<div align="center">

<img src="https://capsule-render.vercel.app/api?type=rect&color=0:0d1117,100:1f2937&height=140&text=Devasya%20Dwivedi&fontSize=42&fontColor=e6edf3&fontAlignY=45&desc=Systems%20%C2%B7%20Machine%20Learning%20%C2%B7%20Quantitative%20Research&descSize=16&descAlignY=72&descColor=9ca3af" width="100%" />

<a href="#flagship">
  <img src="https://readme-typing-svg.demolab.com?font=JetBrains+Mono&weight=500&size=16&pause=1200&color=8B949E&center=true&vCenter=true&width=640&lines=Building+storage+engines+in+modern+C%2B%2B;Leakage-free+probabilistic+forecasting;Problems+that+are+hard+to+fake+and+worth+solving" />
</a>

<br/>

<a href="https://linkedin.com/in/devasya-dwivedi"><img src="https://img.shields.io/badge/LinkedIn-0A66C2?style=flat-square&logo=linkedin&logoColor=white" /></a>
<a href="mailto:devasya3005@gmail.com"><img src="https://img.shields.io/badge/Email-111827?style=flat-square&logo=gmail&logoColor=white" /></a>
<a href="https://github.com/dEvAsTaTiNg0009?tab=repositories"><img src="https://img.shields.io/badge/All%20Projects-111827?style=flat-square&logo=github&logoColor=white" /></a>

<sub>
<a href="#flagship">Flagship</a> &nbsp;·&nbsp;
<a href="#featured">Featured</a> &nbsp;·&nbsp;
<a href="#forecasting-research">Forecasting</a> &nbsp;·&nbsp;
<a href="#applied-systems">Applied</a> &nbsp;·&nbsp;
<a href="#toolkit">Toolkit</a> &nbsp;·&nbsp;
<a href="#contact">Contact</a>
</sub>

</div>

---

## Flagship

### [CASCADE](https://github.com/dEvAsTaTiNg0009/CASCADE)
**Cache-Sensitive Adaptive Storage Architecture for Dynamic and Efficient LSM-Tree Design**

<img src="https://img.shields.io/badge/C%2B%2B-17-00599C?style=flat-square&logo=cplusplus&logoColor=white" /> <img src="https://img.shields.io/badge/YCSB-A%E2%80%93F-374151?style=flat-square" /> <img src="https://img.shields.io/badge/ASan%20%7C%20TSan-clean-374151?style=flat-square" /> <img src="https://img.shields.io/badge/License-MIT-374151?style=flat-square" />

A research-grade LSM key-value storage engine that co-designs three subsystems usually tuned in isolation: the memtable, the compaction policy, and the Bloom filter layer. Each one feeds signals to the next.

<table>
<tr>
<td align="center" width="25%"><h3>5.5M</h3><sub>ops/s · read-only<br/>(YCSB-C)</sub></td>
<td align="center" width="25%"><h3>3.05M</h3><sub>ops/s · 50/50 read-update<br/>(YCSB-A)</sub></td>
<td align="center" width="25%"><h3>≤ 0.4 µs</h3><sub>P99 latency<br/>(YCSB A–D)</sub></td>
<td align="center" width="25%"><h3>3.11× → 1.13×</h3><sub>write amplification<br/>uniform → skewed load</sub></td>
</tr>
</table>

<table>
<tr>
<td width="33%" valign="top">

**① Concurrent CSB⁺-Tree MemTable**

64-byte cache-line-aligned nodes, optimistic lock coupling, epoch-based memory reclamation.

</td>
<td width="33%" valign="top">

**② Adaptive Hybrid Lightweight Compaction**

Switches policy from live telemetry (EWMA write velocity, Gini access skew, level saturation), with a hysteresis FSM to prevent thrashing.

</td>
<td width="33%" valign="top">

**③ Dual-Trigger Blocked Bloom Layer**

Reallocates filter budget on compaction events, and also by access frequency through a sliding-window tracker.

</td>
</tr>
</table>

<details>
<summary><b>Architecture</b> — how the subsystems connect</summary>
<br/>

```mermaid
flowchart LR
    W([Writes]) --> WAL[WAL · group commit]
    W --> M["CSB+ Tree MemTable<br/>64B-aligned · OLC · EBMM"]
    M -- flush --> S[("SSTables<br/>fence pointers · LRU block cache")]
    S <--> A["AHLC Compaction<br/>EWMA velocity · Gini skew · hysteresis FSM"]
    A -- budget events --> B["Dual-Trigger Bloom Layer<br/>structural + frequency tracking"]
    R([Reads]) --> M
    R --> B --> S
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

<sub>Measured with the in-repo YCSB harness (scrambled Zipfian generator). Full read/space amplification figures are in the repo.</sub>

</details>

<details>
<summary><b>Adaptivity under workload skew</b> — Zipfian θ sweep on Workload A</summary>
<br/>

| θ | Distribution | Throughput | Write amp | Policy switches |
|---|---|---|---|---|
| 0.00 | Uniform | 2,363.5K ops/s | 3.11× | 2 |
| 0.80 | Moderate skew | 2,572.7K ops/s | 2.09× | 1 |
| 0.90 | High skew | 3,023.1K ops/s | 1.13× | 1 |
| 0.99 | Extreme skew | **3,157.1K ops/s** | **1.13×** | 1 |

</details>

<details>
<summary><b>Engineering</b> — correctness and tooling</summary>
<br/>

- Write-ahead log with group-commit batching
- Lock-free HDR latency histograms and RUM-triad (read, update, memory) accounting
- AddressSanitizer, UBSan and ThreadSanitizer build targets
- Full test suite across every stage, plus an ablation runner

</details>

---

## Featured

### [Bitcoin Anomaly Detection](https://github.com/dEvAsTaTiNg0009/bitcoin-anomaly-detection)
<!-- TODO: update the link above after renaming the repo -->

<img src="https://img.shields.io/badge/Python-3776AB?style=flat-square&logo=python&logoColor=white" /> <img src="https://img.shields.io/badge/Time%20Series-374151?style=flat-square" /> <img src="https://img.shields.io/badge/Anomaly%20Detection-374151?style=flat-square" />

Detecting anomalous behaviour in Bitcoin time-series data.
<!-- TODO: one line on the data source (market / on-chain), the method, and one headline result -->

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

## Forecasting Research

Both projects share one principle: **no information from the future touches the model.** I prefer honest, calibrated probabilities over inflated accuracy.

<table>
<tr>
<td width="50%" valign="top">

### [Stochastic Cricket Prediction](https://github.com/dEvAsTaTiNg0009/Stochastic-Cricket-Prediction)

Strictly causal IPL match forecasting trained on 18 seasons of ball-by-ball data (2008–2025).

`638 blind walk-forward matches` `10/10 leakage red-team tests` `SHA-256 frozen 2026 holdout`

</td>
<td width="50%" valign="top">

### [Football Prediction](https://github.com/dEvAsTaTiNg0009/football-predictor)

Leakage-free match forecasting with strict temporal validation and calibrated outcome probabilities.

`Temporal validation` `Probabilistic outputs` `Calibration`

</td>
</tr>
</table>

<details>
<summary><b>Cricket — methodology and evaluation</b></summary>
<br/>

**Pipeline:** frozen historical state → causal feature engine (7 interpretable feature families, 69 features) → regularized base models → chronological out-of-fold predictions → Elastic Net meta-learner → isotonic calibration.

**Ensemble after pruning:** Bayesian Bradley-Terry, MLP, Gradient Boosting and Logistic Regression were kept. XGBoost, LightGBM and ExtraTrees were pruned by the meta-learner as redundant.

| Evaluation (2016–2025, pre-XI mode) | Value |
|---|---|
| Blind held-out matches | 638 |
| Accuracy | 51.7% (ELO baseline 49.5%) |
| Log loss / Brier | 0.7245 / 0.2616 |

<sub>T20 outcomes are close to coin-flip before the toss. The project measures how much signal actually exists, without leakage inflating it.</sub>

</details>

---

## Applied Systems

### [Pravesh](https://github.com/dEvAsTaTiNg0009/Pravesh)
**Smart campus entry infrastructure** — a QR-based entry/exit platform running at campus scale: **2,500+ users** and **~500 verified entries per day**.

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

**Going deeper into:** `Probability` `Statistics` `Optimization` `Econometrics` `Time Series` `Statistical Learning` `Quantitative Finance`

---

## Contact

Open to conversations about storage systems, forecasting, and quantitative research.

<a href="https://linkedin.com/in/devasya-dwivedi"><img src="https://img.shields.io/badge/LinkedIn-Devasya%20Dwivedi-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white" /></a>
<a href="mailto:devasya3005@gmail.com"><img src="https://img.shields.io/badge/Email-devasya3005%40gmail.com-111827?style=for-the-badge&logo=gmail&logoColor=white" /></a>

<p align="center"><sub>Build. Test. Question. Repeat.</sub></p>
