# Application-Specific Hardware Protection — Draft Specification v0.1

## 1. Protect commercial applications from low-cost AI-driven application implementation extraction

Use of encrypted applications combined with hardware-enforced application execution protection and independent application/release protection.

Goal is to make it economically difficult for a typical user to extract, given the use of code agents (subscriptions) and typical tools. Not a guarantee of prevention (no forever).

### Anticipated shift in software architecture

Software will probably shift from on-device execution toward frontend wrappers for backend servers, with proprietary implementation running on the backend. This is an anticipated direction, not a certainty.

This specification addresses publishers who still want to offer protected local execution, including responsive software and offline use after initial provisioning, as that broader architectural shift occurs.

## 2. Who?

1. For software publishers with proprietary implementations that they want to protect but still offer local execution.
2. For application developers who want to embed in their build/release pipeline.
3. For customers who want responsive software and no-internet/offline use (post-initial provisioning).

## 3. Success criteria

Initial security bar is for independent evaluation to show no successful extraction of an executable version of the protected implementation within a **?10K budget per evaluated application (per attack)**.

?10K to include:

- Usage of AI/API.
- Compute rent.
- Capital cost of (attributed) equipment.
- People-cost (fixed prior to testing).

Equivalent usage cost (reproducible) for subscription usage should also be reported.

Success at attack is finding the implementation or equivalent (enough extracted stuff to run) outside of the approved protection environment.

Reimplementing some feature from visible behavior is beyond the scope of the claim on extraction.

Failure of evaluation only means a bounded "didn't see any success for extraction in given conditions/budget," which doesn't guarantee a global minimum for attack cost.

## 4. Threat model

Assumptions for attacker:

1. Legitimately licensed copy of software, with control over host PC.
2. Admin, install files, typical debug tools.
3. Use of commercial/open-weight AIs.
4. Access to typical storage, memory, and communication with application.
5. Aware of protection architecture. Able to use publicly available attacks.
6. Possible tinkering with host/loader/application.

Trust is limited to the chosen security boundary and related required firmware/trusted runtime, the publisher's build pipeline, and the key provisioning service.

Initial evaluation does not include physical chip attacks. Also relevant:

- Common software-accessible vulnerabilities.
- Known side-channels.
- Attempts to downgrade configuration.

## 5. Core elements

### R1 — Individual application identity and unique keys

- Every app has its own identity and unique keys (generated individually).
- Extracting an application's content key does not expose the content key for another application.
- "Unique" means cryptographic separation and policy separation (not just different filenames, not just different ciphertext).

### R2 — Independent release protection

- New content keys and approved identity for all the protected code/loader/relevant configuration per release.
- Avoids cross-application substitution. Avoids unauthorized substitution for release.
- Demonstrate offline rollback protection available on the chosen platform or otherwise note lack of guarantee.

### R3 — Encrypted distribution

- Must encrypt protected implementation material for distribution (use of established authenticated crypto).
- Should never see a plaintext copy in the distributed production packaging.
- No unprotected keys for decryption in the production package.

### R4 — Verified key provisioning

- Release keys only to authenticated customer entitlement and an approved (crypto-verified) execution environment.
- Verify that there is no tinkering with the loader, debug-enabled configuration, or security states that are out of policy.
- Provision keys to the protected environment in a way that is not exposed to a typical host application.

### R5 — Protection for load and execution

- Decrypt and execute code inside the approved environment.
- No escape of plaintext code/keys through temporary files, shared memory, logs, crash dumps, or diagnostic interfaces.
- Not meeting the spec if you have execution isolation but lack confidential loading support on the platform.

### R6 — Minimal exposed application interface

- Surrounding application gets to request some operations and get expected output.
- Should not expose arbitrary access to memory.
- Should not expose arbitrary execution.
- No diagnostic facilities that expose protected implementation.
- Must validate input inside the protected component.

### R7 — Application-specific diversification

- Build process should support (optional?) semantics-preserving diversification of internal structure, placement of some of the protection for diversification.
- Correctness/performance/effort for attack (reduced reusability) per transformation should be evaluated.
- Diversification should be a complement to hardware protection (not replacement).
- No custom crypto algorithms.

### R8 — Offline usage

- Supported device(s) should be able to use the protected component without an internet request for each launch/operation (after initial online provisioning).
- Offline storage of keys must be tied to the approved protected environment.
- Requires a lifecycle design for device replacement/recovery/updates.
- Can't promise instant revocation if it's indefinitely offline.

### R9 — No silent degradation of protection

- Error on lack of support for hardware or verification failure (clean compatibility error).
- No silent decryption to typical memory to save compatibility.

## 6. Scope of initial prototype

Build:

1. Single supported hardware/OS configuration.
2. Protected CPU-based engine in a representative application (embedded).
3. Separate application with protection framework (independent keys/identity).
4. Two releases for the first application.
5. Encrypted packaging process, protected loader, provisioning service.
6. Benchmark harness; independent extraction evaluation.

Need a second application and extra release to see if secrets/attacks transfer.

Initial scope does not include:

- Whole game protection.
- Protected graphics pipeline.
- Arbitrary legacy binaries.
- Universal OS support.

## 7. Performance

Tentative goals:

| Metric | Initial target |
| --- | --- |
| Loss for end-to-end throughput — steady state | <5% for agreed representative workload |
| 99th percentile increase for operation latency | <10% |
| Extra time for offline launch | <1 sec |
| Extra peak memory for application | <10% |
| Network dependence for typical operation | 0 (post-provisioning) |

Run three different configurations on the same hardware:

1. Native application without the protection runtime.
2. Application with the required platform security but without protected packaging.
3. Fully protected application.

Report cold/warm startup separately. Report CPU usage, memory, latency distributions, and energy usage (if measurable).

Crypto, boundary, and diversification should be evaluated separately.

Avoid averaging to hide stalls in interactive behavior.

If the target is not met, redesign or document an explicit trade-off. Not "less protection" without disclosure.

## 8. Security evaluation

Tests for:

1. Extraction from distributed files and typical process memory.
2. Tinkering with loader/application/host configuration.
3. Debugging/crash/diagnostic leakage.
4. Key substitution: cross-application and cross-release.
5. Offline execution, copy of device, rollback.
6. Reusability of an attack on the first app against the second.

Known vulnerabilities in chosen hardware/trusted runtime.

Costs for initial breakthrough. Costs for marginal reuse somewhere else.

May affect multiple applications (with unique keys) if there is a common loader/hardware vulnerability. (Key risk in the product.)

## 9. Continuity

Opt-in source/build material escrow and a tested end-of-life plan should be designed before commercial rollout.

- No customer data in continuity materials.
- No customer encryption keys.

Agree on release trigger, allowed redistribution, and dependencies. (Availability of source is not enough to guarantee a usable replacement.)

## 10. Go/no-go

Only continue from prototype if:

1. Demonstrably can support confidential loading/execution for the chosen platform.
2. Functional results match the non-protected baseline.
3. Meets agreed performance targets.
4. Independent keys/identity avoid cross-application secret reuse.
5. No successful extraction path found in the budget-extraction test (given scope).
6. Offline use/recovery supported without exposure of protected material.

Key question of feasibility is if application-specific protection increases cost of extraction substantially while keeping performance acceptable. (Not just: can you encrypt a binary?)
