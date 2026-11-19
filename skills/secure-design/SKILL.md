---
name: secure-design
description: Design-time security - lightweight threat modeling (data-flow sketch, trust boundaries, STRIDE, abuse cases, ranked threats with responses) and secure design principles. Use before building or changing a feature that adds an endpoint, data store, integration, authentication or authorization, a new actor or a new trust boundary, and when reviewing a design for security.
---

# Secure Design - working rules

Answer "**what can go wrong here, and what do we do about it?**" before the code exists, when a
fix costs a line instead of a rewrite. Pair with `clean-architecture` (where code goes) and
`secure-coding` (how to write it safely).

**Which sections to read.** Sections tagged **[write]** when designing or building,
**[review]** when reviewing a design or a change. Section 7 (output format) always. A full model
is for a new system or a major change; for a small slice use the "threat model every story"
shortcut in section 6.

## 0. When to run it [write, review]
Run it when the change does any of these:
- adds an endpoint, queue consumer, scheduled job or admin function;
- stores or exposes personal, financial, credential or secret data;
- adds or changes login, sessions, roles, permissions, tokens or sharing;
- crosses a **trust boundary**: a new client type, a third-party service, a webhook, a file or URL
  from outside, a new network zone, a new tenant;
- moves money, changes limits or alters a workflow that has rules.
Skip it for refactors that change no behavior and no boundary, and for pure UI wording.

## 1. The four questions [write, review]
Keep them in this order. They are a loop, not a one-time gate; revisit when the design changes.
1. **What are we working on?** A small model of the system or the slice (section 2).
2. **What can go wrong?** Enumerate threats systematically (sections 3-4).
3. **What are we going to do about it?** A response for each threat (section 5).
4. **Did we do a good enough job?** Check the model matches reality, every threat has an agreed
   response, and each mitigation is testable (section 6).

## 2. Model the system [write, review]
Draw a **data-flow sketch**; text or Mermaid in the pull request is enough. Show:
- **external entities** (users, other systems), **processes** (your services and jobs), **data
  stores** (databases, caches, files, queues), **data flows** (arrows with what travels);
- **trust boundaries**: lines where the level of trust changes (browser to server, server to
  database, your service to a third party, one tenant to another, public network to internal);
- **entry points** (where outside data enters) and **assets** (what an attacker wants: data,
  money, accounts, availability, reputation).
Rules of thumb: one diagram per level of detail; model only the slice that changes; mark where
authentication and authorization happen; label flows that carry secrets or personal data. A model
that cannot be edited quickly will not be kept current, so keep it small. Several simple views
beat one perfect diagram.

## 3. Enumerate threats with STRIDE [write, review]
Walk **each element and each flow that crosses a trust boundary** and ask the six questions.
| Threat | Violates | Ask | Typical controls |
|---|---|---|---|
| **S**poofing | authentication | Can someone pretend to be a user, service or device? | strong authentication, MFA, signed tokens, mutual TLS, do not store shared secrets |
| **T**ampering | integrity | Can data be changed in transit, at rest, or by the client? | server-side validation, parameter binding, signatures/MACs, hashes, TLS, authorization on writes |
| **R**epudiation | accountability | Can someone deny an action, or erase the evidence? | audit log with who/what/when/outcome, protected and append-only; timestamps; signatures |
| **I**nformation disclosure | confidentiality | Can data leak to someone who should not see it (data, errors, logs, side channels)? | authorization, encryption in transit and at rest, minimal responses, generic errors, no secrets in logs |
| **D**enial of service | availability | Can someone exhaust CPU, memory, connections, storage, or lock others out? | rate limits, size/time limits, quotas, queue bounds, early rejection, no unauthenticated expensive work |
| **E**levation of privilege | authorization | Can a user do what only a higher role may, or reach another tenant's data? | least privilege, deny by default, object-level checks, separation of duties, no self-grant |
Also ask the "boundary" questions: What does this component assume about its input? What if the
assumption is false? What if the dependency is down, slow or hostile? What happens on failure
(open or closed)?

## 4. Add abuse cases and attack paths [write, review]
STRIDE finds technical threats; **abuse cases** find workflow and business-rule abuse. For each
user story ask "how would a malicious or careless user misuse this?":
- do it out of order, repeatedly, concurrently, or for someone else's object;
- send it with a missing, extra, negative, huge or odd value;
- combine two features (create a user, then grant that user rights; export then search);
- make the system call something it should not (a URL, a file, an internal service).
For a high-value goal (take over an account, move money, read all customer data) sketch a short
**attack tree**: the goal at the top, the ways to reach it below, and which of them the design
blocks. Use more than one viewpoint (outsider, ordinary user, malicious insider, compromised
dependency); do not over-focus on one adversary or one technique.

## 5. Decide the response for every threat [write, review]
Rank first, then respond.
- **Rank** with a simple qualitative High/Medium/Low for likelihood and for impact (ease of
  exploitation: remote? authenticated? automatable? Impact: takeover, data breach, money, outage).
  Fix effort is a third input. Skip precise scoring; arguments about numbers waste time.
