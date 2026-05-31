# OWASP Agentic AI Top 10 -- Decision Evidence Mapping

**AgentOS** · May 31, 2026

AgentOS produces a cryptographically signed, tamper-proof artifact for every consequential AI agent decision -- verifiable by any examiner without vendor cooperation.

---

## Evidence Type Definitions

| Term | Meaning |
|---|---|
| **Prevention** | AgentOS blocks or requires authorization before the risk materializes. |
| **Recording** | AgentOS captures immutable cryptographic evidence proving what occurred. |
| **Prevention + Recording** | Both mechanisms active: blocks pre-execution and produces a tamper-proof artifact post-execution. |

---

## Mapping Table

| # | OWASP Risk | Evidence Type | Controls | What the Artifact Proves |
|---|---|---|---|---|
| 1 | Goal hijacking | Prevention + Recording | `input_hash` · substrate enforcement · `AUTHORIZATION_INPUT_MISMATCH` | Input committed in signed token before execution. Substrate blocks if actual input differs. Artifact proves authorized vs presented input. |
| 2 | Tool misuse | Prevention + Recording | `tool_call_log` · `authorization_token_id` · `action_type` · `BLOCK_ON_SERVICE_UNAVAILABLE` | Signed token declares `action_type` pre-execution. `tool_call_log` records every tool invoked. Comparison proves misuse. |
| 3 | Identity abuse | Recording | `principal_identity` · `rsa_cert_key_id` · HSM signing | Every artifact proves which agent acted via HSM-signed identity. Cannot be forged without HSM compromise. |
| 4 | Memory poisoning | Recording | `full_prompt_snapshot` · `rag_evidence_bundle` · `causality_chain_hash` | Complete prompt captured at call time. RAG sources recorded with provenance. Forensic examiner identifies poisoning across chain. |
| 5 | Cascading failures | Recording | `causality_chain_hash` · `instrumentation_coverage` · `PASSPORT_COMMIT_FAILED` | Hash-chained record of every step. Coverage records instrumented vs uninstrumented nodes. Failure patterns reconstructable. |
| 6 | Insecure output handling | Recording | `output_pre_postprocess` · `execution_result` | Raw model output before and after filters. Delta is auditable. Insecure handling detectable from artifact. |
| 7 | Code execution abuse | Prevention + Recording | `action_type` · `tool_call_log` · `BLOCK_ON_SERVICE_UNAVAILABLE` | TIER-A token required before code execution. `action_type` + `tool_call_log` prove authorized vs actual execution. |
| 8 | Human-agent trust exploitation | Prevention + Recording | `policy_version_hash` · `availability_policy` · `CONFIGURABLE_BY_ACTION_TYPE` | Human approval tokens required for high-risk actions. Policy version bound to artifact proves governance in force at decision time. |
| 9 | Supply chain risks | Recording | `model_version_hash` · `registry_validation_status` · `sdk_binary_sha3` · `rekor_log_id` | Exact model, SDK binary hash, Sigstore transparency log entry. Complete forensic supply chain traceability. |
| 10 | Rogue agents | Recording | `deployment_mode` · `evidence_tier` (TIER-A/B/C) · `instrumentation_coverage` | Trust level per artifact. Uninstrumented nodes produce TIER-B boundary records flagging potential rogue paths. |

---

## Differentiation

| Category | What they do | How AgentOS differs |
|---|---|---|
| **Runtime enforcement** (Assury, Microsoft AGT, FriskAI) | Governs what agents can do. | AgentOS proves what agents did do. Per-decision artifact verifiable by OCC examiner or opposing counsel without vendor cooperation. |
| **Hardware TEE** (EQTY Lab / Intel TDX) | Proves computation was genuine: inference ran in attested hardware. | AgentOS proves the decision was authorized under a specific policy version, with pre-committed input hash, in a tamper-proof artifact. |
| **Observability** (Datadog, Arize, LangSmith) | Mutable application logs. Not legal-grade evidence. | AgentOS: HSM-signed, S3 Object Lock COMPLIANCE (WORM), RFC 3161 timestamped. Survives adversarial examination. |
| **AI governance platforms** (Credo AI, Monitaur) | Model-level risk governance. No per-decision cryptographic artifact. | Their customers need AgentOS when the examiner asks about a specific loan refusal, not the model portfolio. |

---

## Regulatory Coverage

Each Decision Passport artifact names the specific controls it addresses.

| Regulation | Status | Controls closed |
|---|---|---|
| OCC Bulletin 2026-13 | In force | `model_version_hash` · `policy_version_hash` · `signing_attestation` · `principal_identity` |
| NYDFS Part 500 | Enforcement risk | `policy_version_hash` · `signing_attestation` · WORM storage |
| EU AI Act Article 12 | August 2, 2026 | `causality_chain_hash` · `full_prompt_snapshot` |
| NY RAISE Act | January 1, 2027 | `output_pre_postprocess` · `execution_result` |
| SOC2 CC6.1 / CC4.1 | Controls addressed | `authorization_token_id` · `execution_result` |
| GDPR Article 22 | In force | `principal_identity` · `policy_version_hash` |

---

## About this mapping

This mapping was produced by [Swapan Shridhar](https://linkedin.com/in/shridharswapan), Founder and CEO of AgentOS Technologies, Inc.

The mapping distinguishes between Prevention and Recording per risk. Naming what each mode actually does is more honest than claiming prevention for everything. Most risks are Recording or Prevention + Recording.

The companion SP 800-53 Rev 5 mapping is at [`docs/sp800-53-mapping.md`](./sp800-53-mapping.md).

Contact: `swapan@getagentos.ai` · [getagentos.ai](https://getagentos.ai)

---

*Published May 23, 2026. Apache License, Version 2.0.*
