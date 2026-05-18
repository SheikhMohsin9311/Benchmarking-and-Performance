# How to Build a Benchmark — Kistowski et al. (ICPE 2015)

> **One-sentence summary:** An insider account from SPEC and TPC committee members of the processes, criteria, and trade-offs involved in designing a standardized benchmark — covering benchmark taxonomy, workload design goals, and quality properties that every serious benchmark must satisfy.

---

## 1. Why This Paper Exists (Context & Motivation)

Benchmark consortia like [[SPEC]] and [[TPC]] operate under **confidentiality agreements**. This means the institutional knowledge about *how* good benchmarks are actually designed has historically been invisible to the outside research community. Calls for a published benchmark development process had grown louder in the field (citing Sachs 2010), and this paper is the direct response.

The paper draws on the lived experience of committee members across multiple benchmark suites:
- **SPEC**: SPECpower_ssj2008, SPEC CPU, SERT
- **TPC**: TPC-C, TPC-H, TPC-DS

> [!important] Why This Matters for Your Project
> This paper gives you the **practitioner's vocabulary** for benchmark quality. When you read MLPerf or SPEChpc papers, they use exactly this framework — relevance, reproducibility, fairness, verifiability, usability — implicitly. This paper makes it explicit.

---

## 2. What Is a Benchmark? (Definitions)

### 2.1 Benchmark (Formal Definition)

> "A **standard tool** for the competitive evaluation and comparison of competing systems or components according to specific characteristics, such as performance, dependability, or security."

Key word: **competitive**. This definition is narrowed specifically to the SPEC/TPC context of comparing products against each other.

### 2.2 Rating Tool (Distinct Concept)

A **rating tool** is for *non-competitive* system evaluation:
- Research purposes
- Regulatory programs
- System improvement/development workflows

Example: SPEC's **SERT** (Server Efficiency Rating Tool) — designed like a benchmark but not intended for product horse-racing. The distinction matters because rating tools may have looser governance than competitive benchmarks.

> [!note] Connection to Your Project
> When you design your benchmarking agenda, decide early: are you building a **benchmark** (ranking systems) or a **rating tool** (diagnosing your system)? The paper maps directly to Phase 1 of your project framework — the *comparative vs. diagnostic* distinction.

---


## 3. Benchmark Taxonomy (Three Types)

### 3.1 Specification-Based

| Property | Detail |
|---|---|
| **What it provides** | A description of a business problem to simulate; required inputs and expected outputs |
| **What it leaves open** | The implementation — the person running the benchmark builds it |
| **Key advantage** | Allows innovative software approaches to solve the benchmark problem |
| **Key disadvantage** | Requires substantial development work before running; proving compliance is hard |
| **Primary example** | TPC benchmarks — define *what* a transaction system must do, not *how* |

Specification-based benchmarks begin with a **business problem definition**. Novelty and relevance are the key criteria at this stage. The flexibility is the point: a new database architecture can prove itself by meeting the spec on its own terms.

### 3.2 Kit-Based

| Property | Detail |
|---|---|
| **What it provides** | A complete runnable implementation as a required artifact |
| **What it leaves open** | Very little — functional differences must be resolved ahead of time |
| **Key advantage** | Near "load and go" — dramatically reduces cost and time to run |
| **Key disadvantage** | May restrict innovative approaches that can't fit the kit's execution model |
| **Primary example** | SPEC CPU — you run the provided workload binary |

For kit-based benchmarks, the specification functions as a **design guide for the implementer of the kit**, not as instructions for the submitter.

### 3.3 Hybrid

When most of the benchmark can be a kit, but there is a desire to allow **some functions to be implemented at the submitter's discretion**.

### 3.4 Current Trend

> "While both specification-based and kit-based methods have both been successful in the past, **current trends have favored kit-based development**."

Rationale: Lower barrier to entry reduces the total cost of the benchmarking ecosystem, increasing participation and breadth of published results.

> [!note] MLPerf is a hybrid
> MLPerf's closed division is kit-like (fixed model, fixed preprocessing, fixed accuracy target). The open division is specification-like (same accuracy target, any implementation). This paper's taxonomy explains *why* MLPerf was designed that way.

---

## 4. Core Thesis: No Single Workload Is Sufficient

> "Since **no single workload can be strong in all of these areas**, there will always be a need for **multiple workloads and benchmarks**."

This is a foundational epistemic claim. The paper argues that the design space of benchmarks is defined by **tensions between competing criteria**, and any single workload embodies a set of trade-off decisions that will make it strong for some scenarios and weak for others.

