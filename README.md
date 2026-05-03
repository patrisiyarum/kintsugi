# Kintsugi

**Try it live:** https://kintsugi-lc08.onrender.com/demo

*金継ぎ (kintsugi) is a Japanese art form. When a bowl breaks, you fill the
cracks with gold instead of hiding them. The bowl is more beautiful than
before because you can see what happened to it.*

This project does the same thing for cancer treatment. There's a kind
of broken DNA repair called HRD (homologous recombination deficiency).
About one in three breast and ovarian cancers have it. When a cancer
has HRD, a specific type of medication called a PARP inhibitor often
works really well. When a cancer doesn't have HRD, those same
medications don't help much.

The problem is that figuring out whether a cancer has HRD is hard.
There are many different tests. They don't always agree. Some patients
who test positive still don't respond to the medication. Some patients
who test negative actually have HRD that the test missed.

Kintsugi helps patients and their doctors look at this question from
several angles at once, and shows them where the cracks are.

---

## What's in this project

Two things, working together.

### 1. The website: https://drug-cell-viz-web.onrender.com/

A free website a patient can use at home before they see their oncologist.

Patients can:

- **Save a profile.** Their name, age, what they were diagnosed with, what
  medications they're on. Their profile remembers everything across visits.
- **Track their symptoms.** Log how they're feeling between appointments.
  Doctors love it when patients bring this kind of journal.
- **Upload a CT scan.** The website shows it as a 3D image they can rotate.
  An AI model looks at the scan and gives a probability that the cancer
  has HRD. The model was trained on real ovarian cancer patients from a
  public research dataset.
- **Check their genetic variants.** If a patient knows about a gene change
  in their tumor (like BRCA1), another AI model gives a second opinion on
  whether that change is harmful. This model is right about 93% of the time
  on a benchmark test.
- **See if their current medication is the right match.** The website
  cross-checks their genetic information against guidelines from the FDA
  and CPIC (the official pharmacogenomics group) and tells them whether
  their current medication is well matched, needs review, or could
  probably be improved.
- **See a 3D model of their medication.** The website shows the actual
  drug attached to the protein it targets, with the patient's specific
  gene change highlighted.
- **Print a one-page report** for their next appointment. Lists what was
  found, what it means, and questions to ask the doctor.

### 2. The research code: [hrd-radiogenomics](https://github.com/patrisiyarum/hrd-radiogenomics)

The code that trains the AI model that looks at CT scans. Anyone can
download it and reproduce what we did. Steps:

1. Get the CT scans and the genetic answers from a public ovarian cancer
   dataset (TCGA-OV via TCIA).
2. Match each patient's scan to their genetic answer.
3. Train an AI model to predict the answer from the scan.
4. Test the model on patients it has never seen, and report how well it did.

The model isn't perfect. With only 135 patients to learn from, it gets the
right answer about 59% of the time on patients it never saw during training.
That's better than guessing (50%) but not good enough for clinical decisions.
The honest reason is that there isn't enough public data to train a really
strong model yet. Bigger collaborations like Hartwig and the PAOLA-1 trial
have hundreds more patients but require special data access agreements.

---

## Why we look at it from many angles

There's a 2024 study where researchers ran 20 different HRD tests on the
same patients. The tests didn't agree. There's no single right answer yet.

About 30% of patients who test positive on the standard scar-based test
still fail PARP inhibitor treatment because their cancer evolved during
treatment. The scar test only sees the past, not the present.

About 30% of HRD-positive cancers are HRD because of changes in the
tumor itself, not changes the patient was born with. A regular genetic
test from a blood sample can't see those.

So no single test is right. The point of Kintsugi is to look at all the
signals at once: genes, scars, imaging, and how the current medication
is matching up. When the signals agree, we have a confident answer. When
they don't, we tell the patient that, plainly. We don't hide the cracks.

---

## How well things work, plainly

| What it does | How well it works | What this means |
|---|---|---|
| Predicts if a BRCA1 gene change is harmful | Right about 93% of the time | Strong, on par with what genetic testing labs use |
| Predicts if a BRCA2 gene change is harmful | Right about 84% of the time | Solid |
| Predicts HRD from a CT scan | Right about 59% of the time | Modest. Better than guessing. The data is limited, not the method. |
| Tells a patient if their current drug is the right match | Pulls directly from FDA labels and CPIC guidelines | The same evidence a pharmacist would use |

---

## What this isn't

Not a medical device. Not approved by the FDA. Not a substitute for a
real doctor.

It's a free, open-source educational tool to help people walk into their
oncology appointments more prepared. Every number on the screen comes
from a public source, and we always show where the number came from.

If you're a patient: bring this to your oncologist. Don't change a
medication based on what you read here. Get genetic testing through a
certified clinical laboratory before any treatment decision.

---

## Where the data comes from

All public, all credited.

- Findlay et al. 2018 for BRCA1 functional data
- Huang et al. 2025 for BRCA2 functional data
- Google DeepMind's AlphaMissense for variant effect predictions
- AlphaFold DB for 3D protein shapes
- Knijnenburg et al. 2018 (via the GDC) for the HRD genetic answers
- TCIA TCGA-OV for the public ovarian CT scans
- Pan et al. 2024 for the imaging model methodology
- CPIC and FDA drug labels for the medication guidance
- BRCA Exchange and ENIGMA for expert panel verdicts on specific gene changes

## License

MIT. Free for anyone to use, study, copy, or build on.