- **Respond** to each threat with one of:
  - **Eliminate** - remove the feature or data (the safest control is the one that is not there).
  - **Mitigate** - add a control; say which and where it is enforced.
  - **Transfer** - someone else carries it (a provider, insurer, customer). This does not reduce the
    risk: name the owner and the remaining controls.
  - **Accept** - record who accepted it and why.
- Turn each mitigation into a **requirement and a test** ("a user cannot read another tenant's
  invoice" with a test that tries it). A mitigation that cannot be tested is a hope.
- Record the **residual risk**: not mitigated, partly mitigated, or fully mitigated.
- Do not stop at "analysis": a threat list without responses is not a result.

## 6. Check your work and keep it current [write, review]
- Does the sketch match what is really built (or will be)? Is every flow across a boundary
  covered? Does every threat have an owner and a response? Are the mitigations testable, and are
  the tests written? Is the model stored with the code?
- **Threat model every story** (a quick pass when a full model is too heavy): list the entry
  points and flows the change touches, run STRIDE over only those, record High/Medium/Low and a
  status per threat. Update the stored model in the same pull request as the code.
- Typical failures: one person owning the whole model ("hero modeler"), a perfect diagram that
  is never updated, analysis without fixes, one favorite threat, modeling the whole system when only a
  slice changed.

## 7. Output format [write, review]
Write the result where the team will see it: a section in the pull request, a design doc, or a
`threat-model.md` next to the code. Use `references/threat-model-template.md`. Keep it short: a
slice model, a table of threats with rank, response, status and test, plus assumptions and
open questions. When you cannot ask the user, list your assumptions first.

## 8. Design principles [write, review]
- **Least privilege.** Give every user, service, token and database account the minimum it
  needs, for the minimum time. Separate roles (reader, writer, admin).
- **Deny by default; fail safe.** New things are closed until opened; when a check fails or
  errors, the answer is no.
- **Complete mediation.** Check authorization on **every** access, not once at the start; one
  central mechanism, not ad hoc checks.
- **Defense in depth.** Independent layers (network, authentication, authorization, validation,
  least-privilege storage, monitoring) so that one failure is not enough.
- **Minimize the attack surface.** Fewer endpoints, parameters, roles, features, dependencies and
  exposed ports. Remove what is not needed.
- **Economy of mechanism.** A simple design is easier to reason about and verify. Reuse
  well-tested components; do not invent crypto, session handling or parsers.
- **Separation of duties.** For high-risk actions require two parties (or two steps), so no
  single account can both create and approve.
- **Never trust the client or the network.** The server verifies everything; internal traffic is
  authenticated too.
- **Open design.** Security must hold when the attacker knows the design; secrets are keys and
  credentials, not the algorithm or a hidden URL.
- **Make security usable and the secure way the easy way.** Safe defaults, one obvious API.
- **Plan for failure and recovery.** Detect (logging, alerting), contain (limits, segmentation),
  recover (backups, revocation, rotation, incident contacts).

## 9. Secure by design in the domain model [write]
Make misuse hard to express in the code itself:
- **Value objects with built-in validation.** Model `EmailAddress`, `Money`, `Quantity`,
  `AccountId` as types that cannot be constructed with an invalid value (create through a
  validating constructor or factory, immutable afterwards). Then every function that accepts one
  is safe from bad input by construction.
- **Make illegal states unrepresentable.** Use enums and types for states and transitions
  instead of free strings and booleans; put the transition rules in one place.
- **Read-once and masked data.** Types for secrets and tokens that print masked, cannot be
  serialized into logs by accident, and are used once.
- **Immutability** for shared and security-relevant data; copy before handing out.
- **Domain errors, not leaks.** Fail with a domain exception with context; the edge turns it
  into a generic message.
- **Limits in the domain.** Maximum sizes, counts and amounts are part of the types and rules, not
  only controller checks.
- **Trust the type, not the caller.** A function takes a `VerifiedToken` or an `AuthorizedUser`
  obtained from the verifying step, not a raw string and a hope.
- **Keep security rules in the domain layer**, separate from the web framework, so they are
  testable without HTTP (this follows the Dependency Rule from `clean-architecture`).

## 10. Quick design checklist [write, review]
1. Have I drawn the flows and trust boundaries for this change?
2. For each element and boundary flow, have I run STRIDE?
3. Who are the actors (including a malicious user and a compromised dependency)?
4. What does the server decide itself (identity, price, role, ownership)?
5. Where is authorization enforced, and can a new route bypass it?
6. What happens on failure: open or closed? What is logged?
7. What limits exist (size, rate, count, time) on anything an outsider can trigger?
8. Which data is sensitive, where does it rest and travel, who can read it?
9. Does every threat have a response, an owner and a test?
10. Is the model saved with the code and updated in this change?

## Reference files (load on demand)
| File | Open when |
|---|---|
| `references/threat-model-template.md` | Writing the threat model for a change or a design review |
