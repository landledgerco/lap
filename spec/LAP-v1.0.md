# LAND Protocol — Specification v1.0

**Status:** Published  
**Published:** April 2026  
**Steward:** Land Ledger (Founding Steward)  
**License:** CC0 1.0 Universal (public domain)  
**Repository:** https://github.com/landledgerco/LAND

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Scope](#2-scope)
3. [Normative Language](#3-normative-language)
4. [Section Status Values](#4-section-status-values)
5. [Five Core Data Sections](#5-five-core-data-sections)
   - 5.1 [Identity](#51-identity)
   - 5.2 [Quality](#52-quality)
   - 5.3 [Valuation](#53-valuation)
   - 5.4 [Legal](#54-legal)
   - 5.5 [Compliance](#55-compliance)
   - 5.6 [Ownership (Optional, v1.1)](#56-ownership-optional-v11)
6. [Hash Construction Algorithm](#6-hash-construction-algorithm)
7. [Root Data Object](#7-root-data-object)
8. [Document Fingerprinting](#8-document-fingerprinting)
9. [Protocol Mechanics](#9-protocol-mechanics)
10. [Compliance Tiers](#10-compliance-tiers)
11. [Attestation Envelope](#11-attestation-envelope)
12. [Verification API Shape](#12-verification-api-shape)
13. [Discovery Endpoint](#13-discovery-endpoint)
14. [Governance and LIPs](#14-governance-and-lips)
15. [Versioning](#15-versioning)
16. [License](#16-license)

---

## 1. Introduction

The LAND Protocol (Ledger-Anchored Notarized Deeds) is an open standard for recording, hashing, and anchoring land asset data onchain. It defines a five-section data schema, a deterministic SHA-256 hash construction algorithm, a public verification API shape, and three progressive compliance tiers that land tokenization platforms can implement and disclose.

LAND Protocol was designed around a single premise: a land record should be cryptographically provable without trusting any single company. Anyone with a root hash and access to a public blockchain explorer can independently confirm that a record has not changed since it was anchored, without creating an account or relying on a proprietary API.

Land Ledger published LAND v1.0 in April 2026 and operates as the reference implementation. The specification is released under CC0 (public domain). Any platform may implement it, fork it, or build tooling around it without restriction.

---

## 2. Scope

LAND v1.0 is scoped to US agricultural and rural land. The five-section schema is designed for farmland, timberland, and similar parcels regardless of the legal entity structure used to hold title (SPV, LLC, trust, corporation, or individual).

Future versions may extend to urban, commercial, or international land. The standard is designed to grow alongside the network of implementers, data providers, and institutions who adopt it.

---

## 3. Normative Language

The key words MUST, MUST NOT, SHOULD, SHOULD NOT, and MAY in this document are to be interpreted as described in RFC 2119.

- **MUST**: Absolute requirement. Non-conformant if absent.
- **SHOULD**: Recommended but not strictly required.
- **MAY**: Optional. Permitted but not required.

---

## 4. Section Status Values

Each data section carries one of three status values:

| Status | Meaning |
|---|---|
| `NOT_CAPTURED` | Data has not yet been collected for this section. The section hash is null. |
| `CAPTURED` | Data has been entered and committed. The section hash is computed and stored. |
| `ATTESTED` | Data has been independently verified by a licensed third party (appraiser, surveyor, title company, or government source) and the attestation has been recorded. |

For a record to qualify as LAND Anchored, all five core sections MUST reach CAPTURED or ATTESTED status.

---

## 5. Five Core Data Sections

Every LAND Protocol record is organized into five core sections. Each section is independently serialized and hashed with SHA-256. See [Section 6](#6-hash-construction-algorithm) for the exact canonicalization procedure.

### 5.1 Identity

The Identity section establishes the legal and geographic identity of the parcel. It MUST unambiguously locate the parcel in public land records.

**Required fields (MUST be present):**

| Field | Type | Description |
|---|---|---|
| `parcelId` | string | Assessor Parcel Number (APN) or equivalent unique parcel identifier. |
| `legalDescription` | string | Legal description of the parcel as it appears in public land records. |
| `county` | string | County where the parcel is located. |
| `state` | string | US state abbreviation (e.g., `"IA"`, `"IL"`). |
| `acreage` | string | Total acreage of the parcel, to the nearest tenth of an acre. |

**Optional fields (SHOULD or MAY be present):**

| Field | Type | Description |
|---|---|---|
| `latitude` | string | Decimal latitude of parcel centroid. |
| `longitude` | string | Decimal longitude of parcel centroid. |
| `townshipRangeSection` | string | PLSS description (e.g., `"T88N R12W S14"`). |
| `propertyAddress` | string | Street address if assigned. Rural parcels may omit. |

**Normative requirements:**

- Identity MUST include either a legal parcel description or an APN. At least one of the two must be present.
- Identity MUST include state, county, and acreage.
- Identity MUST include the name of the entity or individual that holds title (SPV, LLC, trust, corporation, or individual owner). This is captured as the holding entity name within `parcelId` context or a dedicated `holdingEntity` field in implementation.

### 5.2 Quality

The Quality section captures the productive characteristics of the land.

**Required fields (MUST be present for agricultural land):**

| Field | Type | Description |
|---|---|---|
| Agricultural use designation | string | One of: `cropland`, `pasture`, `timberland`, `CRP`, `fallow`, `non-agricultural`. |
| Productivity score | number | At least one of: CSR2 (Iowa/Midwest) or NCCPI (NRCS national index). |

**Optional fields:**

| Field | Type | Description |
|---|---|---|
| `soilTypes` | string[] | List of soil series names present on the parcel. |
| `productivityIndex` | number | Composite productivity index. |
| `csr2Score` | number | Iowa Corn Suitability Rating 2 weighted average. |
| `nccpiScore` | number | NRCS National Commodity Crop Productivity Index. |
| `drainageClass` | string | e.g., `"well drained"`, `"poorly drained"`. |
| `tileDrainageInstalled` | boolean | Whether subsurface tile drainage is present. |
| `cropHistory` | string | e.g., `"corn/soybean rotation"`. |
| `topography` | string | e.g., `"level to gently rolling"`. |
| `irrigationRights` | boolean | Whether irrigation rights are attached to the parcel. |
| `waterRightsDescription` | string | Description of water rights, if any. |

**Normative requirements:**

- Quality MUST include an agricultural use designation.
- For agricultural land, Quality MUST include at least one soil productivity score (CSR2 or NCCPI).
- Non-agricultural land must still include the Quality section but MAY set score fields to null with an explicit `non-agricultural` designation.

### 5.3 Valuation

The Valuation section establishes the property's market value as determined by a licensed appraiser.

**Required fields (MUST be present):**

| Field | Type | Description |
|---|---|---|
| `appraisedValue` | string | Appraised market value in USD (stored as string to avoid float precision issues). |
| `appraisalDate` | string | ISO 8601 date of the appraisal (e.g., `"2026-03-15"`). |
| `appraiserName` | string | Full legal name of the licensed appraiser. |
| `appraiserLicense` | string | Appraiser's state license number. |

**Optional fields:**

| Field | Type | Description |
|---|---|---|
| `appraisalMethod` | string | e.g., `"sales comparison"`, `"income approach"`, `"cost approach"`. |
| `appraiserState` | string | State in which the appraiser is licensed. |
| `ltvApplied` | number | Loan-to-value ratio applied by the lending platform (0.0 to 1.0). |

**Normative requirements:**

- Valuation MUST include the appraised value in USD, the appraisal date, and the appraiser's name and license number.
- Valuation SHOULD include at least 3 comparable sales to support the value opinion (surfaced as a count in `recordCounts.comparableSales`).
- Valuation SHOULD include a government assessed value cross-reference (APN, assessed value, and tax year).

### 5.4 Legal

The Legal section is where LAND Protocol establishes its most significant distinction from self-reported data systems. Lien disclosure is non-negotiable.

**Required fields (MUST be present):**

| Field | Type | Description |
|---|---|---|
| `liens` | LienRecord[] | Array of lien records. An empty array `[]` is valid and means no active liens. Omission is non-conformant. |
| `deedReference` | DeedRecordingReference | At least one deed recording reference with instrument number, recording date, county, and state. |
| Title chain | TitleChainEntry[] | At least 1 entry with grantor, grantee, instrument type, and recording date. |

**LienRecord fields:**

| Field | Type | Description |
|---|---|---|
| `lienHolder` | string | Name of the entity holding the lien. |
| `lienType` | string | One of: `mortgage`, `tax_lien`, `mechanic_lien`, `judgment`, `other`. |
| `amount` | string | Lien amount in USD (optional). |
| `recordingNumber` | string | County recorder instrument number (optional). |
| `recordingDate` | string | ISO 8601 date of recording (optional). |
| `priority` | string | One of: `senior`, `subordinate`. |
| `status` | string | One of: `active`, `released`, `subordinated`. |

**DeedRecordingReference fields:**

| Field | Type | Description |
|---|---|---|
| `instrumentNumber` | string | County recorder instrument number. Required. |
| `recordingDate` | string | ISO 8601 date of recording. Required. |
| `countyRecorder` | string | County recorder office name. Required. |
| `state` | string | State where recorded. Required. |
| `bookVolume` | string | Book/volume reference (optional). |
| `pageNumber` | string | Page number reference (optional). |

**Optional fields:**

| Field | Type | Description |
|---|---|---|
| `spvName` | string | Name of the SPV holding title. |
| `spvJurisdiction` | string | State of SPV formation. |
| `mortgageStatus` | string | One of: `none`, `paid_off`, `active`. |
| `recordingReference` | string | Free-text recording reference. |
| Easements | EasementRecord[] | Easement records with type, holder, and acres affected. |

**Normative requirements:**

- Legal MUST include lien disclosure. Zero active liens is a valid answer. Omission of the `liens` array (or the `totalLiens` count in the verification response) is non-conformant.
- Legal MUST include at least one deed reference with instrument number, recording date, county, and state.
- Legal MUST include a chain of title with at least 1 entry recording grantor, grantee, instrument type, and recording date.
- Legal SHOULD include a complete chain of title going back at least 20 years where public records permit.
- Legal MAY include easement records with type, holder, and acreage affected.

### 5.5 Compliance

The Compliance section captures regulatory and entity status.

**Required fields (MUST be present):**

| Field | Type | Description |
|---|---|---|
| `entityFormationStatus` | string | One of: `confirmed`, `pending`. |
| `kycStatus` | string | One of: `completed`, `pending`, `not-applicable`. |

**Optional fields:**

| Field | Type | Description |
|---|---|---|
| `taxStatus` | string | One of: `current`, `delinquent`, `unknown`. |
| `taxParcelNumber` | string | County tax parcel identifier. |
| `taxYear` | number | Tax year assessed. |
| `annualTaxAmount` | string | Annual property tax in USD. |
| `insuranceStatus` | string | One of: `active`, `lapsed`, `unknown`. |
| `environmentalFlags` | string[] | List of environmental flags, if any. |
| `agriculturalDesignation` | string | USDA/FSA designation (e.g., `cropland`, `pasture`). |
| `fsa_tract_number` | string | FSA tract number. |
| `fsa_farm_number` | string | FSA farm number. |
| `conservationEasement` | boolean | Whether a conservation easement is in place. |
| `conservationEasementDescription` | string | Description of conservation easement, if any. |

**Normative requirements:**

- Compliance MUST include entity formation status: `confirmed` or `pending`.
- Compliance MUST include KYC status for beneficial owners: `completed`, `pending`, or `not-applicable`.
- Compliance MAY include KYC provider name, regulatory jurisdiction, and document reference numbers.

### 5.6 Ownership (Optional, v1.1)

The Ownership section records the legal identity of the holding entity and is optional in v1.0. The Land Ledger reference implementation includes it as schema v1.1. Platforms implementing the base five-section schema remain LAND Protocol-conformant.

When present, the Ownership section is included in the root hash computation. Its inclusion improves audit depth and supports direct onchain ownership verification.

**Required fields (when section is present):**

| Field | Type | Description |
|---|---|---|
| `ownerName` | string | Legal name of the landowner or grantor. |
| `ownerType` | string | One of: `individual`, `trust`, `entity`. |
| `spvEntity` | SpvEntityData | SPV entity details (see below). |

**SpvEntityData fields:**

| Field | Type | Description |
|---|---|---|
| `entityName` | string | Legal name of the SPV (e.g., `"Smith Farm LLLP"`). Required. |
| `entityType` | string | e.g., `LLLP`, `LLC`, `LP`. Required. |
| `jurisdiction` | string | State of formation. Required. |
| `ein` | string | Last 4 digits of Federal EIN (for display/audit, not full disclosure). Optional. |
| `formationDate` | string | ISO 8601 date of entity formation. Optional. |
| `registeredAgent` | string | Registered agent name. Optional. |
| `spvOperatingAgreementHash` | string | SHA-256 of the operating agreement file bytes. Optional. |
| `einConfirmationHash` | string | SHA-256 of the EIN confirmation letter file bytes. Optional. |

**Optional fields:**

| Field | Type | Description |
|---|---|---|
| `deedGrantor` | string | Name as it appears on the deed. |
| `beneficialOwners` | string[] | Names of beneficial owners with >25% interest. |

---

## 6. Hash Construction Algorithm

The following algorithm MUST be followed exactly to produce a LAND Protocol-conformant root hash. Two implementations that follow this algorithm will always produce the same root hash from the same input data.

### Step 1: Read section data

Read all data sections from persistent storage: Identity, Quality, Valuation, Legal, Compliance, and (if present) Ownership.

### Step 2: Compute section hashes

For each captured section:

1. Take the section's data object.
2. Serialize it to a canonical JSON string using the following canonicalization rule:

   **Canonicalization rule:** Use `JSON.stringify` with a function replacer that sorts all object keys alphabetically at every nesting level. All native JSON.stringify semantics are preserved:
   - `undefined` object properties are omitted.
   - `undefined`, function, and symbol values in arrays become `null`.
   - Objects with a `toJSON` method (such as `Date`) are converted via that method before key-sorting.
   - No whitespace is added (`JSON.stringify` with a replacer produces compact output by default).

   **Reference implementation (JavaScript/TypeScript):**
   ```javascript
   function sortedKeyReplacer(key, value) {
     if (value !== null && typeof value === 'object' && !Array.isArray(value)) {
       const sorted = Object.entries(value).sort(([a], [b]) => a < b ? -1 : a > b ? 1 : 0);
       return Object.fromEntries(sorted);
     }
     return value;
   }

   const canonical = JSON.stringify(sectionData, sortedKeyReplacer);
   ```

3. Compute SHA-256 of the canonical string (UTF-8 encoded).
4. Encode the hash as lowercase hexadecimal. This is the **section hash**.

Sections not yet captured (status `NOT_CAPTURED`) have a `null` section hash.

### Step 3: Compute document fingerprints hash

Serialize all accepted and waived document fingerprints as a JSON array and compute SHA-256 using the same canonicalization rule.

Each fingerprint object includes: `docType`, `docHash`, `docStoragePointer`, `docEffectiveDate`, and `hashMethod`.

If no documents exist, `documentFingerprintsHash` is `null`.

### Step 4: Construct the root data object

Construct a root data object containing:

```json
{
  "schemaVersion": "1.1",
  "identity":    { "status": "<status>", "hash": "<section_hash_or_null>" },
  "quality":     { "status": "<status>", "hash": "<section_hash_or_null>" },
  "valuation":   { "status": "<status>", "hash": "<section_hash_or_null>" },
  "legal":       { "status": "<status>", "hash": "<section_hash_or_null>" },
  "compliance":  { "status": "<status>", "hash": "<section_hash_or_null>" },
  "ownership":   { "status": "<status>", "hash": "<section_hash_or_null>" },
  "documentFingerprintsHash": "<hash_or_null>",
  "timestamp": "<ISO_8601_UTC_string>"
}
```

The `timestamp` MUST be the ISO 8601 UTC string recorded as `rootHashComputedAt` in persistent storage. Including the timestamp in the root hash binds the commitment to a specific point in time, making it impossible for two anchoring events to produce the same root hash even if all section data is identical.

### Step 5: Compute the root hash

Apply the same canonicalization rule (sortedKeyReplacer) to the root data object, serialize to a canonical JSON string, and compute SHA-256. Encode as lowercase hexadecimal. This is the **root hash**.

### Step 6: Persist

Store the root hash and all section hashes with their statuses and timestamps in persistent storage.

### Step 7: Anchor

Submit a transaction to the `LandRecordRegistry` contract on the target chain, recording the root hash permanently onchain. Store the returned transaction hash and block number.

### Re-anchoring

Any change to a data field in any section will change that section's hash and generate a new root hash (which also generates a new timestamp). When this happens, the stored root hash will no longer match the most recent onchain anchor. Implementing platforms MUST clearly surface this divergence to users.

---

## 7. Root Data Object

The root data object structure (top-level keys in sorted order as the hash function sees them):

```json
{
  "compliance":              { "hash": "<hex_or_null>", "status": "CAPTURED" },
  "documentFingerprintsHash": "<hex_or_null>",
  "identity":                { "hash": "<hex_or_null>", "status": "CAPTURED" },
  "legal":                   { "hash": "<hex_or_null>", "status": "CAPTURED" },
  "ownership":               { "hash": "<hex_or_null>", "status": "CAPTURED" },
  "quality":                 { "hash": "<hex_or_null>", "status": "CAPTURED" },
  "schemaVersion":           "1.1",
  "timestamp":               "2026-04-01T00:00:00.000Z",
  "valuation":               { "hash": "<hex_or_null>", "status": "CAPTURED" }
}
```

Note: The keys above are shown in alphabetical order as produced by sortedKeyReplacer. This is the exact string fed to SHA-256.

---

## 8. Document Fingerprinting

Documents are anchored using two methods, tracked via the `hashMethod` field:

| Method | When used | How computed |
|---|---|---|
| `file_bytes` | Preferred. File content is available. | SHA-256 of the raw file bytes. |
| `metadata` | Fallback. File content unavailable. | SHA-256 of a canonical object containing `fileUrl`, `docType`, and `uploadedAt`. |

Each document fingerprint object:

```json
{
  "docType": "independent_appraisal",
  "docHash": "<sha256_hex>",
  "docStoragePointer": "<url_or_path>",
  "docEffectiveDate": "2026-03-15T00:00:00.000Z",
  "hashMethod": "file_bytes"
}
```

Implementing platforms SHOULD use `file_bytes` hashing whenever the file content is accessible. `metadata` hashing is a fallback for documents that cannot be re-read from storage.

---

## 9. Protocol Mechanics

The following requirements apply to every LAND Protocol record regardless of tier.

**MUST:**
- Every LAND Protocol record MUST include a `rootHash` field containing the SHA-256 root hash of all section hashes, encoded as lowercase hexadecimal.
- Every LAND Protocol record MUST include a `schemaVersion` field identifying the schema version used to construct the hash.
- Every LAND Protocol record MUST be anchored on at least one supported blockchain: Base (L2) or Ethereum Mainnet.
- Every LAND Protocol record MUST include an `anchoredAt` ISO 8601 timestamp recording when the root hash was first committed onchain.
- The `recordCounts.totalLiens` field MUST be present and numeric. A value of zero is valid but the field must not be omitted. Omitting lien disclosure is non-conformant.

**SHOULD:**
- LAND Protocol records SHOULD be promoted to Ethereum Mainnet after all data is finalized, for maximum long-term permanence.
- Implementing platforms SHOULD publish a `/.well-known/land-protocol` discovery endpoint at their root domain identifying which LAND Protocol version they implement.

**MAY:**
- LAND Protocol records MAY include a `frozen` boolean indicating the record is locked against further updates.

---

## 10. Compliance Tiers

A LAND Protocol record progresses through three tiers as more data is captured and verified. The tiers are additive: a record must satisfy all lower-tier requirements before qualifying for a higher tier.

### LAND Anchored

**Requirements:**
- All five core sections (Identity, Quality, Valuation, Legal, Compliance) at `CAPTURED` or `ATTESTED` status.
- Root hash computed and written to at least one supported blockchain (Base or Ethereum Mainnet).
- `schemaVersion` present.
- `anchoredAt` timestamp present.
- `recordCounts.totalLiens` present (may be zero).

### LAND Verified

**Requirements (all Anchored requirements, plus):**
- At least one deed reference recorded with instrument number, recording date, county, and state.
- Title chain depth of at least 1 entry (grantor, grantee, instrument type, and recording date).
- All active liens disclosed (individual lien records, not just a count).
- At least one independently verified third-party attestation (licensed appraiser, surveyor, or title company). See [Section 11](#11-attestation-envelope).

### LAND Certified

**Requirements (all Verified requirements, plus):**
- Government data bridge linked: county assessor APN, assessed value, and county record URL.
- At least 3 comparable sales recorded.
- An EAS onchain attestation UID or independently verified appraisal credential.

---

## 11. Attestation Envelope

Third-party attestations are credentials from independent professionals recorded outside the five-section hash envelope but part of the public LAND Protocol record. They verify the accuracy of data that was self-reported or admin-entered.

**Attestation types include:** appraiser, surveyor, title company, auditor, government.

**Attestation fields:**

| Field | Type | Description |
|---|---|---|
| `attestationType` | string | Type of attester (appraiser, surveyor, title_company, etc.). |
| `attesterName` | string | Full name of the attesting professional. |
| `attesterOrganization` | string | Organization or firm name. |
| `attesterAddress` | string | Ethereum address, if the attestation is onchain. |
| `attestationUid` | string | EAS attestation UID (0x...), if onchain via the Ethereum Attestation Service. |
| `schemaUid` | string | EAS schema UID defining the attestation structure. |
| `attestationDate` | string | ISO 8601 date the attestation was made. |
| `summary` | string | Human-readable summary of what was attested. |
| `verificationUrl` | string | EAS explorer URL or document link for independent verification. |
| `isOnchain` | boolean | Whether the attestation is recorded onchain via EAS. |
| `isVerified` | boolean | Whether the platform has verified the attestation. |

**Normative requirements:**

- Implementing platforms SHOULD include at least one third-party attestation for a record qualifying as LAND Verified.
- Implementing platforms SHOULD store an EAS attestation UID (verified via `attest.sh`) when an onchain attestation is available.
- Implementing platforms MAY surface attestation counts publicly; individual attester details are not required to be public.

---

## 12. Verification API Shape

Any LAND Protocol-conformant implementing platform MUST expose a public verification endpoint. The endpoint MUST be accessible without authentication.

**Endpoint:** `GET /api/verify/land/:id`

**Response shape:**

```json
{
  "landId": 1,
  "spvTokenName": "FARMLAND-001",
  "rootHash": "<hex>",
  "schemaVersion": "1.1",
  "anchoredAt": "2026-04-01T00:00:00.000Z",
  "lastUpdated": "2026-04-01T00:00:00.000Z",
  "frozen": false,
  "anchors": {
    "base": {
      "anchored": true,
      "transactionHash": "0x...",
      "blockNumber": 12345678
    },
    "mainnet": {
      "anchored": false,
      "transactionHash": null,
      "blockNumber": null,
      "anchoredAt": null,
      "contractAddress": null
    }
  },
  "sections": {
    "identity":   { "status": "CAPTURED",      "hash": "<hex>", "lastUpdated": "2026-04-01T00:00:00.000Z" },
    "quality":    { "status": "CAPTURED",      "hash": "<hex>", "lastUpdated": "2026-04-01T00:00:00.000Z" },
    "valuation":  { "status": "ATTESTED",      "hash": "<hex>", "lastUpdated": "2026-04-01T00:00:00.000Z" },
    "legal":      { "status": "CAPTURED",      "hash": "<hex>", "lastUpdated": "2026-04-01T00:00:00.000Z" },
    "compliance": { "status": "NOT_CAPTURED",  "hash": null,    "lastUpdated": null }
  },
  "recordCounts": {
    "activeLiens": 0,
    "totalLiens": 1,
    "titleChainEntries": 3,
    "deedReferences": 2,
    "easements": 1,
    "attestations": 2,
    "verifiedAttestations": 2,
    "comparableSales": 3,
    "hasGovernmentData": true
  },
  "summary": {
    "state": "IA",
    "county": "Black Hawk",
    "acreage": "160.0",
    "appraisedValue": "1280000",
    "tokenizedAt": "2026-04-01T00:00:00.000Z",
    "blockchainTokenId": "1"
  }
}
```

The machine-readable JSON Schema for this response is published at `/api/schema/land-record` on any conformant implementing platform.

---

## 13. Discovery Endpoint

Implementing platforms SHOULD publish a discovery document at `/.well-known/land-protocol` at their root domain.

**Response shape:**

```json
{
  "protocol": "land-asset-protocol",
  "version": "1.0",
  "implementer": "Land Ledger",
  "license": "CC0-1.0",
  "specUrl": "https://github.com/landledgerco/LAND",
  "schemaUrl": "https://landledger.co/api/schema/land-record",
  "verifyEndpoint": "https://landledger.co/api/verify/land/",
  "hashEndpoint": "https://landledger.co/api/verify/hash/"
}
```

---

## 14. Governance and LIPs

LAND Protocol is maintained as an open standard. Land Ledger serves as the Founding Steward and holds the initial editorial seat on the LAND Steering Committee.

Changes to the protocol are proposed as **LAND Improvement Proposals (LIPs)**. See [CONTRIBUTING.md](../CONTRIBUTING.md) for the full LIP template and lifecycle.

**LIP lifecycle stages:**

| Stage | Description |
|---|---|
| **Draft** | Author proposes a change. Open for public comment. |
| **Review** | Steering committee reviews. Community feedback incorporated. |
| **Final** | Accepted and incorporated into the next protocol version. |
| **Withdrawn** | Proposal withdrawn by author or rejected by committee. |

---

## 15. Versioning

LAND Protocol follows semantic versioning principles:

| Change type | Version bump | Examples |
|---|---|---|
| New optional field or optional section | Minor (1.0 to 1.1) | Adding the Ownership section |
| New required field | Major (1.x to 2.0) | Adding a required appraiser license field |
| Hash algorithm change | Major | Changing canonicalization or digest function |
| Compliance tier criteria change | Major | Changing Anchored/Verified/Certified requirements |

Backward compatibility: a record produced under schema v1.0 MUST still be verifiable under schema v1.1 tools if no new required fields were added in v1.1.

---

## 16. License

The LAND Protocol specification, schema, test vectors, and all normative text are released under the **Creative Commons CC0 1.0 Universal** license (public domain dedication).

Anyone may implement, copy, distribute, adapt, or build on this specification without restriction, without attribution, and without seeking permission.

The CC0 license does not apply to the Land Ledger platform software, brand, trademarks, or platform-specific implementations.

Full license text: https://creativecommons.org/publicdomain/zero/1.0/legalcode

