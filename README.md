# SpecDrift

**A GenLayer Intelligent Contract for consensus-based specification change analysis.**

SpecDrift compares two versions of a public technical document and evaluates whether the changes may break existing integrations, preserve compatibility, introduce no material change, or remain indeterminate.

It uses GenLayer's nondeterministic execution and independent validator verification to turn natural-language document comparison into a structured, consensus-backed result.

## The Problem

Technical specifications, API documentation, and protocol requirements change over time. A change that appears minor in prose may break an existing integration, while a large documentation update may introduce no compatibility issues.

Traditional smart contracts cannot directly retrieve and interpret arbitrary public documents to determine the implications of these changes.

SpecDrift explores how GenLayer can provide a reusable contract primitive for this task.

## How It Works

1. A caller submits a baseline document URL.
2. The caller supplies a candidate document URL.
3. The caller defines the scope of the comparison.
4. The leader independently retrieves and evaluates both documents.
5. A validator independently repeats the evaluation.
6. GenLayer's nondeterministic execution compares the decision-bearing fields.
7. The contract stores the resulting classification, migration recommendation, summary, and evidence.

```text
Baseline Document ─┐
                   ├── Independent Evaluation
Candidate Document ┘           │
                               ▼
                        Leader Result
                               │
                               ▼
                      Independent Validator
                               │
                               ▼
                    Semantic Equivalence Check
                               │
                               ▼
                      Consensus-backed Result
```

## Classifications

| Classification | Meaning |
|---|---|
| `BREAKING` | A documented change appears likely to break existing integrations. |
| `NON_BREAKING` | Meaningful changes appear compatible with the previously documented behavior. |
| `NO_MATERIAL_CHANGE` | No material change was identified within the requested scope. |
| `INDETERMINATE` | The available documents do not support a reliable conclusion. |

Each resolved comparison also includes a migration recommendation:

- `REQUIRED`
- `NOT_REQUIRED`
- `UNKNOWN`

These outputs are assessments of the supplied documents, not guarantees that an integration will or will not fail in practice.

## Consensus Design

SpecDrift uses:

```python
gl.vm.run_nondet_unsafe(
    leader_fn,
    validator_fn,
)
```

The leader retrieves both documents and produces a structured result.

The validator independently retrieves and evaluates the same documents. It does not receive the leader's decision as an instruction for its own evaluation.

The consensus-critical fields are:

- `change_class`
- `migration_action`

The explanatory summary and evidence are stored as supporting information and are not required to match word for word.

This design allows independent executions to express their reasoning differently while requiring agreement on the fields that determine the classification and recommended migration action.

## State Model

Every comparison follows this lifecycle:

```text
PENDING → RESOLVED
```

The contract stores:

| Field | Description |
|---|---|
| `id` | Unique comparison identifier |
| `baseline_url` | URL of the original document |
| `candidate_url` | URL of the document being compared |
| `scope` | Caller-defined comparison scope |
| `status` | Comparison lifecycle state |
| `change_class` | Final classification |
| `migration_action` | Recommended migration action |
| `summary` | Explanation of the result |
| `evidence` | Supporting information from the documents |

A comparison cannot be resolved a second time after it has been completed.

## Example Use Cases

SpecDrift is designed to be reusable across different technical workflows.

**API compatibility analysis**

Compare API documentation to identify potentially breaking changes to endpoints, parameters, or response formats.

**Protocol upgrades**

Compare protocol specifications to identify changes that may require downstream integrations to adapt.

**SDK and library updates**

Compare published specifications across versions to investigate changes in documented interfaces or behavior.

**Smart contract interface changes**

Compare interface documentation to identify potentially incompatible changes in functions, arguments, or expected results.

**Documentation governance**

Check whether a revised technical document changes requirements within a defined scope.

## Using the Contract

### 1. Create a comparison

Call `create_comparison` with:

- `comparison_id`: a unique string identifier;
- `baseline_url`: the original document's HTTPS URL;
- `candidate_url`: the newer document's HTTPS URL;
- `scope`: the aspects that should be compared.

Example:

```python
contract.create_comparison(
    "api-v1-v2",
    "https://example.com/spec-v1",
    "https://example.com/spec-v2",
    "Compare endpoint names, required parameters, response formats, "
    "and backward compatibility."
)
```

The URLs above are illustrative. Use real, publicly accessible documents for an actual comparison.

### 2. Run the comparison

```python
contract.compare_versions("api-v1-v2")
```

The contract retrieves and evaluates the documents through GenLayer's nondeterministic execution and validator process.

### 3. Retrieve the result

```python
result = contract.get_comparison("api-v1-v2")
```

The result includes the comparison status, classification, migration recommendation, summary, and evidence.

Individual getter methods are also available:

```python
contract.get_status("api-v1-v2")
contract.get_change_class("api-v1-v2")
contract.get_migration_action("api-v1-v2")
contract.get_summary("api-v1-v2")
contract.get_evidence("api-v1-v2")
```

## Input Validation and Document Handling

SpecDrift checks that:

- comparison IDs are valid and unique;
- both document URLs use HTTPS;
- baseline and candidate URLs differ;
- the comparison scope is present and within the configured length limit;
- evaluator outputs use recognized classifications and migration actions.

Retrieved documents are treated as untrusted data. Instructions embedded in a document should not be followed as evaluator instructions.

The quality of the outcome depends on document availability, document completeness, the supplied comparison scope, and the reliability of the independent evaluations.

## Technology

- GenLayer Intelligent Contracts
- GenVM
- Nondeterministic web retrieval
- LLM-assisted semantic comparison
- Independent validator execution
- Semantic equivalence checking
- Typed persistent contract storage

## Development and Testing

SpecDrift is designed for GenLayer Studio.

Before treating a version as release-ready:

1. Confirm the contract schema loads in Studio.
2. Deploy the contract successfully.
3. Create a comparison between two accessible documents.
4. Run `compare_versions`.
5. Confirm that validator execution succeeds.
6. Verify the stored classification, migration action, summary, and evidence.
7. Test invalid inputs and attempts to resolve the same comparison twice.
8. Keep the GitHub source identical to the version tested in Studio.

Direct tests and integration tests should cover ordinary classifications, unavailable or incomplete documents, ambiguous changes, and validator disagreement.

## Limitations

SpecDrift does not execute code, run compatibility tests, or prove that an API change will break a production system.

It evaluates the supplied documents within the caller-defined scope. Incomplete documentation, inaccessible pages, ambiguous wording, or differing interpretations can lead to an `INDETERMINATE` result or prevent consensus.

The contract is intended to assist technical change analysis, not replace software testing or human review for high-impact upgrades.

## Project Goal

SpecDrift demonstrates a reusable application of GenLayer consensus: independently interpreting changing external documents and agreeing on a structured assessment of their compatibility implications.

The goal is to make specification-change analysis a reusable Intelligent Contract primitive rather than a one-off AI demonstration.
