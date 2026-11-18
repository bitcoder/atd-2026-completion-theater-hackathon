# CLAUDE.md

Participant/facilitator kit for the **Completion Theater Hackathon** (Agile Testing Days 2026, 18 Nov 2026, Potsdam, with Dragan Spiridonov). Teams build a small seat-reservation app with AI tools, prove what works, then audit another team's frozen submission. The five patterns: Ghost Database, Metric Theater, Test Theater, False Positives, Uncommitted Illusion. Rules/rubric version **1.1**. MIT license. Rewards are undecided: never add prizes, sponsorship claims or product entitlements.

The repo is mostly documentation plus small, dependency-free Python helper tools. It is **not** a finished app.

## Layout

- `docs/` is the canonical event material: `01` PRD, `02` verification checklist, `03` cross-audit checklist, `04` scoring sheets, `05` facilitator notes, `06` pattern card, `07` Codespaces and judging, `08` submission workflow, `api-contract.md`.
- `materials/` holds generated printable PDFs and the deck. `scripts/render_materials.py` renders them from `docs/` (deps in `requirements-render.txt`). Regenerate rather than hand-edit the PDFs.
- `starter-python/` is the optional, **deliberately incomplete** scaffold (transport, static page, schema, reset). All reservation behavior is intentionally unimplemented and `test_app.py` starts RED on purpose. Do not "fix" this in the organizer repo.
- `tools/` holds the helpers: `preflight.py`, `submission.py`, `snapshot.py`, `verify.py` (black-box API challenges via an adapter JSON), `score.py` (score validator). Documented in `tools/README.md`.
- `tests/` has unit tests for the helper tools only.
- `team-templates/` has editable team handoff files.
- `.agents/skills/score-hackathon/` is the canonical judging skill, with `scripts/judge_packet.py`. `.claude/skills/score-hackathon/` only routes to it. Keep one scoring policy, in the canonical file.
- `.devcontainer/` is the Codespaces setup (Node 24, Python 3.12, gh, Claude Code, Codex, Ruflo, Agentic QE).

## Commands

Python **3.11+, standard library only**. There is no package install for the tools. On Windows use `py -3`. Check which interpreter you're running and use the same one throughout.

```sh
python3 tools/preflight.py                 # runtime/SQLite/loopback check (not app correctness)
python3 -m unittest discover -s tests -v   # helper tool tests (this is what CI runs)
python3 -m unittest discover -s tests -v -k <name>   # a single test

# starter app (run from starter-python/)
python3 app.py --reset                      # reset fixture state, then
python3 app.py                              # serve on 127.0.0.1:8765 (omit --reset to keep state)
python3 -m unittest -v test_app             # RED by design in the starter

# helpers: each has --help
python3 tools/verify.py --source DIR --adapter tools/adapter.example.json --evidence NEWDIR
python3 tools/score.py path/to/score.json
```

Helper tests open a loopback listener; a sandbox that denies that will fail, which is an environment problem, so report it instead of working around it. CI (`.github/workflows/checks.yml`) runs preflight and the unit tests on ubuntu/windows/macos with Python 3.11. Env vars for apps: `PORT` (default 8765) and `DATA_DIR` (default `.state/`).

## Rules for working here

- **Don't claim completion without evidence.** This is a repo about completion theater. A green helper test means the helpers work, not that any app works. `verify.py` API PASS is not an acceptance criterion or a score. Report failures and NOT RUN honestly.
- **Tool changes need meaningful positive and failure-path tests.** Keep tools stdlib-only and cross-platform (Windows paths, no reliance on executable bits).
- **Don't change rules mid-stream.** Rubric (40/30/20/10), the 20 mandatory score rows, and rule changes must be announced before scoring. Update `CHANGELOG.md` for user-visible changes, and keep `docs/`, the README, the facilitator material and the skill consistent with one another. Regenerate PDFs after editing docs.
- **Security posture of the helpers:** `verify.py` runs commands with `shell=False`, argv[0] must be an installed interpreter (not a source file), probes `127.0.0.1` only, passes a restricted env allowlist, and is **not an OS sandbox**. Never weaken these. Snapshot extraction never merges into an existing dir, refuses symlinks, and its exclusions are not a secret scanner.
- **Never run or execute submitted participant code** outside an isolated environment, and never inside the trusted organizer checkout. Only fixture data and fictional aliases; no real registration, payment, personal data, tokens or `.env` files. `.gitignore` excludes `evidence/`, `submissions/`, `.state/`, databases, logs and `.env*` on purpose; do not force-add them.
- **Scoring:** points are human decisions. Don't infer creativity or cross-audit points from API results, and don't fill in adjudicator identities. Use the `score-hackathon` skill (`/score-hackathon`) and score the frozen full 40-char commit plus its snapshot, never a branch tip or GitHub merge ref.
- **Submission model:** teams fork, build on `submission/<team-id>`, freeze at 21:55 CET and PR to organizer `main` by 22:00. Submission PRs are review records and are never merged. Don't push directly to organizer branches or amend/force-push a frozen branch.
- Don't commit, push, open PRs or message people unless asked.

