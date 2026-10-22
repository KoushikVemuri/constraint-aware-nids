# Constraint-Aware Adversarial Robustness of ML-Based Network Intrusion Detection Systems

## Overview

Machine learning models are increasingly used to detect malicious network activity. However, their predictions may change when input features are perturbed. This project investigates the adversarial robustness of machine-learning-based network intrusion detection using the UNSW-NB15 dataset.

The experiments compare Logistic Regression, Random Forest, and multilayer perceptrons under gradient-based and random feature perturbations. The study also investigates whether adversarial perturbations generated against a PyTorch MLP transfer to other model architectures.

## Research Questions

1. How does increasing perturbation magnitude affect malicious-traffic detection?
2. Are gradient-based perturbations more effective than random perturbations?
3. Do adversarial perturbations generated against a neural network transfer to Logistic Regression and Random Forest?
4. How do model architecture and decision boundaries influence observed robustness?

## Dataset

The experiments use the UNSW-NB15 training and testing CSV files.

- Target: binary `label` (0 = normal, 1 = attack).
- Excluded identifiers and target-description columns: `id`, `attack_cat`, and `label`.
- Input features include numeric network-flow features and categorical protocol, service, and state features.
- Numeric features are median-imputed and standardized.
- Categorical features are imputed and one-hot encoded.

The supplied training data is split into training and validation subsets using a stratified split and a fixed random seed. The supplied test set is kept separate from the reported validation experiments.

## Models

- Logistic Regression
- Random Forest
- Scikit-learn MLP
- PyTorch MLP

The PyTorch MLP is a separate neural-network implementation used for the gradient-based adversarial experiments. It should not be confused with the scikit-learn MLP used in the baseline comparison.

## Experimental Method

### Baseline evaluation

Models are compared using accuracy, attack precision, attack recall, macro-F1, false-negative rate, and training time.

### Feature-restricted perturbations

The adversarial experiments perturb seven standardized numeric features:

- `dur`
- `sinpkt`
- `dinpkt`
- `sjit`
- `djit`
- `sload`
- `dload`

All other processed features are held fixed. Perturbation budgets are epsilon = 0.05, 0.10, 0.25, and 0.50.

### Gradient-based attack

Input gradients from the PyTorch MLP are used to construct sign-based perturbations intended to reduce the model's probability of classifying malicious traffic as an attack. The same perturbed examples are then evaluated against Logistic Regression and Random Forest to study transferability.

### Random control

A random perturbation is generated within the same feature and magnitude constraints. Its results are compared with the gradient-based attack.

### Evaluation subset

The adversarial evaluation uses validation examples that the PyTorch MLP initially classified correctly as malicious. Therefore, the reported evasion rates describe this selected subset, not all validation examples or the complete test set.

## Baseline Results

See [`results/baseline_validation_metrics.csv`](results/baseline_validation_metrics.csv).

| Model | Accuracy | Attack Recall | Macro-F1 |
|---|---:|---:|---:|
| Random Forest | 0.9582 | 0.9682 | 0.9519 |
| Scikit-learn MLP | 0.9468 | 0.9752 | 0.9378 |
| Logistic Regression | 0.9326 | 0.9507 | 0.9225 |

The Random Forest achieved the highest baseline accuracy and macro-F1 among the three scikit-learn models. The scikit-learn MLP achieved the highest attack recall in this comparison.

## Adversarial Results

At epsilon = 0.50:

| Evaluated model | Gradient-attack evasion rate | Random-control evasion rate |
|---|---:|---:|
| PyTorch MLP | 15.87% | 1.29% |
| Logistic Regression | 73.37% | 18.48% |
| Random Forest | 0.25% | 0.41% |

See [`results/adversarial_evaluation_metrics.csv`](results/adversarial_evaluation_metrics.csv) for all budgets and [`results/transferability_metrics.csv`](results/transferability_metrics.csv) for transferability results.

The transferred perturbations affected the evaluated models differently. Logistic Regression showed a substantial decrease in attack recall as the perturbation budget increased, while Random Forest's attack recall remained high. These findings are specific to the models, preprocessing, feature subset, and perturbation method used here.

## Limitations

- The adversarial evaluation is conducted on a selected validation subset, not the held-out test set.
- Perturbations are applied in standardized feature space. They do not establish that the resulting feature combinations correspond to physically realizable network traffic.
- Network-flow features may have dependencies. Changing them independently can produce inconsistent or infeasible samples.
- The experiment evaluates a particular gradient-based method and random control; it does not establish robustness against all adversarial strategies.
- Results may vary with model initialization, hyperparameters, data preprocessing, and random seed.
- Baseline results for the scikit-learn MLP and adversarial results for the PyTorch MLP refer to different models.

## Reproducibility

The experiment uses Python, pandas, NumPy, scikit-learn, and PyTorch. The notebook and dependency list should be included in the repository so the workflow can be reproduced.

Download the UNSW-NB15 dataset from its official source and place the training and testing CSV files in the expected locations before running the notebook. Dataset redistribution is not included in this repository.

## Future Work

- Evaluate adversarial performance on the untouched test set.
- Add feature-dependency constraints and investigate more realistic perturbations.
- Compare additional attack methods and defenses.
- Report confidence intervals across repeated runs.
- Examine per-class performance and feature sensitivity.

## Disclaimer

This is an academic research project for studying machine-learning robustness in network intrusion detection. It is not a production intrusion detection system.
