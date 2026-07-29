````markdown id="izay9v"
# HDEX Q1 — Hardened Deterministic Execution

HDEX Q1 is a secure, deterministic, offline execution runtime for sensitive workloads.

It is designed for environments where ordinary task runners and workflow systems are not enough because the operator must prove:

- unauthorized actions were blocked,
- the same approved inputs can be reproduced,
- execution can happen without internet access,
- policies were enforced rather than merely documented,
- every important decision is auditable.

HDEX Q1 is not a GUI dashboard, cloud orchestrator, or plugin marketplace. It is a focused local execution system for high-trust workloads where security, reproducibility, and offline operation are product guarantees.

## Core Positioning

HDEX Q1 exists to answer one operational question:

> Can this job run safely, reproducibly, and offline, with evidence?

A valid HDEX Q1 run must be explainable before execution, controlled during execution, and verifiable after execution.

## Primary Guarantees

| Guarantee | Meaning | Proof Surface |
| --- | --- | --- |
| Secure | Unauthorized filesystem, network, process, worker, and secret access fails closed. | Policy decisions, sandbox enforcement, reason codes, security tests. |
| Deterministic | Approved jobs are tied to immutable inputs, pinned workers, stable manifests, and reproducible outputs. | Run manifests, content hashes, replay discipline, golden-output tests. |
| Offline | Core execution works without telemetry, package downloads, cloud activation, or remote services. | Offline tests, local bundles, signed offline updates, local documentation. |
| Auditable | Security-relevant actions produce structured records suitable for inspection and verification. | Audit events, hash-chain logs, run certificates, artifact provenance. |

## What HDEX Q1 Protects Against

HDEX Q1 is built around explicit threat modeling. The system treats submitted jobs, worker output, environment variables, policy files, manifests, paths, and external data as hostile until validated.

Representative threats include:

- unauthorized file access,
- sandbox escape attempts,
- malicious or modified workers,
- policy tampering,
- job definition tampering,
- dependency drift or poisoning,
- data exfiltration through network access,
- unsafe shell expansion,
- symlink and hardlink path escapes,
- environment-variable leakage,
- resource exhaustion,
- audit log tampering,
- secret exposure in logs or artifacts,
- nondeterministic outputs caused by hidden entropy,
- runtime behavior that silently depends on internet access.

## Design Rules

HDEX Q1 follows a small set of non-negotiable engineering rules:

1. Jobs are untrusted.
2. Workers are untrusted or semi-trusted.
3. The trusted core must stay small.
4. Filesystem access is denied unless explicitly mounted.
5. Network access is denied unless explicitly allowed.
6. Shell execution is denied.
7. Unsigned policies, jobs, and workers are rejected.
8. Policy failures fail closed.
9. Ambiguous paths are denied.
10. Secrets are never logged.
11. Runtime downloads are not allowed in normal execution.
12. Audit records are append-oriented and integrity-checkable.
13. Deterministic paths must avoid hidden time, randomness, unordered I/O, and machine-local drift.
14. Offline mode must not phone home.

## Architecture

HDEX Q1 separates trusted control from untrusted execution.