## Conventions

- Docs are Markdown; keep them concise and consistent in tone (blunt, evidence-first). Time references are CET.
- Python style follows the existing files: standard library, argparse CLIs with `--help`, type hints where already used, `unittest` (not pytest).


## Agentic QE v3

This project uses **Agentic QE v3** - a Domain-Driven Quality Engineering platform with 13 bounded contexts, ReasoningBank learning, HNSW vector search, and Agent Teams coordination (ADR-064).

---

### CRITICAL POLICIES

#### Integrity Rule (ABSOLUTE)
- NO shortcuts, fake data, or false claims
- ALWAYS implement properly, verify before claiming success
- ALWAYS use real database queries for integration tests
- ALWAYS run actual tests, not assume they pass

**We value the quality we deliver to our users.**

#### Test Execution
- NEVER run `npm test` without `--run` flag (watch mode risk)
- Use: `npm test -- --run`, `npm run test:unit`, `npm run test:integration` when available

#### Data Protection
- NEVER run `rm -f` on `.agentic-qe/` or `*.db` files without confirmation
- ALWAYS backup before database operations

#### Git Operations
- NEVER auto-commit/push without explicit user request
- ALWAYS wait for user confirmation before git operations

---

### Quick Reference

```bash
# Run tests
npm test -- --run

# Check quality
aqe quality assess

# Generate tests
aqe test generate <file>

# Coverage analysis
aqe coverage <path>
```

### Using AQE MCP Tools

AQE exposes tools via MCP with the `mcp__agentic-qe__` prefix. You MUST call `fleet_init` before any other tool.

#### 1. Initialize the Fleet (required first step)

```typescript
mcp__agentic-qe__fleet_init({
  topology: "hierarchical",
  maxAgents: 15,
  memoryBackend: "hybrid"
})
```

#### 2. Generate Tests

```typescript
mcp__agentic-qe__test_generate_enhanced({
  targetPath: "src/services/auth.ts",
  framework: "vitest",
  strategy: "boundary-value"
})
```

#### 3. Analyze Coverage

```typescript
mcp__agentic-qe__coverage_analyze_sublinear({
  paths: ["src/"],
  threshold: 80
})
```

#### 4. Assess Quality

```typescript
mcp__agentic-qe__quality_assess({
  scope: "full",
  includeMetrics: true
})
```

#### 5. Store and Query Patterns (with learning persistence)

```typescript
// Store a learned pattern
mcp__agentic-qe__memory_store({
  key: "patterns/coverage-gap/{timestamp}",
  namespace: "learning",
  value: {
    pattern: "...",
    confidence: 0.95,
    type: "coverage-gap",
    metadata: { /* domain-specific */ }
  },
  persist: true
})

// Query stored patterns
mcp__agentic-qe__memory_query({
  pattern: "patterns/*",
  namespace: "learning",
  limit: 10
})
```

#### 6. Orchestrate Multi-Agent Tasks

```typescript
mcp__agentic-qe__task_orchestrate({
  task: "Full quality assessment of auth module",
  domains: ["test-generation", "coverage-analysis", "security-compliance"],
  parallel: true
})
```

### MCP Tool Reference

| Tool | Description |
|------|-------------|
| `fleet_init` | Initialize QE fleet (MUST call first) |
| `fleet_status` | Get fleet health and agent status |
| `agent_spawn` | Spawn specialized QE agent |
| `test_generate_enhanced` | AI-powered test generation |
| `test_execute_parallel` | Parallel test execution with retry |
| `task_orchestrate` | Orchestrate multi-agent QE tasks |
| `coverage_analyze_sublinear` | O(log n) coverage analysis |
| `quality_assess` | Quality gate evaluation |
| `memory_store` | Store patterns with namespace + persist |
| `memory_query` | Query patterns by namespace/pattern |
| `security_scan_comprehensive` | SAST/DAST scanning |

### Configuration

- **Enabled Domains**: test-generation, test-execution, coverage-analysis, quality-assessment, defect-intelligence, requirements-validation (+7 more)
- **Learning**: Enabled (auto embeddings)
- **Max Concurrent Agents**: 5
- **Background Workers**: pattern-consolidator, routing-accuracy-monitor

### V3 QE Agents

QE agents are in `.claude/agents/v3/`. Use with Task tool:

```javascript
Task({ prompt: "Generate tests", subagent_type: "qe-test-architect", run_in_background: true })
Task({ prompt: "Find coverage gaps", subagent_type: "qe-coverage-specialist", run_in_background: true })
Task({ prompt: "Security audit", subagent_type: "qe-security-scanner", run_in_background: true })
```

### Data Storage

- **Memory Backend**: `.agentic-qe/memory.db` (SQLite)
- **Configuration**: `.agentic-qe/config.yaml`

---
*Generated by AQE v3 init - 2026-11-18T19:53:49.481Z*
