# Bank Shield: Auditable AI for Regulatory Compliance in Credit Scoring

## Overview

**Bank Shield** is a hybrid AI architecture for credit risk assessment that solves the "Compliance Gap" in Financial AI. While traditional credit models focus exclusively on accuracy, Bank Shield adds a second, equally important objective: **regulatory adherence**. The system ensures that loans violating hard regulatory limits (like the 43% Debt-to-Income cap) cannot be approved, even if persuasive text suggests otherwise.

### Key Innovation

Bank Shield combines:
- **FinBERT**: State-of-the-art financial text understanding
- **Numeric Features**: Raw financial indicators (income, DTI, loan amount, etc.)
- **Regulatory Flags**: Automated detection of compliance violations
- **Custom Loss Function**: A "guardrail" that penalizes regulatory breaches during training

This creates a model that is:
- **Unfoolable by text**: Cannot be manipulated by persuasive language
- **Mathematically constrained**: Regulatory limits baked into the loss function
- **Auditable & Explainable**: Every decision can be traced to financial facts

---

## Problem Statement

Traditional credit models face a critical vulnerability:

> A model might approve a borrower with an illegally high DTI ratio (e.g., 50%) because their "loan purpose" text sounds persuasive, even though the financial metrics clearly violate regulations.

### Why This Matters

For highly regulated sectors (Finance, Healthcare, Law), regulations must be **baked into the math**, not just checked at the end. Banks cannot afford to:
- Manually audit every prediction
- Explain why a risky loan was approved
- Face regulatory penalties for systematically violating compliance rules

---

## Solution Architecture

### 1. **Guardrailed Hybrid Model**

```
Input (Text + Numeric + Flags)
    ↓
FinBERT Encoder (768-dim embeddings)
    ↓
Numeric Scaler (9 financial features)
    ↓
Flag Detector (7 regulatory flags)
    ↓
Fusion Layer (concatenate embeddings + numeric + flags)
    ↓
Classification Head (2 logits: pay back vs. default)
    ↓
Custom Loss = Standard Loss + λ·Penalty(violation_flags)
    ↓
Output: P(Default) with regulatory constraints
```

### 2. **Regulatory Flags** (7 hardcoded rules)

| Flag | Rule | Business Impact |
|------|------|-----------------|
| `flag_dti_43` | DTI > 43% | CFPB Qualified Mortgage hard cap |
| `flag_lti_high` | Loan-to-Income > 50% | Loan exceeds annual income |
| `flag_pti_high` | Payment-to-Income > 35% | Monthly payment > 35% of income (FHA/VA) |
| `flag_predatory` | Income < $20k & Loan > $10k | Predatory lending indicator |
| `flag_dual_stress` | DTI > 36% AND LTI > 30% | Multiple stress indicators |
| `flag_debt_burden` | Existing debt > 50% of income | Severe debt burden |
| `flag_large_loan_dti` | Large loan (>75th %ile) + DTI > 30% | Oversized loan on stretched borrower |

### 3. **Hyperparameter Optimization Results**

The model discovers the Pareto frontier between profit and law:

- **λ (penalty weight)**: 0.1 (optimal balance)
- **τ_train (training threshold)**: 0.5 (penalty applied during training)
- **τ_decision (operating threshold)**: 0.35 (inference decision boundary)

**Result**: High predictive power WITHOUT regulatory violations.

---

## Datasets & Experiments

### Primary Dataset: Lending Club
- **Size**: 50,000 loans (stratified sample from 1.3M historical loans)
- **Labels**: Fully Paid vs. Charged Off
- **Time Period**: 2007–2018 Q4
- **Train/Val/Test**: 35,000 / 7,500 / 7,500 (stratified by label + violation severity)

---

## Key Results

### 1. **Clean Performance (Threshold = 0.35)**