```text
+-------------------------------+
| Trusted HDEX Q1 Control Plane    |
|-------------------------------|
| CLI / Local API               |
| Runtime State Machine         |
| Policy Parser + Compiler      |
| Policy Evaluator              |
| Preflight Validator           |
| Sandbox Launcher              |
| Manifest Generator            |
| Audit Writer                  |
| Artifact Verifier             |
+---------------+---------------+
                |
                | controlled execution contract
                v
+---------------+---------------+
| Isolated Worker Sandbox       |
|-------------------------------|
| Worker Process                |
| Explicit Input Mounts         |
| Explicit Output Mounts        |
| Scrubbed Environment          |
| Resource Limits               |
| Network Policy                |
+---------------+---------------+
                |
                | untrusted output
                v
+---------------+---------------+
| Verification Layer            |
|-------------------------------|
| Output Contract Validation    |
| Artifact Hashing              |
| Quarantine Decisions          |
| Run Certificate               |
| Audit Hash Chain              |
+-------------------------------+
````

## Repository Layout

| Path                          | Purpose                                                                                    |
| ----------------------------- | ------------------------------------------------------------------------------------------ |
| `crates/hdex-core`            | Trusted runtime controller, runtime context, errors, and job state machine.                |
| `crates/hdex-policy`          | Policy parsing, compilation, evaluation, reason codes, and policy signature verification.  |
| `crates/hdex-sandbox`         | Filesystem, process, network, environment, resource, and platform sandbox controls.        |
| `crates/hdex-runner`          | Preflight checks, deterministic execution controls, job queue, executor, and replay.       |
| `crates/hdex-audit`           | Audit event models, tamper-evident log chain, redaction, and audit verification.           |
| `crates/hdex-artifacts`       | Content-addressed storage, artifact certificates, quarantine, and artifact store logic.    |
| `crates/hdex-crypto`          | Hashing, key handling, and signature primitives.                                           |
| `crates/hdex-manifest`        | Canonical JSON, manifest hashing, and run manifest structures.                             |
| `crates/hdex-packager`        | Offline bundle, license, SBOM, and signed update handling.                                 |
| `crates/hdex-api`             | Authenticated local API boundary.                                                          |
| `crates/hdex-cli`             | Operator CLI for run, preflight, explain, audit, offline-test, and security-test commands. |
| `crates/hdex-types`           | Shared typed contracts for jobs, policies, workers, manifests, audit, and artifacts.       |
| `crates/hdex-worker-protocol` | Worker request/response messages, exit states, and schema helpers.                         |
| `workers/python`              | Python worker SDK and example deterministic worker.                                        |
| `workers/node`                | Node worker SDK and example deterministic worker.                                          |
| `workers/rust`                | Rust worker manifest example.                                                              |
| `schemas`                     | JSON schemas for jobs, policies, run manifests, and workers.                               |
| `policies`                    | Default-deny, air-gap, and secure-document-pipeline policies.                              |
| `examples`                    | Demo jobs and policy-violation examples.                                                   |
| `tests`                       | Security, offline, crash-recovery, replay, and determinism test specifications.            |
| `docs`                        | Architecture, CLI, policy, determinism, offline, and security documentation.               |
| `scripts`                     | Developer and release automation.                                                          |
| `packaging`                   | OS-specific packaging and offline license/update examples.                                 |

## Execution Model

A normal HDEX Q1 run follows a strict lifecycle:

```text
CREATED
VALIDATED
QUEUED
RUNNING
COMPLETED
```

A denied or unsafe run follows an explicit failure path:

```text
CREATED
VALIDATED
BLOCKED_BY_POLICY
```

A suspicious or incomplete run is isolated:

```text
CREATED
VALIDATED
RUNNING
FAILED
QUARANTINED
```

Every state transition must be explainable. Silent acceptance is not valid behavior.

## Policy Model

Policies are declarative, human-readable, and compiled into enforceable runtime decisions.

A policy can control:

* allowed workers,
* worker versions,
* signed-worker requirements,
* filesystem read mounts,
* filesystem write mounts,
* network denial or allow-lists,
* shell denial,
* executable allow-lists,
* environment variable exposure,
* secrets availability,
* CPU limits,
* memory limits,
* disk quotas,
* process limits,
* execution timeout,
* deterministic seed,
* fixed clock,
* output constraints.

Policy evaluation is deny-by-default. If a rule is missing, ambiguous, unsupported, unsigned, malformed, or incompatible with the runtime, HDEX Q1 denies the run.

## Security Controls

HDEX Q1 security is built from multiple mutually reinforcing controls.

| Control                         | Purpose                                                                            |
| ------------------------------- | ---------------------------------------------------------------------------------- |
| Signed policies                 | Prevent silent policy tampering.                                                   |
| Signed job definitions          | Prevent approved jobs from being modified after review.                            |
| Signed workers                  | Prevent unknown or modified worker code from executing.                            |
| No-shell launcher               | Blocks shell expansion and command-injection paths.                                |
| Strict argument validation      | Rejects unsafe flags, paths, and malformed worker arguments.                       |
| Canonical path resolver         | Normalizes paths before enforcement and blocks traversal.                          |
| Symlink and hardlink protection | Prevents allowed paths from pointing into restricted locations.                    |
| Environment scrubber            | Removes inherited variables that can leak secrets or alter runtime behavior.       |
| Dynamic loader control          | Blocks unsafe native library injection surfaces.                                   |
| Process tree control            | Prevents background children from escaping job lifecycle management.               |
| Resource limits                 | Restricts CPU, memory, disk, process count, open files, and runtime duration.      |
| Quarantine                      | Keeps failed or suspicious outputs away from the trusted artifact store.           |
| Secret redaction                | Prevents sensitive values from appearing in logs, diagnostics, or support bundles. |
| Audit hash chain                | Detects audit log tampering.                                                       |
| Zero-trust local API            | Requires authentication even for local API calls.                                  |

## Determinism Controls

HDEX Q1 does not treat reproducibility as a side effect. Determinism is a runtime discipline.

Deterministic execution depends on:

* immutable job definitions,
* hashed inputs,
* pinned worker versions,
* dependency lock verification,
* fixed or controlled clocks,
* explicit random seeds,
* stable locale and timezone,
* sorted filesystem traversal where ordering matters,
* canonical JSON for manifests and hashes,
* stable artifact naming,
* controlled environment variables,
* no hidden network calls,
* no runtime dependency downloads.

A run manifest records the evidence needed to replay and compare execution.

## Offline and Air-Gap Operation

HDEX Q1 is designed to run in restricted environments.

Offline mode requires:

* no telemetry,
* no cloud activation,
* no package downloads at runtime,
* no external schema fetching,
* no automatic update checks,
* no remote documentation dependency,
* no DNS dependency for core execution,
* signed offline update bundles,
* locally verifiable licenses,
* bundled worker dependencies,
* local-only diagnostics.

The offline test suite exists to prove this behavior rather than assume it.

## Audit and Evidence

HDEX Q1 audit records are intended for investigation, debugging, and compliance review.

A complete run should produce:

* job hash,
* policy hash,
* worker identity and version,
* input hashes,
* output hashes,
* deterministic controls,
* policy decisions,
* denial reason codes,
* resource limits,
* operator identity where configured,
* run state transitions,
* artifact provenance,
* result status,
* audit log integrity information.

Sensitive data must be redacted before entering logs or exported diagnostics.

## Worker Contract

Workers communicate with HDEX Q1 through a strict protocol.

A worker must declare:

* name,
* exact version,
* supported input schema,
* supported output schema,
* required filesystem permissions,
* required network permissions,
* required executables,
* required environment variables,
* deterministic behavior expectations.

A worker must not:

* assume access to the host filesystem,
* access the network unless policy allows it,
* launch shells,
* depend on unpinned runtime packages,
* write outside approved outputs,
* log secrets,
* modify policy, audit, manifests, or trusted runtime state.

Worker output is untrusted until validated and linked to a run manifest.

## Flagship Demo Direction

The primary demo is a secure document pipeline.

The demo should prove the three core guarantees:

### 1. Security Proof

Attempted unauthorized behavior must fail:

```text
Job attempts to read outside mounted input directory.
HDEX Q1 denies the access.
Audit records DENIED_FS_OUTSIDE_MOUNT.
No artifact is trusted automatically.
```

```text
Job attempts outbound network access.
HDEX Q1 denies the connection.
Audit records DENIED_NETWORK_AIRGAP or DENIED_NETWORK_POLICY.
```

### 2. Determinism Proof

The same approved document job is run twice:

```text
same job spec
same policy
same worker version
same input hash
same seed
same fixed clock
same environment lock
```

Expected result:

```text
same output hash
same run manifest hash where volatile run identifiers are excluded
same artifact fingerprint
```

### 3. Offline Proof

The runtime is executed with network unavailable:

```text
no DNS
no telemetry
no package downloads
no license server call
no update check
```

Expected result:

```text
approved offline job succeeds
attempted network use fails with a policy reason
offline-test passes
```

## Example Operator Flow

Prepare runtime directories:

```bash
make runtime-dirs
```

Validate the development environment:

```bash
make doctor
```

Run standard checks:

```bash
make ci
```

Validate schemas, policies, examples, fixtures, and worker manifests:

```bash
make validate-config
```

Run security and offline proof suites:

```bash
make proof
```

Preflight the secure document pipeline demo:

```bash
make preflight-demo
```

Explain the demo permissions:

```bash
make explain-demo
```

Run the demo:

```bash
make run-demo
```

Run only the offline proof command:

```bash
make offline-demo
```

Run only the security proof command:

```bash
make security-demo
```

## CLI Direction

The HDEX Q1 CLI is the main operator interface.

Expected command surface:

```text
hdex run <job.yaml>
hdex preflight <job.yaml>
hdex explain <job.yaml>
hdex audit verify
hdex offline-test
hdex security-test
```

Command behavior must be deterministic when used in scripts. Human-readable output is acceptable for interactive use, but policy decisions, manifests, and audit exports must remain structured.

## Policy Example

Representative policy shape:

```yaml
id: secure-document-pipeline
version: 1
mode: enforce

