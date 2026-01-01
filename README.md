# /whats-next (Quant Edition)

An autonomous multi-agent system for quantitative trading research that dispatches to **real specialized agents** with hardcoded skepticism.

> **This is the `quant` branch.** For the generic project-agnostic version, see the `main` branch.

## Why Quant Edition?

The generic `/whats-next` uses internal role-play (one Claude instance pretending to be Proposer, Critic, etc.). This works for general software, but **quant research needs structural skepticism that can't be negotiated away**.

This version dispatches to 6 specialized agents via Claude Code's Task tool:

| Agent | Role | Skepticism Level |
|-------|------|------------------|
| `quant-research-generator` | Propose hypotheses | Creative, exploratory |
| `quant-skeptic-redteam` | Attack all assumptions | **Maximum** - cannot be softened |
| `quant-ml-validation-engineer` | Metrics, overfitting, leakage | High - treats improvements as provisional |
| `quant-execution-microstructure` | Slippage, fills, market impact | High - assumes imperfect execution |
| `quant-capital-allocation-risk` | Position sizing, tail risk | High - assumes correlations break |
| `quant-manager-audit` | Final classification | Requires agent consensus |

## Installation

```bash
# Clone the quant branch
git clone -b quant https://github.com/YoungMoneyInvestments/whats-next-agent.git

# Install the command
cp whats-next-agent/commands/whats-next.md ~/.claude/commands/
```

## Requirements

**Critical:** The quant agents must be available in your Claude Code environment. These are defined as Task tool subagent_types:

- `quant-research-generator`
- `quant-skeptic-redteam`
- `quant-ml-validation-engineer`
- `quant-execution-microstructure`
- `quant-capital-allocation-risk`
- `quant-manager-audit`

If these agents aren't configured, the command will fall back to internal role-play (less effective for quant work).

## Usage

```bash
/whats-next "<strategy description>" "<success criteria>" "<constraints>" "<max iterations>"
```

### Examples

**Momentum Strategy:**
```bash
/whats-next "ES futures momentum strategy using 20-day lookback" "Sharpe > 1.5, max DD < 15%, capacity > $10M" "No overnight positions, include 1-tick slippage" "30"
```

**ML Signal:**
```bash
/whats-next "Random forest classifier for SPY direction" "AUC > 0.55 OOS, no lookahead, stable across regimes" "Walk-forward validation only" "25"
```

**Risk Model:**
```bash
/whats-next "VaR model for equity portfolio" "99% VaR accurate within 10% of realized, backtested 10 years" "Must handle fat tails and correlation breakdown" "20"
```

## How It Works

Each iteration runs this pipeline:

```
┌─────────────────────────────────────────────────────────────┐
│  1. STATE READ         Load .whatsnext/state.json           │
│  2. PROPOSE            Task(quant-research-generator)       │
│  3. ATTACK             Task(quant-skeptic-redteam)          │
│  4. VALIDATE           Task(quant-ml-validation-engineer)   │
│  5. EXECUTION CHECK    Task(quant-execution-microstructure) │
│  6. RISK CHECK         Task(quant-capital-allocation-risk)  │
│  7. EXECUTE            Run chosen step via sandbox          │
│  8. AUDIT              Task(quant-manager-audit)            │
│  9. LOG & LOOP         Update state, continue if needed     │
└─────────────────────────────────────────────────────────────┘
```

## Classification System

The `quant-manager-audit` agent assigns one of four classifications:

| Classification | Meaning | Action |
|----------------|---------|--------|
| **Research Only** | Interesting but unproven | Keep iterating |
| **Paper Alpha** | Backtested, not live-ready | Harden execution/risk |
| **Capital Deployable** | Production ready | May complete |
| **Rejected** | Doesn't survive scrutiny | Pivot or stop |

**The loop only completes when classification is `Capital Deployable` AND all agents agree.**

## Mandatory Checks

Every iteration enforces these checks via the specialized agents:

### Anti-Goodhart (quant-ml-validation-engineer)
- How could the optimized metric be misleading?
- What behavior does optimizing this metric incentivize that we don't want?

### Null Hypothesis (quant-skeptic-redteam)
- What would falsify this hypothesis?
- Is there a simpler (non-alpha) explanation?

### Negative Expectation (quant-skeptic-redteam)
- When does this strategy lose money?
- What regime change would break it?

### Execution Reality (quant-execution-microstructure)
- How much edge is consumed by costs?
- Performance with realistic (not ideal) fills?

### Capital Scaling (quant-capital-allocation-risk)
- How does performance degrade with size?
- What's the capacity limit?

## Persistent State

```
.whatsnext/
├── state.json              # Classification, agent consensus, iteration
├── journal.md              # Human-readable decision timeline
├── experiments/            # One JSON per iteration with full agent outputs
├── role_outputs/           # Each agent's output (written before next agent)
│   ├── proposer.md
│   ├── critic.md
│   ├── validator.md
│   ├── execution.md
│   ├── risk.md
│   ├── executor.md
│   └── arbiter.md
└── checkpoints/            # Git snapshots
```

## Stop Conditions

The loop exits when:

1. **Capital Deployable + Consensus** → Success
2. **Max iterations** → Summary with blockers
3. **Rejected by audit** → Explains why
4. **Two consecutive Rejections** → Boundary reached
5. **Critical ambiguity** → Asks ONE question

## Comparison: Generic vs Quant

| Aspect | Generic (main) | Quant (this branch) |
|--------|----------------|---------------------|
| Skepticism | Role-played | Hardcoded in agents |
| Agent separation | Same context | Separate contexts |
| Execution check | Optional | Mandatory |
| Risk check | Optional | Mandatory |
| Classification | None | 4-level system |
| Completion gate | Self-assessed | Agent consensus |

## Integration with Existing Quant Tools

This command works well with:

- **TradingCore** - Data fetching, model registry, risk management
- **DataGuard** - Pre-flight validation for data availability
- **ralph-quant** - Can be used together for different workflows

## License

MIT

## Contributing

PRs welcome. For quant-specific changes, include:
- Which agent behavior is affected
- Whether it changes the skepticism level
- Test case showing the change in action
