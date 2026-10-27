# AI-Assisted Proposal Triage Design Write-Up

**Assessment Reference:** MATACS-TH-2026-02
**Role:** Associate Software Developer Assessment

---

### 1. AI vs. Rules

**Where AI is used:**
- **Entity Extraction:** LLM extracts structured data from proposal text into typed JSON fields (licences, insurance, experience, price, timeline, warranties, team roles).
- **Textual Grounding:** LLM extracts verbatim sentences (`source_quote`) anchoring extracted facts to source text.
- **Threat Isolation:** LLM detects embedded manipulation directives (e.g., Proposal C's Appendix C claiming pre-approval), isolating them as untrusted text and logging security warnings without executing them.

**Where plain code and rules are used:**
- **Mandatory Gates:** Boolean logic checks eligibility: confirming bidder holds Ontario electrical licence (subcontracted deferrals fail), WSIB clearance, and $5M liability insurance.
- **Status Assignment:** Code assigns `ranked`, `ineligible`, or `flagged`.
- **Deterministic Scoring:** Mathematical formulas compute points from admin-defined weights (Experience 30%, Price 25%, Timeline 15%, Warranty 15%, Team 15%).
- **Rank Ordering:** Sorts eligible proposals descending by score.

**Why draw the line there:**
LLMs excel at natural language parsing but are unreliable at arithmetic, non-deterministic, and vulnerable to prompt injection. Scoping AI to extraction and plain code to scoring ensures repeatable arithmetic, explainability, and dynamic admin updates without re-prompting.

---

### 2. Consistency, Scoring Formulas, and Explainability

**Repeatability:**
Scoring repeatability is achieved by running the scoring engine in plain code. LLM extraction runs at `temperature: 0.0` with strict JSON Schema validation. Given identical extracted values, plain code produces identical scores and ranks on every run. Quote validation helps detect unsupported citations, while human review catches subtle interpretation errors.

**Proposed Scoring Method (Design Assumptions):**
*Note:* Weights and pass/fail rules are set by MATACS; point formulas, baselines, and rubric subdivisions below are proposed engineering design assumptions:

1. **Relevant Experience (30% weight):**
   - *Requirement:* Commercial installs $\ge 100\text{ kW}$ in last 5 years.
   - *Design Assumption:* Target of 10 qualifying projects yields full marks (3.0 pts/install, max 30.0): $S_{\text{exp}} = \min(30.0, N \times 3.0)$.
   - *Scores:* A (14 installs) = 30.00; C (9) = 27.00; E (8) = 24.00.
2. **Price (25% weight):**
   - *Requirement:* Lower total cost within $260k budget ceiling.
   - *Design Assumption:* Inverse-price ratio relative to lowest eligible bid ($P_{\text{min}} = \$238,000$ CAD): $S_{\text{price}} = 25 \times (P_{\text{min}} / P_{\text{bid}})$.
   - *Scores:* A ($238k) = 25.00; E ($242.5k) = 24.54; C ($244k) = 24.39.
3. **Timeline (15% weight):**
   - *Requirement:* Shorter completion time within 16 weeks.
   - *Design Assumption:* Inverse-timeline ratio relative to fastest eligible schedule ($T_{\text{min}} = 12\text{ weeks}$): $S_{\text{timeline}} = 15 \times (T_{\text{min}} / T_{\text{bid}})$.
   - *Scores:* A (12 wks) = 15.00; E (13 wks) = 13.85; C (14 wks) = 12.86.
4. **Warranty (15% weight):**
   - *Requirement:* Length of panel and workmanship warranties.
   - *Design Assumption:* 7.5 pts for panel warranty (25-yr benchmark = 7.5) and 7.5 pts for workmanship warranty ($7.5 \times \text{Years} / 12$, max 12 yrs).
   - *Scores:* E (25/12 yrs) = 15.00; A (25/10 yrs) = 13.75; C (25/10 yrs) = 13.75.
5. **Team and Approach (15% weight):**
   - *Requirement:* Named project lead, crew, safety and monitoring plan.
   - *Design Assumption:* 4 rubrics at 3.75 pts each (Lead, crew, safety plan, monitoring plan). In solar projects, "monitoring plan" denotes post-install performance monitoring (e.g., A's 12-month or E's 24-month monitoring). Proposal C specifies "weekly progress reports," which is construction communication during build rather than system performance monitoring, and omits a safety plan. C earns $3.75 + 3.75 = 7.50$ pts.
   - *Scores:* A = 15.00; E = 15.00; C = 7.50.

**Composite Triage Results:**
- **Proposal A (Rank 1, Score 98.8):** Lowest price, fastest schedule, 14 installs, full team compliance.
- **Proposal E (Rank 2, Score 92.4):** Competitive pricing, 13-week schedule, 12-year warranty, 24-month monitoring.
- **Proposal C (Rank 3, Score 85.5):** Eligible with 9 installs; higher price, slower schedule, omitted safety/monitoring plans.
- **Proposal B:** `ineligible` (Score: `null`, Rank: `null`). Disqualified: Bidder lacks required electrical licence.
- **Proposal D:** `flagged` (Score: `null`, Rank: `null`). Held for review: Omitted committed timeline (pending site assessment); indicates pending clarification, not confirmed failure.

**Explainability:**
Reviewers see itemized scorecards displaying extracted values, formulas, design assumptions, and links to verbatim quotes in original proposals.

---

### 3. Human in the Loop & Audit Logging

**Human Checkpoints:**
1. **Intake:** Reviewer uploads proposals and selects category.
2. **Exceptions:** Proposals marked `ineligible` (B) or `flagged` (D) enter an exception queue for confirmation or clarification.
3. **Security Review:** Warnings (e.g., C's Appendix C) alert reviewers without distorting scores.
4. **Overrides:** Reviewers inspect citations and correct extraction errors, triggering instant recalculation.
5. **Award:** Tool assists triage; human officers retain sole authority to award contracts.

**Audit Logging:**
Immutable logs record user IDs, timestamps, document hashes, criteria snapshots, raw LLM payloads, citations, scores, and overrides with rationales.

---

### 4. Changing Criteria or Weights

When an admin updates criteria or weights:
- **Immutable Versioning:** Configurations are stored as versioned entities (e.g., `Solar-v1.0` $\rightarrow$ `v1.1`).
- **Decoupled Architecture:** Modifying weights does not re-run LLM extractions.
- **Instant Re-Scoring:** Cached payloads re-score in milliseconds under updated rules.
- **Audit Comparison:** Historical and updated scores are preserved to assess policy impacts.

---

### 5. Risks and Safeguards

1. **Extraction Errors:** Strict JSON Schema; mandatory `source_quote` extraction; string containment checks against source text. Missing facts default to `null`.
2. **Adversarial Injections:** Proposal text is treated as untrusted data. Explicit prompt boundaries ignore embedded commands (e.g., C's Appendix C). Security warnings are logged for audit.
3. **Missing Evidence:** Incomplete submissions (e.g., D's uncommitted schedule) are marked `status: "flagged"` for human review rather than assumed to pass.
4. **Licence Non-Compliance:** Programmatic checks distinguish bidder-held licences from subcontracted deferrals (e.g., Proposal B).
5. **Cost and Latency:** Fast models (GPT-4o-mini) extract data once; structured payloads are cached in PostgreSQL. Scoring adjustments incur zero LLM API cost.
