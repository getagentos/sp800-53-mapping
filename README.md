# NIST SP 800-53 Control Mapping for AI Agent Decision Evidence

A mapping between [NIST SP 800-53 Rev 5](https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final) and a cryptographic decision-evidence artifact format for AI agents. Offered as a stakeholder contribution to the NIST AI Agent Standards Initiative, the [COSAiS](https://csrc.nist.gov/projects/cosais) overlay project, and the [NCCoE AI Agent Identity and Authorization](https://www.nccoe.nist.gov/projects/software-and-ai-agent-identity-and-authorization) demonstration.

## Why this exists

I read most of the substantive submissions to [NIST docket NIST-2025-0035](https://www.regulations.gov/docket/NIST-2025-0035) when the comment period closed. The banking coalition (BPI, BITS, ABA), Anthropic, the Foundation for Defense of Democracies, the OpenID Foundation, IEEE-USA. The non-repudiation gap kept showing up. The banking coalition asked NIST for guidance on agents that are "attributable, auditable, and stoppable" and proposed practice guides. Anthropic framed the problem as a non-compromised-agent threat model. FDD called out parallel control gaps across AC, IA, AU, and SR.

Nobody proposed an artifact format.

The [NCCoE concept paper](https://www.nccoe.nist.gov/projects/software-and-ai-agent-identity-and-authorization) from February 2026 makes the same observation in different language. It names four agent control dimensions: identification, authorization, auditing, non-repudiation. The OpenID Foundation and the broader OAuth/SPIFFE/MCP ecosystem are actively building the first two. The third and fourth are the empty boxes.

This repository is my attempt at a concrete artifact-level proposal for those third and fourth dimensions. It maps SP 800-53 Rev 5 controls to a cryptographic decision-evidence artifact format. The mapping is technology-neutral in spirit; a reference implementation exists, but the controls close regardless of which implementation satisfies the property column.

## What you'll find here

[`docs/sp800-53-mapping.md`](docs/sp800-53-mapping.md) is the full mapping. Two tables.

The **primary mapping** covers the AU (Audit and Accountability) family in nineteen rows: AU-2, AU-3, AU-3(1), AU-3(2), AU-8, AU-9, AU-9(1), AU-9(3), AU-10, AU-10(1), AU-10(2), AU-10(3), AU-10(5), AU-11, AU-12, AU-12(1), AU-12(2), AU-14, and AU-16.

The **secondary mapping** covers nine controls across AC, IA, SR, and SI where the artifact format crosses into those families.

The headline finding sits in the AU-10 enhancements. AU-10 Non-repudiation, plus 10(1) Association of Identities, 10(2) Validate Binding, 10(3) Chain of Custody, and 10(5) Digital Signatures, describes what an AI agent decision-evidence artifact has to satisfy. The control objectives exist. Implementation guidance is what's missing. That is the gap this mapping proposes to close.

## How to use this mapping

COSAiS contributors will find the AU-family mapping directly applicable to the agentic AI use cases currently in scoping. Feedback to `overlays-securing-ai@list.nist.gov` is welcome.

The NCCoE AI Agent Identity and Authorization demonstration covers four dimensions. This mapping addresses the third and fourth (auditing and non-repudiation). The identification and authorization layers are covered by adjacent OAuth 2.0, OIDC, SPIFFE/SPIRE, SCIM, and MCP work. Combining an identity layer with this evidence layer in a joint demonstration is technically straightforward.

On the [Cyber AI Profile (NIST IR 8596)](https://csrc.nist.gov/pubs/ir/8596/iprd) side, the mapping serves as implementation guidance for the Protect and Respond functions of the CSF 2.0 Core. Protect via pre-execution binding. Respond via the audit trail at incident time.

Regulated financial institutions can apply the mapping under SR 11-7, FFIEC Authentication Guidance, OCC 12 CFR Part 30 Appendix D, NYDFS Part 500, and the NY RAISE Act (effective January 2027). Adverse-action documentation under Regulation B and Regulation Z benefits from the same evidence.

Security architects doing vendor risk review: hand the mapping to your team. It gives them a concrete starting point instead of a blank page.

## Related standards work

The IETF SCITT working group (Supply Chain Integrity, Transparency, and Trust) reached working-group last call on `draft-ietf-scitt-architecture` in October 2025. The decision-evidence artifact format described here is profile-compatible with SCITT Signed Statements. An Internet-Draft profiling the schema is in preparation. <https://datatracker.ietf.org/wg/scitt/>

The IETF RATS working group (Remote Attestation Procedures) supplies the architectural vocabulary: Attester, Verifier, Relying Party, Evidence, Attestation Result. See RFC 9334 (Architecture, January 2023) and RFC 9782 (Entity Attestation Token Media Types). <https://datatracker.ietf.org/wg/rats/>

Other relevant NIST work: NIST IR 8596 Preliminary Draft (Cybersecurity Framework Profile for Artificial Intelligence, December 2025); NCCoE Concept Paper (Accelerating the Adoption of Software and AI Agent Identity and Authorization, February 5, 2026); NIST CAISI AI Agent Standards Initiative (announced February 17, 2026).

## About the author

I'm Swapan Shridhar, founder of AgentOS. I've spent nineteen years in infrastructure engineering, mostly on data and platform infrastructure.

The work most relevant to this mapping was delivering a FedRAMP-compliant Kubernetes platform on GovCloud, on both the control-plane and the workload side. I led the engineering team, owned the technical design, and worked closely with our FedRAMP program counterparts on the engineering implementation of NIST 800-53 controls. The platform had to satisfy federal compliance requirements end to end. I also led FIPS 140-3 compliant image and process work, and spearheaded Chainguard adoption for supply chain hardening.

The delivery cadence was monthly. Each release had to ship with all critical and high CVEs resolved against hard deadlines. Early on, we switched the TLS stack to BoringSSL to meet FedRAMP's FIPS requirements. That was one of the first changes that made the compliance requirement real at the code level, not just a checklist item. Over time, FIPS compliance extended well beyond crypto. The logging framework, monitoring, networking, all of it had to be FIPS-compliant. By the later iterations we were operating under FIPS 140-3.

Before that I was a Project Management Committee member on Apache Ambari for several years (2015 through 2018). Before Ambari, I spent years in the VMware ESX storage stack implementing T10 DIF/DIX, the standard that proves data was not silently corrupted between host and storage device. Same integrity problem as a Decision Passport, one layer down. Earlier work at HP included file system internals on Tru64 UNIX and backup and restore on OpenVMS using tape libraries. Tape is write-once by design. S3 Object Lock enforces the same property, different medium.

The recommendations in this mapping reflect what I have learned about what evidence packages actually need to do to satisfy a reviewer, an examiner, or a court, applied to the AI agent use case. Federal compliance frameworks have already solved the precise evidentiary questions agentic AI now faces. The primitives are in NIST's stewardship. The missing piece is a concrete format that combines them for agent decisions.

Reach me at `swapan@getagentos.ai`. <https://getagentos.ai>

## Status and feedback

This mapping is open and freely available. It is neither endorsed by NIST nor a substitute for official guidance. I expect it to iterate as the COSAiS overlays, the Cyber AI Profile, and the NCCoE demonstration mature.

Feedback channels:

- COSAiS mailing list: `overlays-securing-ai@list.nist.gov`
- Direct email: `swapan@getagentos.ai`
- Pull requests and issues on this repository

## Citation

See [`CITATION.cff`](CITATION.cff) for citation metadata in standard CFF format.

## License

Apache License, Version 2.0. See [`LICENSE`](LICENSE).
