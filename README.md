# Kintsugi

*金継ぎ — the Japanese art of repairing broken pottery with gold, where
the cracks are highlighted rather than hidden.*

A platform for interpreting **homologous-recombination deficiency (HRD)** —
the biomarker that decides whether a cancer will respond to PARP-inhibitor
therapy (olaparib, niraparib, rucaparib). The philosophy of the name is
literal: HRD matters precisely *because* something is broken, and the
point of the platform is to make that brokenness legible to patients
and clinicians from every angle it can be observed — variants, imaging,
tumor scars, drug-match assessments — instead of pretending one assay
gets it right.

---

## The product: [`drug-cell-viz`](https://github.com/patrisiyarum/drug-cell-viz)

A patient-facing web app that puts everything in one screen, designed
for the moment a patient is preparing for an oncology appointment.

### What a patient gets

- **Patient profile** — a stable record per person. Medications they're on,
  symptoms they're tracking, scans + reports they've uploaded. Persists
  across visits via a Postgres-backed API; not localStorage-tier.
- **HR-deficiency status** — a calibrated HRD call combining BRCA1/2
  pathogenic variants, FDA-recognized moderate-penetrance HR genes
  (RAD51C/D, BRIP1, BARD1, PALB2), and ML-predicted BRCA1 loss-of-function.
- **CT-based HRD prediction** — upload a preoperative CT (DICOM zip,
  NIfTI, or even the NBIA `.tcia` manifest TCIA hands you), and the
  radiogenomics 3D CNN scores HR-deficiency probability directly from
  imaging. Renders the volume in a 3D multiplanar viewer (niivue) too.
- **BRCA1 / BRCA2 variant-effect prediction** — XGBoost + AlphaMissense
  ensemble, **AUROC 0.933 on held-out Findlay 2018 SGE data**, with
  calibrated conformal prediction intervals.
- **"Is my current drug right for me?"** — a CPIC + FDA-grounded verdict
  on the patient's current medication. When the imaging model flags HRD
  but the patient is on a non-PARPi, the page explicitly suggests a
  PARP-inhibitor conversation.
- **3D molecular view** — the drug bound to its target protein
  (AlphaFold DB), with the patient's variant residues highlighted.
- **Doctor-visit PDF** — printable one-pager with the HRD result, drug
  verdict, and questions to ask. The closing action of every patient flow.

### Stack

Next.js 15 + TypeScript on the frontend (Mol\* + niivue for 3D),
FastAPI on the backend with SQLModel/Postgres + Redis + an ARQ worker,
38 tests under GitHub Actions CI, deployed via a one-click Render
Blueprint.

### Status

Shipped. Live demo flows for three patient personas (Maya, Diana, Priya)
plus an open `/build` flow for arbitrary input. Open source, MIT-licensed.

---

## The research backing: [`hrd-radiogenomics`](https://github.com/patrisiyarum/hrd-radiogenomics)

The CT → HRD model that powers the imaging arm of the product, plus
its full reproducible training pipeline.

### What's in the repo

- **Snakemake DAG** that joins 10,102 pan-cancer TCGA HRD labels
  (Knijnenburg 2018, via GDC) with TCIA NBIA imaging into a 135-patient
  TCGA-OV paired-imaging-and-genomics cohort, preprocesses each CT
  (DICOM → 96³ HU-windowed cube), and trains end-to-end.
- **Two model arms in the same DAG, evaluated on the same held-out
  test split (108 dev / 27 test, stratified by HRD class × scanner):**
  - **Deep arm** — 3D ResNet-50 (Med3D architecture with the actual
    Tencent MedicalNet pretrained weights, augmentation + class-weighted
    BCE), 5-fold CV ensemble.
  - **Classical-radiomics arm** — PyRadiomics features (intratumoral +
    peritumoral) fused with clinical features (age, FIGO stage, tumor
    grade from GDC), Mann-Whitney univariate filter + LASSO logistic.
    Re-implements Pan et al. 2024 (Front Oncol) on a public cohort.