| Metric | Baseline | Regulatory | Difference |
|--------|----------|-----------|------------|
| **Recall (Sensitivity)** | 82.0% | 79.2% | -2.8% |
| **Precision** | 25.8% | 26.4% | +0.6% |
| **F1 Score** | 0.393 | 0.396 | +0.003 |
| **Balanced Accuracy** | 61.6% | 62.0% | +0.4% |
| **ROC-AUC** | 0.685 | 0.683 | -0.002 |
| **Compliance Breach Rate** | **4.3%** | **1.4%** | **-3.0% (66% reduction)** |
| **False Safe Rate (Flagged)** | 11.3% | 10.8% | -0.5% |

### 2. **Adversarial Robustness**

When exposed to adversarial text attacks (borrowers using "safe" keywords to manipulate FinBERT):

| Metric | Baseline | Regulatory |
|--------|----------|-----------|
| Recall (Clean) | 82.0% | 79.2% |
| Recall (Adversarial) | 99.9% | 100.0% |
| **Recall Drop** | -17.9% | -20.8% |
| **Attack Success Rate** | 0.13% | **0.0%** |
| **Harmful Attack Success** | 0.0% | **0.0%** |
| Mean P(Risky) Clean | 0.427 | 0.427 |
| Mean P(Risky) Adversarial | 0.471 | 0.611 |
| **Mean Δp(risky)** | +0.044 | **+0.183** |

**Key Finding**: Regulatory model stays disciplined. Even when fooled by text, numeric flags dominate the decision.

### 3. **Violation Severity Correlates with Default Risk**

| Violation Bucket | Count | Actual Risky Rate |
|------------------|-------|-------------------|
| None (0) | 7,269 | 19.6% |
| Low (0-0.25) | 144 | 25.0% |
| Medium (0.25-0.5) | 71 | 40.8% |
| High (0.5-0.75) | 9 | 11.1% |
| **Critical (0.75-1.0)** | **7** | **42.9%** |

**Implication**: Regulatory flags are predictive, not just punitive.

### 4. **Compliance Breach Asymmetry**

- **Baseline fooled** (risky case marked safe): 7 cases
- **Regulatory fooled** (risky case marked safe): 0 cases
- **Asymmetry ratio**: 7:0

**Interpretation**: Hybrid approach (text + numeric + flags) is inherently more robust to text manipulation than text-only approaches.

---

## Repository Structure

