---
name: spec-test-driven-development
description: Use when a project has specification documents (PDFs, reference docs) that define what the software must do — extracts features from specs, derives tests per feature, then drives TDD implementation with human verification against the spec
---

# Spec-Test-Driven Development

The spec is the golden rule. Features are extracted from specs. Tests are derived from features. Implementation follows tests.

Every feature traces back to the spec. Every test traces to a feature. No orphans.

**Announce at start:** "I'm using the spec-test-driven-development skill to extract features from specifications and derive tests before implementation."

<HARD-GATE>
Do NOT write any implementation code, invoke writing-plans, or begin TDD until:
1. Specs have been read and understood (Phase 1 complete)
2. ALL features are extracted and listed (Phase 2 complete)
3. Every feature has mapped test cases (Phase 3 complete)
4. Your human partner has approved the feature list and test mapping
This applies regardless of perceived simplicity.
</HARD-GATE>

## The Iron Laws

```
1. THE SPEC IS THE GOLDEN RULE — all features derive from it
2. NO IMPLEMENTATION WITHOUT FEATURE-TO-TEST MAPPING FIRST
3. EVERY FEATURE TRACES TO THE SPEC, EVERY TEST TRACES TO A FEATURE
```

Violating the letter of these rules is violating the spirit. No exceptions without your human partner's permission.

## When to Use

```dot
digraph when {
    rankdir=LR;
    has_specs [label="Project has spec\ndocuments?" shape=diamond];
    specs_define [label="Specs define what\nsoftware must do?" shape=diamond];
    stdd [label="spec-test-driven-\ndevelopment" shape=box style=filled fillcolor="#ccffcc"];
    brainstorm [label="brainstorming\nfirst" shape=box];

    has_specs -> specs_define [label="yes"];
    has_specs -> brainstorm [label="no"];
    specs_define -> stdd [label="yes"];
    specs_define -> brainstorm [label="no"];
}
```

**Use when:**
- Project has PDF specs, reference documents, or standards it must conform to
- Features need to be systematically extracted from these specs
- Each feature needs traceable test coverage before implementation
- Your human partner needs to verify features against the spec

**Not for:**
- Projects without specifications (use brainstorming)
- Simple bug fixes or hotfixes (use TDD directly)
- Exploratory prototypes

## Checklist

You MUST create a task for each of these items and complete them in order:

1. **Read specifications** — use NotebookLM or MarkItDown to ingest spec documents
2. **Ask clarifying questions** — query the spec for ambiguities, ask your human partner
3. **Extract feature list** — enumerate every feature the repo provides, traced to spec sections
4. **Human partner approves feature list** — verify completeness against spec
5. **Map features to tests** — each feature → multiple test cases with acceptance criteria
6. **Human partner approves test mapping** — verify tests cover all spec requirements per feature
7. **Save feature-test document** — commit to `docs/superpowers/specs/YYYY-MM-DD-<project>-features.md` (user preferences for location override this default)
8. **Begin TDD per feature** — invoke writing-plans with pre-derived tests, or invoke TDD directly
9. **Human partner verifies each feature** — confirm implementation matches spec

## Process Flow

```dot
digraph stdd {
    rankdir=TB;

    subgraph cluster_p1 {
        label="Phase 1: Read Specs";
        style=rounded;
        offer_tools [label="Offer NotebookLM /\nMarkItDown / PDF read" shape=box];
        read_specs [label="Read & understand\nall spec documents" shape=box];
        ask_questions [label="Ask clarifying\nquestions" shape=box];
        can_explain [label="Can explain spec\nin own words?" shape=diamond];

        offer_tools -> read_specs -> ask_questions -> can_explain;
        can_explain -> ask_questions [label="no"];
    }

    subgraph cluster_p2 {
        label="Phase 2: Extract Features";
        style=rounded;
        extract [label="Enumerate features\nfrom spec" shape=box];
        registry [label="Build Feature Registry\nwith spec references" shape=box];
        human_approve_features [label="Human partner\napproves feature list?" shape=diamond];

        extract -> registry -> human_approve_features;
        human_approve_features -> extract [label="gaps / changes"];
    }

    subgraph cluster_p3 {
        label="Phase 3: Map Tests";
        style=rounded;
        derive_tests [label="Derive test cases\nper feature" shape=box];
        matrix [label="Build Traceability\nMatrix" shape=box];
        human_approve_tests [label="Human partner\napproves test mapping?" shape=diamond];

        derive_tests -> matrix -> human_approve_tests;
        human_approve_tests -> derive_tests [label="gaps / changes"];
    }

    subgraph cluster_p4 {
        label="Phase 4: TDD per Feature";
        style=rounded;
        write_plan [label="Invoke writing-plans\nwith pre-derived tests" shape=box];
        tdd_cycle [label="RED → GREEN → REFACTOR\nper feature" shape=box];

        write_plan -> tdd_cycle;
    }

    subgraph cluster_p5 {
        label="Phase 5: Verification";
        style=rounded;
        human_verify [label="Human partner verifies\nfeature against spec" shape=diamond];
        done [label="Feature verified" shape=box style=filled fillcolor="#ccffcc"];

        human_verify -> done [label="matches spec"];
    }

    can_explain -> extract [label="yes"];
    human_approve_features -> derive_tests [label="approved"];
    human_approve_tests -> write_plan [label="approved"];
    tdd_cycle -> human_verify;
    human_verify -> tdd_cycle [label="needs revision"];
}
```

