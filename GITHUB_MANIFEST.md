# Bank Shield GitHub Repository - File Manifest & Setup Guide

## 📋 Quick Navigation

This document helps you understand and navigate the Bank Shield repository structure.

---

## 📁 Repository Structure

### Root Level Files

| File | Purpose | Size |
|------|---------|------|
| `GITHUB_README.md` | Main project documentation | 22 KB |
| `ARCHITECTURE.md` | Technical system design | 18 KB |
| `REGULATIONS.md` | Regulatory framework mappings | 10 KB |
| `RESULTS_SUMMARY.md` | Key findings & results analysis | 11 KB |
| `requirements.txt` | Python dependencies | <1 KB |
| `LICENSE` | MIT License | <1 KB |
| `CITATION.md` | How to cite this project | <1 KB |

### Code Structure (src/)

```
src/
├── __init__.py
├── config.py                          # Configuration management
├── data/
│   ├── __init__.py
│   ├── loader.py                      # Load & preprocess data
│   ├── features.py                    # Feature engineering (flags, derived features)
│   └── dataset.py                     # PyTorch Dataset class
├── models/
│   ├── __init__.py
│   ├── finbert_hybrid.py              # Hybrid FinBERT + Numeric + Flags model
│   ├── loss.py                        # Custom loss with regulatory penalty
│   └── training.py                    # Training & evaluation functions
├── evaluation/
│   ├── __init__.py
│   ├── metrics.py                     # Compliance & performance metrics
│   ├── calibration.py                 # Calibration analysis
│   └── adversarial.py                 # Adversarial robustness testing
└── utils/
    ├── __init__.py
    ├── seed.py                        # Reproducibility & seed management
    └── logging.py                     # Logging utilities
```

### Scripts (scripts/)

```
scripts/
├── train.py                           # Main training script
├── evaluate.py                        # Evaluate on test set
├── adversarial_test.py                # Adversarial robustness tests
├── hpo.py                             # Hyperparameter optimization
└── reproduce_results.py               # Reproducibility check
```

### Configuration (configs/)

```
configs/
└── default.yaml                       # Default hyperparameters & settings
```

### Notebooks (notebooks/)

```
notebooks/
└── bank_shield_full_pipeline.ipynb    # Complete end-to-end analysis
```

### Results (results/)

This folder contains all experiment results and should NOT be committed to Git (too large):

```
results/
├── clean_performance_table.csv
│   └── Performance metrics (accuracy, precision, recall, compliance)
│
├── threshold_sweep_table.csv
│   └── Metrics across different decision thresholds (0.05 to 0.95)
│
├── violation_score_table.csv
│   └── Metrics broken down by violation severity
│
├── adversarial_robustness_table.csv
│   └── Attack success rates & probability shifts
│
├── compliance_table.csv
│   └── Per-violation compliance breakdown
│
├── synthetic_shift_results.csv
│   └── Domain adaptation results
│
├── temporal_shift_results.csv
│   └── Temporal drift analysis
│
├── plots/                             # All visualizations (PNG)
│   ├── roc_pr_calibration.png
│   ├── confusion_matrices.png
│   ├── compliance_heatmap.png
│   ├── compliance_by_violation.png
│   ├── violation_score_plot.png
│   ├── pareto_frontier.png
│   ├── threshold_sweep_plot.png
│   ├── adversarial_shift_plots.png
│   ├── probability_distributions.png
│   ├── training_curves.png
│   ├── synthetic_shift_comparison.png
│   └── temporal_shift_comparison.png
│
└── sample_predictions/                # Example predictions for analysis
    ├── baseline_fooled_examples.csv
    ├── regulatory_fooled_examples.csv
    ├── baseline_fooled_flagged_examples.csv
    └── regulatory_fooled_flagged_examples.csv
```

---

## 🚀 Getting Started

### Step 1: Clone & Install

```bash
# Clone repository
git clone https://github.com/yourusername/bank-shield.git
cd bank-shield

# Create virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt
```

### Step 2: Download Data

The Lending Club dataset is available from Kaggle:
- https://www.kaggle.com/datasets/wordsforthewise/lending-club

Download and place in a `data/` directory:
```
data/
├── accepted_2007_to_2018Q4.csv.gz
└── rejected_2007_to_2018Q4.csv.gz
```

### Step 3: Run the Pipeline

```bash
# Preprocess data
python scripts/data_prep.py

# Train baseline model
python scripts/train.py --model baseline

# Train regulatory model
python scripts/train.py --model regulatory

# Run evaluation
python scripts/evaluate.py

# Test adversarial robustness
python scripts/adversarial_test.py

# Generate final report
python scripts/generate_report.py
```

### Step 4: View Results

```bash
# Open Jupyter notebook
jupyter notebook notebooks/bank_shield_full_pipeline.ipynb

# Or view CSV results directly
head results/clean_performance_table.csv
head results/compliance_table.csv
```

---

## 📖 Documentation Guide

**What to read based on your role:**

### 👨‍💼 For Business / Product Managers
1. `GITHUB_README.md` — Overview & key findings (section "Key Results")
2. `RESULTS_SUMMARY.md` — Detailed results & business implications

### 👨‍💻 For Software Engineers / ML Engineers
1. `ARCHITECTURE.md` — System design & components
2. `notebooks/bank_shield_full_pipeline.ipynb` — Implementation details
3. `src/models/finbert_hybrid.py` — Model architecture code

