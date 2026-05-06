# Feature Extraction Guide

How to systematically read specification documents and extract a complete list of features.

## What Is a Feature?

A feature is a single, discrete, testable capability that the software provides. It answers: "What does this software do?"

**Features are:**
- A specific behavior or capability: "vector-vector addition", "JSON config parsing", "email validation"
- Derived from spec requirements: every feature points to a section of the spec
- Testable: you can write a test that proves the feature works or doesn't

**Features are NOT:**
- Categories: "arithmetic operations" is a group, not a feature
- Implementation details: "uses HashMap internally" describes how, not what
- Parameters: "SEW=32" is a configuration, not a capability
- Quality attributes: "fast", "reliable", "secure" are non-functional requirements

## How to Read a Spec for Features

### Step 1: Identify requirement language

Scan the spec for requirement keywords. These signal features:

| Keyword | Meaning | Example |
|---------|---------|---------|
| **SHALL** | Mandatory requirement | "The system SHALL validate input" |
| **MUST** | Mandatory requirement | "The encoder MUST support UTF-8" |
| **WHEN...THEN** | Behavioral requirement | "WHEN input is empty THEN return error" |
| **IF...THEN** | Conditional requirement | "IF user is admin THEN allow deletion" |
| **supports** | Capability | "The processor supports 64-bit operations" |
| **provides** | Capability | "The API provides batch processing" |

Each requirement is a candidate feature or part of one.

### Step 2: Group by domain concept

Cluster related requirements into features. Requirements that describe different aspects of the same capability belong to the same feature.

Example: These three requirements describe one feature ("email validation"):
- "The system SHALL reject emails without @ symbol"
- "The system SHALL reject emails with spaces"
- "The system SHALL accept emails with + in the local part"

But these describe two features:
- "The system SHALL validate email format" → email validation
- "The system SHALL send confirmation email" → confirmation workflow

### Step 3: Check feature granularity

For each candidate feature, apply the granularity test:

- **Too coarse?** Can you describe two independent test scenarios that would pass/fail independently? If yes, split into two features.
- **Too fine?** Does the test for this only make sense combined with another feature's test? If yes, merge them.
- **Just right?** The feature has 2-5 meaningful test cases that all relate to the same capability.

### Step 4: Assign spec references

Every feature MUST trace to a specific location in the spec. Not "somewhere in chapter 3" — a section number, page number, or requirement ID.

```markdown
| ID | Feature | Spec Reference |
|----|---------|----------------|
| F1 | User registration | CUM022 §4.2.1, page 15 |
| F2 | Email validation | CUM022 §4.2.2, pages 16-17 |
```

If a feature doesn't trace to the spec, it's either:
- Missing from the spec (flag to your human partner)
- Not a real feature (remove it)
- Implied by the spec (note the implication explicitly)

### Step 5: Identify implicit features

Specs often assume certain features without stating them explicitly:

- **Error handling**: What happens when inputs are invalid? The spec may not say, but the software must handle it.
- **Defaults**: What happens when optional parameters are omitted?
- **Boundaries**: What are the limits of each parameter? What happens at the boundary?
- **Initialization**: How does the system start up? What state is it in initially?

Mark implicit features with their source: "Implied by §4.2 — spec defines valid inputs but not error behavior."

## Feature Registry Format

```markdown
## Feature Registry

| ID | Feature Name | Spec Reference | Category | Status |
|----|-------------|----------------|----------|--------|
| F1 | [name] | [doc §section, page] | [group] | proposed |
| F2 | [name] | [doc §section, page] | [group] | proposed |
| F3 | [name] | implied by [doc §section] | [group] | proposed |

### Feature Details

#### F1: [Feature Name]
**Spec:** [document, section/page]
**Description:** [1-2 sentences: what this feature does in domain terms]
**Spec requirements:**
- [Quoted or paraphrased requirement from spec]
- [Another requirement]
**Notes:** [Any ambiguities, open questions, dependencies on other features]
```

## Feature Categories

Group features by domain concept, not by implementation layer. Good categories describe what the user cares about, not how the code is organized.

**Good categories:** "Authentication", "Vector arithmetic", "Configuration parsing", "Error reporting"

**Bad categories:** "Frontend", "Database", "Utilities", "Helpers"

## Common Mistakes

### Listing chapters as features
The spec has a chapter called "Security". You create one feature "Security". This is too coarse — security includes authentication, authorization, encryption, audit logging, etc. Each is its own feature.

### Listing parameters as features
The spec says "supports SEW values 8, 16, 32, 64". You create four features, one per value. These are configurations of one feature ("configurable SEW"), not four separate features.

### Missing error-path features
The spec defines happy paths. You extract only those. But "rejects invalid input" and "handles timeout gracefully" are features too — often the most important ones.

### Inventing features not in the spec
You think the software should also do X. But the spec doesn't mention X. Don't add it. If you think it's important, flag it to your human partner: "The spec doesn't mention X — should it be a feature?"

### Inconsistent granularity
Some features are huge ("process all transactions") and some are tiny ("trim whitespace"). Aim for features that each take roughly the same effort to test: 2-5 test cases per feature.