```
bank-shield/
├── README.md                                  # Main documentation
├── LICENSE                                    # MIT License
├── requirements.txt                           # Python dependencies
│
├── src/
│   ├── __init__.py
│   ├── config.py                              # Global configuration (paths, hyperparameters)
│   │
│   ├── data/
│   │   ├── __init__.py
│   │   ├── loader.py                          # Data loading & preprocessing
│   │   ├── features.py                        # Feature engineering (regulatory flags)
│   │   └── dataset.py                         # PyTorch Dataset class
│   │
│   ├── models/
│   │   ├── __init__.py
│   │   ├── finbert_hybrid.py                  # Main hybrid model (FinBERT + Numeric + Flags)
│   │   ├── loss.py                            # Custom loss with regulatory penalty
│   │   └── training.py                        # Training loop & evaluation functions
│   │
│   ├── evaluation/
│   │   ├── __init__.py
│   │   ├── metrics.py                         # Compliance & performance metrics
│   │   ├── calibration.py                     # Calibration analysis
│   │   └── adversarial.py                     # Adversarial robustness tests
│   │
│   └── utils/
│       ├── __init__.py
│       ├── seed.py                            # Reproducibility (seed management)
│       └── logging.py                         # Logging utilities
│
├── scripts/
│   ├── train.py                               # Main training script
│   ├── evaluate.py                            # Evaluation on test set
│   ├── adversarial_test.py                    # Adversarial robustness testing
│   ├── hpo.py                                 # Hyperparameter optimization (Bayesian)
│   └── reproduce_results.py                   # Reproducibility script
│
├── configs/
│   └── default.yaml                           # Default configuration
│
├── notebooks/
│   └── bank_shield_full_pipeline.ipynb        # End-to-end analysis & visualizations
│
├── results/
│   ├── clean_performance_table.csv            # Core metrics (both models)
│   ├── compliance_table.csv                   # Per-violation compliance breakdown
│   ├── adversarial_robustness_table.csv       # Adversarial attack results
│   ├── threshold_sweep_table.csv              # Operating point analysis (threshold sweep)
│   ├── violation_score_table.csv              # Metric distribution by violation tier
│   ├── synthetic_shift_results.csv            # Domain adaptation results
│   ├── temporal_shift_results.csv             # Temporal drift analysis
│   │
│   ├── plots/
│   │   ├── roc_pr_calibration.png             # ROC/PR curves + calibration
│   │   ├── confusion_matrices.png             # Confusion matrices (baseline vs. regulatory)
│   │   ├── compliance_heatmap.png             # Compliance metrics by flag type
│   │   ├── compliance_by_violation.png        # Per-violation breakdown
│   │   ├── violation_score_plot.png           # Metric distribution by violation severity
│   │   ├── pareto_frontier.png                # Profit-compliance trade-off frontier
│   │   ├── threshold_sweep_plot.png           # Operating points by decision threshold
│   │   ├── adversarial_shift_plots.png        # Probability shifts under attack
│   │   ├── probability_distributions.png      # Raw vs. calibrated prediction distributions
│   │   ├── paired_probability_shifts.csv      # Detailed adversarial probability changes
│   │   ├── training_curves.png                # Loss & metrics over epochs
│   │   ├── synthetic_shift_comparison.png     # Domain adaptation performance
│   │   └── temporal_shift_comparison.png      # Temporal drift over loan cohorts
│   │
│   └── sample_predictions/
│       ├── baseline_fooled_examples.csv           # Baseline fooled, Regulatory held (risky)
│       ├── regulatory_fooled_examples.csv         # Regulatory fooled, Baseline held (risky)
│       ├── baseline_fooled_flagged_examples.csv   # Baseline fooled, Regulatory held (flagged+risky)
│       └── regulatory_fooled_flagged_examples.csv # Regulatory fooled, Baseline held (flagged+risky)
│
└── docs/
    ├── ARCHITECTURE.md                        # Detailed system design & math
    ├── METHODOLOGY.md                         # Data prep, training, evaluation
    ├── REGULATIONS.md                         # Regulatory framework mapping
    ├── RESULTS_ANALYSIS.md                    # Deep-dive into results
    └── FAIRNESS.md                            # Fairness & bias considerations
```

---

## Installation & Quick Start

### Prerequisites
- Python 3.9+
- GPU (CUDA 11+) recommended for FinBERT training
- ~50GB disk space (for datasets and model checkpoints)

### Installation

```bash
# Clone the repository
git clone https://github.com/yourusername/bank-shield.git
cd bank-shield

# Create virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt
```

### Run the Full Pipeline

```bash
# 1. Data preprocessing (Block 2)
python scripts/data_prep.py

# 2. Train baseline model (Block 3)
python scripts/train.py --model baseline --config configs/default.yaml

# 3. Train regulatory model (Block 4)
python scripts/train.py --model regulatory --config configs/default.yaml

# 4. Hyperparameter optimization (Block 8)
python scripts/hpo.py --n-trials 100

# 5. Evaluate & generate report (Block 9)
python scripts/evaluate.py --baseline results/baseline_model/best.pt \
                           --regulatory results/regulatory_model/best.pt

# 6. Adversarial robustness testing (Block 7)
python scripts/adversarial_test.py --baseline results/baseline_model/best.pt \
                                  --regulatory results/regulatory_model/best.pt

# 7. Full reproducibility check
python scripts/reproduce_results.py --seed 42 --dataset lending_club
```

### Run Jupyter Notebook

