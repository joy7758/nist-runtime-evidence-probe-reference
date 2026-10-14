> **Moved:** this repository has been merged into [joy7758/ans-runtime-evidence-reference](https://github.com/joy7758/ans-runtime-evidence-reference/tree/main/nist-runtime-evidence-probe-reference) with full history. This repo is archived and read-only.

# NIST-style Runtime Execution Evidence Probe Reference

中文：NIST风格运行时执行证据探针参考实现。

> This is an independent, development-stage reference implementation.
>
> It is not an official NIST implementation. It is not endorsed, certified, or
> formally evaluated by NIST.

## Problem

An agent can produce a plausible answer while its runtime behavior differs from
what was authorized or claimed. A bounded evaluator therefore needs to ask
more than whether the final answer looks correct:

- Was the named tool actually invoked?
- Was the invocation authorized for this test case?
- Is required execution evidence present?
- Do the evidence bytes still match the recorded digest?
- Does the reported outcome agree with the retained evidence?

This reference evaluates supplied records. It does not execute an agent,
authorize a tool, collect production evidence, or change runtime state.

## Relationship to existing NIST probes

Existing NIST research probes evaluate whether outputs are faithful, complete,
and sufficient relative to trusted sources. NIST also describes structured
audit trails and probes that can run during an agentic workflow or post hoc.

This reference explores a complementary question:

> Are the agent's claimed runtime actions supported by independently
> verifiable execution evidence?

It does not replace factual-grounding probes. The labels and Schema in this
repository are community-authored and are not a NIST taxonomy or official NIST
interface.

References:

- [NIST: Building Evaluation Probes into Agentic AI](https://www.nist.gov/programs-projects/building-evaluation-probes-agentic-ai)
- [NIST evaluation-probe research repository](https://github.com/usnistgov/agentic-research-evaluation-probes)
- [Issue #1: Measurement scope vs execution conditions](https://github.com/usnistgov/agentic-research-evaluation-probes/issues/1)

## Runtime execution evidence dimension

The deterministic verifier compares:

```text
expected authorization and outcome
  + observed actor, trace, tool calls, and outcome
  + referenced evidence bytes
  -> independently computed scoped result
```

The checked concerns are:

1. authorization-to-execution binding;
2. tool invocation evidence;
3. evidence presence or expected-evidence absence;
4. evidence integrity;
5. outcome-to-evidence consistency; and
6. an independent verification result that does not trust the fixture's
   self-declared verdict.

## Failure cases

| Example | Expected result |
|---|---|
| [`successful_execution.json`](examples/successful_execution.json) | `PASS` |
| [`unauthorized_tool_call.json`](examples/unauthorized_tool_call.json) | `FAIL_UNAUTHORIZED_TOOL_CALL` |
| [`missing_evidence.json`](examples/missing_evidence.json) | `FAIL_MISSING_EVIDENCE` |
| [`evidence_tampering.json`](examples/evidence_tampering.json) | `FAIL_EVIDENCE_TAMPERING` |
| [`outcome_mismatch.json`](examples/outcome_mismatch.json) | `FAIL_OUTCOME_MISMATCH` |

All examples and evidence files are synthetic. They contain no customer,
patient, production, or private operational data.

## Machine-readable schema

[`schema/nist-evaluation-probe.schema.json`](schema/nist-evaluation-probe.schema.json)
is a strict JSON Schema Draft 2020-12 contract requiring:

```text
probe_id
agent_id
test_case
expected_behavior
observed_behavior
evidence_reference
verification_result
```

The input `verification_result` is a test-vector assertion only. The verifier
computes its own result and rejects malformed records, missing evidence,
unauthorized calls, changed evidence, trace inconsistencies, and mismatched
outcomes.

## Quick validation

With [uv](https://docs.astral.sh/uv/) installed:

```bash
uv run --with pytest --with jsonschema pytest -q
```

Evaluate the successful fixture:

```bash
uv run --with jsonschema python -m nist_probe.validator \
  examples/successful_execution.json
```

The CLI exits with `0` only for `PASS` and prints a machine-readable result.

## Repository structure

```text
README.md
LICENSE
schema/
examples/
  evidence/
nist_probe/
tests/
docs/
IMPLEMENTATION_REPORT.md
PUBLIC_RELEASE_READINESS_REPORT.md
```

Source provenance:

```text
SOURCE_REPOSITORY=digital-biosphere-architecture-specification
SOURCE_REPOSITORY_VISIBILITY=PRIVATE_URL_INTENTIONALLY_OMITTED
IMPLEMENTATION_SOURCE_COMMIT=e82a8aca2b324b2f65c7f1a08acb2beab8ac79a3
TECHNICAL_INPUT_SOURCE_COMMIT=c58e91026a831d5abd44586fbff1d8f90d92d7fc
SOURCE_AUTHOR=Bin Zhang
SOURCE_LICENSE=Apache-2.0
```

The standalone layout, public documentation, fifth static test vector, and
modified Schema metadata are changes made for this candidate package. Modified
source-derived files contain a modification notice where their file format
permits it.

## Scope and limitations

- SHA-256 detects byte changes; it is not a digital signature, trusted
  timestamp, custody proof, or non-repudiation mechanism.
- Evidence-capture completeness and activity outside the supplied trace are
  not proven.
- The verifier checks a supplied authorization reference; it does not prove
  that the issuer was legitimate.
- Local synthetic success does not prove production safety, legal compliance,
  domain fitness, interoperability, or system trustworthiness.
- No official NIST Schema, API, harness, conformance test, endorsement,
  certification, or formal compatibility determination is claimed.

## AI assistance disclosure

This reference implementation and its public-preparation materials were
prepared with assistance from OpenAI Codex and reviewed by Bin Zhang, who
remains responsible for the technical content and licensing. The earlier
private-source implementation also received a bounded review from Alibaba Cloud
Qwen; that review was input to development, not an external certification.
