# Benchmarking and Performance

This repository is a working research space for a literature survey on benchmarking modern computing systems. The project is currently focused on understanding how strong benchmarks are designed, justified, reported, and made reproducible.

The immediate milestone is a polished paper presentation on **"How to Build a Benchmark"** by Kistowski et al. (ICPE 2015). That paper is being used as the first framing piece for the survey because it gives a practical vocabulary for benchmark quality: relevance, reproducibility, fairness, verifiability, and usability.

## Current Project State

The project has moved from initial paper collection into the first structured reading and presentation phase.

- Collected a small methodology-focused paper set in [`Literature-Survey/`](Literature-Survey/).
- Drafted detailed notes for **How to Build a Benchmark**.
- Built a 16:9 LaTeX presentation in [`Presentation/`](Presentation/).
- Established the current design direction for presentations: dark manuscript style, cream text, muted gold accents, and sparse academic layout.

## First Presentation

The first completed draft is:

- [`Presentation/01 - How-to-Build-a-Benchmark.md`](Presentation/01%20-%20How-to-Build-a-Benchmark.md) — detailed notes and reading synthesis
- [`Presentation/01 - How-to-Build-a-Benchmark.tex`](Presentation/01%20-%20How-to-Build-a-Benchmark.tex) — LaTeX source
- [`Presentation/01 - How-to-Build-a-Benchmark.pdf`](Presentation/01%20-%20How-to-Build-a-Benchmark.pdf) — compiled presentation

This presentation frames benchmarking as more than running a workload. A benchmark is treated as a full measurement system: workload, metric, run rules, validation, reporting, and governance.

## Survey Questions

The current survey is organized around these questions:

- What makes a benchmark scientifically useful rather than only a ranking?
- How do benchmark designers choose workloads that represent real systems and users?
- How do established suites balance relevance, reproducibility, fairness, verifiability, and usability?
- Which details must be reported for another researcher to interpret or reproduce a result?
- How do benchmark rules prevent gaming, cherry-picking, and misleading comparisons?
- How do modern settings such as MLPerf, cloud systems, and energy measurement complicate older benchmark methodology?

## Literature Survey Materials

Current papers and resources are stored in [`Literature-Survey/`](Literature-Survey/). The collection currently includes work on:

- Benchmark design methodology
- Reliable benchmarking practice
- Fairness and reproducibility
- Common benchmarking pitfalls
- Algorithm benchmarking
- HPC and large-scale system benchmarking

The notes file [`Literature-Survey/Paper Notes.md`](Literature-Survey/Paper%20Notes.md) is used for ongoing reading synthesis and cross-paper connections.

## Working Direction

The goal at this stage is not yet to define a final benchmark. The goal is to build a rigorous methodological foundation for later benchmark design.

Near-term work:

- Continue turning individual papers into notes and short presentations.
- Compare how SPEC, TPC, MLPerf, and academic benchmark papers define quality.
- Build a reusable template for analyzing each benchmark paper.
- Identify which benchmark design choices matter most for the eventual project direction.

The current guiding idea: a strong benchmark should produce evidence that other people can interpret, challenge, and reproduce.