## Phase 1: Specification Reading

Read and understand the spec documents before anything else.

**Offer optional tools:**

Check for available tools and offer them to your human partner:

- **NotebookLM** (if `notebooklm` skill is available): "I can query your NotebookLM notebooks for source-grounded answers about the specification. Would you like to use it?"
- **MarkItDown** (if installed — check via `which markitdown`): "I can convert your PDF specs to markdown for direct analysis. Would you like me to run `markitdown spec/your-file.pdf`?"
- **Native PDF reading**: Claude Code reads PDFs directly — use as fallback when neither tool is available
- **Human partner input**: Always ask your human partner about ambiguities or missing context

Tools are optional. Spec reading is not.

**Key activities:**

- Read all spec documents cover-to-cover (or query systematically)
- Identify the domain vocabulary and key concepts
- Note all "SHALL", "MUST", "WHEN...THEN" requirements
- Ask your human partner about ambiguities or contradictions
- Build a mental model of what the software must do

**Gate:** You can explain what the spec requires in your own words before proceeding. If you can't, keep reading.

## Phase 2: Feature Extraction

Systematically enumerate every discrete feature the repo provides or should provide. Read @feature-extraction-guide.md for the full methodology.

A feature is a single, testable capability — not a chapter, not a category, not a parameter. "Vector-vector addition" is a feature. "Arithmetic operations" is a category.

**Output — Feature Registry:**

```markdown
## Feature Registry

| ID | Feature Name | Spec Reference | Category | Status |
|----|-------------|----------------|----------|--------|
| F1 | [capability name] | [spec doc, section/page] | [group] | proposed |
| F2 | [capability name] | [spec doc, section/page] | [group] | proposed |
```

Each feature MUST have:
- **Unique ID** (F1, F2, ...) — stable across the project lifecycle
- **Clear name** describing the capability in domain terms
- **Spec reference** — document name + section or page number. No orphan features.
- **Category** for grouping related features
- **Status**: proposed → approved → tested → implemented → verified

**Gate:** Your human partner reviews and approves the complete feature list before proceeding. Ask:

> "Here is the complete feature list extracted from the spec. Please review — are there features missing? Any that don't belong? Any that should be split or merged?"

## Phase 3: Feature-to-Test Mapping

For each approved feature, derive test cases from the spec's requirements. Read @feature-test-mapping-guide.md for the full methodology.

**Output — Traceability Matrix:**

For each feature:

```markdown
### F1: [Feature Name]

**Spec ref:** [document, section/page]

**Acceptance criteria (from spec):**
- WHEN [condition] THEN [system] SHALL [behavior]
- IF [precondition] THEN [system] SHALL [behavior]
- ...

**Test cases:**

| Test ID | Test Name | What it verifies | Type |
|---------|-----------|-----------------|------|
| F1.T1 | test_[behavior]_[condition] | [specific assertion] | unit |
| F1.T2 | test_[behavior]_[edge_case] | [specific assertion] | unit |
| F1.T3 | test_[integration_scenario] | [end-to-end check] | integration |
```

**Rules:**
- Every feature has ≥1 test. Most have 3+: happy path, edge cases, error cases.
- Every test traces to exactly one feature.
- Test names describe behavior, not implementation.
- Include both unit and integration tests where appropriate.
- Write test skeletons with concrete assertions — not placeholders.

**Gate:** Your human partner reviews and approves the test mapping. Ask:

> "Here is the test mapping for all features. Each feature has tests derived from spec requirements. Please review — are tests missing for any requirement? Any tests that don't make sense?"

## Phase 4: TDD Implementation

For each feature in dependency order, implement using RED-GREEN-REFACTOR with the pre-derived tests from Phase 3.

**For larger scope (multiple features):** Invoke `writing-plans` to produce a detailed implementation plan. The plan receives the pre-derived tests, so each task already has its test skeletons:

```markdown
### Task N: Feature F1 — [Feature Name]

**Spec ref:** [document, section]
**Pre-derived tests:** F1.T1, F1.T2, F1.T3

- [ ] Step 1: Write test F1.T1 (from Phase 3 skeleton)
- [ ] Step 2: Run — verify RED
- [ ] Step 3: Implement minimal code — verify GREEN
- [ ] Step 4: Write test F1.T2 (from Phase 3 skeleton)
- [ ] Step 5: Run — verify RED
- [ ] Step 6: Implement — verify GREEN
- [ ] Step 7: Refactor
- [ ] Step 8: Commit "feat(F1): implement [feature name]"
```

