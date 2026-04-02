# LAP Hash Algorithm Test Vectors

This directory contains canonical test vectors for the LAP v1.0 hash construction algorithm. Any implementation that produces these exact outputs from these exact inputs is correctly implementing the LAP canonicalization and hashing procedure.

---

## Algorithm Summary

LAP uses **SHA-256** as the hash function and **sorted-key JSON serialization** as the canonicalization function.

**Canonical serialization rule:** Use `JSON.stringify` with a function replacer that recursively sorts all object keys alphabetically. All native `JSON.stringify` semantics are preserved (undefined omission, Date.toJSON(), array null coercion). No whitespace is added.

**Reference implementation (JavaScript/TypeScript):**

```javascript
function sortedKeyReplacer(key, value) {
  if (value !== null && typeof value === 'object' && !Array.isArray(value)) {
    const sorted = Object.entries(value).sort(([a], [b]) => a < b ? -1 : a > b ? 1 : 0);
    return Object.fromEntries(sorted);
  }
  return value;
}

function computeHash(data) {
  const canonical = JSON.stringify(data, sortedKeyReplacer);
  return crypto.createHash('sha256').update(canonical).digest('hex');
}
```

**Python equivalent:**

```python
import json
import hashlib

def canonical(obj):
    return json.dumps(obj, sort_keys=True, separators=(',', ':'))

def compute_hash(obj):
    return hashlib.sha256(canonical(obj).encode('utf-8')).hexdigest()
```

Note: Python's `json.dumps(sort_keys=True, separators=(',', ':'))` is semantically equivalent to the JavaScript `sortedKeyReplacer` for inputs that contain only JSON-native types (strings, numbers, booleans, null, arrays, objects). Both produce the same output.

---

## Test Vector 1: Identity Section Hash

### Input

```json
{
  "acreage": 160,
  "county": "Black Hawk",
  "holdingEntity": "Green Acres LLC",
  "legalDescription": "NW1/4 Section 14, Township 88N, Range 12W",
  "state": "IA"
}
```

Note: The input object above uses unquoted numbers for acreage (`160`, not `"160"`). The sorted-key serializer sorts all five keys alphabetically: `acreage`, `county`, `holdingEntity`, `legalDescription`, `state`.

### Canonical JSON string (Step 2 output, fed to SHA-256)

```
{"acreage":160,"county":"Black Hawk","holdingEntity":"Green Acres LLC","legalDescription":"NW1/4 Section 14, Township 88N, Range 12W","state":"IA"}
```

### Identity section hash (SHA-256 of the string above)

```
9a46cba34c65bb55f9d5757653739d203c3725ae1fbc1ae2541ec2d832dab173
```

---

## Test Vector 2: Root Hash

This vector tests the full root hash computation using:
- Schema version: `1.1`
- Identity section: CAPTURED, hash from Test Vector 1
- All other sections: NOT_CAPTURED, hash null
- No document fingerprints
- Timestamp: `2026-04-01T00:00:00.000Z`

### Root data object (before canonicalization)

```json
{
  "schemaVersion": "1.1",
  "identity":    { "status": "CAPTURED",     "hash": "9a46cba34c65bb55f9d5757653739d203c3725ae1fbc1ae2541ec2d832dab173" },
  "quality":     { "status": "NOT_CAPTURED", "hash": null },
  "valuation":   { "status": "NOT_CAPTURED", "hash": null },
  "legal":       { "status": "NOT_CAPTURED", "hash": null },
  "compliance":  { "status": "NOT_CAPTURED", "hash": null },
  "ownership":   { "status": "NOT_CAPTURED", "hash": null },
  "documentFingerprintsHash": null,
  "timestamp": "2026-04-01T00:00:00.000Z"
}
```

### Canonical JSON string (Step 4 output, fed to SHA-256)

The root data object after applying `sortedKeyReplacer` (all keys sorted alphabetically at every level):

```
{"compliance":{"hash":null,"status":"NOT_CAPTURED"},"documentFingerprintsHash":null,"identity":{"hash":"9a46cba34c65bb55f9d5757653739d203c3725ae1fbc1ae2541ec2d832dab173","status":"CAPTURED"},"legal":{"hash":null,"status":"NOT_CAPTURED"},"ownership":{"hash":null,"status":"NOT_CAPTURED"},"quality":{"hash":null,"status":"NOT_CAPTURED"},"schemaVersion":"1.1","timestamp":"2026-04-01T00:00:00.000Z","valuation":{"hash":null,"status":"NOT_CAPTURED"}}
```

Note: The top-level keys appear in alphabetical order: `compliance`, `documentFingerprintsHash`, `identity`, `legal`, `ownership`, `quality`, `schemaVersion`, `timestamp`, `valuation`. The nested keys within each section object also appear in alphabetical order: `hash`, `status`.

### Root hash (SHA-256 of the canonical string above)

```
720c84c75e8f8a63e1ef5a597ae8b5730309f9b446e40a246d07dddbb9602e2f
```

---

## Verification Script (JavaScript/Node.js)

Save the following as `verify-test-vectors.mjs` and run with `node verify-test-vectors.mjs`:

```javascript
import crypto from 'crypto';

function sortedKeyReplacer(key, value) {
  if (value !== null && typeof value === 'object' && !Array.isArray(value)) {
    const sorted = Object.entries(value).sort(([a], [b]) => a < b ? -1 : a > b ? 1 : 0);
    return Object.fromEntries(sorted);
  }
  return value;
}

function computeHash(data) {
  const canonical = JSON.stringify(data, sortedKeyReplacer);
  return crypto.createHash('sha256').update(canonical).digest('hex');
}

// Test Vector 1: Identity section hash
const identityInput = {
  acreage: 160,
  county: 'Black Hawk',
  holdingEntity: 'Green Acres LLC',
  legalDescription: 'NW1/4 Section 14, Township 88N, Range 12W',
  state: 'IA',
};

const identityHash = computeHash(identityInput);
const expectedIdentityHash = '9a46cba34c65bb55f9d5757653739d203c3725ae1fbc1ae2541ec2d832dab173';

// Test Vector 2: Root hash
const rootData = {
  schemaVersion: '1.1',
  identity:   { status: 'CAPTURED',     hash: identityHash },
  quality:    { status: 'NOT_CAPTURED', hash: null },
  valuation:  { status: 'NOT_CAPTURED', hash: null },
  legal:      { status: 'NOT_CAPTURED', hash: null },
  compliance: { status: 'NOT_CAPTURED', hash: null },
  ownership:  { status: 'NOT_CAPTURED', hash: null },
  documentFingerprintsHash: null,
  timestamp: '2026-04-01T00:00:00.000Z',
};

const rootHash = computeHash(rootData);
const expectedRootHash = '720c84c75e8f8a63e1ef5a597ae8b5730309f9b446e40a246d07dddbb9602e2f';

// Report
const identityOk = identityHash === expectedIdentityHash;
const rootOk = rootHash === expectedRootHash;

console.log(`Identity hash: ${identityOk ? 'PASS' : 'FAIL'}`);
if (!identityOk) {
  console.log(`  Expected: ${expectedIdentityHash}`);
  console.log(`  Got:      ${identityHash}`);
}

console.log(`Root hash:     ${rootOk ? 'PASS' : 'FAIL'}`);
if (!rootOk) {
  console.log(`  Expected: ${expectedRootHash}`);
  console.log(`  Got:      ${rootHash}`);
}

if (!identityOk || !rootOk) process.exit(1);
console.log('\nAll test vectors passed.');
```

---

## Verification Script (Python)

```python
import json
import hashlib

def canonical(obj):
    return json.dumps(obj, sort_keys=True, separators=(',', ':'))

def compute_hash(obj):
    return hashlib.sha256(canonical(obj).encode('utf-8')).hexdigest()

# Test Vector 1
identity_input = {
    'acreage': 160,
    'county': 'Black Hawk',
    'holdingEntity': 'Green Acres LLC',
    'legalDescription': 'NW1/4 Section 14, Township 88N, Range 12W',
    'state': 'IA',
}

identity_hash = compute_hash(identity_input)
expected_identity = '9a46cba34c65bb55f9d5757653739d203c3725ae1fbc1ae2541ec2d832dab173'

# Test Vector 2
root_data = {
    'compliance':  {'hash': None, 'status': 'NOT_CAPTURED'},
    'documentFingerprintsHash': None,
    'identity':    {'hash': identity_hash, 'status': 'CAPTURED'},
    'legal':       {'hash': None, 'status': 'NOT_CAPTURED'},
    'ownership':   {'hash': None, 'status': 'NOT_CAPTURED'},
    'quality':     {'hash': None, 'status': 'NOT_CAPTURED'},
    'schemaVersion': '1.1',
    'timestamp': '2026-04-01T00:00:00.000Z',
    'valuation':   {'hash': None, 'status': 'NOT_CAPTURED'},
}

root_hash = compute_hash(root_data)
expected_root = '720c84c75e8f8a63e1ef5a597ae8b5730309f9b446e40a246d07dddbb9602e2f'

print(f"Identity hash: {'PASS' if identity_hash == expected_identity else 'FAIL'}")
print(f"Root hash:     {'PASS' if root_hash == expected_root else 'FAIL'}")
```

Note: The Python verification script passes the root data pre-sorted (alphabetical key order) because Python's `sort_keys=True` and the JavaScript `sortedKeyReplacer` produce the same canonical string when all values are JSON-native types. If your Python input data has unsorted keys, `sort_keys=True` will handle them correctly.

---

## Common Implementation Mistakes

| Mistake | Effect | Correct approach |
|---|---|---|
| Adding whitespace to JSON output | Hash mismatch | Use compact serialization (no spaces) |
| Sorting only top-level keys | Hash mismatch on nested objects | Sort keys at every nesting level recursively |
| Using a hand-rolled serializer that doesn't handle null | Root hash mismatch | Use native JSON.stringify or json.dumps with sort_keys |
| Using Date objects in section data without serializing first | Non-deterministic output | Serialize dates to ISO 8601 strings before hashing |
| Including undefined fields in the canonical string | Hash mismatch | Let JSON.stringify omit undefined fields naturally |

---

## Adding New Test Vectors

If you add a test vector via a LIP (see CONTRIBUTING.md), include:

1. The input data object, pretty-printed.
2. The canonical string (compact, no whitespace) fed to SHA-256.
3. The expected hash output (lowercase hex).
4. A verification script snippet in at least JavaScript and Python.

All test vectors must be independently reproducible using only a SHA-256 implementation and a JSON serializer. No LAP-specific libraries should be required to verify a test vector.
