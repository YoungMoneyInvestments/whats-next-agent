# /whats-next

An autonomous multi-role self-play agent for Claude Code that advances your project with minimal intervention.

One command. Loops until done or blocked.

> **Looking for quant-specific features?** See the [`quant` branch](https://github.com/YoungMoneyInvestments/whats-next-agent/tree/quant) which dispatches to real specialized agents with hardcoded skepticism.

## What it does

`/whats-next` runs an autonomous improvement cycle that:

1. **Reads repo state** - Understands what exists and what's been done
2. **Self-plays 6 roles** - Proposer, Critic, Alternatives, Arbiter, Executor, Referee
3. **Executes via sandbox** - Runs actual commands, not just suggestions
4. **Validates with gates** - Tests, lints, backtests must pass
5. **Logs everything** - Persistent memory in `.whatsnext/`
6. **Loops automatically** - Until done, blocked, or stalled

## Installation

### Quick install

```bash
# Create commands directory if it doesn't exist
mkdir -p ~/.claude/commands

# Download the command
curl -sL "https://raw.githubusercontent.com/YoungMoneyInvestments/whats-next-agent/main/commands/whats-next.md" \
  > ~/.claude/commands/whats-next.md
```

### Or clone and symlink

```bash
git clone https://github.com/YoungMoneyInvestments/whats-next-agent.git
ln -s $(pwd)/whats-next-agent/commands/whats-next.md ~/.claude/commands/whats-next.md
```

## Usage

### JSON format (recommended)

```bash
/whats-next '{"description": "Build customer churn predictor", "success_criteria": "AUC > 0.85", "constraints": "No PII", "max_iterations": 20}'
```

### Natural language format

```bash
/whats-next Build user authentication with JWT tokens and proper session handling
```

When using natural language, the agent infers success criteria and constraints from context.

### Examples

**ML Pipeline:**
```bash
/whats-next '{"description": "Build customer churn predictor", "success_criteria": "AUC > 0.85, inference < 100ms", "constraints": "No PII in features", "max_iterations": 20}'
```

**Web App Feature:**
```bash
/whats-next '{"description": "Add user authentication", "success_criteria": "All tests pass, secure session handling", "constraints": "Use existing auth library", "max_iterations": 15}'
```

**Refactoring:**
```bash
/whats-next '{"description": "Migrate from REST to GraphQL", "success_criteria": "All endpoints covered, no breaking changes", "constraints": "Maintain backwards compat", "max_iterations": 30}'
```

**Minimal (let it figure things out):**
```bash
/whats-next Improve this codebase
```

---

## How it works

### The 6-Role Self-Play Chain

Each iteration runs these roles **sequentially**, with each role receiving the previous roles' outputs **inline**:

```
┌─────────────────────────────────────────────────────────────┐
│  PROPOSER                                                    │
│  ├─ Input: state.json, previous iteration results           │
│  ├─ Output: Single proposed step with success criteria      │
│  └─ Write to: role_outputs/proposer.md                      │
├─────────────────────────────────────────────────────────────┤
│  CRITIC                                                      │
│  ├─ Input: state + PROPOSER output (inline)                 │
│  ├─ Output: Attack on proposal, failure modes identified    │
│  └─ Write to: role_outputs/critic.md                        │
├─────────────────────────────────────────────────────────────┤
│  ALTERNATIVES                                                │
│  ├─ Input: state + PROPOSER + CRITIC outputs (inline)       │
│  ├─ Output: 3-5 competing steps with success criteria       │
│  └─ Write to: role_outputs/alternatives.md                  │
├─────────────────────────────────────────────────────────────┤
│  ARBITER                                                     │
│  ├─ Input: state + all previous role outputs (inline)       │
│  ├─ Output: Scored ranking, ONE selected step               │
│  └─ Write to: role_outputs/arbiter.md                       │
├─────────────────────────────────────────────────────────────┤
│  EXECUTOR                                                    │
│  ├─ Input: state + ARBITER decision (inline)                │
│  ├─ Output: Execution results, artifacts, evidence          │
│  └─ Write to: role_outputs/executor.md                      │
├─────────────────────────────────────────────────────────────┤
│  REFEREE                                                     │
│  ├─ Input: state + all role outputs + gate results (inline) │
│  ├─ Output: Convergence assessment, continue/pivot/stop     │
│  └─ Write to: role_outputs/referee.md                       │
└─────────────────────────────────────────────────────────────┘
```

### Context Passing (v2 improvement)

Each role receives previous outputs **inline in its prompt**, not by file reference. This prevents context loss between roles.

### Scoring Rubric (0-5 each)

| Criterion | Description |
|-----------|-------------|
| Information gain | How much do we learn? |
| Uncertainty reduction | Does this narrow unknowns? |
| Risk reduction | Does this reduce project risk? |
| Measurability | Can we objectively evaluate? |
| Reproducibility | Can results be replicated? |
| Impact on success criteria | Direct progress toward goal? |
| Convergence contribution | Move toward done? |

**Penalties:**
- Parameter tuning without validation: -2
- Complexity increase without learning: -2
- Steps that cannot be objectively evaluated: -3

### Stop Conditions

| Condition | Action |
|-----------|--------|
| All success criteria met | `<DONE>` with summary |
| Max iterations reached | `<DONE>` with blockers |
| 2 consecutive low-info-gain | `<DONE>` explaining boundary |
| Critical ambiguity | Asks ONE clarifying question |

---

## Persistent State

### Directory Structure

```
.whatsnext/
├── config.json           # Parsed input configuration
├── state.json            # Current loop state
├── journal.md            # Human-readable decision timeline
├── experiments/          # One JSON per iteration
│   └── iteration_NNN.json
├── role_outputs/         # Each role's output per iteration
│   ├── proposer.md
│   ├── critic.md
│   ├── alternatives.md
│   ├── arbiter.md
│   ├── executor.md
│   └── referee.md
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

  "confidence": "low",
  "entropy": "high",

  "completed": [],
  "unknowns": [],
  "risks": [],

  "role_consensus": {
    "proposer": null,
    "critic": null,
    "alternatives": null,
    "arbiter": null,
    "executor": null,
    "referee": null
  },

  "chosen_step": null,
  "last_gate_results": {},
  "stall_count": 0,
  "low_info_streak": 0
}
```

---

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

---

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

---

## Error Handling (v2)

| Error | Recovery |
|-------|----------|
| Gate command fails | Log error, mark as "error", continue |
| File write fails | Retry once, then log and continue |
| Git command fails | Log warning, continue without checkpoint |
| Sandbox timeout | Log timeout, reduce scope, retry |

Never crashes on recoverable errors.

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
├─ Confidence: [low/medium/high]
└─ Entropy: [low/medium/high]

SELF-PLAY RESULTS
├─ Proposer: [proposed step summary]
├─ Critic: [key objections]
├─ Alternatives: [count] alternatives generated
├─ Arbiter: Selected [step] with score [N]
├─ Executor: [execution summary]
└─ Referee: [continue/pivot/stop] - [reasoning]

CHOSEN STEP
├─ Step: [what was selected]
├─ Why: [arbiter reasoning]
└─ Success Criteria: [how we measure]

EXECUTION AND EVIDENCE
├─ Commands run: [list]
├─ Artifacts: [list]
└─ Metrics: [measurements]

GATE RESULTS
├─ Tests: [pass/fail/skipped]
├─ Lints: [pass/fail/skipped]
└─ Custom: [pass/fail/skipped]

CONVERGENCE
├─ Info Gain: [high/medium/low]
├─ Stall Count: [N]
└─ Decision: [continue/pivot/stop]

LOG WRITEBACK: [confirmed]
═══════════════════════════════════════════════════════════════
```

---

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

---

## Version History

### v2 (Current)
- **JSON input format** - Structured input with fallback to natural language
- **Inline context passing** - Each role receives previous outputs directly in prompt
- **Structured state.json** - Explicit schema with role_consensus tracking
- **Error handling** - Recovery procedures for common failures
- **Output contract** - Consistent iteration output format

### v1
- Initial release with 6-role self-play
- File-based context passing (prone to loss)
- Unstructured argument parsing

---

## Branches

| Branch | Purpose |
|--------|---------|
| `main` | Generic project-agnostic version (internal role-play) |
| `quant` | Quant-specialized with real agent dispatch |

For quant/trading work, the `quant` branch dispatches to 6 specialized agents with hardcoded skepticism that cannot be negotiated away.

---

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