**For smaller scope (one or two features):** Invoke TDD directly — follow `superpowers:test-driven-development` with the Phase 3 test skeletons as input.

The terminal state of Phase 4 per feature is: all pre-derived tests pass, code is committed with feature ID reference.

## Phase 5: Human Verification

After each feature is implemented, your human partner verifies it against the spec.

Present each feature for verification:

> "Feature F1 ([name]) is implemented. All [N] tests pass. The spec requires [brief summary of spec requirement]. Please verify this matches the spec and mark it as verified or needs-revision."

- **verified** → Update feature status to `verified` in the registry
- **needs-revision** → Go back to Phase 4 with specific feedback. Do NOT re-derive tests unless the spec interpretation was wrong.

**Final gate:** All features in the registry are `verified`. Only then is the project complete.

## Constantly Consulting the Spec

Throughout ALL phases, repeatedly query the spec — not just in Phase 1:

- **During feature extraction:** "Let me check the spec for section X to confirm this feature's requirements"
- **During test derivation:** "Let me query the spec about edge cases for this behavior"
- **During implementation:** "Let me verify this matches the spec's definition of [concept]"
- **During verification:** "Let me re-read the spec section to confirm the implementation is correct"

If NotebookLM is available, prefer it for ongoing spec queries — source-grounded, citation-backed answers reduce hallucination risk. If MarkItDown was used, the markdown version is available for re-reading throughout.

The spec is not a one-time input. It is a living reference consulted at every decision point.

## Common Rationalizations

| Excuse | Reality |
|--------|---------|
| "I already understand the spec" | Understanding fades. Query the spec for every feature. |
| "This feature is obvious, no need to trace to spec" | Orphan features become orphan code. Every feature has a spec reference. |
| "One test per feature is enough" | One test = one happy path. Specs define edge cases too. |
| "I'll extract more features later" | Incomplete feature list = incomplete project. Extract ALL features first. |
| "The spec is ambiguous here" | Ask your human partner. Don't guess. |
| "NotebookLM/MarkItDown aren't available" | Read the PDF directly. Tools are optional, spec reading is not. |
| "Let me just start coding, I'll map tests after" | That's TDD without spec-derivation. Tests come from features come from specs. |
| "The feature list is too long" | That's how specs work. Prioritize with your human partner, but list them all. |
| "This test is too hard to write from spec alone" | Hard-to-test = unclear spec requirement. Clarify with your human partner first. |
| "I already know what tests to write" | Knowledge is not documentation. Derive from specs for traceability. |
| "Some requirements don't need tests" | If it's not testable, it's not a requirement. Remove or rewrite it. |
| "The spec is the brainstorming output, same thing" | Brainstorming produces a design. STDD derives testable features from authoritative spec documents. Different artifact. |

## Red Flags — STOP

- Implementation started without complete feature list
- Features without spec references (orphans)
- Tests without feature tracing
- Feature list not reviewed by human partner
- Spec not consulted during implementation
- "I'll add the remaining features later"
- Test mapping skipped for "simple" features
- Tests written AFTER implementation
- Traceability matrix has gaps
- "I'll map tests as I go"

**All of these mean: stop and go back to the appropriate phase.**

## Integration with Existing Skills

**Upstream (can invoke STDD):**
- `superpowers:brainstorming` — after producing a design, can transition to STDD for spec-driven test derivation

**Downstream (STDD invokes):**
- `superpowers:writing-plans` — Phase 4 output; receives pre-derived tests. writing-plans then offers subagent-driven-development or executing-plans as normal.
- `superpowers:test-driven-development` — Phase 4 for smaller scope; subagents follow RED-GREEN-REFACTOR with pre-derived test skeletons.

**Complementary:**
- `superpowers:verification-before-completion` — verify all spec-derived tests pass before claiming done

**Optional tools (soft references):**
- `notebooklm` skill — query knowledge bases for source-grounded spec answers
- `markitdown` CLI — convert PDF/Word/PowerPoint specs to markdown (`pip install 'markitdown[all]'`)

**Decision hierarchy for the agent:**
1. No idea / no specs → `brainstorming`
2. Have specs, need feature extraction + test derivation → **`spec-test-driven-development`**
3. Have specs + features + tests, need plan → `writing-plans`
4. Have plan, ready to execute → `subagent-driven-development` or `executing-plans`

## Verification Checklist

Before transitioning from Phase 3 to Phase 4:

- [ ] Every feature in registry has a spec reference
- [ ] Every feature has ≥1 test case
- [ ] Every test traces to exactly one feature
- [ ] No orphan features or orphan tests
- [ ] Test skeletons have concrete assertions (not placeholders)
- [ ] Human partner approved feature list
- [ ] Human partner approved test mapping
- [ ] Feature-test document saved and committed

Can't check all boxes? You're not ready for Phase 4. Fix the gaps first.
