# /whats-next (Quant Edition)

An autonomous multi-agent system for quantitative trading research that dispatches to **real specialized agents** with hardcoded skepticism.

> **This is the `quant` branch.** For the generic project-agnostic version, see the [`main` branch](https://github.com/YoungMoneyInvestments/whats-next-agent/tree/main).

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

---

## Installation

### Quick install

```bash
mkdir -p ~/.claude/commands

curl -sL "https://raw.githubusercontent.com/YoungMoneyInvestments/whats-next-agent/quant/commands/whats-next.md" \
  > ~/.claude/commands/whats-next-quant.md
```

### Or clone and symlink

```bash
git clone -b quant https://github.com/YoungMoneyInvestments/whats-next-agent.git
ln -s $(pwd)/whats-next-agent/commands/whats-next.md ~/.claude/commands/whats-next-quant.md
```

---

## Requirements

**Critical:** The quant agents must be available in your Claude Code environment. These are defined as Task tool subagent_types:

```
~/.claude/agents/
├── quant-research-generator.md
├── quant-skeptic-redteam.md
├── quant-ml-validation-engineer.md
├── quant-execution-microstructure.md
├── quant-capital-allocation-risk.md
└── quant-manager-audit.md
```

If these agents aren't configured, the command will fall back to internal role-play (less effective for quant work).

---

## Usage

### JSON format (recommended)

```bash
/whats-next-quant '{"description": "ES momentum strategy", "success_criteria": "Sharpe > 1.5, DD < 15%", "constraints": "1-tick slippage", "max_iterations": 25}'
```

### Natural language format

```bash
/whats-next-quant Build a mean reversion strategy for SPY with walk-forward validation
```

When using natural language, the agent infers success criteria (defaults to quant-appropriate constraints).

### Examples

**Momentum Strategy:**
```bash
/whats-next-quant '{"description": "ES futures momentum strategy using 20-day lookback", "success_criteria": "Sharpe > 1.5, max DD < 15%, capacity > $10M", "constraints": "No overnight positions, include 1-tick slippage", "max_iterations": 30}'
```

**ML Signal:**
```bash
/whats-next-quant '{"description": "Random forest classifier for SPY direction", "success_criteria": "AUC > 0.55 OOS, no lookahead, stable across regimes", "constraints": "Walk-forward validation only", "max_iterations": 25}'
```

**Risk Model:**
```bash
/whats-next-quant '{"description": "VaR model for equity portfolio", "success_criteria": "99% VaR accurate within 10% of realized, backtested 10 years", "constraints": "Must handle fat tails and correlation breakdown", "max_iterations": 20}'
```

---

## How It Works

### Pipeline Architecture (v2)

Each iteration runs this pipeline with **parallel critique phase**:

```
┌─────────────────────────────────────────────────────────────┐
│  1. STATE READ                                               │
│     └─ Load state.json, summarize current position          │
├─────────────────────────────────────────────────────────────┤
│  2. PROPOSE (Task: quant-research-generator)                │
│     └─ Pass: state summary, previous results (inline)       │
│     └─ Capture output → write to role_outputs/proposer.md   │
├─────────────────────────────────────────────────────────────┤
│  3. PARALLEL CRITIQUE PHASE                                  │
│     Run 4 agents IN PARALLEL (no dependencies):             │
│                                                              │
│     ├─ Task: quant-skeptic-redteam                          │
│     ├─ Task: quant-ml-validation-engineer                   │
│     ├─ Task: quant-execution-microstructure                 │
│     └─ Task: quant-capital-allocation-risk                  │
│                                                              │
│     Each receives: state + proposer output (inline)         │
│     Each returns: APPROVE / BLOCK / CONDITIONAL verdict     │
├─────────────────────────────────────────────────────────────┤
│  4. EXECUTE CHOSEN STEP                                      │
│     └─ Select step with fewest blocking objections          │
│     └─ Run via sandbox, produce artifacts                   │
│     └─ Run objective gates                                  │
├─────────────────────────────────────────────────────────────┤
│  5. AUDIT (Task: quant-manager-audit)                        │
│     └─ Pass: ALL agent outputs inline + gate results        │
│     └─ Receive: classification + consensus status           │
├─────────────────────────────────────────────────────────────┤
│  6. LOG AND LOOP                                             │
│     └─ Update state.json with votes, classification         │
│     └─ Write experiment JSON, append to journal             │
│     └─ If not done, start next iteration                    │
└─────────────────────────────────────────────────────────────┘
```

### Inline Context Passing (v2 Critical Fix)

Each agent receives all relevant context **inline in its prompt**, not by file reference:

```python
# Pseudocode for agent dispatch
Task(
  subagent_type: "quant-skeptic-redteam",
  prompt: """
You are reviewing a quant research proposal.

## CURRENT STATE
- Iteration: {iteration}
- Classification: {classification}
- Completed: {completed_list}

## PROPOSAL TO ATTACK
{full proposer.md content here}

## YOUR TASK
Attack all assumptions. Return verdict: APPROVE / BLOCK / CONDITIONAL
"""
)
```

This prevents context loss between agent dispatches.

### Parallel Dispatch (v2 Performance)

The critique phase runs 4 agents simultaneously:

```python
# Single message with multiple Task calls
Task(subagent_type: "quant-skeptic-redteam", ..., run_in_background: true)
Task(subagent_type: "quant-ml-validation-engineer", ..., run_in_background: true)
Task(subagent_type: "quant-execution-microstructure", ..., run_in_background: true)
Task(subagent_type: "quant-capital-allocation-risk", ..., run_in_background: true)

# Then collect with TaskOutput
```

---

## Classification System

The `quant-manager-audit` agent assigns one classification per iteration:

| Classification | Meaning | Consensus Required | Next Action |
|----------------|---------|-------------------|-------------|
| `research_only` | Interesting but unproven | No | Continue iterating |
| `paper_alpha` | Backtested, not live-ready | Partial | Harden execution/risk |
| `capital_deployable` | Production ready | **Yes - ALL agents** | May output `<DONE>` |
| `rejected` | Does not survive scrutiny | No | Pivot or stop |

### Consensus Rules

```json
{
  "consensus": true,
  "conditions": [
    "No agents have verdict = BLOCK",
    "quant-manager-audit classification = capital_deployable",
    "All mandatory checks passed"
  ]
}
```

**The loop only completes when classification is `capital_deployable` AND `consensus = true`.**

---

## Mandatory Checks

Every iteration enforces these checks via specialized agents:

### Anti-Goodhart (quant-ml-validation-engineer)
- How could the optimized metric be misleading?
- What perverse behavior does optimizing this incentivize?

### Null Hypothesis (quant-skeptic-redteam)
- What would falsify this hypothesis?
- Is there a simpler (non-alpha) explanation?

### Negative Expectation (quant-skeptic-redteam)
- When does this strategy lose money?
- What regime change would break it?

### Execution Reality (quant-execution-microstructure)
- How much edge is consumed by costs?
- Performance with realistic fills?
- Capacity constraints?

### Capital Scaling (quant-capital-allocation-risk)
- How does performance degrade with size?
- Correlation with existing strategies?
- Tail risk under stress?

---

## Persistent State

### Directory Structure

```
.whatsnext/
├── config.json           # Parsed input configuration
├── state.json            # Current loop state (schema below)
├── journal.md            # Human-readable decision timeline
├── experiments/          # One JSON per iteration
│   └── iteration_NNN.json
├── role_outputs/         # Agent outputs per iteration
│   ├── proposer.md
│   ├── critic.md
│   ├── validator.md
│   ├── execution.md
│   ├── risk.md
│   ├── executor.md
│   └── arbiter.md
└── checkpoints/          # Git context snapshots
```

### state.json Schema (v2)

```json
{
  "iteration": 1,
  "status": "running",
  "description": "...",
  "success_criteria": "...",
  "constraints": "...",
  "max_iterations": 25,

  "classification": "research_only",
  "confidence": "low",

  "completed": [],
  "unknowns": [],
  "risks": [],

  "agent_votes": {
    "quant-research-generator": null,
    "quant-skeptic-redteam": null,
    "quant-ml-validation-engineer": null,
    "quant-execution-microstructure": null,
    "quant-capital-allocation-risk": null,
    "quant-manager-audit": null
  },

  "blocking_agents": [],
  "consensus": false,

  "chosen_step": null,
  "last_gate_results": {},
  "low_info_streak": 0
}
```

---

## Output Contract

Each iteration outputs this structure:

```
═══════════════════════════════════════════════════════════════
ITERATION N
═══════════════════════════════════════════════════════════════

STATE ASSESSMENT
├─ Completed: [list]
├─ Unknown: [list]
├─ Key Risks: [list]
├─ Classification: [research_only / paper_alpha / capital_deployable / rejected]
└─ Consensus: [true/false] - Blocking: [agent list]

AGENT DISPATCHES
├─ quant-research-generator: [proposal summary]
├─ quant-skeptic-redteam: [verdict] - [key objections]
├─ quant-ml-validation-engineer: [verdict] - [validation status]
├─ quant-execution-microstructure: [verdict] - [execution concerns]
├─ quant-capital-allocation-risk: [verdict] - [risk assessment]
└─ quant-manager-audit: [classification] - [reasoning]

CHOSEN STEP
├─ Step: [what was executed]
├─ Why: [selection reasoning]
└─ Evidence: [artifacts produced]

GATE RESULTS
├─ Tests: [pass/fail/skipped]
├─ Backtest: [metrics or skipped]
└─ Validation: [pass/fail/skipped]

MANDATORY CHECKS
├─ Anti-Goodhart: [status]
├─ Null Hypothesis: [status]
├─ Negative Expectation: [status]
├─ Execution Reality: [status]
└─ Capital Scaling: [status]

CONVERGENCE
├─ Classification: [current]
├─ Consensus: [true/false]
├─ Info Gain: [high/medium/low]
└─ Decision: [continue/pivot/stop]

LOG WRITEBACK: [confirmed]
═══════════════════════════════════════════════════════════════
```

---

## Stop Conditions

| Condition | Action |
|-----------|--------|
| Capital Deployable + Consensus | `<DONE>` with success summary |
| Max iterations reached | `<DONE>` with classification and blockers |
| Rejected by audit | `<DONE>` explaining why |
| Two consecutive Rejections | Stop, explain boundary |
| Critical ambiguity | Asks ONE question and stops |

---

## Error Handling (v2)

### Agent Fallback Protocol

| Failure | Recovery |
|---------|----------|
| Task tool error | Retry once, then fall back to internal role-play |
| Agent returns empty | Log warning, treat as CONDITIONAL |
| Agent timeout | Log timeout, treat as CONDITIONAL with note |

When falling back:
1. Log to journal: "Agent {name} unavailable, using internal role-play"
2. Reduce confidence by one level
3. Add to risks: "Reduced scrutiny from {agent}"

Never skip a perspective entirely.

### General Errors

| Error | Recovery |
|-------|----------|
| Gate command fails | Log error, mark as "error", continue |
| File write fails | Retry once, then log and continue |
| Sandbox timeout | Log timeout, reduce scope, retry |

---

## Comparison: Generic vs Quant

| Aspect | Generic (main) | Quant (this branch) |
|--------|----------------|---------------------|
| Skepticism | Role-played | Hardcoded in agents |
| Agent separation | Same context | Separate contexts |
| Critique phase | Sequential | **Parallel** |
| Execution check | Optional | Mandatory |
| Risk check | Optional | Mandatory |
| Classification | None | 4-level system |
| Completion gate | Self-assessed | Agent consensus |
| Context passing | Inline | Inline + explicit capture |

---

## Version History

### v2 (Current)
- **JSON input format** - Structured input with fallback to natural language
- **Inline context passing** - Each agent receives all context directly in prompt
- **Parallel critique phase** - 4 agents run simultaneously
- **Structured agent verdicts** - APPROVE / BLOCK / CONDITIONAL
- **Agent votes tracking** - Explicit state.json schema with blocking_agents
- **Fallback protocol** - Recovery when agents fail
- **Output contract** - Consistent iteration output with mandatory checks section

### v1
- Initial release with real agent dispatch
- Sequential agent calls
- File-based context passing (prone to loss)
- Unstructured argument parsing

---

## Integration with Existing Quant Tools

This command works well with:

- **TradingCore** - Data fetching, model registry, risk management
- **DataGuard** - Pre-flight validation for data availability
- **ralph-quant** - Can be used together for different workflows
- **BrokerBridge** - IBKR connectivity for live validation

---

## Requirements

- [Claude Code](https://claude.ai/code) CLI
- Git (for checkpoints)
- Quant agents configured in `~/.claude/agents/`
- Project-specific tooling (pytest, make, etc.)

## License

MIT

## Contributing

PRs welcome. For quant-specific changes, include:
- Which agent behavior is affected
- Whether it changes the skepticism level
- Test case showing the change in action
