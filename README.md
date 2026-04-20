# PaedOx Solver Validation
Repository for [PaedOx](https://www.paedox.com) predictive model optimizer/solver validation work.

## Support Vector Machines
Support Vector Machines (SVMs) are the recommended predictive model for the PaedOx platform. PaedOx uses a custom, generalized solver based on Sequential Minimal Optimization (SMO)<sup>1</sup> and working set selection using second-order information.<sup>2</sup>

The PaedOx solver supports several forms of SVM through different problem encodings:

| Model Type | Description |
|-----------|------------|
| **SVC**   | Support Vector Classification |
| **$\varepsilon$-SVR**   | Support Vector Regression |
| **OC-SVM**| One-Class SVM (anomaly detection and Support Vector Clustering) |

MATLAB and libsvm implement similar solvers that are trusted by industry. The following experiments use a dataset containing 10-engineered features from open source oximetry data from the Childhood Adenotonsillectomy Trial (CHAT)<sup>3</sup> and Pediatric Adenotonsillectomy Trial for Snoring (PATS)<sup>4</sup> datasets.
- [MATLAB](https://www.mathworks.com)
- [libsvm](https://www.csie.ntu.edu.tw/~cjlin/libsvm/)

The PaedOx solver is compared to MATLAB and libsvm solvers in the following experiments. In these experiments, PaedOx matches MATLAB and libsvm dual objectives, rho, and support vector sparsity, and does so in fewer iterations.

### References
1. Platt, J. (1998). *Sequential Minimal Optimization: A Fast Algorithm for Training Support Vector Machines.* Microsoft Research Technical Report MSR-TR-98-14.  
2. Fan, R.-E., Chen, P.-H., & Lin, C.-J. (2005). *Working Set Selection Using Second Order Information for Training Support Vector Machines.* Journal of Machine Learning Research, 6, 1889–1918. 
3. Marcus, C. L., et al. (2013). *A Randomized Trial of Adenotonsillectomy for Childhood Sleep Apnea (CHAT).* New England Journal of Medicine, 368(25), 2366–2376.  
4. Mitchell, R. B., et al. (2020). *The Pediatric Adenotonsillectomy Trial for Snoring (PATS).*  

## Experiment 1 (Gaussian SVC)
- 780 recordings with 10-dimensions
- Hyperparameters: $C=100, γ=0.01$
- Standardization: $z = \frac{x - \mu}{\sigma}$
- Stopping criterion: $G(I_{\mathrm{up}}) - G(I_{\mathrm{low}}) < 0.001$

### MATLAB
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
    'IterationLimit', 1e6);
```

### libsvm
```
-s 0 -t 2 -g 0.01 -c 100 -e 0.001 -h 0
```

### Results
Outputs for MATLAB, libsvm, and PaedOx solvers are located within the 'Experiment 1' folder.

| Solver | Iterations | Support Vectors | Objective | Rho (ρ)
|-----------|------------|------------|------------|------------|
| **MATLAB** | 4,682 | 289 | -26118.6528084 | 0.795813141832127*
| **libsvm** | 4,896 | 289 | -26118.634311 | 0.794652
| **PaedOx** | 4,607 | 289 | -26118.653023437815 | 0.7951236991689781

<sub>*MATLAB Bias converted to Rho (-Bias).</sub>

## Experiment 2 (Gaussian SVR)
- 780 recordings with 10-dimensions
- Hyperparameters: $C=100, γ=0.01, \varepsilon = 1$
- Standardization: $z = \frac{x - \mu}{\sigma}$
- Stopping criterion: $G(I_{\mathrm{up}}) - G(I_{\mathrm{low}}) < 0.001$

### MATLAB
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
    'IterationLimit', 1e6);
```

### libsvm
```
-s 3 -t 2 -g 0.01 -c 100 -p 1 -e 0.001 -h 0
```

### Results
Outputs for MATLAB, libsvm, and PaedOx solvers are located within the 'Experiment 2' folder.

| Solver | Iterations | Support Vectors | Objective | Rho (ρ)
|-----------|------------|------------|------------|------------|
| **MATLAB** | 6,564 | 464 | -86744.3269316 | -13.598481776474713*
| **libsvm** | 6,475 | 464 | -86744.294430 | -13.598689
| **PaedOx** | 5,424 | 464 | -86744.32640719382 | -13.598054612932991

<sub>*MATLAB Bias converted to Rho (-Bias).</sub>
