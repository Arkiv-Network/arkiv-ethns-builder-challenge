# Reference Pattern — ZK Proof of Attribute

A starter pattern for the **Privacy** theme — prove a fact about yourself (e.g., "I'm over 18", "I have a verified GitHub contribution count above 100", "I'm a registered DAO member") **without revealing the underlying data**.

This is the harder of the two Privacy reference patterns. It pairs ZK circuits with Arkiv entities — Arkiv stores the proofs (and optionally the public commitments / nullifiers); the actual sensitive data never touches the chain.

> Code samples are illustrative. Confirm exact SDK API against the [Arkiv TypeScript SDK docs](https://arkiv.network/getting-started/typescript). For the ZK side, this pattern uses [Noir](https://noir-lang.org/) — but [circom + snarkjs](https://github.com/iden3/snarkjs) work equally well; adapt to the toolchain you know.

---

## Conceptual model

Three actors:

- **Issuer** — verifies the underlying fact and issues a credential (e.g., a government, a GitHub-attestation service, a DAO admin)
- **Prover** — the user, who holds the credential privately and wants to demonstrate something about it
- **Verifier** — anyone who wants to confirm the prover's claim

Three Arkiv entity types:

### `IssuerKey`
Public-read. The issuer's verification key (the public key of the credential signing scheme). Verifiers fetch this to validate proofs.

| Field | Type | Notes |
|------|------|-------|
| `issuerName` | string | Human-readable issuer label |
| `pubkey` | bytes | Public key for verifying credential signatures |
| `circuitHash` | bytes | Hash of the verification circuit this issuer issues for |
| `expiration` | timestamp | Long — issuers are long-lived, but rotate keys eventually |

### `AttributeProof`
A ZK proof that the prover holds a credential satisfying some statement. **No PII**.

| Field | Type | Notes |
|------|------|-------|
| `issuerKeyId` | reference → IssuerKey | Which issuer's credential is proven |
| `claim` | string | Public — `over-18`, `gh-contributions-above-100`, `dao-member` |
| `proof` | bytes | The ZK proof itself |
| `publicInputs` | object | Public inputs to the circuit (e.g., the threshold value) |
| `nullifier` | bytes (optional) | Prevents reuse, if your scheme requires single-use proofs |
| `prover` | wallet address | Optional — the wallet who submitted the proof |
| `expiration` | timestamp | Short for one-time use, longer for "still over 18" claims |

### `Nullifier`
Optional. Records used nullifiers to prevent proof reuse for single-shot claims.

| Field | Type | Notes |
|------|------|-------|
| `value` | bytes | The nullifier hash |
| `claim` | string | What claim it was used for |
| `expiration` | timestamp | Matches the validity window of the claim |

---

## Minimal flow

```text
ISSUER (offline)
  1. Verify the underlying fact (e.g., user is over 18).
  2. Issue a signed credential to the user (signed by the IssuerKey).
  3. Publish IssuerKey to Arkiv.

PROVER (client-side)
  4. Use a circuit (Noir / circom) that proves:
       - "I hold a credential signed by IssuerKey"
       - "The credential's `birthYear` field, when subtracted from today, is >= 18"
     without revealing birthYear.
  5. Publish AttributeProof to Arkiv with the circuit's proof + public inputs.

VERIFIER
  6. Fetch IssuerKey from Arkiv.
  7. Fetch AttributeProof from Arkiv.
  8. Run the verification circuit's verifier function on (proof, publicInputs, IssuerKey.pubkey).
  9. Optionally check Nullifier hasn't been used.
```

---

## Code outline (TypeScript, Noir-based)

```typescript
import { Arkiv } from "@arkiv-network/sdk";
import { Noir } from "@noir-lang/noir_js";
import circuit from "./over18.json";

const arkiv = new Arkiv({ rpc: "https://kaolin.hoodi.arkiv.network/rpc" });

// 1. Prover generates the proof from their private credential
async function proveOver18(privateCredential: Credential): Promise<{ proof: Uint8Array; publicInputs: any }> {
  const noir = new Noir(circuit);
  const inputs = {
    credential: privateCredential.payload,
    signature: privateCredential.signature,
    todayYear: new Date().getFullYear(),
  };
  return noir.execute(inputs);
}

// 2. Prover submits the proof to Arkiv (only public bits)
async function submitProof(issuerKeyId: string, claim: string, proof: Uint8Array, publicInputs: any) {
  return arkiv.entities.create({
    type: "AttributeProof",
    payload: { claim, proof, publicInputs },
    references: { issuerKeyId },
    expiration: addDays(new Date(), 30), // valid for 30 days
  });
}

// 3. Verifier checks the proof
async function verifyProof(proofId: string): Promise<boolean> {
  const proofEntity = await arkiv.entities.get({ type: "AttributeProof", id: proofId });
  const issuerKey = await arkiv.entities.get({ type: "IssuerKey", id: proofEntity.references.issuerKeyId });

  const noir = new Noir(circuit);
  const valid = await noir.verify(
    proofEntity.payload.proof,
    proofEntity.payload.publicInputs,
    issuerKey.payload.pubkey,
  );
  if (!valid) return false;

  if (proofEntity.payload.nullifier) {
    const used = await arkiv.entities.query({
      type: "Nullifier",
      filters: { "payload.value": proofEntity.payload.nullifier },
    });
    if (used.length > 0) return false;
    await arkiv.entities.create({
      type: "Nullifier",
      payload: { value: proofEntity.payload.nullifier, claim: proofEntity.payload.claim },
      expiration: proofEntity.expiration,
    });
  }

  return true;
}
```

---

## Demo flow

1. Issuer publishes an `IssuerKey` for an "age verification service" (Arkiv).
2. Prover (Alice) holds a private credential signed by Issuer (off-chain).
3. Alice runs the circuit locally → produces proof of "I am ≥ 18".
4. Alice publishes the `AttributeProof` to Arkiv. The proof contains zero information about her birthday.
5. Verifier fetches the proof + IssuerKey from Arkiv, runs verification, gets `true`.
6. Verifier confirms that no PII was revealed in the process.

---

## Where to go from here

- **Anonymous credential:** Prove "I have *some* credential from issuer X" without revealing which credential — a step toward anonymous attestations / DAO membership proofs.
- **Threshold proofs:** "I have at least N credentials from issuer X" or "I'm in the top P% by some metric".
- **Time-bounded claims:** AttributeProof expirations enforce that claims must be re-proven periodically (e.g., monthly age re-verification).
- **Selective revelation:** Reveal *some* fields, prove *others*. (Mix this pattern with [`privacy-encrypted-payload`](privacy-encrypted-payload.md) for hybrid disclosure.)

---

## Honest caveats

- ZK is a serious time investment. Most builders without prior ZK experience will not get further than a working over-18 demo in 9 days. **That's fine** — a clean implementation of one statement scores well.
- Pick a toolchain you can ship: Noir is the most beginner-friendly in 2026; circom is more mature; Halo2 / STARK toolchains are heavier.
- The proof goes on Arkiv. The witness (the secret data) **never** does. Double-check before you ship.

---

## Scoring tips for this theme

- A clean ZK round-trip (proof generation → Arkiv submit → independent verification) is the unlock for high marks on **Arkiv integration depth** + **Functionality** here.
- Differentiated expirations on `AttributeProof` vs. `Nullifier` vs. `IssuerKey` = high score on **Expiration dates**.
- Demonstrating that the wrong proof or expired proof gets rejected = high score on **Data integrity**.
- Anonymous-credential extension or time-bounded re-proving = high score on **Advanced features**.
