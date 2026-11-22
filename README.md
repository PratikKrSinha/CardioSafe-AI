# CardioSafe AI

**Calibrated ECG analysis with evidence-linked medication cardiac-safety checking.**
A student research project for an AI in Healthcare course.

> **Research prototype, not a medical device.** It must not be used for diagnosis or to change treatment.
> "None found" never means a drug is safe.

**Live demo:** [add Hugging Face Space link] &nbsp;|&nbsp; **Demo video:** [add YouTube link] &nbsp;|&nbsp; **Report:** [add link or `report/` folder]

---

## What it does

Given a 12-lead ECG and a prescription, CardioSafe AI:

1. **Classifies the ECG** into five PTB-XL diagnostic superclasses (Normal, Myocardial infarction, ST/T change, Conduction disturbance, Hypertrophy) with calibrated probabilities and an uncertainty check.
2. **Estimates QT and heart rate** with a learned model, computes QTc (Fridericia) and reports a normal / borderline / prolonged status with an uncertainty band.
3. **Reads the prescription** (OCR plus a line parser), then matches each drug to its active ingredient. Names that cannot be verified are marked *unrecognised*, never guessed.
4. **Links the two.** For each ECG finding and each medicine, it looks for sentences in the FDA drug label and reports one of three outcomes: *relationship found* (with the quoted sentence and a link), *none found in available sources*, or *insufficient evidence*.

## Main results (PTB-XL test fold 10, 2,158 ECGs)

| Part | Result |
|---|---|
| ECG classification (15-model ensemble) | macro AUROC **0.934** (95% CI 0.927 to 0.940); single architectures 0.927 to 0.932 |
| Calibration | expected calibration error 0.025 after temperature scaling |
| Conformal prediction (90% target) | empirical coverage 0.901 |
| Prolonged-QTc flag vs 12SL reference flag | specificity 0.987, PPV 0.844, **sensitivity 0.685** |
| Prescription OCR (synthetic prescriptions, Tesseract) | drug-level F1 0.996 (clean), 0.987 (medium), 0.935 (hard) |
| Combined module | 13 of 13 designed scenarios behave as specified |

The deep architectures perform almost the same, so the project emphasises trust (calibration, uncertainty, explanations) rather than a "best network".

## Repository layout

```
cardiosafe_common.py      data loading, metrics, training loop
cardiosafe_features.py    features for the classical baselines
cardiosafe_models.py      network architectures and augmentation
cardiosafe_trust.py       temperature scaling, ensemble disagreement, conformal prediction
cardiosafe_explain.py     Grad-CAM and Integrated Gradients
cardiosafe_qt.py          signal-processing QT measurement (documented negative result)
cardiosafe_qtnet.py       learned QT/RR regression
cardiosafe_kb.py          openFDA label extraction, RxNorm normalisation, evidence tiers
cardiosafe_rx.py          synthetic prescriptions, OCR wrappers, drug-line parser
cardiosafe_infer.py       ECG inference engine (ensemble + conformal + QT)
cardiosafe_combine.py     combined ECG x prescription assessor
cardiosafe_view.py        structured HTML result view
cardiosafe_app.py         Gradio demo app
notebooks/                01 to 15 (see below)
results/                  result CSVs, FINAL_METRICS.md, report figures
report/                   LaTeX source of the project report
```

## Notebooks (run in order on Google Colab)

| Notebook | Purpose |
|---|---|
| `01_ptbxl_data_prep` | download PTB-XL, build labels and the official train/validation/test split |
| `02_baselines` | classical models (logistic regression, decision tree, random forest, XGBoost) |
| `03_deep_models` | deep models (CNN, ResNet, xResNet, Inception, BiGRU, CNN-Transformer), 3 seeds each |
| `04_calibration_uncertainty` | temperature scaling, ensemble, MC dropout, conformal prediction |
| `05_explainability` | Grad-CAM, Integrated Gradients, deletion test |
| `06_qtc_measurement`, `06b_learned_qtc` | signal-processing QT (failed) and the learned QT/RR estimator |
| `07_medication_kb` to `07d_role_fixes` | FDA label knowledge base, sentence roles, drug-name resolver |
| `08_prescription_pipeline` | synthetic prescriptions and OCR benchmark |
| `09_combined_module` | combined assessor and scenario tests |
| `10_demo_app` | launches the Gradio demo |
| `11_collect_metrics` | collects all metrics into `FINAL_METRICS.md` |
| `12_report_figures` | regenerates the report figures and tables |
| `13_update_and_launch_app`, `14/15_export_*` | demo helpers and test-input export |

All notebooks read from and write to `MyDrive/CardioSafe` on Google Drive.

## How to run

**Easiest: open the live demo** (link above).

**On Google Colab**
1. Put the `cardiosafe_*.py` files in `MyDrive/CardioSafe`.
2. Run notebooks 01 to 09 in order (training needs a GPU; the demo itself runs on CPU).
3. Run `10_demo_app` and open the `gradio.live` link it prints.

**Requirements:** Python 3, PyTorch, scikit-learn, XGBoost, SciPy, pandas, matplotlib, Gradio, pytesseract and the Tesseract OCR engine (`apt install tesseract-ocr`).

## Data and sources

- **ECG data:** [PTB-XL](https://physionet.org/content/ptb-xl/1.0.3/) (public, not included in this repository) and PTB-XL+ for QT/RR reference values.
- **Drug information:** [openFDA drug labels](https://open.fda.gov/apis/drug/label/) and [RxNorm](https://www.nlm.nih.gov/research/umls/rxnorm/) (fetched at run time and cached).
- Demo prescriptions are **synthetic**. OCR was not tested on handwriting or real photographs.

## Limitations

- ECG and QT models are trained and tested on one dataset (PTB-XL) with no external validation.
- QT/RR training targets are algorithm outputs (12SL / Uni-G), not manual measurements. The prolonged-QTc flag misses about one third of reference-flagged cases.
- AV block is represented only by the conduction-disturbance class; PR interval is not measured.
- Drug evidence comes from US FDA label sentences. Labels can omit known risks, and a few sentence roles are still tagged incorrectly.
- DDInter and CredibleMeds were not used.
- The label link shows what a label says, not that a drug caused an ECG finding.

## Authors

[Add team names and enrollment numbers] &nbsp;|&nbsp; Mentor: [add name] &nbsp;|&nbsp; Bennett University, SCSET

## License

[Add a license, for example MIT for the code. PTB-XL, openFDA and RxNorm have their own terms.]