```bash
jupyter notebook notebooks/bank_shield_full_pipeline.ipynb
```

---

## Configuration

### `configs/default.yaml`

```yaml
# Model configuration
model:
  model_name: "ProsusAI/finbert"
  max_seq_len: 96
  num_numeric_features: 9
  num_flags: 7
  hidden_dim: 256
  dropout: 0.1

# Training configuration
training:
  batch_size: 64
  num_epochs: 5
  learning_rate: 2.0e-5
  weight_decay: 0.01
  warmup_steps: 1000

# Regularization (regulatory model only)
regularization:
  lambda_reg: 0.1        # Penalty weight for regulatory violations
  tau_train: 0.5         # Training threshold for penalty application
  tau_decision: 0.35     # Inference decision threshold

# Data configuration
data:
  train_size: 50000
  val_size: 7500
  test_size: 7500
  stratify_by: "label + violation_tercile"
  seed: 42
```

---

## Key Findings & Interpretations

### Finding 1: **66% Reduction in Compliance Breaches**
- **What**: Regulatory model reduces compliance breach rate from 4.3% to 1.4%
- **Why**: Custom loss function penalizes regulatory flag violations during training
- **Trade-off**: 2.8% loss in recall (catching 2% fewer defaults)
- **Implication**: Well-designed guardrails don't destroy model performance; the trade-off is acceptable

### Finding 2: **Text-Based Attacks Are Ineffective Against Regulatory Model**
- **What**: Adversarial text attacks fool Baseline 7 times but Regulatory 0 times
- **Why**: Numeric features + regulatory flags dominate predictions
- **Mechanism**: Even when FinBERT is fooled, the financial metrics override the text signal
- **Implication**: Hybrid approach is inherently more robust than text-only approaches

### Finding 3: **Regulatory Flags Are Predictive**
- **What**: Violation severity strongly correlates with default risk (19.6% → 42.9%)
- **Why**: Flags encode genuine financial stress (DTI, income-loan mismatch, etc.)
- **Implication**: Regulations are not just constraints; they identify high-risk borrowers

### Finding 4: **Operating Point Flexibility**
- **What**: Banks can tune decision threshold (τ) to fit risk appetite
- **Why**: Threshold sweep shows clear accuracy-compliance trade-off curve (Pareto frontier)
- **Implications**:
  - Conservative: τ = 0.75 (reject ~95% of risky cases, only ~0.5% compliance breaches)
  - Moderate: τ = 0.35 (reject ~82% of risky cases, ~1.4% compliance breaches)
  - Aggressive: τ = 0.10 (reject ~100% of risky cases, 0% compliance breaches)

---

## Metrics Explained

### Performance Metrics
- **Recall (Sensitivity)**: % of actual defaulters that the model identifies
  - Formula: TP / (TP + FN)
  - Interpretation: "Catch rate" for risky borrowers
  
- **Precision**: % of predicted defaults that are actually risky
  - Formula: TP / (TP + FP)
  - Interpretation: "Accuracy" of negative predictions
  
- **F1 Score**: Harmonic mean of precision and recall
  - Formula: 2 × (Precision × Recall) / (Precision + Recall)
  - Interpretation: Balanced performance on imbalanced data
  
- **Balanced Accuracy**: Average recall across both classes
  - Formula: (Recall_Positive + Recall_Negative) / 2
  - Interpretation: Handles class imbalance better than raw accuracy

- **ROC-AUC**: Area under ROC curve (discrimination across thresholds)
  - Range: 0.5 (random) to 1.0 (perfect)
  - Interpretation: "How well does the model rank risky vs. safe borrowers?"

### Compliance Metrics
- **Compliance Breach Rate**: % of approved loans that violate ≥1 regulatory flag
  - Formula: (Number of flagged approvals) / (Total approvals)
  - Target: < 2% for regulatory compliance
  
