# Bank Shield - Quick Reference Card

## 🎯 Project at a Glance

**What**: Auditable AI for regulatory compliance in credit scoring  
**Why**: 66% reduction in compliance breaches while maintaining accuracy  
**How**: Hybrid FinBERT + Numeric + Regulatory Flags + Custom Loss Function  

---

## 📊 Key Numbers

| Metric | Value | Impact |
|--------|-------|--------|
| **Compliance Breach Reduction** | 66% | Baseline 4.3% → Regulatory 1.4% |
| **Recall Trade-off** | -2.8% | 82.0% → 79.2% (acceptable) |
| **Adversarial Attack Success** | 0.0% | Immune to text manipulation |
| **Truly Risky Cases Fooled** | 7:0 | Baseline 7× more vulnerable |
| **F1 Score Improvement** | +0.003 | 0.393 → 0.396 |
| **Test Set Size** | 7,500 | Lending Club loans |
| **Default Rate** | 13.3% | Among test set loans |

---

## 🏗️ Architecture (5 Components)

```
1. FinBERT Encoder
   └─ Input: Text (purpose, employment, grade, etc.)
   └─ Output: 768-dim embeddings

2. Numeric Scaler
   └─ Input: 9 financial features (income, DTI, LTI, etc.)
   └─ Output: Scaled numeric vector

3. Flag Detector
   └─ Input: Derived financial metrics
   └─ Output: 7 binary regulatory flags

4. Fusion Layer
   └─ Concatenate: [FinBERT (768) + Numeric (9) + Flags (7)]
   └─ Dense: 784 → 256 → 2 (logits)

5. Custom Loss
   └─ Loss = CrossEntropy + λ × Penalty(flags)
   └─ λ = 0.1 (penalty weight)
```

---

## 🚩 7 Regulatory Flags

| # | Flag | Rule | Basis |
|---|------|------|-------|
| 1 | `dti_43` | DTI > 43% | CFPB QM |
| 2 | `lti_high` | LTI > 50% | Predatory indicator |
| 3 | `pti_high` | PTI > 35% | FHA guideline |
| 4 | `predatory` | Income < $20k & Loan > $10k | Predatory lending |
| 5 | `dual_stress` | DTI > 36% AND LTI > 30% | VA compensating factors |
| 6 | `debt_burden` | Existing debt > 50% income | Debt sustainability |
| 7 | `large_loan_dti` | Large loan + DTI > 30% | Oversized loan |

---

## 💻 Installation (3 Lines)

```bash
git clone https://github.com/yourusername/bank-shield.git
cd bank-shield
pip install -r requirements.txt
```

---

## 📁 Documentation Map

| File | Purpose | Audience | Read Time |
|------|---------|----------|-----------|
| GITHUB_README.md | Overview & features | Everyone | 15 min |
| ARCHITECTURE.md | Technical deep-dive | Engineers | 20 min |
| REGULATIONS.md | Regulatory mappings | Compliance | 15 min |
| RESULTS_SUMMARY.md | Findings & analysis | Decision makers | 15 min |
| GITHUB_MANIFEST.md | Navigation guide | Developers | 10 min |

---

## 🎓 Key Insights

### Finding #1: Accuracy ≠ Compliance Sacrifice
- F1 Score: 0.393 → 0.396 (+0.003, improved!)
- Balanced Accuracy: 61.6% → 62.0% (+0.4%, improved!)
- Recall: 82% → 79% (-2.8%, acceptable trade-off)

### Finding #2: Text Can Be Defeated
- Baseline fooled on 7 risky cases (0.13% attack success rate)
- Regulatory fooled on 0 risky cases (0.0% attack success rate)
- Numeric + flags override persuasive language

### Finding #3: Violations Predict Risk
- No flags: 19.6% default rate
- Critical violations: 42.9% default rate
- 2.2× higher default when flagged

### Finding #4: Operating Point Matters
- τ = 0.25: 93.7% recall, 1.4% compliance breaches (conservative)
- τ = 0.35: 82.0% recall, 1.4% compliance breaches (balanced)
- τ = 0.50: 59.5% recall, 10.1% compliance breaches (risky)

---

## 📈 Performance at Recommended Threshold (τ = 0.35)

### Regulatory Model (RECOMMENDED)
```
Recall (Sensitivity)......79.2%  [catches 4 out of 5 defaulters]
Precision.................26.4%  [2 out of 8 approvals will default]
F1 Score..................0.396  [balanced metric]
Compliance Breach Rate....1.4%   [1.4% of approvals violate rules]
ROC-AUC...................0.683  [good discrimination]
Adversarial Robustness...0.0%    [immune to text attacks]
```

---

## 🔧 Hyperparameter Tuning

Three hyperparameters control the regulatory model:

```yaml
regularization:
  lambda_reg: 0.1        # Penalty weight (default: 0.1)
  tau_train: 0.5         # Training threshold (default: 0.5)
  tau_decision: 0.35     # Decision threshold (default: 0.35)
```

**How to tune**:
- **Increase λ** if compliance breaches still too high
- **Decrease λ** if too many false rejections
- **Sweep τ_decision** to find best accuracy-compliance tradeoff

---

## ✅ Reproducibility Guarantee

✓ Fixed seed (SEED=42)  
✓ Stratified splits  
✓ Frozen weights  
✓ Full logging  
✓ Deterministic preprocessing  
✓ GPU reproducibility  

**To reproduce**:
```bash
python scripts/reproduce_results.py --seed 42
```

---

## ⚖️ Regulatory Compliance

### Regulations Covered
- ✅ CFPB Qualified Mortgage (DTI ≤ 43%)
- ✅ FHA Guidelines (PTI ≤ 35%)
- ✅ VA Loan Rules (DTI with compensating factors)
- ✅ Predatory Lending (ECOA considerations)

### Regulations NOT Covered
- ❌ Interest rate appropriateness (requires APR modeling)
- ❌ Mortgage insurance calculation
- ❌ Points & fees analysis
- ❌ Broker compensation oversight

---

## 🚀 Deployment Checklist

- [ ] Legal review of flag definitions
- [ ] Fairness audit (disparate impact testing)
- [ ] Backtesting on 2022-2025 data
- [ ] Integration with loan origination system
- [ ] Regulatory pre-approval (CFPB, state regulators)
- [ ] Staff training on guardrails & explainability
- [ ] Monitoring dashboard for compliance breaches
- [ ] Quarterly fairness audits

---

## 📚 Citation Format

```bibtex
@article{bankshield2026,
  title={Bank Shield: Auditable AI for Regulatory Compliance in Credit Scoring},
  author={Your Name},
  journal={Journal Name},
  year={2026},
  doi={10.xxxx/xxxxx}
}
```

---

## 🎯 Use Cases

### ✅ Good Fit
- Credit scoring (unsecured consumer loans)
- Mortgage underwriting
- Risk assessment
- Regulatory audits
- Fair lending monitoring

### ❌ Poor Fit
- Auto insurance pricing
- Employment decisions
- Medical underwriting
- Public benefits eligibility
- Law enforcement decisions

---

## 💾 File Sizes (Documentation Only)

```
GITHUB_README.md........22 KB
ARCHITECTURE.md.........18 KB
REGULATIONS.md..........10 KB
RESULTS_SUMMARY.md......11 KB
GITHUB_MANIFEST.md......10 KB
requirements.txt........<1 KB
────────────────────────────
TOTAL..................~71 KB
```

**Note**: Large files (datasets, models) stored separately

---

## 🔒 Security Notes

**Do NOT commit to GitHub**:
- ❌ Training data (contains PII)
- ❌ Model weights (440 MB FinBERT + checkpoints)
- ❌ API keys or credentials
- ❌ Internal compliance policies

**DO commit to GitHub**:
- ✅ Documentation (no sensitive data)
- ✅ Code structure (no hardcoded secrets)
- ✅ Requirements.txt (package versions)
- ✅ LICENSE file

---

## 📞 Quick Support

### "How do I use this?"
→ Read GITHUB_README.md (Section: Quick Start)

### "How does it work?"
→ Read ARCHITECTURE.md (Section: System Overview)

### "Is it compliant?"
→ Read REGULATIONS.md (all sections)

### "What are the results?"
→ Read RESULTS_SUMMARY.md (Section: Main Results)

### "How do I set this up?"
→ Read GITHUB_MANIFEST.md (Section: Getting Started)

---

## ⚠️ Legal Disclaimer

**For Educational/Research Use Only**

This project should NOT be deployed in production without:
1. Legal review by internal counsel
2. Fairness audit (disparate impact testing)
3. Regulatory pre-approval (CFPB, OCC, Fed, etc.)
4. Integration with institutional risk framework
5. Board/Management approval

**No liability accepted** for:
- Financial losses from deployment
- Regulatory penalties
- Unfair lending violations
- Model failure or inaccuracy

---

**Quick Facts Summary**

- **Problem**: AI credit models ignore regulations
- **Solution**: Bake regulations into the math (loss function)
- **Result**: 66% fewer compliance breaches, immune to text attacks
- **Trade-off**: 2.8% fewer defaults caught (acceptable)
- **Status**: Research project, ready for publication
- **Data**: 50,000 Lending Club loans (2007-2018)
- **Model**: FinBERT + Numeric + Flags + Custom Loss
- **Performance**: 79% recall, 26% precision, 1.4% compliance breaches

---

**Version**: 1.0.0 | **Status**: Research Release | **Date**: May 2026
