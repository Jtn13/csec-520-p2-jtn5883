# Submission Manifest — Project 2

Fill this in before submitting. The grading agent reads it first.

## Student
- Name (RIT ID): Jack Nardi (jtn5883)

## Course level
- [x] 520 (undergraduate)

## Dataset
- Name / source / version / URL:
- How to obtain it (command or steps):
- Label column (used for EVALUATION ONLY): attack_cat
- Subsample size and how you chose it: Cap of 4000

## How to reproduce
```bash
make setup
# Download the course Drive archive and extract data/UNSW_NB15_training-set.csv
# See data/README.md; make data is an optional mirror fallback / checksum check.
make reproduce
```
Anything non-default the grader must know (data download, runtime):

## Your implementation
- Confirm `config.yaml` has `kmeans.implementation: scratch`: [x]
- Initialization used `kmeans++`:
- How you handle **empty clusters**: Left in place: centroid keeps its previous position if no points are assigned.
- Convergence criterion and tolerance: Stop when the toal squared centroid shift is below tol = 1e-4 or after max_iter = 300
- Agreement with the reference (each run’s `reference_check` in metrics.json; same geometry):
  - Euclidean `inertia_ratio` / `ari_vs_reference`: 
  - Mahalanobis `inertia_ratio` / `ari_vs_reference`:

## Choosing k
- k you report, and the evidence (elbow / silhouette): 11 clusters vs 10 classes. 
- If the best silhouette k differs from the number of true classes, explain: Several clusters map to the same class (three to Fuzzers), while Analysis, Reconnaissance, and Shellcode are never a majority, since these attacks look similar in flow features

## Claimed results (must match `results/metrics.json`)
| Metric | Euclidean | Mahalanobis |
|---|---|---|
| Silhouette (common Euclidean) | | |
| Silhouette (configured geometry) | | |
| V-measure | | |
| Accuracy | | |
| Macro F1 | | |

- Primary comparison criterion declared before weight experiments: V-measure. It balances homogeneity (each cluster contains one class) and completeness (each class stays in one cluster), so it captures overall cluster-class agreement in a single score that doesn't depend on majority-vote mapping.
- The diagonal C you chose, its feature order, and why:
- Same sample, k, seed and restart budget across the primary runs: [x]
- Tradeoffs and whether any independent validation was used:

## AI-use acknowledgment
Per the syllabus policy, briefly note any substantive use of AI assistants.
