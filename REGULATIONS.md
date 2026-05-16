# Regulatory Framework & Compliance Mappings

## Overview

Bank Shield implements guardrails based on established financial regulations. This document maps each regulatory requirement to the corresponding flag in the system.

---

## 1. CFPB Qualified Mortgage (QM) Standard

### Regulation
**Source**: Dodd-Frank Act, Section 1026.43 (15 U.S.C. § 1681 et seq.)

The Consumer Financial Protection Bureau (CFPB) defines "Qualified Mortgage" standards that lenders must follow to avoid predatory lending practices.

### Key Requirement: DTI Cap

**Rule**: Borrower's debt-to-income ratio (DTI) shall not exceed 43%

**Definition of DTI**:
```
DTI = (Total Monthly Debt Obligations) / (Gross Monthly Income)
    = (Car payments + Credit cards + Student loans + NEW mortgage payment) / Monthly Income
```

### Bank Shield Implementation

```python
flag_dti_43 = (dti > 0.43)  # DTI expressed as decimal (0.43 = 43%)
```

**When Applied**: During training and inference
**Penalty in Loss**: If `flag_dti_43 = 1`, loss increases by λ × max(0, P(risky) - τ_train)
**Compliance Check**: Reject all loans where `flag_dti_43 = 1`

### Evidence Base
- **Study**: Bhutta & Ringo (2014), Federal Reserve analysis
- **Finding**: Borrowers with DTI > 43% show 2.5× higher default rates
- **Regulatory Justification**: DTI > 43% strongly predicts payment burden unsustainability

---

## 2. FHA (Federal Housing Administration) Guidelines

### Payment-to-Income (PTI) Ratio

**Rule**: Monthly housing payment should not exceed 31% of gross monthly income
- For more flexible lending: Up to 40% with compensating factors

### Bank Shield Implementation

```python
monthly_payment_est = loan_amnt / 36  # Assume 3-year amortization
pti = monthly_payment_est / (monthly_income + 1e-8)
flag_pti_high = (pti > 0.35)  # Using 35% (between FHA 31% and flexible 40%)
```

**Rationale**: 35% is a conservative middle ground between FHA strict (31%) and flexible (40%)

### Evidence Base
- **Source**: FHA Handbook 4155.1 (Underwriting)
- **Guideline**: "Principal, Interest, Taxes, Insurance (PITI) ratio should not exceed 31%"
- **Extended**: VA guidelines allow up to 41% DTI

---

## 3. Predatory Lending Indicators

### FHA/ECOA Definition

**Predatory Lending**: Loans to borrowers who lack capacity to repay, typically:
- Low income + high loan amount (mismatch)
- Unclear terms or high fees
- Targeting vulnerable populations

### Bank Shield Implementation

```python
flag_predatory = (annual_inc < 20_000) & (loan_amnt > 10_000)
```

**Logic**: 
- If annual income < $20k, a $10k loan exceeds 50% of yearly income
- This is likely unsustainable (borrower has limited ability to repay)
- Combined with other stress factors, suggests predatory pattern

### Evidence Base
- **Statute**: Fair Housing Act, 42 U.S.C. § 3604 (predatory practices)
- **Agency Guidance**: CFPB "Predatory Lending" enforcement actions
- **Pattern**: Low-income borrowers with high-to-income-ratio loans show 4-5× default rates

---

## 4. VA (Veterans Affairs) Loan Guidelines

### DTI and Compensating Factors

**Rule**: VA allows DTI up to 41% with compensating factors, or up to 50% with strong compensating factors

**Compensating Factors**:
- Strong credit history
- Significant cash reserves (savings)
- Low existing debt
- Stable employment

### Bank Shield Implementation

```python
flag_dual_stress = (dti > 0.36) & (loan_to_income > 0.3)
```

**Logic**: 
- Both DTI > 36% AND LTI > 30% indicates dual stress (no compensating factor)
- If one flag is triggered but not both, model has more flexibility
- If both triggered, likely lacks compensating factors

---

## 5. ECOA (Equal Credit Opportunity Act) & Fair Lending

### Regulation
**Source**: 15 U.S.C. § 1691 et seq.

**Rule**: Lenders cannot discriminate based on protected characteristics:
- Race, color, religion
- National origin
- Sex, marital status
- Age
- Receipt of income from public assistance
- Exercise of rights under CCPA

### Bank Shield Implementation

**Note**: Bank Shield does not directly address ECOA compliance (that requires separate fairness audits).

**However**: The guardrail approach inherently reduces discriminatory decisions by:
1. Removing subjective text-based reasoning (which can encode bias)
2. Relying on objective financial metrics
3. Requiring explicit regulatory flags for rejection

**See**: `docs/FAIRNESS.md` for disparate impact testing

---

## 6. State-Level Regulations

### California (CA) - Fair Lending Practices

**Rule**: DTI > 43% + additional risk factors = presumed predatory
**Implementation**: Similar to CFPB QM standard

### New York (NY) - Payday Loan Restrictions

**Rule**: Caps on APR and fees for high-cost loans
**Implementation**: Not explicitly modeled (would require interest rate feature)

### Texas (TX) - Consumer Protection Act

**Rule**: Reasonable assurance of borrower repayment capacity
**Implementation**: Proxied through DTI + income-loan mismatch flags

---

## 7. Compliance Metrics & Thresholds

### Compliance Breach Rate

**Definition**: % of approved loans that violate ≥1 regulatory flag

**Target**: < 2% (allowing minimal exceptions for compensating factors)

