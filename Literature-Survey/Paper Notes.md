# A Methodology for Scientific Benchmarking with Large-Scale Applications

---

## 0. Read This First — How to Use This Note

This note is designed to be read **before** the paper. It maps the full intellectual territory so that when you encounter each section in the original, you already know what argument it's making, why it matters, and how it connects to your own project.

The note follows the paper's structure but goes deeper on implications and makes explicit the things the paper assumes you already know.

> [!tip] Reading Strategy
>  Read sections 0–2 of this note, then read the paper's Introduction (Section 1) and Section 2. Return here for sections 3–5 before reading Sections 3–4 of the paper. The case study (Section 4 of the paper) is dense with figures — use the figure-by-figure breakdown in section 5 of this note as a companion while reading.

---

## 1. The Big Idea — One Paragraph Summary

The paper argues that the HPC research community has a benchmark problem: existing suites are either **representative** (reflecting real workloads) or **shared** (used by enough researchers to enable comparisons), but not both at once. Armstrong and Eigenmann propose a structured **characterization methodology** — a taxonomy of information categories that every benchmark should document — and demonstrate it on the SPECseis seismic processing application. The goal is to make large-scale, realistic benchmarks _feasible_ for ordinary research groups to use, by telling users exactly what they need to know and telling developers exactly what they must provide.

---

## 2. Background You Need Before Reading

### 2.1 What is SPEC?

**SPEC** = Standard Performance Evaluation Corporation. An industry consortium that produces standardized benchmark suites. At the time of this paper, SPEC had a **High-Performance Group (SPEC/HPG)** focused on parallel and HPC workloads. SPEC benchmarks come with strict **run rules** — legally-defined requirements for how you must run the benchmark and what you must disclose to publish a valid result.

> [!note] Why SPEC Matters for This Paper The paper is written by academic participants in SPEC/HPG. They have a different mission from the industrial members: they want to use these large benchmarks _for scientific research_, not just for marketing comparisons. This creates the core tension the paper resolves.

### 2.2 The Two Properties a Benchmark Must Have

The paper opens with two requirements. Memorize these — they are referenced implicitly throughout:

|Property|Definition|Why It Matters|
|---|---|---|
|**Representative**|The suite reflects important properties of realistic problems|Ensures performance gains measured will hold in real applications|
|**Shared**|The same test suite is used by different researchers under comparable conditions|Enables comparison of results across research groups|

> [!warning] The Core Problem In HPC, you almost never get both. Realistic applications are large, proprietary, hard to distribute, and require enormous machines to run. Scaled-down proxies are shared but not representative. This is the gap the paper addresses.

### 2.3 The Abstraction Problem

This is the philosophical foundation of the entire paper and it is stated once, clearly, in the Introduction. Every reduction of a problem is an abstraction that **assumes certain aspects are more important than others.**

The paper gives a concrete example: suppose you reduce the number of time steps in a simulation, assuming each time step is equivalent. This is fine if you're measuring raw FLOPS. But if you're evaluating _adaptive recompilation_ techniques, the number of time steps matters enormously — the optimization overhead must be amortized over remaining steps. A reduced benchmark would give you completely wrong conclusions.

> [!important] The General Principle The higher the optimization capability of the system you're studying, the more dangerous it is to use abstracted, reduced benchmarks. Systems that adapt, recompile, or learn from runtime behavior need full-scale, realistic workloads to be evaluated correctly.

### 2.4 Prior Benchmark Efforts (Referenced in the Paper)

The paper situates itself against several prior efforts. You'll see these names cited:

- **Perfect Benchmarks / Perfect Club** — proposed "benchmark diaries" where users reported code modifications and their performance impacts. Methodology-oriented but not widely adopted as a shared suite.
- **Parkbench** — defined several system levels and provided benchmarks for each level. More structured than Perfect Club.
- **NAS Parallel Benchmarks** — NASA's parallel benchmark suite. Widely used but kernel-level, not full applications.
- **LINPACK / HPL** — measures dense linear algebra; became the basis for the TOP500 list. Highly shared but not representative of most HPC workloads.
- **SPEC CPU** — the industry standard for single-node CPU benchmarking. Strong run rules and disclosure requirements.

