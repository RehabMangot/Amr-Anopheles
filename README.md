# *Anopheles albimanus* Midgut Microbiome and Insecticide Resistance

A Google Colab pipeline that profiles the midgut microbiome of insecticide-resistant vs. susceptible *Anopheles albimanus* mosquitoes, and screens it for microbial genes linked to insecticide detoxification.

> **Note on scope:** this notebook analyzes the *An. albimanus* / fenitrothion dataset only. 
## Background

Insecticide resistance in *Anopheles* mosquitoes is usually studied at the host genome level (target-site mutations, detoxification gene amplification). This pipeline instead asks whether the **midgut bacterial community** itself differs between resistant and susceptible mosquitoes, and whether that community carries genes capable of degrading insecticides — a possible microbiome-mediated contribution to resistance.

## Data

| Source | Species | Insecticide | Samples |
|---|---|---|---|
| [PRJNA388280](https://www.ncbi.nlm.nih.gov/bioproject/PRJNA388280) | *An. albimanus* | Fenitrothion (organophosphate) | SRR5630719 (resistant), SRR5630720 (susceptible) |

One sample per phenotype, subsampled to 100,000 read pairs each, to keep runtime and storage manageable on Colab.

**⚠️ Statistical caveat:** every downstream comparison (diversity, differential abundance, gene screening) is n = 1 per phenotype — two individual mosquitoes, no biological replicates. Results should be read as a **pipeline demonstration**, not a validated population-level finding, until replicated with more samples.

## Pipeline

1. **Environment setup** — mount Google Drive (for lightweight outputs only), install command-line tools.
2. **Metadata & data fetch** — pull run accessions via NCBI Entrez Direct, download subsampled reads with `fastq-dump`.
3. **Quality control** — adapter trimming and quality filtering with `fastp`.
4. **Host depletion** — align reads to the *An. albimanus* reference genome (BWA) and keep only unmapped (non-host, presumed microbial) reads.
5. **Taxonomic classification** — classify host-depleted reads with **Kraken2** (Standard-8 database) to genus level.
6. **Alpha & beta diversity** — Shannon diversity index per sample; Bray–Curtis distance and PCoA between samples.
7. **Differential abundance** — Welch's t-test with Benjamini–Hochberg FDR correction on genus-level abundances; volcano plot and comparative barplot.
8. **Resistance/detox gene screening** — nucleotide motif screening of host-depleted reads for microbial detoxification gene signatures (opd/mpd, carboxylesterase, GST, P450-like, estP), normalized to reads per million (RPM).
9. **Functional prediction** — map classified genera to KEGG pathways (xenobiotics biodegradation, cytochrome P450 metabolism, glutathione metabolism) and visualize pathway enrichment.
10. **Results sync** — copy only the small result folders (CSVs + PNGs, a few MB) to Google Drive; heavy intermediate files (raw reads, reference genome, ~8 GB Kraken2 DB) stay on Colab's local disk and are discarded when the session ends.

## Requirements

**Command-line tools** (installed in-notebook via `apt-get`, designed for a Debian/Ubuntu-based Colab runtime):
- `sra-toolkit` (`fastq-dump`)
- `bwa`
- `samtools`
- `fastp`
- `kraken2` (+ Standard-8 prebuilt database, downloaded in-notebook, ~8 GB)
- NCBI Entrez Direct (`esearch`/`efetch`, installed via NCBI's install script)

**Python** (3.11) — see `requirements.txt`:
- biopython, pandas, numpy, scipy
- matplotlib, seaborn, adjustText

## Usage

1. Open the notebook in Google Colab (or a local Jupyter environment with the tools above installed and on `PATH`).
2. Run cells top to bottom. The first cell mounts Google Drive — only the final `results` folders are written there; everything else stays on local/ephemeral disk.
3. Expect the Kraken2 database download (~8 GB) to take several minutes; keep the runtime active during this step.
4. Final outputs (tables + figures) land under `taxonomy_results_albimanus/`, `phase4_diversity_analysis/`, `phase4_functional_prediction/`, and `resistance_gene_screening/`, and are synced to Drive at the end.

## Repository structure

```
.
├── albimanus_midgut_microbiome_AMR.ipynb   # main pipeline notebook
├── requirements.txt                        # Python dependencies
├── README.md
```

Raw reads, the reference genome, and the Kraken2 database are **not** tracked in this repo (see `.gitignore`) — they're multi-gigabyte and fully reproducible by re-running the notebook.

## Limitations

- **n = 1 per phenotype** — see caveat above; treat all statistics as exploratory.
- Kraken2 classification uses the capped Standard-8 database, not the full standard DB or PlusPF, so rare/unusual taxa may be missed.
- Motif-based detox gene screening detects sequence signatures, not confirmed functional expression.

## Citation

If this pipeline is useful in your work, please cite this repository and the original data source (BioProject PRJNA388280).