This is why SPEC CPU exists alongside SPECpower, and why MLPerf has separate training/inference/power benchmarks. They are not redundant; they are solving different trade-offs.

---

## 5. The Five Quality Criteria (Core of the Paper)

The paper organizes all benchmark quality properties into five groups. These are not independent — they trade off against each other.

```
Quality Criteria
├── Relevance         ← Most important; hardest to achieve broadly
├── Reproducibility   ← Run-to-run consistency + cross-site portability
├── Fairness          ← No artificial advantages; governance matters
├── Verifiability     ← Results must be trustworthy
└── Usability         ← Barrier to entry must be manageable
```

---

### 5.1 Relevance

**Relevance** = how closely benchmark behavior correlates with behaviors that matter to the consumers of the results.

#### Two Dimensions of Relevance

| Dimension | Meaning |
|---|---|
| **Breadth of applicability** | How many deployment scenarios does this benchmark speak to? |
| **Degree of relevance** | For the scenarios it covers, how faithfully does it represent them? |

These two dimensions **trade off**:
- An XML parsing benchmark → *high degree* of relevance for XML performance, *narrow breadth*
- SPEC CPU2006 → *moderate degree* of relevance for a *wide range* of computing environments

> "Benchmarks that are designed to be highly relevant in a specific area tend to have **narrow applicability**, while benchmarks that attempt to be applicable to a broader spectrum of uses tend to be **less meaningful** for any particular scenario."

#### Scalability as a Relevance Concern

Scalability is specifically called out as an aspect of relevance for server benchmarks. Most relevant server benchmarks must be **multi-process and/or multi-threaded** to stress the full resources of the system under test. The challenge is:
- Avoid artificial scaling limits
- But also behave like real applications, which often have their own scalability limits

#### Implications for Benchmark Designers

Relevance means first asking: **who are the consumers of these results, and what decisions will they make with them?** The benchmark is then designed backwards from that question. This is exactly the "Purpose & Scientific Claim" step in Phase 1 of your project framework.

---

### 5.2 Reproducibility

**Reproducibility** = the capability to consistently produce the same results for a given test configuration.

This has two sub-components:
1. **Run-to-run consistency** — same machine, same config, repeated runs
2. **Cross-site reproducibility** — independent tester, different physical system, equivalent config

#### Sources of Variability (Exhaustive List from Paper)

| Source | Type |
|---|---|
| Thread scheduling timing | OS non-determinism |
| Dynamic compilation (JIT) | Runtime non-determinism |
| Physical disk layout | Hardware non-determinism |
| Network contention | Environmental |
| User interaction during run | Operator error |
| Power management (DVFS, turbo boost) | Hardware-level interference |
| Temperature changes affecting power consumption | Physical environment |

> [!warning] Energy Benchmarks Are Harder
> "Energy efficiency benchmarks often have **additional sources of variability** due to power management technologies dynamically making changes to system performance and temperature changes affecting power consumption."

This is directly relevant to MLPerf Power and any energy-aware benchmarking you plan to do.

#### Mechanisms to Address Variability

| Mechanism | How It Works |
|---|---|
| **Run length** | Run long enough that variable behaviors appear in representative proportions |
| **Multiple run requirement** | Submit multiple runs; scores must be near each other as evidence of consistency |
| **Steady-state operation** | Benchmark runs at steady load, unlike real applications with bursty user demand |
| **Configuration disclosure** | Hardware, software versions, firmware, tuning options must all be stated |
| **Auditing (TPC)** | Certified auditor reviews results and ensures reporting compliance |
| **Automatic validation (SPEC)** | Combination of runtime checks and committee review |

#### What "Configuration Disclosure" Actually Requires

The paper is specific about what must be documented for reproducibility:
- **Hardware**: sufficient detail for another person to obtain identical hardware
- **Software versions**: exact version numbers so the same software can be used
- **Tuning options**: firmware, OS, and application software configuration must all be documented

> "In both of these cases, the description may **not provide enough detail** for an independent tester to be able to assemble an equivalent environment."

This is an acknowledged gap — even good-faith disclosure often falls short of true reproducibility. This limitation maps directly to the [[Reliable Benchmarking Tools and Practices]] paper's concerns.

> [!note] Connection to Your SUT Definition
> Phase 1 of your project asks: "What is the exact definition of the System Under Test?" This section tells you *exactly* what that definition must contain: hardware, OS, software stack, compiler flags, firmware, driver versions, tuning options.

---

### 5.3 Fairness

**Fairness** = ensuring systems can compete on their merits without artificial constraints imposed by the benchmark design.

#### The Artificiality Problem