The key contrast the paper draws: commercial suites (SPEC CPU, PC benchmarks) have strong formal run requirements but are designed for comparison, not flexible scientific research. Academic benchmarks (NAS, Perfect) allow flexibility but lack the infrastructure to make large-scale codes accessible.

---

## 3. The Methodology — Information Categories in Full Detail

Section 3 of the paper is the intellectual core. It defines a taxonomy of **information categories** — things you must document about any large-scale benchmark to make it usable for scientific research.

The categories are organized into three tables in the paper (Tables 2, 3, 4). Here they are with expanded definitions.

### 3.1 Table 2 — Characteristics Gathered from the Code (Static Analysis)

These are things you can determine by reading and analyzing the source code, before running anything.

#### Application Description

- **What it is:** A plain-language explanation of the problem being solved
- **Who needs it:** All user classes — this is the entry point for any new user
- **What it must answer:** What real-world problem does this application model? How representative of a specific scientific domain is it? Would domain experts recognize their work in it?
- **Why it matters:** Users can only evaluate the _quality_ of benchmark results if they understand what the application is actually doing. A benchmark measuring the "wrong" thing for a domain is worse than useless — it produces misleading conclusions.

#### Program Description

- **What it is:** Software engineering facts about the code
- **Includes:** Programming languages, lines of code, number of subroutines, system requirements (CPU, memory, I/O), software libraries used
- **Why it matters:** Tells users whether they can run it (do they have enough memory? The right compiler?), and gives developers a baseline for understanding code complexity.

#### Application Component Structure

- **What it is:** The coarse-grain decomposition of the application into major phases or components
- **Key questions:** Can components run in parallel with each other? What are the data dependencies between phases? Which phases are compute-bound vs. communication-bound?
- **Why it matters:** Determines what architecture is appropriate for the application. A benchmark with phases that must run sequentially has different machine requirements than one with independent parallel components.

#### Algorithmic Structure

- **What it is:** The mathematical/algorithmic decomposition of the application
- **Key questions:** What are the core algorithms? What is their theoretical complexity? What opportunities exist for algorithmic improvements?
- **Target audience:** Algorithm researchers specifically
- **Includes:** Measured improvements from algorithmic changes, not just theoretical analysis
- **Why it matters:** Researchers evaluating new algorithmic techniques need to know the baseline algorithms before they can measure improvement.

> [!note] The Uniform Naming Problem The paper identifies a critical practical problem: different tools use different naming schemes. Instruction analyzers use program counters. Call graphs use subroutine names. Compilers report on loops. There is no universal way to link these together. The paper calls development of unified naming schemes "an important future goal." This remains partially unsolved today.

### 3.2 Table 3 — Characteristics Gathered from Runtime Measurements (Dynamic Analysis)

These require actually running the code on hardware.

#### Measured Performance

- **What it is:** Baseline scalability data — how execution time changes with problem size and processor count
- **Includes:** Performance across different architectures, sensitivity to input parameter changes
- **Why it matters:** This is the "headline number" every user needs first. It tells you whether the benchmark is worth running on your machine at all, and provides the baseline that all improvements are measured against.

#### Analysis with Respect to Programming Models

- **What it is:** Separate analysis of the message-passing (MPI/PVM) and shared-memory (OpenMP) variants
- **MPI profile answers:** Is this architecture suitable for distributed memory? How does communication scale? What are the communication optimization opportunities? Is the application heterogeneous (different phases have different communication characteristics)?
- **OpenMP analysis answers:** Where are the sources of performance loss? What are the data-sharing patterns? What opportunities exist for shared-data optimizations?
- **Why it matters:** The same application can behave very differently under MPI vs. OpenMP. Understanding both tells you which programming model and architecture is the right fit.

