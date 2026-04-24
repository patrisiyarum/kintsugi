# Kintsugi

*金継ぎ — the Japanese art of repairing broken pottery with gold, where
the cracks are highlighted rather than hidden.*

Kintsugi is a two-repo platform for interpreting **homologous-recombination
deficiency (HRD)** — a biomarker for broken DNA-repair machinery that
determines whether a cancer will respond to PARP-inhibitor therapy
(olaparib, niraparib, rucaparib). The philosophy of the name is literal:
HRD matters precisely *because* something is broken, and the point of
the platform is to make that brokenness legible to patients and clinicians
from every angle it can be observed.

## Repositories

| Repo | Layer | What it does |
|---|---|---|
| [**drug-cell-viz**](https://github.com/patrisiyarum/drug-cell-viz) | Product | Patient-facing web app. Takes variants (23andMe file, clinical VCF, catalog, or pasted protein sequence) and returns an HRD composite score, a "right drug?" second-opinion verdict, a 3D molecular view, and a printable doctor-visit PDF. Next.js 15 + FastAPI + Postgres + Redis + ARQ. |
| [**hrd-radiogenomics**](https://github.com/patrisiyarum/hrd-radiogenomics) | Research | CT → HRD 3D deep-learning pipeline. Snakemake + MONAI on TCGA-OV × TCIA paired data. Trained checkpoint drops into `drug-cell-viz` via a shared preprocessing contract. |

## What's the actual output?

Given a patient, Kintsugi looks at every evidence layer it can reach and
produces a single calibrated HRD call with the receipts attached:

- **Germline + somatic variants** — BRCA1/2 pathogenic calls, moderate-
  penetrance HR genes (RAD51C/D, BRIP1, PALB2, BARD1), Fanconi anemia
  family, plus ML-predicted BRCA1 loss-of-function.
- **Structural-variant HRD scars** — LOH + LST + NtAI aggregated from a
  pangenome-graph SV caller into a Myriad-compatible HRD-sum.
- **Radiogenomic signal** — 3D CNN prediction from a preoperative CT
  (TCGA-OV-trained).
- **Drug match assessment** — is the patient's current medication well-
  matched to their HRD status, and are better-matched alternatives
  available?

Every prediction reports held-out performance metrics and cites its
source. Nothing is presented as a treatment recommendation.

## Why multi-modal

The HRD label is contested. The Friends of Cancer Research HRD
Harmonization Project (2024) ran 20 independent HRD assays on matched
tumor samples and declined to declare a gold standard. Roughly 30% of
Myriad-HRD-positive patients still fail PARP inhibitors because
scar-score tests are **historical** — they see past damage but miss
live reversion mutations that happen during therapy.

Kintsugi's answer isn't "pick the right assay." It's "combine the
imperfect signals we have into a single calibrated call that shows its
work" — genomic variants, SV-derived scars, imaging, and (when
available) functional assays.

## Status

- **`drug-cell-viz`** — shipped, deployed on Render (one-click Blueprint),
  38 tests under GitHub Actions CI. BRCA1 classifier: XGBoost + AlphaMissense
  ensemble, AUROC 0.933 with conformal prediction intervals. BRCA2-DBD
  classifier: AUROC 0.842.
- **`hrd-radiogenomics`** — v0 trained on Lambda Labs A10, mean CV AUROC
  0.62 (random-init DenseNet121 baseline). v1 with Med3D ResNet-50
  architecture + augmentation + class-weighted loss in progress. TRIPOD+AI
  and CLAIM 2.0 reporting checklists completed pre-release.

## Tech stack

- **Frontend** — Next.js 15, TypeScript, Mol* 3D viewer, client-side
  23andMe parsing
- **Backend** — FastAPI (async Python), Postgres, Redis, ARQ worker
- **ML** — PyTorch, MONAI, XGBoost, scikit-learn, AlphaMissense, conformal
  prediction
- **Genomics** — cyvcf2, pangenome graphs (`vg giraffe` on HPRC),
  NVIDIA Clara Parabricks (GPU FASTQ → VCF)
- **Infra** — Snakemake pipelines, `uv` Python package management,
  Docker, Render Blueprint deploy, Lambda Labs A10 for training
- **Reporting** — TRIPOD+AI, CLAIM 2.0, PROBAST+AI checklists

## Data sources

All public, all cited in the relevant repo. See each sub-repo's README
for attribution.

- [Findlay et al. 2018](https://pmc.ncbi.nlm.nih.gov/articles/PMC6181777/) — BRCA1 SGE functional scores
- [Huang et al. 2025](https://www.nature.com/articles/s41586-024-08388-8) — BRCA2 DBD SGE functional scores
- [AlphaMissense](https://www.science.org/doi/10.1126/science.adg7492) — 216M precomputed missense scores
- [AlphaFold DB](https://alphafold.ebi.ac.uk/) — per-gene protein structures
- [Knijnenburg et al. 2018 (GDC)](https://gdc.cancer.gov/about-data/publications/panimmune) — pan-cancer HRD labels
- [TCIA](https://www.cancerimagingarchive.net/) — TCGA-OV CT imaging
- [CPIC](https://cpicpgx.org/) / FDA labels — pharmacogenomic guidance
- [BRCA Exchange](https://brcaexchange.org/) / ENIGMA — expert panel classifications

## Disclaimer

Research and educational use only. **Not a medical device.** Not FDA-cleared.
Every prediction reports held-out performance metrics and known limitations.
All evidence is summarized from public sources with citations. Consult a
qualified oncologist and clinical pharmacogenomicist before any treatment
decision. Genetic testing must be performed by a CLIA-certified laboratory.

## License

MIT for source in both sub-repos. Pretrained model weights inherit the
license of their source corpora where applicable (see each repo).
