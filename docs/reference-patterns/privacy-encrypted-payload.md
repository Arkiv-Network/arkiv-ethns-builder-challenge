# Reference Pattern — Encrypted Payload + Access List

A starter pattern for the **Privacy** theme. Use it as the on-ramp; depart from it as your build demands.

The goal: store sensitive data on Arkiv as encrypted entities, grant time-bounded decryption access to specific wallets, revoke (or auto-expire) access cleanly.

> Code samples are illustrative. Confirm exact SDK API against the [Arkiv TypeScript SDK docs](https://arkiv.network/getting-started/typescript) — version `@arkiv-network/sdk` v0.6.0 or newer.

---

## Encryption model

The pattern is **envelope encryption**:

1. Generate a fresh symmetric key (the *record key*) per record.
2. Encrypt the record's plaintext payload with the record key (e.g., AES-256-GCM).
3. Encrypt the record key once per grantee, using each grantee's public key (e.g., x25519 / ECIES).
4. Store the ciphertext payload as the `Record` entity.
5. Store each per-grantee encrypted-key + expiration as an `AccessGrant` entity referencing the `Record`.

**Why envelope:** rotating access for one grantee (revoke or grant) doesn't require re-encrypting the payload — just adding or removing an `AccessGrant`. To rotate the record key entirely (e.g., after a leak), generate a new key, re-encrypt the payload, and replace all grants.

---

## Entity schema

### `Record`

| Field | Type | Notes |
|------|------|-------|
| `recordType` | string | Public — `medical-note`, `bid`, `document`, ... |
| `payload` | bytes (ciphertext) | Encrypted with the record key |
| `nonce` | bytes | AES nonce / IV |
| `metadata` | object | Public, queryable (timestamps, tags) |
| `owner` | wallet address | Set automatically |
| `expiration` | timestamp | When the record itself expires |

### `AccessGrant`

| Field | Type | Notes |
|------|------|-------|
| `recordId` | reference → Record | Which record this grants access to |
| `grantee` | wallet address | Who can decrypt |
| `wrappedKey` | bytes | Record key, encrypted to grantee's public key |
| `grantedBy` | wallet address | Owner who issued the grant |
| `expiration` | timestamp | **Auto-revokes when this passes** |

To revoke before expiration: delete the `AccessGrant`. To grant: create a new one.

---

## Minimal code outline (TypeScript)

```typescript
import { Arkiv } from "@arkiv-network/sdk";
import { randomBytes, createCipheriv, createDecipheriv } from "crypto";
import { box, randomBytes as nacl_random } from "tweetnacl";

const arkiv = new Arkiv({ rpc: "https://kaolin.hoodi.arkiv.network/rpc" });

// 1. Owner creates an encrypted record
async function createRecord(plaintext: Buffer, recordType: string, expiresInDays: number) {
  const recordKey = randomBytes(32);
  const nonce = randomBytes(12);
  const cipher = createCipheriv("aes-256-gcm", recordKey, nonce);
  const ciphertext = Buffer.concat([cipher.update(plaintext), cipher.final()]);
  const tag = cipher.getAuthTag();

  const record = await arkiv.entities.create({
    type: "Record",
    payload: {
      recordType,
      payload: Buffer.concat([ciphertext, tag]),
      nonce,
      metadata: { createdAt: new Date().toISOString() },
    },
    expiration: addDays(new Date(), expiresInDays),
  });

  return { recordId: record.id, recordKey };
}

// 2. Owner grants access to another wallet
async function grantAccess(
  recordId: string,
  recordKey: Buffer,
  grantee: { wallet: string; pubkey: Uint8Array },
  expiresInDays: number,
  ownerKeypair: nacl.BoxKeyPair,
) {
  const ephemeralNonce = nacl_random(box.nonceLength);
  const wrappedKey = box(recordKey, ephemeralNonce, grantee.pubkey, ownerKeypair.secretKey);

  return arkiv.entities.create({
    type: "AccessGrant",
    payload: {
      grantee: grantee.wallet,
      wrappedKey: Buffer.concat([Buffer.from(ephemeralNonce), Buffer.from(wrappedKey)]),
      grantedBy: ownerKeypair.publicKey,
    },
    references: { recordId },
    expiration: addDays(new Date(), expiresInDays),
  });
}

// 3. Grantee decrypts
async function decryptRecord(recordId: string, granteeKeypair: nacl.BoxKeyPair) {
  const record = await arkiv.entities.get({ type: "Record", id: recordId });
  const grants = await arkiv.entities.query({
    type: "AccessGrant",
    filters: {
      "references.recordId": recordId,
      "payload.grantee": granteeKeypair.publicKeyAsWallet,
    },
  });
  if (grants.length === 0) throw new Error("No active access grant");

  const grant = grants[0]; // pick the active, non-expired one
  const recordKey = unwrapKey(grant.payload.wrappedKey, granteeKeypair.secretKey, grant.payload.grantedBy);

  const ciphertext = record.payload.payload.slice(0, -16);
  const tag = record.payload.payload.slice(-16);
  const decipher = createDecipheriv("aes-256-gcm", recordKey, record.payload.nonce);
  decipher.setAuthTag(tag);
  return Buffer.concat([decipher.update(ciphertext), decipher.final()]);
}

// 4. Owner revokes access
async function revokeAccess(grantId: string) {
  return arkiv.entities.delete({ type: "AccessGrant", id: grantId });
}
```

---

## Demo flow

To satisfy the minimum requirements:

1. Owner creates a `Record` containing some plaintext (e.g., `"Patient: J. Doe — medication X 200mg"`).
2. Owner grants access to Doctor B with a 30-day expiration.
3. Doctor B decrypts and reads the record.
4. Owner revokes access (delete the grant) — or, alternatively, wait for the grant to expire.
5. Doctor B attempts to decrypt again — fails. **Show this in the demo** (it's the part that proves the access control actually works).

---

## Where to go from here

- **Audit log:** Every successful decrypt creates an `AccessEvent` entity (grantee + record + timestamp), public-read. Owner sees who accessed what, when.
- **Selective field disclosure:** Encrypt only some fields of an entity, leave the rest public-queryable (e.g., `recordType`, `createdAt` public, body encrypted). Lets you query without decrypting.
- **Group access:** A `Group` entity holds member wallets; an `AccessGrant` references the group instead of a single grantee. Membership changes don't require re-issuing grants.
- **Auto-revocation cascade:** When a `Record` expires, all its `AccessGrant`s expire with it (matching expirations).

---

## Scoring tips for this theme

- Demonstrating revocation that *actually works* (failed decrypt after revoke) = high score on **Functionality / Data integrity**.
- Different expiration durations on `Record` vs. `AccessGrant` vs. `AccessEvent` = high score on **Expiration dates**.
- Selective disclosure or group-based access = high score on **Advanced features**.
- Production-ready key handling (no plaintext keys in client logs, no hard-coded keys) = high score on **Code quality**.