- **Held-out test discipline** — the test patients are split off
  *before* any training touches the data; one-shot evaluation at the
  end. Numbers are generalization, not optimism.
- **TRIPOD+AI / CLAIM 2.0 / PROBAST+AI** reporting checklists filled
  pre-release (`reports/`).

### Honest results (small-N reality)

| Model | Test AUROC | Notes |
|---|---|---|
| BRCA1 variant classifier (in `drug-cell-viz`) | **0.933** | 729 held-out Findlay 2018 SGE variants. Real research-tier number. |
| BRCA2-DBD classifier (in `drug-cell-viz`) | **0.842** | 881 held-out Huang 2025 SGE variants. |
| Radiogenomics CT model — deep, v2 | **0.59** (test) / 0.69 (CV mean) | 27-patient held-out. The dataset is the bottleneck, not the model. |
| Radiogenomics CT — classical radiomics + clinical | *pipeline built; not yet run* | Predicted 0.65–0.72 based on Pan et al. on a similar cohort. |

The BRCA-variant classifiers are publishable-quality at this scale.
The CT model is research-tier — within the range of what published
radiogenomics work gets at ~135 patients, and a clear demonstration
that the bottleneck is paired imaging-genomics data (not algorithms).

---

## Why multi-modal — the design thesis

The HRD label is contested. The Friends of Cancer Research HRD
Harmonization Project (2024) ran 20 independent HRD assays on matched
tumor samples and declined to declare a gold standard. Roughly 30% of
Myriad-HRD-positive patients still fail PARP inhibitors because
scar-score tests are **historical** — they see past damage but miss
live reversion mutations that happen during therapy. ~30% of
HRD-positive ovarian tumors are HR-deficient via somatic events
(BRCA1 promoter methylation, somatic BRCA loss) that germline panels
can't see.

Kintsugi's answer isn't "pick the right assay." It's "combine the
imperfect signals we have into a single calibrated call that shows its
work" — variants, SV-derived scars, imaging, drug match — and surface
the gaps explicitly (the patient sees both the prediction and the
caveat, in the same UI element).

That's the gold seam. Brokenness made visible, not hidden.

---

## Data sources

All public, all cited in each sub-repo.

- [Findlay et al. 2018](https://pmc.ncbi.nlm.nih.gov/articles/PMC6181777/) — BRCA1 SGE functional scores (3,893 SNVs)
- [Huang et al. 2025](https://www.nature.com/articles/s41586-024-08388-8) — BRCA2 DBD SGE functional scores (4,404 missense)
- [AlphaMissense (DeepMind)](https://www.science.org/doi/10.1126/science.adg7492) — 216M precomputed missense scores
- [AlphaFold DB](https://alphafold.ebi.ac.uk/) — per-gene protein structures
- [Knijnenburg et al. 2018 (GDC)](https://gdc.cancer.gov/about-data/publications/panimmune) — pan-cancer HRD labels
- [TCIA TCGA-OV](https://www.cancerimagingarchive.net/collection/tcga-ov/) — paired CT imaging (CC-BY 3.0, Holback et al. 2016)
- [Pan et al. 2024 (Front Oncol)](https://www.frontiersin.org/journals/oncology/articles/10.3389/fonc.2024.1477759/full) — radiomics nomogram baseline we re-implement
- [CPIC](https://cpicpgx.org/) / FDA labels — pharmacogenomic guidance
- [BRCA Exchange](https://brcaexchange.org/) / ENIGMA — expert panel classifications

---

## Disclaimer

Research and educational use only. **Not a medical device.** Not FDA-cleared.
Every prediction reports held-out performance metrics and known limitations.
All evidence is summarized from public sources with citations. Consult a
qualified oncologist and a clinical pharmacogenomicist before any
treatment decision. Genetic testing must be performed by a CLIA-certified
laboratory.

## License

MIT for source in both sub-repos. Pretrained model weights inherit the
license of their source corpora where applicable.
