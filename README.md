# Supplementary Data — Machine-Learning Attribution of MP/NP Toxicity
**Manuscript:** EN-ART-07-2026-000594 (Environmental Science: Nano, revision)
**Topic:** Micro/nanoplastic toxicity attribution with IBR index + Random Forest + SHAP
(*Chlorella vulgaris* – *Daphnia magna* two-trophic-level system)

---

## 1. Contents

| File | Purpose |
|---|---|
| `RF_IBR_SHAP_Reproducible_Analysis.py` | **Main executable script.** Reproduces every model result in the manuscript and SI. |
| `SI_Dataset_unified.csv` | **Unified dataset (144 rows, both species)** used for all unified-model analyses. |
| `_check_fit_metrics.py` / `_check_fit_metrics.csv` | EC20 interpolation and dose–response fit metrics for **Fig. 8 / SI Table S8**. |
| `model_results_final.txt` | Full console log of the main run. |
| `results_main_final.csv` | Unified baseline / enhanced model performance on the independent test set (**Section 3.5**). |
| `results_cv_final.csv` | Repeated 5-fold cross-validation, 50 random seeds (**Table 3 / SI Table S11c**). |
| `results_species_specific_final.csv` | Species-specific models (**SI Table S7**). |
| `results_leakage_final.csv` | Target-leakage diagnostics for the Fold-EC50 input (**SI Table S9**). |
| `si_table_S10_prediction_intervals_final.csv` | 95% prediction intervals, leave-one-experimental-series-out (**SI Table S10**). |
| `si_table_S11_grouped_cv_pooled.csv` | Per-fold grouped cross-validation: LOPTO and LOSeries (**SI Table S11a/b**). |
| `si_table_S12_ec20_flags.csv` | EC20 interpolation / extrapolation flags (**SI Table S12**). |
| `si_shap_importance_baseline.csv` / `si_shap_importance_enhanced.csv` | SHAP mean |SHAP| importances (**Fig. 7 / Fig. S21**). |
| `si_split_indices.csv` | Exact train/test split indices (stratified by species, seed 42). |

## 2. Dataset columns

`Species` (0 = *C. vulgaris*, 1 = *D. magna*), `SpeciesName`, `Plastic`,
`Concentration_mgL`, `Fold_EC50` (concentration ÷ species-specific EC50),
`Size_um` (PS 0.08; PVC 0.8; PE/PP 7.0), `SurfaceGroup` (**PS = 0, PVC = 1,
PS-COOH = 2, PE = 3, PP = 4, PS-NH2 = 5**), `Time_h` (24/48/72), `SeriesID`,
`Inhibition_pct` (response), `MDA`, `SOD`, `GSH` (post-exposure biomarkers).

EC50 values used for normalization (SI Table S5):
*C. vulgaris* (72 h): PS 198.0, PS-COOH 733.4, PS-NH2 120.3, PVC 972.6, PP 1395.7, PE 1309.9 mg/L;
*D. magna* (48 h): PS 23.1, PS-COOH 24.2, PS-NH2 21.6, PVC 127.7, PP 179.8, PE 155.5 mg/L.

## 3. How to run

- **Requirements:** Python ≥ 3.8, `numpy`, `pandas`, `scikit-learn` (≥ 1.0), `matplotlib`.
- **Run (from this folder):**
  ```
  python RF_IBR_SHAP_Reproducible_Analysis.py
  ```
  The script reads `SI_Dataset_unified.csv` from the same folder and writes the
  `*_final` output files listed above. Random seeds are fixed
  (model seed 42; CV seeds 0–49; split: `StratifiedShuffleSplit(test_size=0.2,
  random_state=42)` stratified by species), so results are fully reproducible.
- `_check_fit_metrics.py` reproduces Fig. 8 / SI Table S8:
  ```
  python _check_fit_metrics.py
  ```

## 4. Model configuration

- Random forest: `n_estimators = 200`, `max_depth = 8` (grid-searched, SI Table S6).
- Baseline features: `Fold_EC50, Size_um, SurfaceGroup, Time_h, Species`.
- Enhanced model: baseline + post-exposure biomarkers `MDA, SOD, GSH`
  (interpreted strictly as a post-exposure biomarker model).
- Raw-concentration model: `Concentration_mgL` replaces `Fold_EC50`
  (target-leakage check, SI Table S9).
- SHAP: `shap.TreeExplainer`; mean absolute SHAP values used for ranking.

## 5. Key results reproduced by this package

- Unified baseline: test R² = 0.940 (MAE = 1.12%); enhanced: test R² = 0.942 (MAE = 1.34%).
- Repeated 5-fold CV (50 seeds): baseline 0.908 ± 0.014 [0.876, 0.931];
  enhanced 0.901 ± 0.016 [0.861, 0.922]; raw-concentration baseline 0.829 ± 0.037 [0.737, 0.872].
- Grouped CV: leave-one-plastic-type-out pooled R² = 0.905; leave-one-experimental-series-out = 0.908.
- Target-leakage: Fold-EC50 alone R² = 0.529 ± 0.168 in 5-fold CV — far below the full model;
  36 of 48 (species, material, Fold-EC50) groups have non-unique responses.
- Size-related attribution: SHAP identifies Fold-EC50 > exposure time > particle size >
  species > surface group; a model-interpolated size transition at ≈0.44 µm is reported
  as exploratory (requires factorial validation, no experimental data at intermediate sizes).
