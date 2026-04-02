# Land Asset Protocol (LAP)

[![License: CC0-1.0](https://img.shields.io/badge/License-CC0_1.0-lightgrey.svg)](https://creativecommons.org/publicdomain/zero/1.0/)
[![Version](https://img.shields.io/badge/LAP-v1.0-green.svg)](spec/LAP-v1.0.md)

---

Land is the world's most enduring store of wealth. Most of it has never been liquid.

Across the world, families and communities hold their futures in land, yet that wealth remains locked behind fragmented records, opaque ownership structures, and capital markets that cannot see it clearly. We see a world where land liquidity unlocks generational prosperity, where a landowner can access the same capital markets as a publicly traded REIT, and where lenders can underwrite with the same confidence they bring to any transparent, verifiable asset class.

The Land Asset Protocol exists to make that world possible. When landowners, lenders, capital providers, and government entities can all read and verify the same onchain record, land stops being an illiquid footnote and starts becoming the foundation it was always meant to be.

LAP is the common language.

---

## What is LAP?

The **Land Asset Protocol** is an open standard for recording, hashing, and anchoring land asset data onchain. It defines:

- A **five-section data schema** covering Identity, Quality, Valuation, Legal, and Compliance
- A **deterministic SHA-256 hash construction algorithm** that produces a cryptographic commitment to all land record data
- A **public verification API shape** that any implementing platform exposes
- **Three progressive compliance tiers** (LAP Anchored, LAP Verified, LAP Certified) that platforms disclose

A land record conforming to LAP is cryptographically provable without trusting any single company. Anyone with a root hash and a public blockchain explorer can independently confirm that a record has not changed since it was anchored.

LAP v1.0 was published by Land Ledger in April 2026 as a CC0 public domain standard. The standard begins with US agricultural and rural land as the proving ground. Future versions will extend further.

---

## Who Benefits

| Participant | What LAP gives them |
|---|---|
| **Landowners** | A verifiable, tamper-evident record that unlocks access to transparent capital markets |
| **Lenders** | Standardized land records they can read onchain and underwrite against without opaque data rooms |
| **Capital Providers** | A consistent, open data standard for evaluating tokenized land exposure that anyone can audit |
| **Government Entities** | A public verification layer they can connect county assessor records and GIS data to |

---

## Five Data Sections

Every LAP record is organized into five core sections. Each section is independently serialized and hashed with SHA-256 to produce a section hash. The section hashes are combined with schema version, document fingerprints, and a timestamp into a single root hash.

| Section | Key Data |
|---|---|
| **Identity** | Parcel ID, legal description, county, state, acreage, holding entity |
| **Quality** | Soil productivity scores (CSR2/NCCPI), drainage, crop history, water rights |
| **Valuation** | Appraised value (USD), appraisal date, appraiser name and license number |
| **Legal** | Lien disclosure, deed references, chain of title, easements |
| **Compliance** | Entity formation status, KYC status, tax status, environmental flags |

The Land Ledger reference implementation also includes an optional **Ownership** section (schema v1.1) that records the holding entity, deed grantor, and beneficial owners.

---

## Compliance Tiers

LAP tiers are additive. A record must satisfy all lower-tier requirements before qualifying for a higher tier.

| Tier | Requirements |
|---|---|
| **LAP Anchored** | All five sections CAPTURED or ATTESTED. Root hash written to at least one supported blockchain (Base or Ethereum Mainnet). Schema version, anchor timestamp, and lien disclosure present. |
| **LAP Verified** | All Anchored requirements, plus: at least one deed reference with instrument number, recording date, county, and state; title chain depth of at least 1 entry; all active liens disclosed; at least one verified third-party attestation (appraiser, surveyor, or title company). |
| **LAP Certified** | All Verified requirements, plus: government data bridge linked (county assessor APN, assessed value, and county record URL); at least 3 comparable sales recorded; EAS onchain attestation UID or independently verified appraisal credential. |

---

## Implementer Quick-Start

Any platform can implement LAP. To get started:

**1. Check the discovery document (Land Ledger reference implementation):**
```sh
curl https://landledger.co/.well-known/lap-protocol
```

**2. Verify a record:**
```sh
curl https://landledger.co/api/verify/land/1
```

**3. Fetch the JSON Schema:**
```sh
curl https://landledger.co/api/schema/land-record
```

**4. Read the full spec:**

See [spec/LAP-v1.0.md](spec/LAP-v1.0.md) for the complete normative specification including all field definitions, the hash construction algorithm, and compliance tier requirements.

**5. Validate your implementation:**

Use the test vectors in [test-vectors/README.md](test-vectors/README.md) to verify your canonicalization and hashing produce the correct output.

---

## Repository Contents

```
README.md                   This file
spec/LAP-v1.0.md            Full normative specification
schemas/land-record-v1.1.json  JSON Schema (draft 2020-12)
test-vectors/README.md      Hash algorithm test vectors
CONTRIBUTING.md             LAP Improvement Proposal (LIP) process
LICENSE                     CC0 1.0 Universal (public domain)
```

---

## Reference Implementation

[Land Ledger](https://landledger.co) is the founding steward and reference implementation of LAP. The Land Ledger platform implements LAP v1.0 and exposes the verification API described in this specification.

---

## License

This specification, schema, test vectors, and all normative text are released under **CC0 1.0 Universal (public domain)**. Anyone may implement, copy, distribute, adapt, or build on this specification without restriction, without attribution, and without seeking permission. See [LICENSE](LICENSE).

The CC0 license does not apply to the Land Ledger platform software, brand, trademarks, or any platform-specific implementations.