defaults:
  filesystem: deny
  network: deny
  shell: deny
  unsigned_workers: deny
  telemetry: deny

workers:
  - name: pdf_redactor
    version: 1.0.0
    signature_required: true

filesystem:
  read:
    - mount: input_docs
      path: examples/secure-document-pipeline/input
  write:
    - mount: output_reports
      path: examples/secure-document-pipeline/output

network:
  mode: deny

execution:
  allow_exec:
    - python3
  timeout_seconds: 30
  max_processes: 4

resources:
  max_memory_mib: 512
  max_workspace_mib: 256
  max_artifact_mib: 64

determinism:
  enabled: true
  seed: "hdex-demo-seed"
  timezone: "UTC"
  clock: "2026-01-01T00:00:00Z"

audit:
  decision_log: true
  redact_secrets: true
```

Policies are not trusted merely because they parse. They must also be valid, signed where required, compiled, and enforceable by the runtime.

## Job Example

Representative job shape:

```yaml
id: redact-sample-document
worker:
  name: pdf_redactor
  version: 1.0.0

policy: secure-document-pipeline

inputs:
  - mount: input_docs
    path: sample.txt

outputs:
  - mount: output_reports
    path: sample.redacted.txt

determinism:
  mode: strict
```

A job definition becomes trustworthy only after validation, hashing, signature verification where required, policy compilation, preflight checks, and audit recording.

## Development Standards

HDEX Q1 code must preserve the project guarantees.

Rust source files must forbid unsafe code:

```rust
#![forbid(unsafe_code)]
```

Rust production code must not use:

* `unsafe`,
* `unwrap()`,
* `expect()`,
* `panic!()`,
* `todo!()`,
* `unimplemented!()`,
* `unreachable!()`,
* `unwrap_or_else()`,
* panic-driven control flow.

All external input must be validated with typed structures, clear errors, explicit bounds, and fail-closed handling.

Python code must avoid:

* `eval()`,
* `exec()`,
* `pickle`,
* `os.system()`,
* unsafe subprocess execution,
* unvalidated dictionary indexing,
* unsafe YAML loaders.

JSON and YAML must be treated as hostile input. Malformed documents must produce structured errors rather than crashes.

## Directory Placeholder Convention

Required runtime/generated directories are tracked with `.gitignore` placeholders.

The placeholder format is:

```gitignore
# This directory is required at runtime.
# Generated contents are intentionally ignored.