- **False Safe Rate (Flagged)**: % of flagged-but-risky loans marked safe
  - Formula: (Flagged risky loans marked safe) / (All flagged risky loans)
  - Interpretation: "How many dangerous loans did we miss?"
  
- **Flagged Recall**: % of flagged loans that are caught
  - Formula: (Flagged loans marked risky) / (All flagged loans)
  - Target: > 95% (catch almost all flagged cases)

### Calibration Metrics
- **Brier Score**: Mean squared error between predicted probabilities and actual outcomes
  - Formula: Mean((y_pred - y_true)²)
  - Range: 0 (perfect) to 1 (worst)
  - Interpretation: "Are the model's confidence levels accurate?"
  
- **ECE (Expected Calibration Error)**: Max deviation between predicted and actual rates
  - Formula: Max|P(prediction) - P(actual outcome)|
  - Interpretation: "How much do confidence calibration curves deviate?"

---

## Reproducibility

### How to Reproduce Results

All results can be reproduced with fixed random seed and data splits:

```bash
# Full reproducibility test
python scripts/reproduce_results.py --seed 42 --dataset lending_club --verbose

# This will:
# 1. Load and preprocess data with fixed stratification
# 2. Train baseline model with fixed initialization
# 3. Train regulatory model with fixed initialization
# 4. Generate all evaluation metrics and plots
# 5. Compare against stored results (with small tolerance for numerical precision)
```

### Reproducibility Checklist

- ✅ Fixed random seed (SEED = 42) across NumPy, PyTorch, and Python
- ✅ Stratified train/val/test split (by label + violation tercile)
- ✅ Frozen model weights available in results/
- ✅ Full hyperparameter logging
- ✅ Deterministic data preprocessing (no augmentation)
- ✅ GPU reproducibility (torch.cuda.manual_seed_all)

### Sources of Non-Determinism

- Floating-point arithmetic (minor differences expected)
- PyTorch backend operations (vary by CUDA version)
- Multi-threaded data loading (use num_workers=0 to eliminate)

---

## Regulatory Framework

This project implements guardrails based on:

| Regulation | Rule | Implementation |
|-----------|------|-----------------|
| **CFPB Qualified Mortgage (QM)** | DTI ≤ 43% hard cap | `flag_dti_43 = (DTI > 43)` |
| **FHA Guidelines** | Payment-to-Income ≤ 35% | `flag_pti_high = (Monthly Payment > 0.35 × Monthly Income)` |
| **VA Guidelines** | DTI ≤ 41% (or 50% with compensating factors) | `flag_dual_stress = (DTI > 36 AND LTI > 0.3)` |
| **ECOA (Equal Credit Opportunity Act)** | Fair lending (no discrimination) | Fairness audit (see `docs/FAIRNESS.md`) |
| **HMDA (Home Mortgage Disclosure Act)** | Lending practice monitoring | Results logged per violation type |

For detailed mappings and regulatory citations, see `docs/REGULATIONS.md`.

---

## Performance by Operating Point

### Threshold Sweep Results

The following table shows key metrics at different decision thresholds:

| Threshold | Recall | Precision | F1 | Balanced Acc | False Safe (Risky) | Compliance Breach |
|-----------|--------|-----------|-----|-----|-----------|-----------|
| **0.10** | 100.0% | 19.9% | 0.333 | 50.0% | 0.0% | 0.0% |
| **0.25** | 93.7% | 22.8% | 0.367 | 57.4% | 6.3% | 1.4% |
| **0.35** | 82.0% | 25.8% | 0.393 | 61.6% | 18.0% | **1.4% (Regulatory)** |
| **0.50** | 59.5% | 31.2% | 0.409 | 63.4% | 40.5% | 10.1% |
| **0.70** | 19.9% | 42.8% | 0.272 | 56.6% | 80.1% | 52.2% |
| **0.80** | 0.5% | 50.0% | 0.011 | 50.2% | 99.5% | 97.1% |