### ⚖️ For Compliance / Legal
1. `REGULATIONS.md` — Regulatory framework mappings
2. `RESULTS_SUMMARY.md` — Compliance metrics (section "Regulatory Compliance Breakdown")
3. `GITHUB_README.md` — Regulatory framework section

### 🔬 For Researchers / Data Scientists
1. `ARCHITECTURE.md` — Detailed technical design
2. `RESULTS_SUMMARY.md` — Complete experimental results
3. `notebooks/bank_shield_full_pipeline.ipynb` — Full reproducible pipeline

---

## 📊 Key Results at a Glance

### Main Finding
**Regulatory model reduces compliance breaches by 66% with only 2.8% loss in recall**

| Metric | Baseline | Regulatory | Winner |
|--------|----------|-----------|--------|
| Recall | 82.0% | 79.2% | Baseline |
| Precision | 25.8% | 26.4% | Regulatory |
| **Compliance Breach** | **4.3%** | **1.4%** | **Regulatory ✓** |
| Adversarial Attack Success | 0.13% | 0.0% | **Regulatory ✓** |

### Adversarial Robustness
- Baseline fooled on 7 risky cases
- Regulatory fooled on 0 risky cases
- **Conclusion**: Hybrid approach (text + numeric + flags) is robust to text manipulation

---

## ✅ Reproducibility Checklist

- [x] Fixed random seed (SEED=42)
- [x] Stratified train/val/test split
- [x] Frozen model weights
- [x] Full hyperparameter logging
- [x] Deterministic preprocessing
- [x] GPU reproducibility (manual_seed_all)

**To reproduce**:
```bash
python scripts/reproduce_results.py --seed 42 --dataset lending_club
```

---

## 📝 Important Files

### For Model Inspection

**File**: `src/models/finbert_hybrid.py`
**Contains**: 
- FinBERTHybrid class (main model)
- Fusion layer implementation
- Forward pass logic

```python
model = FinBERTHybrid(
    model_name="ProsusAI/finbert",
    num_numeric_features=9,
    num_flags=7,
    hidden_dim=256,
    dropout=0.1
)
```

### For Loss Function Inspection

**File**: `src/models/loss.py`
**Contains**:
- BaselineLoss (standard cross-entropy)
- RegulatoryLoss (custom loss with penalty)

```python
loss_fn = RegulatoryLoss(
    lambda_reg=0.1,
    tau_train=0.5
)
```

### For Feature Engineering

**File**: `src/data/features.py`
**Contains**:
- Regulatory flag computation (7 flags)
- Derived feature engineering
- Feature scaling

---

## 🎯 Common Tasks

### How to Change Hyperparameters?

Edit `configs/default.yaml`:
```yaml
training:
  batch_size: 64
  num_epochs: 5
  learning_rate: 2.0e-5
  weight_decay: 0.01

regularization:
  lambda_reg: 0.1        # Penalty weight
  tau_train: 0.5         # Training threshold
  tau_decision: 0.35     # Decision threshold
```

### How to Add a New Regulatory Flag?

1. Add flag computation in `src/data/features.py`
2. Update `FLAG_COLS` list
3. Update `VIOLATION_WEIGHTS` dictionary
4. Retrain model

### How to Test on Your Own Data?

```python
# Load your data
df = pd.read_csv('your_data.csv')

# Prepare features
df = prepare_df(df)

# Predict
model = load_model('results/regulatory_model/best.pt')
predictions = model.predict(df)
```

---

## 🐛 Troubleshooting

### Issue: Out of Memory (OOM)

**Solution**: Reduce batch_size in `configs/default.yaml`
```yaml
training:
  batch_size: 32  # from 64
```

### Issue: Model Overfitting to Compliance

**Solution**: Reduce penalty weight in `configs/default.yaml`
```yaml
regularization:
  lambda_reg: 0.05  # from 0.1
```

### Issue: Results Don't Match Paper

**Solution**: Ensure you're using the exact same configuration and seed:
```bash
python scripts/reproduce_results.py --seed 42 --dataset lending_club --verbose
```

---

## 📚 Additional Resources

### Regulatory References
- CFPB Qualified Mortgage Rule: https://www.consumerfinance.gov/rules-policy/regulations/1026/
- FHA Handbook 4155.1: https://www.fha.gov/
- FinBERT Model: https://github.com/ProsusAI/finbert

### Recommended Reading
- "Auditable AI" concept (this project)
- FinBERT paper: Araci, D. (2019)
- CFPB Mortgage Compliance Guide

### Citation

```bibtex
@article{bankshield2026,
  title={Bank Shield: Auditable AI for Regulatory Compliance in Credit Scoring},
  author={Your Name},
  year={2026}
}
```

---

## 📞 Contact & Support

- **Issues & Bugs**: GitHub Issues
- **Questions**: GitHub Discussions
- **Regulatory Questions**: See REGULATIONS.md or contact your compliance officer

---

## ⚠️ Important Disclaimer

**This is a research project for educational purposes only.** Do NOT deploy in production without:
1. Legal review
2. Fairness audit (disparate impact testing)
3. Regulatory pre-approval
4. Integration with your institution's risk framework

See GITHUB_README.md for full disclaimer.

---

**Last Updated**: May 2026  
**Version**: 1.0.0  
**Status**: Research Release