*

!.gitignore
```

This keeps required directories present while preventing generated logs, caches, artifacts, secrets, temporary files, and local runtime state from entering Git.

## Build and Validation

Common commands:

```bash
make fmt-check
make check
make clippy
make test
make validate-config
make security-test
make offline-test
make proof
```

Release readiness:

```bash
make release-check
```

Generate SBOM:

```bash
make sbom
```

Build offline bundle:

```bash
make offline-bundle
```

Sign bundle:

```bash
make sign-bundle
```

## Testing Strategy

HDEX Q1 requires tests that prove invariants, not only happy paths.

| Test Area            | Required Behavior                                                             |
| -------------------- | ----------------------------------------------------------------------------- |
| Policy parser        | Malformed JSON/YAML fails safely with structured errors.                      |
| Policy compiler      | Human-readable rules become enforceable runtime decisions.                    |
| Policy evaluator     | Denials include reason codes and fail closed.                                 |
| Filesystem sandbox   | Traversal, symlink escape, and unauthorized reads are blocked.                |
| Network sandbox      | Network is denied by default and in air-gap mode.                             |
| Process launcher     | Workers launch without shell expansion.                                       |
| Environment scrubber | Dangerous inherited variables are removed.                                    |
| Resource limits      | CPU, memory, disk, process, and timeout limits are enforced.                  |
| Determinism          | Repeated approved runs produce stable output hashes.                          |
| Replay               | Previous runs can be reconstructed from manifests.                            |
| Audit                | Hash-chain integrity detects tampering.                                       |
| Artifacts            | Outputs are hashed, linked to runs, and quarantined when unsafe.              |
| Offline              | Core execution passes with internet unavailable.                              |
| Crash recovery       | Partial writes, interrupted runs, and incomplete audit events recover safely. |

## Non-Goals

HDEX Q1 intentionally avoids early expansion into:

* generic workflow orchestration,
* cloud-first execution,
* remote managed services,
* broad dashboard development,
* marketplace-style plugin hosting,
* AI feature branding without security value,
* implicit network integrations,
* policy warnings without enforcement.

Depth is more important than breadth. The system must be hard to bypass, easy to prove, and simple to deploy.

## Release Discipline

A release must provide:

* pinned dependencies,
* reproducible build metadata,
* SBOM,
* signed bundle,
* signed update manifest,
* offline install path,
* security self-test,
* offline self-test,
* audit continuity checks,
* documented migration notes.

A release must not depend on hidden local state, runtime downloads, unpinned workers, unsigned policies, unsigned jobs, or silent configuration drift.

## Current Development Focus

The first production milestone is the proof-first core:

1. strict execution sandbox,
2. enforced policy engine,
3. deterministic execution controls,
4. run manifest generation,
5. offline operation,
6. tamper-evident audit logging,
7. secure document pipeline demo.

Secondary hardening layers include:

* signed policies,
* signed jobs,
* signed workers,
* zero-trust local API,
* preflight validation,
* policy reason codes,
* security invariant tests,
* environment scrubbing,
* canonical path resolution,
* process tree cleanup,
* resource exhaustion guards,
* artifact quarantine,
* run certificates,
* SBOM and offline bundle signing.

## License

HDEX Q1 is free-to-use proprietary software owned by Kevin George. The public
release does not include or license HDEX Q1 core source code.

Users may download and use the official signed HDEX Q1 binary release for free,
subject to the license terms. Modification, reverse engineering, redistribution,
resale, sublicensing, repackaging, fake builds, signature tampering, and
derivative works are restricted. Public documentation, verification commands,
public key fingerprints, SDK usage material, and worker SDK usage material may
be used for legitimate HDEX Q1 operation and integration. See `LICENSE` for the
full terms.

```
```