#### I/O Characterization

- **What it is:** Analysis of disk read/write behavior — how much, when, and how it scales
- **Why it matters:** I/O is often the invisible bottleneck. A benchmark that looks compute-bound in a short run may be I/O-bound at production scale. Understanding I/O scaling tells you whether adding processors will help (if I/O doesn't scale, adding compute is useless).

#### Cache Analysis

- **What it is:** The application's locality properties — how it uses the memory hierarchy
- **Includes:** Data reuse behavior, data sharing in shared-memory versions, cache miss rates across program sections
- **Reference:** The paper cites a specific cache characterization model from related work (Kim, Voss, Eigenmann 1998)
- **Why it matters:** Cache behavior is increasingly the dominant factor in modern CPU performance. Applications that look identical in FLOPS counts can differ by 10× in actual runtime due to cache behavior.

#### Instruction Analysis

- **What it is:** The instruction mix — how many instructions of each type are executed
- **Categories:** Integer, floating-point, loads, stores, conditional branches
- **Includes:** Timing information (pipeline stalls), per-phase breakdowns
- **Why it matters:** Reveals whether the application is compute-bound, memory-bound, or branch-bound. Determines which hardware features (FPUs, prefetchers, branch predictors) are most critical.

#### Program Analysis

- **What it is:** Static compiler analysis combined with runtime profiling
- **Includes:** Call graphs, data use and access patterns, control-flow structure, compiler optimization results (what was applied, what failed and why)
- **Target audience:** Compiler researchers primarily; also valuable for manual code optimization
- **Why it matters:** If you're building a new compiler pass or optimization, you need to know what the existing compiler already handles before you can claim your technique adds value.

### 3.3 Table 4 — Characteristics Gathered from Performance Modeling

These require simulation or analytical modeling beyond raw measurement.

#### Simulation Analysis

- **What it is:** An open-ended category — simulations of the application on ideal or hypothetical machines
- **Why it matters:** Simulators can reveal the _theoretical_ maximum performance — what would happen if memory latency were zero, or if parallelism were infinite. This upper bound is critical context for interpreting measured results.

#### Advanced Model Analysis

- **What it is:** Application of formal performance prediction models from the literature
- **Examples cited:** Hierarchical Performance Bound Model (Boyd et al.), PTOPP model (Eigenmann), Perfore methodology (Armstrong & Eigenmann)
- **What they produce:** Upper bounds on achievable performance, identification of specific bottlenecks and their locations in the code, predictions of how performance will scale to unmeasured configurations
- **Why it matters:** Advanced models can give insight into a code's _complex behavior_ that raw measurements alone cannot — they can distinguish between causes of performance loss that look identical in profiling data.

---

## 4. Run Requirements — The Scientific vs. Industrial Tension

Section 3.1 is short but important. It makes a deliberate design choice that distinguishes this methodology from industrial benchmark suites.

### The Tension

|Industrial Benchmark (e.g., official SPEC)|This Methodology|
|---|---|
|Strict run rules — you must do X, Y, Z exactly|Flexible — you can modify the code|
|Formal result submission process|Informal reporting requirements|
|Reproducibility guaranteed by compliance|Reproducibility guaranteed by disclosure|
|Optimized for apples-to-apples comparison|Optimized for scientific research flexibility|

### The Solution

Instead of prescribing _what you must do_, the methodology prescribes _what you must report_:

1. **Precise execution environment** — machine parameters, operating system, compiler flags (everything that affects the result)
2. **All code modifications** — every change made to the released benchmark version, with rationale
3. **Subset justification** — if only some benchmarks were run, explain why the others were omitted

> [!important] The Principle Behind This This is the scientific equivalent of a methods section in a journal paper. You are allowed to deviate from the standard protocol — but you must fully disclose what you did so others can interpret your results and reproduce your work. Transparency replaces compliance.
> 
> This maps directly to your own project's need: your Fisher-Yates shuffle and worst-case memory fragmentation are exactly this kind of methodological choice — legitimate, but requiring clear documentation and justification.

---

## 5. The Case Study — SPECseis, Step by Step

Section 4 of the paper applies the full methodology to the seismic processing benchmark. This section walks through every part of the case study with annotations.

### 5.1 The Application

**SPECseis** (also called ARCO suite, Seismic) processes seismic signals from a source moving along a grid, reflected off earth's interior structures, received by receptor arrays. The signals are _seismic traces_, and the application applies a sequence of data transformations to them.

**Why this application as a benchmark?** It was designed explicitly for parallel computer benchmarking, reflects real computational methods from the seismic industry, is public domain (avoids proprietary code problems), and has been included in multiple benchmark suites (so it's already somewhat shared).

### 5.2 The Four Phases

The application is organized into four sequential phases. Understanding the phase structure is essential for interpreting every figure in the paper:

|Phase|Name|What It Does|Key Characteristics|
|---|---|---|---|
|1|Data Generation|Generates synthetic seismic data, applies filters and corrections|Highly parallel, little communication, write-heavy I/O|
|2|Stacking of Data|Sums traces into zero-offset section (compression step)|High communication-to-computation ratio, read+write heavy|
|3|Time Migration|3-D Fourier domain migration|Very short, only 3 communication operations regardless of input size|
|4|Depth Migration|3-D finite difference migration|Large computational load, communication overhead hidden by computation|

> [!note] Key Architectural Implication The phases have different optimal architectures. Phase 1 is suitable for distributed-memory (loosely coupled) machines. Phases 2 and 4 scale best on machines with low communication latency (tightly coupled). Phase 3 is so short it barely matters. This heterogeneity is a core finding — the "right" machine for this application depends on which phase dominates your workload.

### 5.3 Program Description Facts

- **Size:** 20,000 lines of Fortran and C
- **Structure:** ~230 Fortran subroutines (computational), ~120 C routines (I/O, data partitioning, message passing)
- **Libraries:** None external — entirely self-contained
- **Datasets:** 5 datasets ranging from 16 MB (test) to 1,536 MB; the largest requires ~1 day on a 1 GFlops machine and 100 GB disk

### 5.4 Figure-by-Figure Breakdown

The paper includes 13 figures. Here is what each shows and why it matters:

**Figure 1 — Performance vs. dataset size on 4-processor SUN Ultra Enterprise 4000**

- Shows 5 different datasets (DS1–DS5) across different numbers of processors (1–10)
- Key observation: execution time scales predictably with dataset size on a log-log scale
- DS4 and DS5 are both 96 MB but have different dimensions — shows that data layout within a fixed size matters, not just total size

**Figure 2 — Per-phase performance on 16-processor SGI PowerChallenge**

- Breaks down execution by Phase 1–4 at different processor counts
- Key observation: Phase 1 scales well (parallel-friendly), Phase 2 scales worst (communication bottleneck), Phase 4 scales well despite high communication because computation dominates

**Figure 3 — Cross-architecture performance comparison**

- Shows Seismic on SUN, SGI, HP, Dell machines at varying processor counts
- Key observation: good scalability is maintained across all architectures — the benchmark is portable, not tuned to one machine

**Figure 4 — Communication profiles for Phase 2 and Phase 4 (3-second windows)**

- Shows how many processors are waiting for communication at any point in time
- Key observation: Phase 2 has much denser communication barriers (more processors idle simultaneously) than Phase 4, confirming the communication-to-computation ratio difference

**Figure 5 — Total communication time per processor (4, 8, 16 processors)**

- Shows the distribution of communication time across processor IDs
- Key observation: **significant imbalance** — some processors spend much more time communicating than others. This is a hidden bottleneck not visible in aggregate timing.

**Figure 6 — Average time between communication points per phase**

- Shows computation granularity: the average time between two consecutive communication operations
- Key observation: Phase 2 has the shortest computation bursts between communications, making it most sensitive to communication latency

**Figure 7 — Shared-memory analysis: productive vs. barrier wait time per thread**

- From the OpenMP implementation
- Key observation: wait times are very small (barriers are cheap), which explains the good scalability despite regular synchronization. Number of barriers per processor also shown.

**Figure 8 — Percentage of execution time in parallel loops (Polaris compiler analysis)**

- The Polaris automatic parallelizing compiler identifies which loops are parallel
- Key observation: varies significantly by phase. Some phases have most execution time in loops Polaris can parallelize; others have a large serial fraction. This limits the speedup achievable without manual parallelization.

**Figure 9 — Disk I/O read/write time by phase at varying processor counts**

- Separates read and write operations for each phase
- Key observation: Phase 1 writes 1.5 GB of trace data. Phase 2 reads this and writes a 70 MB compressed file. Phases 3 and 4 read the compressed file. Write time for Phases 2 and 4 stays near 4 seconds regardless of processor count — a fixed I/O bottleneck that limits overall scaling.

**Figure 10 — Instruction mix by phase (serial run, Dataset 3)**

- Breaks instructions into: Integer, Floating-Point, Loads, Stores, Conditional, Control
- Key observation: Despite being a scientific application, integer and floating-point instruction counts are roughly equal. Phase 4 is particularly load-heavy (1/4 of instructions are loads).

**Figure 11 — Branch outcomes (taken vs. not taken) for integer and FP branches**

- Shows branch predictability per phase
- Key observation: considerable variation — some phases have highly predictable branches, others have near-50/50 splits (hardest for hardware predictors)

**Figure 12 — Call graph and loop table for DCON subroutines (Phase 1)**

- Shows the nesting structure of subroutine calls and the parallel/serial status of each loop
- The loop table links compile-time information (serial/parallel classification) with runtime timing
- Key observation: the most time-consuming loops are the parallel ones, confirming that parallelization targets the right bottlenecks

**Figure 13 — Maximum theoretical parallelism in Phase 2 (MaxPar simulation)**

- Simulates execution on an ideal parallel machine with no communication overhead
- Key observation: up to **several million** operations could execute concurrently at any point. The actual exploited parallelism (from earlier figures) is orders of magnitude lower. This demonstrates that the current parallel implementation barely scratches the code's inherent potential.

> [!important] The Most Striking Result Figure 13 is the paper's most provocative finding. Even a well-parallelized code running on real hardware with 16 processors exploits only a tiny fraction of the theoretical parallelism. This has two implications: (1) there is enormous room for improvement, and (2) measuring parallelism on real hardware massively underestimates what the algorithm itself is capable of.

---

## 6. Key Arguments and How They Connect

### Argument 1: Realism is not optional for certain research questions

The paper argues this with the "adaptive optimization" example (Introduction). The implication is that you cannot know _in advance_ whether a scaled-down benchmark is adequate — it depends on your research question. Therefore, having realistic large-scale benchmarks available is necessary infrastructure, even if you don't always use them.

### Argument 2: Benchmark characterization must be structured and shared

Individual research groups characterize benchmarks ad-hoc (cache studies, compiler analyses) using incompatible metrics and naming conventions. The methodology enforces a common vocabulary that makes these contributions additive rather than isolated.

### Argument 3: Flexibility and comparability are not mutually exclusive

The run requirements section (3.1) resolves the apparent conflict: you can allow code modifications (flexibility for research) and still have comparable results, as long as disclosure requirements are strict. The methodology trades formal compliance for formal transparency.

### Argument 4: Understanding a benchmark has multiple levels

The information categories are not just a checklist — they form a hierarchy from surface (application description) to deep (simulation analysis). A user who only needs to run the benchmark needs level 1–2. A user building a compiler pass needs levels 5–8. A user building a new architecture needs all levels. The methodology serves all three.

---

## 7. Terminology Glossary

|Term|Definition in this paper|
|---|---|
|**SPEChpc**|SPEC's High-Performance computing benchmark suite; includes SPECseis, SPECchem, SPECclimate|
|**SPEC/HPG**|SPEC High-Performance Group — the working group that produced SPEChpc|
|**Run rules**|Formal requirements governing how a benchmark must be executed for a result to be officially valid|
|**Benchmark diary**|Perfect Club's methodology — users document all code modifications and their performance effects|
|**Parkbench**|Parallel benchmark effort defining multiple system levels; organized by level of abstraction|
|**PVM**|Parallel Virtual Machine — a message-passing library (precursor to MPI)|
|**OpenMP**|Shared-memory parallelism via compiler directives; used in the directive-parallel variant of SPEChpc|
|**Seismic trace**|The fundamental data unit in SPECseis — a time-series of reflected acoustic signals|
|**Common-midpoint stacking**|The key operation in Phase 2 — averages multiple traces to amplify true reflections, reduces data volume|
|**Polaris**|Automatic parallelizing compiler from UIUC used for program analysis in this paper|
|**MaxPar**|Tool that simulates execution on an ideal parallel machine to reveal maximum theoretical parallelism|
|**PTOPP model**|Performance model defining overhead factors (globalization penalty, parallelization overhead, spreading overhead)|
|**Perfore**|Methodology that extracts Resource Usage Equations to characterize application scaling|
|**Information category**|The paper's term for each type of data that must be collected about a benchmark|

---

## 8. Limitations the Paper Acknowledges (and Those It Doesn't)

### Explicitly Acknowledged

- The scope of one paper cannot fully characterize even a single SPEChpc benchmark — Figure 13 (simulation) and the advanced model section are deferred to a companion web repository
- The uniform naming scheme problem (linking counters to subroutines to loops) is identified as a future goal, not a solved problem
- The Web repository (www.ece.purdue.edu/ParaMount/Benchmark) is described as "being created" — the paper is announcing intent as much as describing a completed system

### Implicitly Not Addressed

- The methodology says nothing about statistical rigor — no guidance on run counts, variance reporting, or confidence intervals. This is the gap that the Reliable Benchmarking paper (your second Week 1 reading) directly addresses.
- The run requirements are still vague about _how_ to disclose execution environment — what level of detail about compiler flags, OS configuration, etc.
- The methodology is HPC-centric and assumes MPI + OpenMP as the relevant programming models. It does not address GPU computing, which was emerging but not yet dominant.
- All case study work is on one benchmark (SPECseis). The claim that the methodology generalizes to SPECchem and SPECclimate is asserted, not demonstrated.

---

## 9. Connection to Your Project (The Memory Wall)

Use this section to bridge the paper's ideas to your own benchmarking work.

### Where the Paper's Methodology Applies Directly

|Paper Category|Your Project's Current State|Gap / Action|
|---|---|---|
|Application description|README explains the Memory Wall concept|Needs workload representativeness argument — _which real applications are traversal-dominated?_|
|Program description|16 structures, C11, ~6641 samples documented|Complete — this is well done|
|Component structure|Three families (Contiguous/Pointer/Cache-Aware)|Good taxonomy; needs literature grounding for the classification|
|Algorithmic structure|Traversal as the measured operation|Needs formal statement of _why_ traversal and what it assumes away|
|Measured performance|Extensive — 5 orders of magnitude|Strong; needs cross-architecture data to match paper's Figure 3|
|Programming model analysis|Not applicable (sequential access patterns)|N/A for your scope|
|I/O characterization|Not measured|Out of scope — but should be stated explicitly|
|Cache analysis|LLC miss rate measured; 0.90 correlation found|Core finding — needs to be _argued_, not just reported|
|Instruction analysis|IPC measured|Needs per-structure breakdown like Figure 10|
|Program analysis|Not done|Future work|
|Simulation analysis|Not done|Future work — MaxPar equivalent would be interesting|

### The Run Requirements Principle Applied to Your Project

The paper's run requirement philosophy (disclose everything, prescribe nothing) maps to your project as follows. For every result you report, a reader needs:

1. Exact CPU model and microarchitecture (not just "Zen 3")
2. RAM speed, capacity, channel count, and latency timings
3. OS distribution and kernel version
4. GCC/Clang version and complete flag string
5. glibc version (malloc implementation matters for your results)
6. perf_event_paranoid setting
7. CPU frequency governor confirmation method
8. The Fisher-Yates shuffle (you have this — foreground it, it's a methodological contribution)
9. Run count policy and its justification

### The "Abstraction Problem" Applied to Your Project

Your traversal-bound benchmark makes a specific abstraction: that mutation (insertion, deletion) performance is not measured. The paper's abstraction principle says this is acceptable _if your research question is about traversal-dominated workloads_. You need to explicitly state which real workloads justify this scope:

- OLAP query engines (full table scans)
- Neural network inference (sequential weight reads)
- Graph analytics (BFS/DFS traversal)
- Network packet classification (sequential rule matching)
- Scientific simulation (array sweeps)

These are your representativeness argument. Without them, the benchmark appears arbitrarily scoped.

---

## 10. Questions to Answer While Reading the Paper

Use these as active reading prompts. Try to answer each before moving to the next section.

1. **Introduction:** What two properties does Armstrong say a benchmark needs? Why does the research community currently lack suites with both?
    
2. **Section 2:** What criteria does SPEC/HPG use to select applications for SPEChpc? How many are in the suite at time of writing?
    
3. **Section 3 opening:** How does this methodology differ from the Perfect Club benchmark diary approach? What is the tradeoff?
    
4. **Table 2:** Which information category is aimed at _all_ user classes? Which is primarily for _compiler researchers_?
    
5. **Section 3.1:** What three things must every evaluation project report to maintain comparability while still allowing code modifications?
    
6. **Section 4 — Application Description:** Why was SPECseis created as a benchmark rather than using a proprietary application? What is the "relevance as a benchmark" claim?
    
7. **Figure 2:** Which phase scales best? Which scales worst? What is the communication-to-computation ratio for Phase 2 compared to Phase 4?
    
8. **Figure 5:** What does the imbalance in per-processor communication time imply for potential optimization?
    
9. **Figure 13:** How many concurrent operations does MaxPar find available in Phase 2? How does this compare to the parallelism actually exploited on real hardware?
    
10. **Section 5 — Conclusions:** What three specific limitations does the paper identify in the Seismic benchmark's parallel implementation?
    

---

## 11. What to Look for When You Read

### Sentences That Signal Key Claims

Watch for the word _"however"_ — every time it appears, a limitation or complication is being introduced. These are often the most important sentences.

Watch for _"we recommend"_ — these signal the normative/prescriptive content, as distinct from the descriptive content.

Watch for _"for example"_ — Armstrong uses concrete examples to anchor abstract methodology claims. These examples carry the argument.

### The Tables

Tables 2, 3, and 4 are the deliverable of the paper — everything else motivates and illustrates them. Read them slowly. Each row is a design decision about what matters.

### The Figures

The figures are not just results — they are demonstrations that each information category can be measured and is meaningful. Figure 1 demonstrates "Measured Performance." Figure 5 demonstrates why "Analysis with respect to programming models" matters (the imbalance would be invisible from Figure 1 alone). Keep this in mind: each figure is evidence for one information category in the taxonomy.

---

_Note created: {{date}}_ _Source paper: Armstrong, B. & Eigenmann, R. (2000). A Methodology for Scientific Benchmarking with Large-Scale Applications. Purdue University._ _Project: [[Benchmarking Project]] | Phase: [[Week 1 — Foundational Methodology]]_ _Next reading: [[Reliable Benchmarking Tools and Practices — Leufelder et al.]]_