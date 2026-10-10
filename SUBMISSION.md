# Submission Manifest — Project 2

Fill this in before submitting. The grading agent reads it first.

## Student
- Name (RIT ID): Jack Nardi (jtn5883)

## Course level
- [x] 520 (undergraduate)

## Dataset
- Name / source / version / URL:
- How to obtain it (command or steps): Download the course dataset ZIP from Google Drive: https://drive.google.com/file/d/1jDsXYALnEsLzYYGCEtxgkAYygS-vKIt7/view?usp=sharing (about 20.6 GB). Extract UNSW_NB15_training-set.csv and place it at data/UNSW_NB15_training-set.csv. Use this partitioned training CSV for the primary experiment. See data/README.md for an optional mirror fallback (make data) and checksum details. Do not commit UNSW-NB15.
- Label column (used for EVALUATION ONLY): attack_cat
- Subsample size and how you chose it: Cap of 4000 seed 13

## How to reproduce
```bash
make setup
# Download the course Drive archive and extract data/UNSW_NB15_training-set.csv
# See data/README.md; make data is an optional mirror fallback / checksum check.
make reproduce
```
Anything non-default the grader must know (data download, runtime):
seed: 13
restarts: 25

## Your implementation
- Confirm `config.yaml` has `kmeans.implementation: scratch`: [x]
- Initialization used `kmeans++`:
- How you handle **empty clusters**: Left in place: centroid keeps its previous position if no points are assigned.
- Convergence criterion and tolerance: Stop when the toal squared centroid shift is below tol = 1e-4 or after max_iter = 300
- Agreement with the reference (each run’s `reference_check` in metrics.json; same geometry):
  - Euclidean `inertia_ratio` / `ari_vs_reference`: 
  - Mahalanobis `inertia_ratio` / `ari_vs_reference`:

## Choosing k
- k you report, and the evidence (elbow / silhouette): 16 clusters vs 10 classes. 
- If the best silhouette k differs from the number of true classes, explain: Several clusters map to the same class (three to Fuzzers), while Analysis, Reconnaissance, and Shellcode are never a majority, since these attacks look similar in flow features

## Claimed results (must match `results/metrics.json`)
| Metric | Euclidean | Mahalanobis |
|---|---|---|
| Silhouette (common Euclidean) | 0.467 | 0.435 |
| Silhouette (configured geometry) | 0.467 | 0.449 |
| V-measure | 0.316 | 0.302 |
| Accuracy | 0.351 | 0.347 |
| Macro F1 | 0.281 | 0.272 |

- Primary comparison criterion declared before weight experiments: V-measure. It balances homogeneity (each cluster contains one class) and completeness (each class stays in one cluster), so it captures overall cluster-class agreement in a single score that doesn't depend on majority-vote mapping.
- The diagonal C you chose, its feature order, and why: Vector: `[1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 0.1, 0.1, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0]`
- Why: Features are already standardized to variance 1, so inverse-variance weights would all equal 1 (plain Euclidean). Weights were instead chosen from the paper's feature groups. Three cumulative candidates were tested at identical settings: Stage 1 (`stcpb`, `dtcpb` = 0.1, TCP sequence numbers appeared random), Stage 2 (+ `tcprtt` = 0.1 since it equals `synack` + `ackdat`; volume features `spkts, dpkts, sbytes, dbytes, rate, sload, dload` = 0.5 as overlapping measures), Stage 3 (+ `is_ftp_login`, `is_sm_ips_ports` = 0.25, rare flags become extreme after standardization). V-measures: Stage 1 0.302, Stage 2 0.283, Stage 3 0.296. Stage 1 was best by the declared criterion, so it is the final C. 0.1 is used instead of 0 because weights must be strictly positive. TTL features were kept at 1 to avoid boosting a possible data-generation artifact.
- Same sample, k, seed and restart budget across the primary runs: [x]
- Tradeoffs and whether any independent validation was used:  No weighting beat Euclidean on V-measure (0.316 vs. 0.302 best). Stage 1 slightly improved Generic separation (353 → 359 of 400) but lowered completeness (0.375 → 0.348), Normal separation (268 → 206), silhouette, and Davies-Bouldin (0.787 → 0.888). Halving the volume features (Stage 2) hurt most, suggesting volume carries real class signal; the sequence-number fields likely act as a TCP indicator (0 for non-TCP flows). The same direction held at k = 11 (Euclidean 0.304 vs. Stage 2 0.284). Weighted runs had reference ratios of 1.04–1.09, so their optima are a few percent worse than the reference. No independent validation was used: all metrics are in-sample cluster agreement on the same 3,730-row sample, not held-out detection performance. Exploratory runs (seed 42 / n_init 10, k = 11) and Stage 2/3 candidates are disclosed above and in the report.

## AI-use acknowledgment
Per the syllabus policy, briefly note any substantive use of AI assistants.
AI was used in assistance with debugging my functions in the from scrratch implementation. It was also used in reviewing results and comparing different experiements. It was also used in sythesysing findings for my report and writting out the tables in LATeX using my data.