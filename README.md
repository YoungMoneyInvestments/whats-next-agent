# /whats-next

An autonomous multi-role self-play agent for Claude Code that advances your project with minimal intervention.

One command. Loops until done or blocked.

## What it does

`/whats-next` runs an autonomous improvement cycle that:

1. **Reads repo state** - Understands what exists and what's been done
2. **Self-plays 6 roles** - Proposer, Critic, Alternatives, Arbiter, Executor, Referee
3. **Executes via sandbox** - Runs actual commands, not just suggestions
4. **Validates with gates** - Tests, lints, backtests must pass
5. **Logs everything** - Persistent memory in `.whatsnext/`
6. **Loops automatically** - Until done, blocked, or stalled

## Installation

### Quick install (copy command file)

```bash
# Create commands directory if it doesn't exist
mkdir -p ~/.claude/commands

# Copy the command
cp commands/whats-next.md ~/.claude/commands/
```

### Or clone and symlink

```bash
git clone https://github.com/YOUR_USERNAME/whats-next-agent.git
ln -s $(pwd)/whats-next-agent/commands/whats-next.md ~/.claude/commands/whats-next.md
```

## Usage

In any project directory with Claude Code:

```bash
/whats-next "<project description>" "<success criteria>" "<constraints>" "<max iterations>"
```

### Examples

**ML Pipeline:**
```bash
/whats-next "Build customer churn predictor" "AUC > 0.85, inference < 100ms" "No PII in features" "20"
```

**Trading Strategy:**
```bash
/whats-next "Momentum strategy for ES futures" "Sharpe > 1.5, max DD < 15%" "No lookahead bias, include slippage" "25"
```

**Web App Feature:**
```bash
/whats-next "Add user authentication" "All tests pass, secure session handling" "Use existing auth library" "15"
```

**Refactoring:**
```bash
/whats-next "Migrate from REST to GraphQL" "All endpoints covered, no breaking changes" "Maintain backwards compat for 2 weeks" "30"
```

### Minimal usage

If you just want it to figure things out:

```bash
/whats-next "Improve this codebase" "Tests pass, no regressions" "" "10"
```

## How it works

### The 6-Role Self-Play Loop

Each iteration runs these roles sequentially:

| Role | Purpose |
|------|---------|
| **Proposer** | Suggests ONE next step that reduces uncertainty |
| **Critic** | Attacks the proposal, finds failure modes |
| **Alternatives** | Generates 3-5 competing approaches |
| **Arbiter** | Scores all options, picks the winner |
| **Executor** | Runs the chosen step, produces evidence |
| **Referee** | Decides: continue, pivot, or stop |

### Scoring Rubric (0-5 each)

Steps are scored on:
- Information gain
- Uncertainty reduction
- Risk reduction
- Measurability
- Reproducibility
- Leakage safety (if ML/quant)
- Execution realism (if trading)
- Impact on success criteria
- Convergence contribution

### Stop Conditions

The loop exits when:
- All success criteria met → `<DONE>`
- Max iterations reached → `<DONE>` + summary
- 2 consecutive low-info-gain iterations → `<DONE>` + boundary explanation
- Critical ambiguity → Asks ONE clarifying question

## Persistent State

Everything is logged to `.whatsnext/` in your project:

```
.whatsnext/
├── state.json           # Current progress, confidence, risks
├── journal.md           # Human-readable decision timeline
├── experiments/         # One JSON per iteration
│   ├── iteration_001.json
│   ├── iteration_002.json
│   └── ...
├── role_outputs/        # Each role's reasoning (prevents silent rewrites)
│   ├── proposer.md
│   ├── critic.md
│   ├── alternatives.md
│   ├── arbiter.md
│   ├── executor.md
│   └── referee.md
└── checkpoints/         # Git context snapshots
```

## Priority Order

The agent addresses issues in this order:

### General Projects
1. Correctness and reproducibility
2. Tests and evaluation harness
3. Data integrity
4. Baselines and comparisons
5. Performance optimization
6. Deployment safety

### Quant/Trading Projects (auto-detected)
1. Data correctness (timestamps, survivorship)
2. Leakage-safe validation
3. Baselines and null models
4. Execution realism (slippage, fees, latency)
5. Risk limits and drawdowns
6. Regime robustness
7. Monitoring and kill switches

## Objective Gates

The agent runs available validation automatically:

```bash
# If Makefile exists:
make test
make whatsnext-checks
make backtest          # if relevant
make ml-eval           # if relevant

# Otherwise:
pytest
npm test
go test
# etc.
```

**No PASS without running gates.**

## Tips

### Let it run
The default behavior is to loop without asking questions. Trust the process.

### Check the journal
`.whatsnext/journal.md` is the best place to understand what happened and why.

### Resume after interruption
State is persistent. Just run `/whats-next` again with the same parameters.

### Increase iterations for complex projects
Default is 25. Large refactors or research projects may need 50+.

### Add project-specific gates
Create a `Makefile` with `whatsnext-checks` target for custom validation.

## Requirements

- [Claude Code](https://claude.ai/code) CLI
- Git (for checkpoints)
- Project-specific tooling (pytest, make, etc.)

## License

MIT

## Contributing

PRs welcome. Please include:
- Clear description of the change
- Why it improves the autonomous loop
- Any new failure modes it might introduce
