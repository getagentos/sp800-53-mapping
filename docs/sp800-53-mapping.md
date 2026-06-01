# [NIST SP 800-53 Rev 5](https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final) Control Mapping for AI Agent Decision Evidence

| Metadata | |
|---|---|
| Version | 1.0.0 |
| Author | Swapan Shridhar. Founder, AgentOS. Apache Ambari PMC alumnus (2015–2018). 19 years infrastructure engineering, including FedRAMP GovCloud delivery and FIPS 140-3 compliance work. |
| Contact | swapan@getagentos.ai |
| License | Apache License, Version 2.0 |
| Repository | https://github.com/getagentos/sp800-53-mapping |
| DOI | [10.5281/zenodo.20486631](https://doi.org/10.5281/zenodo.20486631) |
| Status | Stakeholder contribution. Not endorsed by NIST. |

## Abstract

I map NIST SP 800-53 Rev 5 controls to a cryptographic decision-evidence artifact format for AI agents. The headline finding is short: AU-10 Non-repudiation, plus enhancements 1, 2, 3, and 5, already describes what an AI agent decision-evidence artifact has to satisfy. The control objectives exist. What is missing is implementation guidance.

Primary mapping is the AU family. Secondary mappings touch AC, IA, SR, and SI where the artifact format crosses into those control families.

The mapping draws on practical experience building infrastructure under FedRAMP and FIPS 140-3 compliance requirements. The framework primitives I rely on (SP 800-53, FIPS 186-5, FIPS 202, RFC 3161, RFC 8785) are already in NIST's stewardship. What I am proposing is the specification that ties them together for the agentic AI use case.

## 1. Motivation

When the comment period closed on [NIST docket NIST-2025-0035](https://www.regulations.gov/docket/NIST-2025-0035), I read most of the substantive submissions. Several of them converge on the same observation, in different words.

The BPI / BITS / American Bankers Association joint submission asked NIST for guidance covering "machine-to-machine access controls, traceability, audit logging, and mechanisms for orderly shutdown of misbehaving agents." Their phrase was "attributable, auditable, and stoppable." The Foundation for Defense of Democracies submission named parallel control gaps across the AC, IA, AU, and SR families and called for engineering updates to SP 800-160 and SP 800-218. The Anthropic submission framed the problem as a non-compromised-agent threat model, observing that current NIST publications assume external adversaries or deliberate misuse and do not address agents operating inside permissions that nonetheless cause harm. The OpenID Foundation AIIM submission addressed the identity and authorization layer. IEEE-USA contributed a threat catalogue.

The auditability and non-repudiation gap is named in submission after submission. None of them proposed an artifact format that would close it.

The [NCCoE Concept Paper on Software and AI Agent Identity and Authorization](https://www.nccoe.nist.gov/projects/software-and-ai-agent-identity-and-authorization), dated February 5, 2026, frames the problem in a complementary way. It identifies four agent control dimensions: identification, authorization, auditing, non-repudiation. The first two are addressed by the OpenID Foundation work and by the broader NCCoE OAuth / SPIFFE / SCIM / MCP scoping. The third and fourth, in the same paper's words, "are an area where considerable additional guidance is needed."

That is the gap this document proposes to close.

The mapping below addresses it in two layers. First, by identifying the existing SP 800-53 controls and enhancements that already specify the control objective. Second, by describing concretely what an artifact has to do to satisfy them. The second part is what is missing from NIST publications today.

## 2. Terminology

A small set of terms appears throughout the mapping. Each names a general capability rather than any specific product. The implementation example is included to show the controls are realisable end to end.

| Term | Meaning |
|---|---|
| Decision Passport | A signed, sealed record of one consequential AI agent action. Each passport is cryptographically signed (Ed25519), committed to write-once storage (WORM), and timestamped by an independent authority (RFC 3161). Schema v1.1.0 defines twenty-nine fields (see getagentos.ai). |
| Authorization Token | A signed token issued before agent execution that commits the agent to a specific input. Carries `input_hash`, `action_type`, `policy_version_hash`, time-to-live, and the producing-service identity. |
| WSS Protocol | The write-storage-seal sequence used to commit a Decision Passport to write-once-read-many (WORM) storage and obtain an independent time-stamping authority signature. |
| ABSA Interceptor | A reference monitor at the agent-tool transport boundary that hashes the actual call payload for comparison with the Authorization Token. |
| Evidence Tier | A per-artifact classification of evidentiary weight. TIER-A means full WORM commit plus RFC 3161 timestamp, treated as primary evidence. TIER-B means a boundary record with a declared 60-second attestation uncertainty window, treated as corroborating evidence. TIER-C means an uninstrumented node, lowest weight. |
| Causality Chain | A hash-chained record linking sequential agent decisions across multi-agent workflows, enabling forensic reconstruction. |

A reference implementation of all the above is available at https://getagentos.ai. The control mapping in Sections 3 and 4 is implementation-neutral. Any artifact format that satisfies the property column closes the listed control.

## 3. Primary Mapping: AU (Audit and Accountability) Family

The AU family is the natural locus for any AI agent decision-evidence artifact. The mapping below pairs each in-scope control or enhancement with the artifact properties that satisfy it.

| SP 800-53 Control / Enhancement | Control intent | Artifact property that satisfies it |
|---|---|---|
| **AU-2 Event Logging** | Define which events the system audits. Coordinate event logging with other organizational entities. Provide rationale for selected event types. | The Decision Passport schema (29 fields, v1.1.0) is the explicit definition of the audit event for any consequential AI agent action. The `action_type` field captures the event class. The schema is published and version-locked. |
| **AU-3 Content of Audit Records** | Audit records contain event type, time, location, source, outcome, and identity of users, subjects, objects. | All five required content elements are first-class fields: event type (`action_type`), time (`rfc_3161_timestamp`), location (`deployment_mode` and `region`), source (`principal_identity`), outcome (`execution_result`). The schema additionally records `causality_chain_hash` and `policy_version_hash` for forensic reconstruction. |
| **AU-3(1) Additional Audit Information** | Generate audit records containing additional information beyond the baseline. | `input_hash`, `output_pre_postprocess`, `full_prompt_snapshot`, `rag_evidence_bundle`, `model_version_hash`, and `tool_call_log` extend baseline content. Each is forensically necessary for an OCC examiner, supervisory authority, or opposing counsel reconstructing a specific agent decision. |
| **AU-3(2) Centralized Management** | Centrally manage and configure the content of audit records. | The Decision Passport schema is centrally version-controlled. The schema version is included in every artifact. Content changes are themselves an auditable schema-bump event. |
| **AU-8 Time Stamps** | Use internal system clocks and synchronize with an authoritative time source. | RFC 3161 timestamping by an independent time-stamping authority, applied after WORM commit, anchors every artifact to an authoritative external time source. The `lock_verification_latency_ms` field records elapsed time between commit and timestamp precisely. This closes a control that internal NTP synchronisation alone does not satisfy for legal-standing evidence. |
| **AU-9 Protection of Audit Information** | Protect audit information and audit logging tools from unauthorized access, modification, and deletion. | The customer owns the WORM storage bucket. The producing service has no write access post-provisioning. The bucket is in S3 Object Lock COMPLIANCE mode (or equivalent WORM-enforced storage). The cryptographic signature is independently verifiable without the producing service's infrastructure. Any modification or deletion attempt is mathematically detectable. |
| **AU-9(1) Hardware Write-Once Media** | Write audit trails to hardware-enforced write-once, read-many media. | S3 Object Lock in COMPLIANCE mode is the AWS implementation of hardware-enforced WORM. Per-object retention verification, custody-chain fields, and the `lock_verification_latency_ms` boundary statement document compliance per artifact. Equivalent implementations exist on Azure (Immutable Blob Storage) and GCP (Bucket Lock). |
| **AU-9(3) Cryptographic Protection** | Implement cryptographic mechanisms to protect the integrity of audit information. | Ed25519 signature (FIPS 186-5) on every artifact. Production artifacts are signed inside a FIPS 140-2 Level 3 HSM; design partner phase uses software signing with the same Ed25519 algorithm. SHA3-256 (FIPS 202) with JCS canonicalization (RFC 8785) for schema integrity. RFC 3161 timestamp adds independent third-party temporal proof. The combination produces a tamper-evident record verifiable by any third party in possession of the artifact bytes and the published public key. |
| **AU-10 Non-repudiation** | Provide non-repudiation evidence of producer identity and content. | This is the headline control. The entire decision-evidence artifact architecture is purpose-built for AU-10. Signed artifact, plus WORM commit, plus RFC 3161 timestamp, equals legally non-repudiable evidence that a specific agent took a specific action under a named policy at a documented time. The four enhancements below specify this further. |
| **AU-10(1) Association of Identities** | Bind identities to information. | The `principal_identity` field is cryptographically bound to the artifact via the signature. The binding cannot be forged without HSM compromise. |
| **AU-10(2) Validate Binding of Information Producer Identity** | Validate the binding of producer identity to the information. | The `rsa_cert_key_id` field plus the HSM signature allow any verifier to validate the binding. A standalone verifier (typically a CLI distributed with the schema) uses only the published public key. No cooperation from the producing service is required. |
| **AU-10(3) Chain of Custody** | Maintain reviewer identity and the chain of custody. | The custody-chain fields document the sequence: pre-execution Authorization Token issuance, execution, post-execution WORM commit, RFC 3161 timestamp. Each transition is cryptographically linked. The `causality_chain_hash` field extends this across multi-agent workflows. |
| **AU-10(5) Digital Signatures** | Use digital signatures to enforce non-repudiation. | Ed25519 (FIPS 186-5) digital signature is the cornerstone primitive of every Decision Passport. |
| **AU-11 Audit Record Retention** | Retain audit records consistent with the records retention policy. | The `retain_until_date` field is set at PutObject time and recorded in the artifact itself. Retention is enforced by the underlying WORM storage layer, not by the producing service. The retention period is auditable from the artifact alone. |
| **AU-12 Audit Record Generation** | Provide audit record generation capability for the events identified. | Two complementary instrumentation modes are supported: a business-function decorator pattern at the consequential-action boundary, and automatic LLM-API monkey-patching at process startup. Both modes produce identical Decision Passports and are testable in CI/CD. |
| **AU-12(1) System-Wide / Time-Correlated Audit Trail** | Compile audit records into a system-wide audit trail that is time-correlated. | A per-deployment ledger (typically DynamoDB) collects all `PASSPORT_COMMIT_FAILED`, `PASSPORT_INTEGRITY_WARNING`, and successful seal events with UUID primary keys and timestamp sort keys, providing system-wide time-correlated retrieval across all instrumented agents. |
| **AU-12(2) Standardized Formats** | Produce audit records in a standardized format. | The schema is published, version-controlled, and canonically serialized (JCS per RFC 8785). A standalone verifier confirms format compliance independent of the producing service. |
| **AU-14 Session Audit** | Provide capability to capture sessions for review. | `full_prompt_snapshot`, plus `tool_call_log`, plus `rag_evidence_bundle`, reconstruct the agent session at decision time. The `causality_chain_hash` links session events across multi-step reasoning. |
| **AU-16 Cross-Organizational Audit Logging** | Coordinate audit information among external organizations when audit information is transmitted across boundaries. | Standalone verifiability is the core mechanism. A regulator, opposing counsel, or partner organisation verifies the artifact using only the published public key. No back-channel, no API access, no trust in the producing service is required. The artifact is portable across organisational boundaries by design. |

## 4. Secondary Mappings: AC, IA, SR, SI Families

The artifact format additionally satisfies, in part, controls from four other families. These mappings are secondary in the sense that AU is the natural primary fit. The controls below are closed in conjunction with the AU evidence.

| SP 800-53 Control | Control intent | Artifact property |
|---|---|---|
| **AC-3 Access Enforcement** | Enforce approved authorizations. | The Authorization Token is issued before execution. A substrate check (`SHA3-256(actual_input) == token.input_hash`) blocks unauthorised execution. The declared `action_type` in the token gates downstream tool calls. |
| **AC-4 Information Flow Enforcement** | Enforce approved authorizations for controlling the flow of information. | `input_hash` binding ensures the agent acts on the exact authorised input. The `tool_call_log` plus `execution_result` allow post-hoc verification that information flowed as authorised. |
| **AC-25 Reference Monitor** | Implement a reference monitor mediating all access. | The ABSA Interceptor at the agent-tool transport boundary is a reference monitor for tool calls. The Decision Passport is the evidence that the reference monitor functioned at decision time. |
| **IA-9 Service Identification and Authentication** | Identify and authenticate non-human services. | `principal_identity` uniquely identifies the agent. `rsa_cert_key_id` plus the HSM signing key authenticate the producing service. |
| **IA-12 Identity Proofing** | Verify identity claims with appropriate evidence. | The Decision Passport provides the cryptographic evidence that an identity claim made by an agent is valid for a specific decision. |
| **SR-3 Supply Chain Controls and Processes** | Establish supply chain risk management controls. | `model_version_hash` plus `sdk_binary_sha3` plus `registry_validation_status` form the supply chain provenance for the model and the SDK. This maps to OWASP Agentic AI Risk 9 (supply chain risks). |
| **SR-4 Provenance** | Document, monitor, and maintain valid provenance. | Same fields as SR-3. The Decision Passport itself is also provenance evidence for any downstream artifacts produced by the agent. |
| **SI-4 System Monitoring** | Monitor the system for indicators of compromise and unusual behavior. | Per-decision artifacts enable longitudinal monitoring across an agent's lifetime. Anomalies in `action_type` frequency, `instrumentation_coverage`, or `causality_chain` depth are detectable from the artifact set. |
| **SI-7 Software, Firmware, and Information Integrity** | Verify integrity of software, firmware, and information. | The `passport_hash` forensic invariant covers all non-signing schema fields. Any tamper attempt is detectable by recomputing the hash and comparing. |

## 5. How to Use This Mapping

The most direct application is [COSAiS](https://csrc.nist.gov/projects/cosais). The AU-family mapping in Section 3 is implementation guidance for the agentic AI use cases currently in scoping (single-agent and multi-agent). Feedback to `overlays-securing-ai@list.nist.gov` is welcome.

The NCCoE AI Agent Identity and Authorization demonstration project covers four dimensions. This mapping addresses auditing and non-repudiation. The identification and authorization dimensions are addressed by adjacent work on OAuth 2.0, OIDC, SPIFFE / SPIRE, SCIM, and MCP. A joint demonstration combining an identity layer with this evidence layer is technically straightforward.

On the Cyber AI Profile (NIST IR 8596), the mapping serves as implementation guidance for the Protect and Respond functions of the CSF 2.0 Core. Protect via pre-execution binding. Respond via the audit trail at incident time.

Financial services institutions can apply the mapping under SR 11-7 (model risk management), FFIEC Authentication Guidance, OCC 12 CFR Part 30 Appendix D (Heightened Standards), NYDFS Part 500, the NY RAISE Act (effective January 2027), Regulation B, and Regulation Z. Adverse-action disclosure under Reg B and Reg Z benefits from the same evidence set. Healthcare, defence, and other regulated sectors have adjacent applicability.

Security teams doing vendor risk review can use the AU-10 row of Section 3 as a control-mapping reference. Hand it to the team and the review acceleration is concrete.

The artifact format is also profile-compatible with IETF SCITT Signed Statements. An Internet-Draft profiling a Decision Statement for SCITT is in preparation.

## 6. References

### Normative

- [NIST SP 800-53 Rev 5](https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final). Security and Privacy Controls for Information Systems and Organizations.
- FIPS 186-5. Digital Signature Standard (DSS). Specifies Ed25519.
- FIPS 202. SHA-3 Standard: Permutation-Based Hash and Extendable-Output Functions.
- RFC 3161. Internet X.509 Public Key Infrastructure Time-Stamp Protocol (TSP).
- RFC 8785. JSON Canonicalization Scheme (JCS).

### Informative

- [NIST IR 8596 Preliminary Draft](https://csrc.nist.gov/pubs/ir/8596/iprd) (December 2025). Cybersecurity Framework Profile for Artificial Intelligence (Cyber AI Profile).
- NIST SP 800-218A. Secure Software Development Practices for Generative AI and Dual-Use Foundation Models.
- NIST AI 100-2 (2025). Adversarial Machine Learning: A Taxonomy and Terminology of Attacks and Mitigations.
- [NCCoE Concept Paper](https://www.nccoe.nist.gov/projects/software-and-ai-agent-identity-and-authorization) (February 5, 2026). Accelerating the Adoption of Software and AI Agent Identity and Authorization.
- IETF SCITT WG. `draft-ietf-scitt-architecture` (working-group last call, October 2025). <https://datatracker.ietf.org/wg/scitt/>
- IETF RATS WG. RFC 9334 (Architecture, January 2023). RFC 9782 (Entity Attestation Token Media Types). <https://datatracker.ietf.org/wg/rats/>
- Federal Reserve SR 11-7. Guidance on Model Risk Management (April 4, 2011).
- OCC 12 CFR Part 30 Appendix D. Heightened Standards.
- NYDFS 23 NYCRR 500. Cybersecurity Requirements for Financial Services Companies.
- NY RAISE Act (signed December 2025, effective January 1, 2027).
- OWASP Top 10 for Agentic Applications (December 2025).

### Public submissions to [NIST docket NIST-2025-0035](https://www.regulations.gov/docket/NIST-2025-0035) referenced in this mapping

- Anthropic. Response to NIST RFI on Agentic Security (March 9, 2026).
- Foundation for Defense of Democracies. Regarding Security Considerations for Artificial Intelligence Agents (March 9, 2026).
- OpenID Foundation AIIM Threat Modeling Subgroup. Response to NIST AI Agent Security RFI (March 2026).
- [BPI / BITS / American Bankers Association](https://www.aba.com/advocacy/policy-analysis/joint-letter-on-ai-security-considerations-rfi). Joint Letter to NIST on the RFI re: Security Considerations for AI Agent Systems (March 9, 2026).
- IEEE-USA. Response to NIST RFI on Agentic AI (March 9, 2026).

## 7. Feedback and Iteration

I expect this mapping to iterate. COSAiS will publish the agentic overlays. The Cyber AI Profile will move from preliminary draft to IPD. The SCITT working group is progressing toward consensus. As those pieces land, this mapping should update to reference them directly.

Feedback channels:

- COSAiS mailing list: `overlays-securing-ai@list.nist.gov`
- Direct email: `swapan@getagentos.ai`
- Pull requests and issues on this repository

This document is not endorsed by NIST and is not a substitute for official guidance. It is offered openly for anyone working on the problem.

Contact: Swapan Shridhar. `swapan@getagentos.ai`. <https://getagentos.ai>
