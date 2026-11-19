# Threat model template

Copy into the pull request, a design doc, or `threat-model.md`. Keep each section short; delete
what does not apply. A one-slice model should fit on one screen.

```markdown
# Threat model: <feature or change>
Date: <yyyy-mm-dd>   Owner: <name>   Reviewers: <names>   Version: <n>

## 1. Scope
What is changing, in two or three sentences. What is **out** of scope.

## 2. Data-flow sketch
<Mermaid or ASCII. Mark trust boundaries with a dashed box/line.>

    [Browser] --HTTPS--> |=== trust boundary ===| [API] --SQL--> [DB]
                                                  |--HTTP--> [Payment provider]

## 3. Entry points, assets, actors
| ID | Entry point | Who can reach it | Notes |
|---|---|---|---|
| E1 | POST /transfers | authenticated user | moves money |

| ID | Asset | Why it matters |
|---|---|---|
| A1 | Account balance | financial integrity |

Actors: anonymous visitor, ordinary user, user of another tenant, admin, a compromised
dependency, a malicious insider.

## 4. Assumptions
- <e.g. "TLS terminates at the load balancer", "admins are trusted", "the queue is private">

## 5. Threats
| ID | Element / flow | STRIDE | Threat (attacker does what) | Likelihood | Impact | Response | Control and where it lives | Status | Test |
|---|---|---|---|---|---|---|---|---|---|
| T1 | E1 | E | User transfers from another user's account by changing `from_id` | M | H | Mitigate | Owner-scoped account lookup in `TransferService` | Mitigated | `test_cannot_transfer_from_foreign_account` |
| T2 | E1 | T | Negative amount reverses the transfer direction | M | H | Mitigate | `Money` rejects amount <= 0 | Mitigated | `test_rejects_non_positive_amount` |
| T3 | E1 | D | Burst of requests exhausts DB connections | M | M | Mitigate | Per-user rate limit at gateway and in app | Partial | `test_rate_limit_returns_429` |
| T4 | Payment provider | I | Webhook forged to mark an order as paid | L | H | Mitigate | Signature check + replay window | Open | pending |

Likelihood/Impact: H / M / L. Response: Eliminate / Mitigate / Transfer / Accept.
Status: Mitigated / Partial / Open. For **Accept**, record who accepted and why. For
**Transfer**, record the owner and the remaining controls.

## 6. Abuse cases considered
- Out-of-order calls, repeat/replay, parallel submission, another user's ids, missing or extra
  fields, huge or negative numbers.

## 7. Open questions and follow-ups
- <question> - owner - due

## 8. Review checklist
- [ ] Diagram matches what is (to be) built
- [ ] Every flow across a boundary was walked with STRIDE
- [ ] Every threat has a response and an owner
- [ ] Every mitigation is testable and has a test (or a ticket)
- [ ] Model stored with the code and updated in this change
```

## Tips for doing it fast
- Model only the slice that changes; link to the system-level model if one exists.
- Prefer a concrete fix over a long analysis. One sentence per threat is enough.
- Run it as a short conversation with the people who know the feature; write down what you decide.
- A threat with a High impact and no response blocks the change; Medium and Low can become
  tickets with owners.
