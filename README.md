# PaedOx Solver Validation
Repository for [PaedOx](https://www.paedox.com) predictive model optimizer/solver validation work.

Support Vector Machines (SVMs) are the recommended predictive model for the PaedOx platform. PaedOx uses custom solvers including Sequential Minimal Optimization (SMO)<sup>1,2</sup> (PaedOx Version 2.0+), Iterative Single Data Algorithm (ISDA)<sup>3</sup> (PaedOx Version 6.8+), and Quadratic Programming (QP) (PaedOx Version 6.9+).

See Experiment 5 for a head-to-head across PaedOx and other solvers on a large regression problem.

Summary: PaedOx SMO is the recommended solver for the platform. For smaller datasets, PaedOx QP provides more a precise solution.

## Sequential Minimal Optimization
PaedOx SMO is recommended for most use-cases and supports several types of SVM through different problem encodings:

| Model Type | Description |
|-----------|------------|
| **SVC**   | Support Vector Classification |
| **$\varepsilon$-SVR**   | Support Vector Regression |
| **OC-SVM**| One-Class SVM (anomaly detection and Support Vector Clustering) |

MATLAB and LIBSVM implement similar SMO-based solvers that are trusted by industry. The following experiments use a dataset containing 10-engineered features from open source oximetry data from the Childhood Adenotonsillectomy Trial (CHAT)<sup>4</sup> and Pediatric Adenotonsillectomy Trial for Snoring (PATS)<sup>5</sup> datasets.
- [MATLAB](https://www.mathworks.com)
- [LIBSVM](https://www.csie.ntu.edu.tw/~cjlin/libsvm/)

PaedOx SMO is compared to MATLAB and LIBSVM SMO in the following experiments.

### Algorithm
PaedOx SMO uses second order information in working set selection, and makes a pairwise (2D) update with the largest predicted objective improvement until the gradient difference between upper and lower violators is less than a preset tolerance. PaedOx implements caching of the Q matrix, and PaedOx SMO does not implement shrinking.

### Experiment 1
- Gaussian SVC
- 780 recordings with 7-dimensions
- Hyperparameters: $C=100, γ=0.01$
- Standardization: $z = \frac{x - \mu}{\sigma}$
- Stopping criterion: $G(I_{\mathrm{up}}) - G(I_{\mathrm{low}}) < 0.001$

#### MATLAB
```
classificationSVM = fitcsvm(...
    data, ...
    y, ...
    'KernelFunction', 'gaussian', ...
    'BoxConstraint', 100, ...
    'KernelScale', 10, ...
    'Standardize', true, ...
    'GapTolerance', 0, ...
    'DeltaGradientTolerance', 1e-3, ...
    'KKTTolerance', 0, ...
    'Verbose', 2, ...
    'IterationLimit', 1e6, ...
    'Solver', 'SMO');
```

#### LIBSVM
```
-s 0 -t 2 -g 0.01 -c 100 -e 0.001
```

#### Results
Outputs for MATLAB, LIBSVM, and PaedOx SMO are located within the 'Experiment 1' folder.

| Solver | Iterations | Support Vectors | Objective | Rho (ρ)
|-----------|------------|------------|------------|------------|
| **MATLAB SMO** | 4,410 | 274 | -25094.8029098 | 2.527222310883571*
| **LIBSVM SMO** | 3,868 | 274 | -25094.873281 | 2.527749
| **PaedOx SMO** | 4,211 | 274 | -25094.802535791718 | 2.5273942768169446

<sub>*MATLAB b converted to ρ.</sub>

### Experiment 2
- Gaussian SVR
- 780 recordings with 7-dimensions
- Hyperparameters: $C=100, γ=0.01, \varepsilon = 1$
- Standardization: $z = \frac{x - \mu}{\sigma}$
- Stopping criterion: $G(I_{\mathrm{up}}) - G(I_{\mathrm{low}}) < 0.001$

#### MATLAB
```
regressionSVM = fitrsvm(...
    data, ...
    y, ...
    'KernelFunction', 'gaussian', ...
    'BoxConstraint', 100, ...
    'KernelScale', 10, ...
    'Epsilon', 1, ...
    'Standardize', true, ...
    'GapTolerance', 0, ...
    'DeltaGradientTolerance', 1e-3, ...
    'KKTTolerance', 0, ...
    'Verbose', 2, ...
    'IterationLimit', 1e6, ...
    'Solver', 'SMO');
```

#### LIBSVM
```
-s 3 -t 2 -g 0.01 -c 100 -p 1 -e 0.001
```

#### Results
Outputs for MATLAB, LIBSVM, and PaedOx SMO are located within the 'Experiment 2' folder.

| Solver | Iterations | Support Vectors | Objective | Rho (ρ)
|-----------|------------|------------|------------|------------|
| **MATLAB SMO** | 3,856 | 453 | -75398.419356 | -16.451949250670143*
| **LIBSVM SMO** | 3,631 | 453 | -75398.414787 | -16.458473
| **PaedOx SMO** | 3,601 | 453 | -75398.4195178858 | -16.452431241671576

<sub>*MATLAB b converted to ρ.</sub>

## Iterative Single Data Algorithm
PaedOx ISDA is available as of Version 6.8 and supports the following types of SVM using different problem encodings:

| Model Type | Description |
|-----------|------------|
| **SVC**   | Support Vector Classification |
| **$\varepsilon$-SVR**   | Support Vector Regression |

MATLAB implements a similar ISDA-based solver. The below experiment uses an identical dataset to SMO-based experiments above.

### Algorithm
PaedOx ISDA selects a single coordinate with the largest KKT violation and makes a 1D update each iteration until the maximum KKT violation is less than a preset tolerance. PaedOx implements caching of the Q matrix, and PaedOx ISDA does not implement shrinking.

Rho (ρ) is not learned explicitly, removing the equality constraint and enabling 1D updates. This makes the solver incompatible with OC-SVM. To incorporate ρ, a constant equal to a fraction (1/10) of the mean of the Q diagonal is added to the Q matrix.<sup>3</sup> Once ISDA has converged, the resulting equivalent term is stored in ρ.

PaedOx ISDA 1D updates are faster than PaedOx SMO 2D updates, though many more iterations are typically required. It is a simpler, less constrained optimization, but is slower to converge, and so SMO is the recommended solver for most use-cases.

### Experiment 3
- Gaussian SVR
- 780 recordings with 7-dimensions
- Hyperparameters: $C=10, γ=0.01, \varepsilon = 1$
- Q matrix added constant (ISDA): $0.01$
- Standardization: $z = \frac{x - \mu}{\sigma}$
- Stopping criterion (SMO): $G(I_{\mathrm{up}}) - G(I_{\mathrm{low}}) < 0.001$
- Stopping criterion (ISDA): Maximum KKT Violation $< 0.001$

#### MATLAB
```
regressionSVM = fitrsvm(...
    data, ...
    y, ...
    'KernelFunction', 'gaussian', ...
    'BoxConstraint', 10, ...
    'KernelScale', 10, ...
    'Epsilon', 1, ...
    'Standardize', true, ...
    'GapTolerance', 0, ...
    'DeltaGradientTolerance', 0, ...
    'KKTTolerance', 1e-3, ...
    'Verbose', 2, ...
    'IterationLimit', 20e6, ...
    'Solver', 'ISDA');
```
#### Results
Outputs for MATLAB ISDA, PaedOx ISDA, and PaedOx SMO are located within the 'Experiment 3' folder.

| Solver | Iterations | Support Vectors | Objective | Rho (ρ)
|-----------|------------|------------|------------|------------|
| **PaedOx SMO** | 495 | 460 | -9676.640482640165 | -7.405392647449415
| **PaedOx ISDA** | 10,298 | 465 | -9731.771315884413 | -1.510151029711426
| **MATLAB ISDA** | 10,298 | 465 | -9731.77131588 | -1.510151029711403*

<sub>*MATLAB b converted to ρ.</sub>

## Quadratic Programming
PaedOx QP is available as of Version 6.9 and supports the following types of SVM using different problem encodings:

| Model Type | Description |
|-----------|------------|
| **SVC**   | Support Vector Classification |
| **$\varepsilon$-SVR**   | Support Vector Regression |
| **OC-SVM**| One-Class SVM (anomaly detection and Support Vector Clustering) |

Comparisons to MATLAB L1QP (quadprog) and other methods are below. Note that MATLAB quadprog uses an interior point method for SVM and so operates differently to PaedOx QP.

### Algorithm
PaedOx QP combines Alternating Direction Method of Multipliers (ADMM) and a polishing solve, and so is similar in spirit to [OSQP](https://osqp.org/). PaedOx QP supports box- and equality-constrained QPs.

PaedOx QP is slower and more memory-intensive than decomposition methods such as SMO and ISDA, so it is not intended for large datasets. It is best suited to problems with at most a few thousand training examples. For smaller datasets, it can produce solutions with strong primal feasibility and machine-precision optimality.

PaedOx QP solves iterateively until primal and dual residuals are simultaneously less than a preset tolerance. This is used to infer which coordinates are at the box bounds, after which a reduced equality-constrained QP is solved over the remaining free variables. This polishing step improves the solution to machine precision. Polishing is successful if (1) all coordinates remain within the box, and (2) both primal and dual residuals are less than those of solution prior to polishing. If polishing is not successful, the solution prior to polishing is used instead.

### Experiment 4
This experiment runs PaedOx QP on the problem posed in Experiment 3. Other solver results are re-used.
- Stopping critierion (PaedOx QP): ‖x<sup>k+1</sup> − z<sup>k+1</sup>‖<sub>∞</sub> ≤ 10<sup>−3</sup> and ρ ‖z<sup>k+1</sup> − z<sup>k</sup>‖<sub>∞</sub> ≤ 10<sup>−3</sup>

#### Results
Outputs for MATLAB ISDA, PaedOx ISDA, and PaedOx SMO are located within the 'Experiment 3' folder. PaedOx QP results are located within the 'Experiment 4' folder.

| Solver | Iterations | Support Vectors | Objective | Rho (ρ)
|-----------|------------|------------|------------|------------|
| **PaedOx SMO** | 495 | 460 | -9676.640482640165 | -7.405392647449415
| **PaedOx ISDA** | 10,298 | 465 | -9731.771315884413 | -1.510151029711426
| **MATLAB ISDA** | 10,298 | 465 | -9731.77131588 | -1.510151029711403*
| **PaedOx QP** | 266 | 460 | -9676.640602336578 | -7.4023527993285825
| **MATLAB L1QP** | 14 | 460 | -9676.640602336614 | -7.402351875288217

PaedOx QP converged and the returned solution was polished to machine precision. The objective value is near-identical to MATLAB L1QP (quadprog), and slightly improved from PaedOx SMO.

## Large Regression (Experiment 5)
This experiment is a head-to-head comparison using a large dataset containing data from two open source datasets<sup>4,5</sup> and two paediatric tertiary centres (for a total of four datasets), to train an SVR configured similarly (hyperparameters and features) to models used in production. The size of this problem serves as a difficult stress test for PaedOx and other solvers.

### Methods
- Gaussian SVR
- 7,103 recordings with 11-dimensions
- Hyperparameters: $C=250, γ=0.01, \varepsilon = 1$
- Q matrix added constant (ISDA): $0.01$
- Standardization: $z = \frac{x - \mu}{\sigma}$
- Stopping criterion (SMO): $G(I_{\mathrm{up}}) - G(I_{\mathrm{low}}) < 0.001$
- Stopping criterion (ISDA): Maximum KKT Violation $< 0.001$
- Stopping criterion (PaedOx QP): ‖x<sup>k+1</sup> − z<sup>k+1</sup>‖<sub>∞</sub> < 0.001 and ρ ‖z<sup>k+1</sup> − z<sup>k</sup>‖<sub>∞</sub> < 0.001

MATLAB and LIBSVM solvers configured as above, with relevant changes to C.

### Results
Outputs are located within the 'Experiment 5' folder. Middle portions of PaedOx ISDA log are removed to reduce file size. MATLAB ISDA log omitted due to size.

| Solver | Iterations | Support Vectors | Objective | Rho (ρ) | Note
|-----------|------------|------------|------------|------------|------------|
| **MATLAB SMO** | 58,429 | 4,573 | -3998822.54687 | -50.174433278612604* | |
| **MATLAB ISDA** | 2,222,127 | 4,572 | -4005889.66636| -28.038084931358030* | |
| **LIBSVM SMO** | 63,202 | 4,573 | -3998825.062475 | -50.174449 | |
| **PaedOx SMO** | 63,718 | 4,573 | -3998822.551904961 | -50.17419408734025 | |
| **PaedOx ISDA** | 2,222,127 | 4,572 | -4005889.66636264 | -28.038084931308095 | |
| **PaedOx QP** | 7,861 | 4,573 | -3998822.592545736 | -50.17239936999981 | Polish unsuccessful
| **MATLAB L1QP** | 4 | 979 | 1.291016323602368e-17 | -1.000070397747272* | Solve failed

<sub>*MATLAB b converted to ρ.</sub>

ISDA objective values differ from other methods as the problem is posed differently (see above for details). MATLAB ISDA and PaedOx ISDA return identical solutions.

SMO solvers and PaedOx QP provide good solutions. PaedOx QP introduced a box violation during polishing and so the pre-polished solution was accepted and reported.

MATLAB L1QP (quadprog) converged to a degenerate near-zero QP objective and so may not be suitable for large kernel SVR.

## Outlier detection using One-Class SVM / Support Vector Clustering (Experiment 6)
This experiment is a head-to-head comparison using the large dataset from Experiment 5 to train an OC-SVM to identify outliers or anomalies. In this scenario, 5% of recordings are assumed to be outliers. PaedOx uses the rescaled formulation, where $0 \le \alpha_i \le 1$ and $\sum_{i=1}^{l} \alpha_i = \nu l$.

OC-SVM using the Gaussian kernel (equivalent to SVDD)<sup>7</sup> forms the basis of Support Vector Clustering<sup>8</sup> for the PaedOx platform. OC-SVM outputs a support boundary. If during separate cluster analysis this boundary breaks into multiple disconnected regions in input space, those regions become clusters.

### Methods
- Gaussian OC-SVM
- 7,103 recordings with 11-dimensions
- Hyperparameters: $ν=0.05, γ=0.01$
- Standardization: $z = \frac{x - \mu}{\sigma}$
- Stopping criterion (SMO): $G(I_{\mathrm{up}}) - G(I_{\mathrm{low}}) < 0.001$

#### MATLAB
```
oneClassSVM = fitcsvm(...
    data, ...
    y, ...
    'KernelFunction', 'gaussian', ...
    'OutlierFraction', 0.05, ...
    'KernelScale', 10, ...
    'Standardize', true, ...
    'DeltaGradientTolerance', 1e-3, ...
    'KKTTolerance', 0, ...
    'Verbose', 2, ...
    'IterationLimit', 1e6, ...
    'Solver', 'SMO');
```

#### LIBSVM
```
-s 2 -t 2 -g 0.01 -n 0.05 -e 0.001
```

### Results
Outputs are located within the 'Experiment 6' folder.

| Solver | Iterations | Support Vectors | Objective | Rho (ρ)
|-----------|------------|------------|------------|------------|
| **MATLAB SMO** | 6,776 | 357 | 24445.3789039 | -1.853217837248648
| **LIBSVM SMO** | 370 | 357 | 24445.378903 | 172.219744
| **PaedOx SMO** | 375 | 357 | 24445.378903642497 | 172.21974160499542
| **PaedOx QP** | 1,333 | 358 | 24552.884315529594 | 172.613591933053

MATLAB uses a different initialization scheme, where every coordinate set to the value of ν. This dramatically slows convergence. MATLAB also appears to use a different scoring convention and so a different kind of offset. Logging suggests the dual state at convergence is similar to LIBSVM and PaedOx.

## References

1. Platt, J. C. (1998). *Sequential minimal optimization: A fast algorithm for training support vector machines* (Technical Report MSR-TR-98-14). Microsoft Research.

2. Fan, R.-E., Chen, P.-H., & Lin, C.-J. (2005). Working set selection using second order information for training support vector machines. *Journal of Machine Learning Research, 6*, 1889–1918.

3. Kecman, V., Huang, T.-M., & Vogt, M. (2005). Iterative single data algorithm for training kernel machines from huge data sets: Theory and performance. In L. Wang (Ed.), *Support vector machines: Theory and applications* (pp. 255–274). Springer-Verlag.

4. Marcus, C. L., Moore, R. H., Rosen, C. L., Giordani, B., Garetz, S. L., Taylor, H. G., Mitchell, R. B., Amin, R., Katz, E. S., Arens, R., Paruthi, S., Muzumdar, H., Gozal, D., Thomas, N. H., Ware, J., Beebe, D., Snyder, K., Elden, L., Sprecher, R. C., … Redline, S. (2013). A randomized trial of adenotonsillectomy for childhood sleep apnea. *New England Journal of Medicine, 368*(25), 2366–2376.

5. Mitchell, R. B., et al. (2020). *The Pediatric Adenotonsillectomy Trial for Snoring (PATS).*

6. Boyd, S., Parikh, N., Chu, E., Peleato, B., & Eckstein, J. (2011). Distributed optimization and statistical learning via the alternating direction method of multipliers. *Foundations and Trends in Machine Learning, 3*(1), 1–122.

7. Görnitz, N., Kloft, M., Rieck, K., & Brefeld, U. (2018). Support vector data descriptions and k-means clustering: One class? *IEEE Transactions on Neural Networks and Learning Systems, 29*(9), 3994–4006.

8. Ben-Hur, A., Horn, D., Siegelmann, H. T., & Vapnik, V. (2001). Support vector clustering. *Journal of Machine Learning Research, 2*, 125–137.