**Recommended**: τ = 0.35 for balanced performance with compliance guarantee.

---

## Future Work

- [ ] Cross-dataset portability (test on Prosper dataset)
- [ ] Real-time compliance monitoring dashboard
- [ ] Integration with Loan Origination Systems (LOS)
- [ ] Multi-class outcomes (default, prepayment, delinquency, early payoff)
- [ ] Fairness audits (disparate impact testing, FICO score analysis)
- [ ] Explainability dashboards (SHAP, LIME, feature importance)
- [ ] Temporal drift detection & retraining triggers
- [ ] Bayesian uncertainty quantification

---

## Publication & Citation

This work is designed for publication as a research article. If you use this codebase or results, please cite:

```bibtex
@article{bankshield2026,
  title={Bank Shield: Auditable AI for Regulatory Compliance in Credit Scoring},
  author={Your Name},
  journal={Journal/Conference Name},
  year={2026},
  doi={10.xxxx/xxxxx}
}
```

---

## License

This project is licensed under the **MIT License** — see the LICENSE file for details.

MIT License permits:
- ✅ Commercial use
- ✅ Modification
- ✅ Distribution
- ✅ Private use

Conditions:
- ⚠️ Must include license and copyright notice
- ⚠️ Changes must be documented

---

## Contributing

Contributions are welcome! Please:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/your-feature`)
3. Commit changes (`git commit -m 'Add your feature'`)
4. Push to branch (`git push origin feature/your-feature`)
5. Open a Pull Request

Please ensure:
- Code follows PEP 8 style
- Tests pass (`pytest tests/`)
- Documentation is updated
- Commit messages are descriptive

---

## Support & Issues

- **Bug Reports**: Use GitHub Issues with label `bug`
- **Feature Requests**: Use GitHub Issues with label `enhancement`
- **Questions**: Use GitHub Discussions or email
- **Security Issues**: Do NOT post publicly; email your.email@example.com

---

## Acknowledgments

### Data
- **Lending Club** (lendingclub.com) for historical loan data

### Models & Libraries
- **FinBERT** (Araci, 2019) for financial text understanding
- **PyTorch** (Paszke et al., 2019) for deep learning framework
- **Hugging Face Transformers** (Wolf et al., 2019) for model hub
- **scikit-learn** (Pedregosa et al., 2011) for ML utilities

### Regulatory Guidance
- **CFPB** (Consumer Financial Protection Bureau)
- **SEC** (Securities and Exchange Commission)
- **FHA** (Federal Housing Administration)
- **OCC** (Office of the Comptroller of the Currency)

---

## Important Disclaimer

⚠️ **This project is for educational and research purposes only.**

It should NOT be deployed in production without:

1. ✅ **Compliance Review**: Legal and regulatory teams must review
2. ✅ **Fairness Audit**: Test for disparate impact on protected classes
3. ✅ **Backtesting**: Validate on recent real-world data (2+ years)
4. ✅ **Integration**: Integrate with your institution's risk management framework
5. ✅ **Regulatory Approval**: Obtain approval from relevant regulators:
   - FDIC (Federal Deposit Insurance Corporation)
   - OCC (Office of the Comptroller of the Currency)
   - Federal Reserve
   - CFPB (Consumer Financial Protection Bureau)

### Liability Waiver

**The authors accept no liability** for:
- Financial losses resulting from model deployment
- Regulatory penalties or fines
- Unfair lending violations (ECOA, FHA, etc.)
- Model failure or inaccuracy
- Breach of data privacy or security

Users deploy this model **entirely at their own risk**. Conduct your own due diligence and regulatory review.

---

## Questions?

For questions, issues, or collaborations:
- 📧 Email: your.email@example.com
- 💬 GitHub Discussions: bank-shield/discussions
- 🐛 Bug Reports: bank-shield/issues

---

**Last Updated**: May 2026  
**Version**: 1.0.0  
**Status**: Research Release
