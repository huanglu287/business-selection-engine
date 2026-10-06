# CODEX TASK — Business Selection Engine V0

## Mission

Act as lead engineer. Implement working code in this repository, not just a plan. Inspect existing files, proceed incrementally, run tests, and report results. Make reasonable low-complexity decisions without repeatedly asking for permission.

**Product principle:** Selection > effort. Screen many opportunities cheaply, eliminate weak ones, research only decision-critical unknowns, and ultimately specialize in a single winning business. V0 builds only the **selection engine**, not the autonomous business operation.

## V0 outcomes / definition of done

A user must be able to:
1. Start locally via Docker Compose (API + PostgreSQL).
2. Seed 15–30 clearly labeled **fictional/demo** opportunity records, varied in quality.
3. List, inspect, create and edit candidates using API.
4. Score all candidates deterministically using editable weights.
5. See score decomposition, penalty contributions, model version and timestamp.
6. See confidence **separately** from opportunity score, with assumptions distinct from sourced claims.
7. View a ranked shortlist (top 3–5) and rejected candidates.
8. View a prioritized queue of **specific** decision-critical research questions.
9. Update evidence and rerun to observe appropriate score/confidence/ranking changes.
10. Run automated tests successfully and follow instructions in README.

No frontend and no real-money integrations required.

## In-scope pipeline

Input/seed → Opportunity DB → Cheap eligibility screen → Configurable scoring → Ranking → Research prioritization → Top candidates marked VALIDATION_READY (when evidence sufficient).

Future, **out of scope**: web crawler/automatic scouting, deep-research agents, external paid API calls, outreach, advertising, payment/account setup, self-directed software business, automatically switching a live business.

## Eligibility / hard gates

Prefer: online and digitally deliverable, AI operational share >=70%, low capital, no physical inventory, little founder daily involvement, plausible revenue test within weeks, B2B recurring workflows, urgent measurable buyer pain, fast learning cycle, margin, accessible distribution, accumulating proprietary experience.

Penalize: regulatory complexity, substantial manual work, founder dependency, SEO-only long feedback, platform dependency, commodity content, copyable AI wrappers, expensive inference, spam-dependent acquisition and disproportionate startup cost.

Hard failures: illegal business model, prohibited platform abuse, no identifiable buyer, structurally infeasible delivery. Track rejection reason. Low confidence alone is **not** proof of a bad business.

## Data model

`Opportunity` minimum:
id, name, description, buyer, buyer_type, pain, current_solution,
estimated_willingness_to_pay, purchase_frequency, recurring_revenue_potential,
ai_automation_ratio, ai_asymmetry, ai_asymmetry_reason, human_dependency,
distribution_accessibility, competition_intensity, platform_dependency,
feedback_speed, gross_margin_potential, startup_cost, estimated_token_cost,
regulatory_risk, reputation_risk, moat_potential, proprietary_data_potential,
learning_density, confidence, status, created_at, updated_at, last_researched_at.

Use normalized 0–100 scores where appropriate; `ai_automation_ratio` should be a documented 0–1 fraction or percent consistently (prefer 0–1). Validate ranges and clarify units for all monetary and token estimates.

`Evidence`:
id, opportunity_id, claim, evidence_type, source, source_url,
observed_value, confidence, collected_at, notes.
Evidence types: market, customer, pricing, competitor, regulatory, distribution,
technical, experiment, assumption. Never pass invented values off as observed data.

`InformationGap`:
id, opportunity_id, question, category, importance, uncertainty,
estimated_research_cost, expected_decision_value, priority_score, status.
Gaps must have a *specific answerable question*, not “research this niche.”

`ScoringRun` or an equivalent audit record:
opportunity_id, version, weights/config snapshot, input snapshot,
factor contributions, penalties, base_score, penalty_score,
final_score, confidence, timestamp.

## Scoring

Keep score calculation deterministic and fully configurable (e.g. `config/scoring.yaml`).

Positive normalized weights, totaling 1:
- buyer pain × willingness-to-pay: 0.25
- AI asymmetry: 0.20
- distribution accessibility: 0.15
- recurring revenue potential: 0.15
- feedback speed / learning density: 0.10
- moat / proprietary data: 0.10
- gross margin: 0.05

Subtract clearly documented, sensible initial configurable penalties for:
competition, regulation, startup cost, human dependency, inference/token cost,
platform dependency, reputation risk.

Keep `base_score`, `penalty_score`, `final_score` and `confidence` separate. Clamp final score 0–100. Return per-factor calculations to explain comparisons. Do not fabricate objective market data; scoring seeded values are illustrative.

Rubric for `ai_asymmetry`: savings in labor/time, reliability, task AI-fit, and how commoditized the advantage is. Rubric for `learning_density`: frequency, speed and quality of meaningful buyer/user feedback.

**Never let a confident-sounding model opinion count as verified evidence.**