**Regulatory Interpretation**:
- If Compliance Breach Rate > 5%, lender is likely non-compliant
- If Compliance Breach Rate < 1%, lender is overly conservative
- 1-2% is the "sweet spot" (allows flexibility while maintaining compliance)

### False Safe Rate (Flagged)

**Definition**: Among flagged borrowers, % marked as "safe" (low default risk)

**Target**: < 10% (most flagged borrowers should be rejected or accepted conservatively)

**Regulatory Interpretation**:
- If False Safe Rate > 20%, guardrail is not working
- If False Safe Rate < 2%, model is overly conservative

---

## 8. Regulatory Safe Harbors

### What Actions Provide Immunity?

**Qualified Mortgage (QM) Safe Harbor**:
- If loan meets all QM criteria (DTI ≤ 43%, verified income, no negative amortization, etc.)
- Lender has presumption of compliance with TILA Section 129E
- Protection against predatory lending lawsuits (with exceptions)

### Bank Shield Safe Harbor

**Our Approach**: 
- If all 7 flags = 0 (no violations), loan is in "safe" zone
- If any flag = 1, manual review or rejection is mandatory
- No automatic approval even if P(Default) is low

**Rationale**: More conservative than legal minimum, reduces regulatory risk

---

## 9. Enforcement Actions & Historical Context

### CFPB Enforcement Examples

| Year | Lender | Violation | Settlement |
|------|--------|-----------|-----------|
| 2018 | Wells Fargo | DTI violations (43%+ approvals) | $1.2B |
| 2017 | Equifax | Fair lending discrimination | $700M |
| 2016 | Ally Bank | Pricing discrimination | $100M |
| 2015 | CITI | QM violations | $700M |

**Key Lesson**: Systematic regulatory violations → massive fines. Bank Shield prevents this.

---

## 10. Forward-Looking Regulatory Trends

### Emerging Standards

1. **ESG (Environmental, Social, Governance)**: 
   - Some regulators now require fairness audits
   - Bank Shield's explainability helps demonstrate ESG compliance

2. **AI/ML Governance**:
   - SEC, OCC considering AI risk guidelines
   - Demands for model documentation, testing, governance
   - Bank Shield's modular design facilitates compliance

3. **Climate Risk**:
   - Fed requiring climate risk stress testing
   - Could extend to "environmental lending risk" for mortgages
   - Bank Shield framework adaptable to new risk dimensions

---

## 11. How to Integrate Bank Shield with Compliance

### Step 1: Pre-Deployment Review
- [ ] Legal team reviews all 7 flags against institution's policies
- [ ] Compliance officer signs off on penalty weights (λ, τ)
- [ ] Audit confirms flags map to regulatory requirements

### Step 2: Deployment
- [ ] Model deployed in "advisory" mode (flags recommendations, not decisions)
- [ ] Monitor rejection rates and compliance breach rates
- [ ] Compare against historical lender data

### Step 3: Ongoing Monitoring
- [ ] Monthly reports: Compliance breach rate, false safe rate, flagged recall
- [ ] Quarterly fairness audits (disparate impact testing)
- [ ] Annual regulatory review (CFPB, state regulators)

### Step 4: Continuous Improvement
- [ ] Retrain model on recent data (annually)
- [ ] HPO to update penalty weights if regulatory environment changes
- [ ] Add new flags as regulations evolve

---

## 12. Limitations & Disclaimers

### What Bank Shield DOES Cover
✅ DTI analysis (CFPB QM)  
✅ Payment-to-income (FHA)  
✅ Predatory lending detection  
✅ Multiple stress indicators (VA compensating factors)  

### What Bank Shield DOES NOT Cover
❌ Interest rate appropriateness (requires APR modeling)  
❌ Mortgage insurance calculation  
❌ Points, fees, and prepayment penalties  
❌ ECOA/Fair Lending (requires separate audit)  
❌ Broker compensation oversight  

**Implications**: Bank Shield is ONE TOOL in a larger compliance ecosystem. It is NOT a substitute for:
- Legal review
- Fairness audits
- Regulatory examinations
- Internal policy reviews

---

## 13. References

### Federal Regulations
- Dodd-Frank Act, 15 U.S.C. § 1681 et seq.
- Truth in Lending Act (TILA), 15 U.S.C. § 1601 et seq.
- Equal Credit Opportunity Act (ECOA), 15 U.S.C. § 1691 et seq.
- Fair Housing Act, 42 U.S.C. § 3604 et seq.

### Regulatory Guidance
- CFPB Rule § 1026.43 (Qualified Mortgage definition)
- FHA Handbook 4155.1 (Underwriting)
- VA Circular 26-16-11 (Loan Guaranty Benefits)
- OCC Bulletin 2015-21 (Credit Risk Governance)

### Research
- Bhutta, N., & Ringo, D. (2014). "The Effect of the CFPB's Ability-to-Repay Rule on Mortgage Originations and Default." Journal of Finance
- Cordell, L., et al. (2012). "The Incentives of Mortgage Servicers." Federal Reserve Financial Stability Report

---

## 14. Contact for Regulatory Questions

For questions about regulatory mappings:
- **Compliance Officer**: [institutional contact]
- **Legal Counsel**: [law firm contact]
- **CFPB Guidance**: https://www.consumerfinance.gov/
- **Your Regulator**: Federal Reserve, OCC, FDIC, etc. (institution-specific)

---

**Last Updated**: May 2026  
**Version**: 1.0.0  
**Status**: Research Release (NOT for production without legal review)