Benchmarks are always simplifications of reality. This creates a risk: the benchmark's simplicity can be *exploited* in ways that real applications cannot be. Fairness mechanisms are designed to close these loopholes.

Examples:
- Enterprise software typically requires **security features** that a benchmark doesn't directly exercise. A vendor who strips security features could run faster — but that software wouldn't be usable in the real-world scenarios the benchmark targets.
- A benchmark might effectively require a minimum number of disks to achieve reasonable I/O rates — which constrains what hardware configurations can participate.

#### Why Consortium Governance Promotes Fairness

> "Benchmarks developed by a **consensus of experts** is generally perceived as being more fair than a benchmark designed by a single company."

SPEC and TPC derive their legitimacy partly from their membership structure: companies *and* academic institutions. "Design by committee" is inefficient but produces compromises that multiple stakeholders can accept as fair.

#### The Balance Problem in Run Rules

| Too restrictive | Too permissive |
|---|---|
| Disallows results that are relevant for legitimate scenarios | Pollutes the result pool with "inappropriate" submissions |
| Reduces participation | Reduces the number of *relevant* results (legitimate vendors can't compete with gaming) |

Both failure modes reduce the utility of the benchmark.

#### Fair Use Rules

Run rules must also govern **how results are used in comparisons**, not just how they are produced. Example: SPECpower_ssj2008 requires that if you compare power consumption at 50% load, you must also state the overall ssj_ops/watt value. This prevents cherry-picked single-metric comparisons.

> [!note] Connection to MLPerf
> MLPerf's closed/open division distinction is a fairness mechanism. Closed division rules prevent the most aggressive optimizations (like model quantization) that would make comparisons meaningless. The open division allows them but with different interpretation rules.

---

### 5.4 Verifiability

**Verifiability** = results can be deemed trustworthy because the benchmark provides mechanisms to confirm results are accurate.

#### The Industry Problem

In industry, benchmarks are typically run by **vendors with a vested interest in the results**. This is not a moral failing — it's a structural reality. Academia has peer review; industry needs verifiability mechanisms built into the benchmark itself.

#### Mechanisms for Verifiability

| Mechanism | What It Does |
|---|---|
| **Self-validation at runtime** | Benchmark checks that run rules are being followed during execution |
| **Configuration option control** | Benchmark limits compliant values and can verify them at runtime (more trustworthy than user-documented options) |
| **Functional output verification** | Check that outputs are correct — catches cases where aggressive compiler optimizations produce wrong results |
| **Extra data in results** | Include more information than strictly needed; inconsistencies raise red flags |

The last mechanism is clever: a throughput benchmark that also reports response time distributions and transaction counts provides *redundant* information. If the throughput claim is inflated, inconsistencies will appear in the correlated data.

> "Configuration details that must be documented by the user are **less trustworthy** since they could have been entered incorrectly."

This argues for benchmark-collected system information over user-declared information wherever possible.

---

### 5.5 Usability

**Usability** = avoiding roadblocks for users to run the benchmark in their test environments.

#### Self-Validation as a Usability Feature

The most important ease-of-use feature is **self-validation** — giving the tester confidence that the workload is running correctly. This also appears under verifiability; the two criteria are intertwined here.

#### The Cost Barrier Problem

The paper gives a striking concrete example:

> "The current top TPC-C result has a system under test with over **100 distinct servers**, over **700 disk drives** and **11,000 SSD flash modules** (with a total capacity of **1.76 petabytes**), and a system cost of **over $30 million USD**."

The median cost of TPC-C submissions between 2010–2013 was $776,627. These configurations "aren't economical for most potential users." High usability cost reduces participation and thus the breadth and representativeness of the result pool.

#### Configuration Documentation as a Usability Challenge

Accurately documenting the SUT is **both a reproducibility requirement and a usability challenge**. Systems are complex; describing them fully is labor-intensive. Benchmarks can improve this by providing tooling to assist with automated system configuration collection.

---

## 6. Cross-Cutting Tensions Between Criteria

The paper does not make this table explicit, but the following tensions are implicit throughout:

| Tension | Description |
|---|---|
| Relevance ↔ Reproducibility | High-fidelity real-world workloads introduce more variability (real apps have bursty behavior, user interaction) |
| Relevance ↔ Usability | Representative workloads may require expensive configurations (TPC-C) |
| Fairness ↔ Relevance | Restricting configurations for fairness may exclude legitimate real-world setups |
| Fairness ↔ Usability | Run rules and reporting requirements add complexity for the submitter |
| Verifiability ↔ Usability | More self-validation = more runtime overhead and complexity |
| Specificity ↔ Breadth (Relevance) | High relevance in one domain = low relevance everywhere else |

> [!important] Design Implication
> Every benchmark is a **set of deliberate trade-off decisions**. The job of the benchmark designer is not to maximize all criteria simultaneously (impossible) but to identify which criteria matter most for the intended consumers and optimize for those while remaining acceptable on the others.

---

## 7. Annotation Using Your Per-Paper Template

### Claim
This benchmark measures: *nothing directly* — it is a **meta-paper** about benchmark design, not a benchmark itself. Its "measurement" is of the design space of benchmarks, for the audience of researchers and practitioners who build or evaluate benchmarks.

### Workload Selection Method
N/A — uses existing SPEC and TPC benchmarks as illustrative examples rather than selecting new workloads.

### Reproducibility Mechanism
Discusses reproducibility as a first-class criterion; describes the mechanisms (multi-run requirements, configuration disclosure, auditing) used by SPEC and TPC. Does not introduce new mechanisms — catalogs existing practice.

### Reporting Standard
Advocates for: full configuration disclosure (hardware + software + tuning), multiple runs, self-validation, auditing for competitive results. References SPEC fair use rule and TPC auditor requirements.

### Limitations Acknowledged
The paper is primarily descriptive of SPEC/TPC practice as of 2015. It does not address:
- ML system benchmarking (pre-MLPerf)
- Energy-aware evaluation beyond SPECpower
- The reproducibility crisis in academic ML research
- Non-determinism from random seeds in training workloads
- Cloud/virtualized environments where the SUT boundary is ambiguous

---

## 8. Key Quotes to Remember

> "Since **no single workload can be strong in all of these areas**, there will always be a need for **multiple workloads and benchmarks**."

> "Benchmarks that are designed to be highly relevant in a specific area tend to have **narrow applicability**, while benchmarks that attempt to be applicable to a broader spectrum of uses tend to be **less meaningful** for any particular scenario."

> "Reproducibility is the capability of the benchmark to produce the **same results consistently** for a particular test environment."

> "Fairness ensures that systems can compete on **their merits without artificial constraints**."

> "Within the industry, benchmarks are typically run by **vendors who have a vested interest in the results**."

---

## 9. Connections to Other Papers in Your Survey

| Paper | Connection |
|---|---|
| [[Purdue Benchmarking Methodology]] | That paper operationalizes *reproducibility* with statistical rigor; this paper defines it conceptually |
| [[Reliable Benchmarking Tools and Practices]] | Drills into variance control and experimental discipline — the practical mechanisms behind *reproducibility* here |
| [[MLPerf Inference]] | Applies *all five criteria* in an ML context; closed/open division = fairness mechanism; LoadGen = verifiability mechanism |
| [[SPEChpc 2021]] | Applies this framework to HPC; highlights where *relevance* becomes especially hard (real HPC apps are enormous) |
| [[SPEC CPU2017]] | Canonical kit-based benchmark; exemplifies the trade-off between narrow relevance and broad applicability |
| [[TPC-C]], [[TPC-H]], [[TPC-DS]] | Canonical specification-based benchmarks; exemplify auditing as verifiability and the cost-usability tension |

---

## 10. Open Questions Raised for Your Research Agenda

1. **Where does ML non-determinism fit?** The variability sources listed (thread scheduling, JIT, disk layout) are all *deterministic in principle* — they are hard to control but not fundamentally random. ML training introduces *genuine* non-determinism from random seeds, data shuffling, and hardware-level floating-point non-associativity. How does this change the reproducibility framework?

2. **Is the closed/open MLPerf division the right fairness mechanism?** This paper argues that fairness requires balancing restrictions vs. innovation freedom. Does MLPerf's binary division (closed = locked down, open = anything goes) create a two-tier system that obscures meaningful comparisons?

3. **How do you define the SUT when the system is a cloud service?** The paper assumes a physical system with hardware you can describe. Cloud benchmarking (where the "hardware" is abstracted away) breaks the configuration disclosure requirement.

4. **Kit-based vs. specification-based for ML?** The paper says current trends favor kit-based. But ML models evolve so rapidly that a kit-based benchmark can become irrelevant within a year. Is there a principled argument for specification-based ML benchmarks?

5. **Energy as a first-class metric:** The paper notes that energy benchmarks face extra variability challenges. As energy efficiency becomes a first-class metric (see [[MLPerf Power]]), does the five-criteria framework need to be extended?

---

*Notes created: May 2026 | Project: Benchmarking Survey | Phase: Week 1 — Foundational Methodology*