## Confidence and evidence

Explicitly distinguish:
- verified observations with source and recency
- estimates
- assumptions
- unknowns

Define a deterministic, bounded confidence aggregation rule from evidence coverage/quality, not a free-floating subjective number. Low confidence/high score must trigger research priority; it must NOT be auto-promoted into “proven winner”. Updating relevant evidence should affect confidence and/or assessed factors predictably.

## Budgeted research

Tokens are capital. V0 uses abstract `research_cost_units` (no LLM billing integration required).

Default stage allowances (configurable):
- cheap screen: ~500 units per idea
- preliminary: ~3,000 per surviving idea
- deep research: ~20,000 per finalist

Budget allocator should primarily research the top candidates and stop wasting units on weak ones. Rank each gap by something like:

`candidate_value * uncertainty * decision_impact / max(research_cost, epsilon)`

Normalize factors, guard against division by zero, and prioritize **expected decision value**, not curiosity. Respect a global run budget, allocate no more than the cap, and generate a queue with reasons. Make reruns idempotent when inputs unchanged.

## Selection

Allowed states:
DISCOVERED → SCREENING → SHORTLISTED → DEEP_RESEARCH → VALIDATION_READY;
plus REJECTED and ARCHIVED.

Configurable gates, clear rejection reasons, and enough confidence required before “validation ready”. Select ~3–5 finalists where candidates are available. High score/low confidence should move to research, not ready.

Document future (do not implement): after proving one business, devote ~90% resources to that specialist and ~10% to scouting, consider switching only when replacement's expected value convincingly exceeds incumbent (illustrative 2× hurdle).

## API

Implement:
- POST /opportunities
- GET /opportunities
- GET /opportunities/{id}
- PATCH /opportunities/{id}
- POST /opportunities/{id}/evidence
- POST /opportunities/{id}/gaps
- POST /opportunities/{id}/score
- GET /rankings
- GET /research-queue
- POST /selection/run

`/rankings` exposes rank/name/score/confidence/status/critical_gaps.
`/selection/run` returns scores, winners, rejection reasons, gap queue and readable rationale.

## CLI

Support:
`python -m app.cli seed`
`python -m app.cli list`
`python -m app.cli score`
`python -m app.cli rank`
`python -m app.cli research-queue`
`python -m app.cli select`

## Tech and repo shape

Prefer Python 3.12+, FastAPI, Pydantic, SQLAlchemy, PostgreSQL, Alembic, pytest, Docker Compose.

Aim for:
```text
app/
  main.py
  cli.py
  api/{opportunities,rankings,research}.py
  core/{config,scoring,selection,research_allocator}.py
  models/{opportunity,evidence,information_gap}.py
  schemas/
  services/
  db/
config/scoring.yaml
tests/
scripts/seed_demo.py
alembic/
docker-compose.yml
Dockerfile
pyproject.toml
.env.example
README.md
```

Avoid Redis, Celery, Kafka, Temporal, a vector DB, microservices, Kubernetes and a frontend in V0.

## Seed examples

Create 15–30 diverse **demo-only** candidate businesses:
AI tender monitoring, agency client reporting, lead qualification,
compliance document review, ecommerce price monitoring, vertical AI receptionist,
generic SEO content farm, generic copywriting agency, affiliate content site,
niche intelligence subscription, back-office document automation, etc.

Strong, defensible vertical workflows should generally rank above commodity content **because of the model criteria**, not forced output. Test with edge cases too.

## Mandatory tests

- greater pain/WTP increases score
- regulation increases penalty
- high score + low confidence gets research, not validation-ready
- weak candidates rejected with reason
- strong high-confidence candidate shortlisted
- cheaper equally valuable research gaps rank first
- evidence quality/coverage updates confidence
- unchanged config/inputs produce deterministic scores
- ranking deterministic with stable tie-breaks
- config edits affect ranking when relevant
- repeated selection run does not duplicate evidence or spend
- API smoke tests and migrations

## Security and commercial guardrails

V0 must NEVER spend money, create ads, send cold emails, scrape forbidden content,
create external accounts, open payment/bank accounts, sign contracts, purchase domains,
deploy customer-facing businesses, manipulate reviews or evade platform rules.
It can only evaluate opportunities. No exposed credentials in source files.

## Delivery plan

1. Inspect repo and initialize backend/dependencies/config.
2. Add data models/migration.
3. Implement deterministic scoring.
4. Implement evidence/confidence.
5. Implement gating/selection and gap allocator.
6. REST API + CLI.
7. Demo seed, tests, Docker config.
8. Run tests and fix failures.
9. Update README with architecture, assumptions, exact launch/seed/test commands,
   limitations and next milestone.

Finish with repo tree, commands, test results, scoring rationale, and unresolved issues.

**Critical:** Selection quality > agent complexity; real buyer signal > more AI reasoning. Do not claim market validation or real revenue in V0.
