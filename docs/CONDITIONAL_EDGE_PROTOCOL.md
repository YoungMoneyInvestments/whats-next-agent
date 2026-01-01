# Conditional Edge Discovery Protocol

## Problem Statement

Quantitative research agents often reach premature conclusions when unconditional results are weak or flat. The classic failure mode:

```
Unconditional Sharpe: 0.3
Agent Conclusion: "No edge exists. Reject."
```

This is epistemically lazy. A 0.3 unconditional Sharpe could mask a 1.5 Sharpe in specific market regimes, instruments, or conditions. The problem isn't that edge doesn't exist—it's that edge may be **conditional**.

## The Solution: Structured Conditional Search

The Conditional Edge Discovery Protocol ensures agents cannot conclude "no edge" without systematically searching for conditional pockets of alpha.

### Core Principle

> "No edge exists" is FORBIDDEN as a conclusion from unconditional averages alone.
>
> The only valid conclusions are:
> - "No UNCONDITIONAL edge exists" (reframing)
> - "No edge found after exhaustive conditional search" (earned conclusion)

## How It Works

### 1. Trigger Conditions

The protocol activates when:
- Unconditional results are flat (Sharpe < 0.5)
- Unconditional results are negative
- Metrics disagree (positive returns but negative Sharpe)
- Results are surprisingly weak given hypothesis

### 2. Segmentation Axes

When triggered, agents must generate hypotheses across these axes:

| Axis | Examples | Rationale |
|------|----------|-----------|
| **Instrument** | ES vs NQ vs individual stocks | Edge may concentrate in specific products |
| **Sector/Asset Class** | Tech vs Financials, Equity vs Commodities | Different market dynamics |
| **Market Cap** | Large vs Mid vs Small cap | Liquidity and efficiency differences |
| **Volatility Regime** | VIX < 15 vs VIX 15-25 vs VIX > 25 | Strategy behavior varies by regime |
| **Liquidity** | High vs low volume periods | Execution and edge differ |
| **Trend Regime** | Trending vs mean-reverting | Strategy type dependencies |
| **Calendar/Event** | FOMC days, earnings, month-end | Event-driven concentration |
| **Time of Day** | Open, close, overnight | Microstructure effects |

### 3. P-Hacking Prevention

Testing multiple segments creates multiple comparison risk. The protocol enforces:

| Segments Tested | Required Correction |
|-----------------|---------------------|
| 1-3 | Document all tested |
| 4-10 | Bonferroni correction (α / N) |
| 11+ | FDR control (Benjamini-Hochberg) |

**Additionally:**
- Minimum 30% holdout for pocket validation
- All segments tested must be documented (not just winners)
- Pre-registration of hypotheses before testing
- Walk-forward validation required for production consideration

### 4. State Tracking

The protocol maintains persistent state across iterations:

```json
{
  "segmentation_hypotheses": [
    {
      "axis": "volatility_regime",
      "hypothesis": "Edge concentrates in low-vol (VIX < 20)",
      "test_design": "Split sample by VIX quintile",
      "status": "tested",
      "result": "Sharpe 1.2 in lowest quintile vs 0.1 overall"
    }
  ],
  "segmentation_tests_run": ["volatility_regime", "time_of_day"],
  "best_pockets_found": [
    {
      "segment": "VIX < 20",
      "sharpe": 1.2,
      "sample_size": 245,
      "holdout_validated": false
    }
  ],
  "p_hacking_risk_notes": "2 segments tested, both documented, no correction needed yet"
}
```

### 5. Classification Integration

The protocol introduces a new classification:

| Classification | Meaning |
|----------------|---------|
| `conditional_alpha_candidate` | Pocket identified but not holdout-validated |

**Upgrade path:** conditional_alpha_candidate → paper_alpha
- Pocket validated on holdout (min 30%)
- P-hacking controls documented
- Multiple testing correction applied

**Downgrade triggers:**
- Pocket fails holdout
- P-hacking detected
- Pocket too small for capacity

## Agent-Specific Requirements

### quant-research-generator
- MUST include pocket search plans when unconditional results weak
- MUST propose 2-3 segmentation hypotheses per iteration
- MUST specify falsification criteria for each hypothesis

### quant-skeptic-redteam
- MUST include "Where Could Edge Still Be Hiding" section
- FORBIDDEN from concluding "no edge" from unconditional averages
- MUST propose at least 3 conditional hypotheses even when skeptical

### quant-ml-validation-engineer
- MUST apply multiple comparisons correction
- MUST specify holdout validation design
- MUST document all segments tested, not just winners

### quant-execution-microstructure
- MUST analyze execution in specific segments/pockets
- MUST calculate segment-specific capacity (often << unconditional)
- MUST identify adverse selection risks per pocket

### quant-capital-allocation-risk
- MUST analyze pocket-specific capacity and crowding
- MUST evaluate pocket correlation (pockets often correlate in stress)
- MUST size for pocket-level, not unconditional, capacity

### quant-manager-audit
- MUST verify conditional edge protocol was followed
- MUST check p-hacking controls before classification
- MUST require holdout validation for paper_alpha upgrade

## Why This Prevents Premature Conclusions

### Before (Failure Mode)
```
Iteration 1: "Sharpe 0.3, no edge, reject"
Result: Potentially profitable strategy abandoned
```

### After (Protocol Enforced)
```
Iteration 1: "Unconditional Sharpe 0.3, reframing to conditional search"
Iteration 2: "Testing volatility regime hypothesis"
Iteration 3: "Low-vol pocket shows Sharpe 1.2, needs holdout validation"
Iteration 4: "Holdout confirms Sharpe 0.9, upgrade to paper_alpha"
Result: Conditional edge discovered and validated
```

## Definition Unpacking Requirement

Any agent using dismissal terms must define them:

```
DEFINITION CHECK
├─ Term: "broken"
├─ Defined As: Sharpe < 0.2 AND negative expectation
├─ Working Would Look Like: Sharpe > 0.7, positive expectation, <20% drawdown
├─ Evidence: [specific data points]
└─ Could Also Explain: Sample too small, wrong segment, regime mismatch
```

Undefined dismissals are flagged as incomplete analysis.

## Implementation Checklist

- [ ] Command file includes CONDITIONAL EDGE DISCOVERY PROTOCOL section
- [ ] State schema includes segmentation tracking fields
- [ ] MANDATORY CHECKS include Definition Unpacking and Conditional Edge checks
- [ ] All 6 agent templates updated with required sections
- [ ] Classification system includes `conditional_alpha_candidate`
- [ ] Uniqueness verification has minimum search budgets

## Summary

The Conditional Edge Discovery Protocol transforms the quant research workflow from:

**"Does edge exist?"** (binary, premature)

To:

**"Where might edge be hiding, and how do we find it without fooling ourselves?"** (systematic, rigorous)

This preserves skepticism while preventing lazy dismissal of potentially profitable conditional strategies.
