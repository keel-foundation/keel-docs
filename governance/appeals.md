# Appeals

Every enforcement action is appealable. This is the public summary of the appeal process.

## Timelines

| Stage | Deadline |
|---|---|
| Acknowledgment | 24 hours |
| Temporary hold review | 72 hours |
| Channel-level appeal decision | 7 days |
| Complex appeal decision | 14 days |
| Second appeal on new evidence or procedural error | 21 days |
| Ombudsperson complaint acknowledgment | 5 days |
| Ombudsperson report | 60 days |

## Process

1. The creator receives a Statement of Reasons with the policy clause, evidence, decision-maker, and appeal deadline.
2. The creator files an appeal with grounds and a statement.
3. A reviewer independent of the original decision reviews the full record.
4. The reviewer issues a written decision that addresses the evidence.
5. The decision and reasoning are recorded in the audit ledger.
6. A second appeal is available on new evidence or a procedural error.

## Grounds

- `new_evidence`
- `procedural_error`
- `policy_misapplication`
- `identity`
- `other`

## Current demo

The local demo implements the action, appeal, independent review, reversal, and ledger record. It uses a synthetic signal and a fixed demo reviewer. The production appeals service, SLA timers, reviewer assignment, and second-level escalation are backlog work.

## Independent oversight

The Constitution provides for an Appeals Council and an Ombudsperson. They are not yet seated. Until they are, appeal decisions are made by the interim review process described in Article XV.
