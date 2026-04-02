# Contributing to LAP

Thank you for your interest in improving the Land Asset Protocol. LAP is an open standard, and contributions from implementers, land professionals, legal experts, and technologists are what will make it better.

---

## How Changes Are Made

Changes to LAP are proposed through **LAP Improvement Proposals (LIPs)**. A LIP is a structured document that describes the rationale, scope, and technical detail of a proposed change to the protocol. All normative changes to the specification, schema, or hash algorithm must go through the LIP process.

Non-normative contributions (documentation improvements, typo fixes, new test vectors for existing algorithms) can be submitted as pull requests directly without a formal LIP.

---

## LIP Template

Copy the template below into a new file at `lips/LIP-XXXX.md` (replacing XXXX with the next available number).

```markdown
# LIP-XXXX: [Short title]

**Status:** Draft  
**Author:** [Your name or GitHub handle]  
**Created:** YYYY-MM-DD  
**Type:** [Spec / Schema / Hash Algorithm / Compliance Tiers / Process]  
**Version Impact:** [Minor (additive only) / Major (breaking change)]

---

## Summary

One paragraph summary of what this LIP changes and why.

---

## Motivation

Why is this change needed? What problem does it solve for implementers, landowners, lenders, or other participants?

---

## Specification

Precise description of the proposed change. For field additions, include the field name, type, whether it is required or optional, and a description. For algorithm changes, include the before and after behavior side by side.

---

## Backward Compatibility

Is this change backward compatible with existing LAP records? If not, explain what migration path implementers should follow.

---

## Test Vectors

If this LIP changes the hash algorithm or canonicalization, provide new test vectors following the format in `test-vectors/README.md`.

---

## Open Questions

Any unresolved issues or tradeoffs the committee should consider before accepting this LIP.

---

## References

Links to relevant standards, research, or prior art.
```

---

## LIP Lifecycle

| Stage | Description |
|---|---|
| **Draft** | Author opens a pull request with the LIP file. Open for public comment. Any party may comment in the PR. |
| **Review** | The LAP Steering Committee marks the LIP as under formal review. Steering Committee members provide structured feedback within 30 days. |
| **Final** | Steering Committee accepts the LIP. The changes are incorporated into the next protocol version. |
| **Withdrawn** | Author withdraws the LIP, or the Steering Committee rejects it with explanation. |

---

## LAP Steering Committee

The Steering Committee is responsible for reviewing LIPs, maintaining the specification, and managing version releases.

**Founding Steward:** Land Ledger LLC

Additional seats on the Steering Committee are available to organizations that implement LAP and commit to operating a publicly accessible verification endpoint for at least 12 months. To request a seat, open an issue with the subject line: "Steering Committee Seat Request: [Organization Name]".

---

## What Makes a Good LIP

**Additive changes are preferred.** A LIP that adds an optional field or optional section is much easier to accept than one that changes a required field or the hash algorithm. Optional additions are backward compatible and do not require implementers to re-anchor existing records.

**Provide test vectors.** Any change that touches the hash algorithm or the canonical serialization format must include updated test vectors. A LIP without test vectors cannot be accepted.

**Justify new required fields carefully.** Requiring a new field makes every existing LAP record non-conformant until implementers update their data. The bar for adding a required field is high.

**Engage before writing a full LIP.** Open an issue to discuss the idea first. This saves time for both you and the Steering Committee.

---

## Code of Conduct

All contributors are expected to engage in good faith, focus on the technical merits of proposals, and treat other contributors respectfully. The goal is the best possible open standard for the global land community.
