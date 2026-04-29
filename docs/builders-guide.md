# Arkiv × ETHNS Builder Challenge — Builder's Guide

---

## What you're building

A **web3-native application** where all data lives on Arkiv. Users own their data — not the platform.

Pick one of three open themes:

| Theme       | The pitch                                                  | Concrete examples                                                                                       |
| ----------- | ---------------------------------------------------------- | ------------------------------------------------------------------------------------------------------- |
| **AI**      | Agents whose memory you actually own                       | Personal research assistant, coding-agent project context, MCP memory backend, agent reputation log    |
| **Privacy** | Confidential data patterns on a public, tamper-proof layer | Encrypted records with access control, anonymous attestations, sealed-bid auctions, selective disclosure |
| **DePIN**   | A queryable data layer for sensor / telemetry / device data | Proof-of-coverage map, solar tracker, air-quality network, fleet telemetry                             |

**Pick the one that excites you, or combine them.** All themes are scored on the same rubric. There's no advantage to choosing one over another.

---

## How Arkiv works (60-second mental model)

### 1. Entities = payload + typed attributes + `expiresIn`

Every Arkiv entity has:

- **Payload** — the actual data (JSON, text, or bytes). Stored as-is.
- **Attributes** — typed key-value pairs. **This is your index.** Filtering, sorting, and lookups happen against attributes, not against the payload.
- **Content type** — MIME type of the payload (e.g., `application/json`).
- **`expiresIn`** — a **duration in seconds** set at creation.

### 2. Attributes have types — pick the right one

- **String attributes** support equality and glob matching (`~`). Use for tags, statuses, names, identifiers.
- **Numeric attributes** support range queries (`gt`, `lt`, `gte`, `lte`). Use for any value you'll filter or sort by range — timestamps, scores, counts.

If you store `priority` as the string `"5"`, you lose range queries. Always store numerics as numbers.

### 3. Relationships are shared attribute keys

There is no built-in foreign-key field. To link entities, use a shared attribute key with the parent's entity key as the value (e.g., `{ key: "agentKey", value: agentEntityKey }`). Querying children of a parent is then a single `where(eq(...))`.

### 4. Two metadata fields matter: `$owner` and `$creator`

Every entity has both:

- **`$owner`** — the wallet that currently controls the entity. Mutable — ownership can be transferred. Only the owner can `updateEntity` / `deleteEntity` / `extendEntity`.
- **`$creator`** — the wallet that originally created the entity. **Immutable** — set at creation, never changes. Cannot be spoofed.

---

## Best Practice: PROJECT_ATTRIBUTE

**Arkiv is a shared, public database.** Every entity from every project lives in the same store. Without a project namespace, your queries return everyone else's data — and vice versa.

Every Arkiv project must:

1. Define a unique `PROJECT_ATTRIBUTE` constant in code (e.g., `lib/arkiv.ts`).
2. Stamp it on **every** create and update call.
3. Filter on it in **every** query.

```typescript
// lib/arkiv.ts
export const PROJECT_ATTRIBUTE = {
  key: "project",
  value: "myteam-agents-7x9k",  // globally unique to your project
} as const;
```

Pick a value unique enough to never collide (project name + suffix is fine). This is judged.

---

## Minimum Requirements (all themes)

Regardless of theme, your submission must:

### Technical baseline
- [ ] Define and use a unique `PROJECT_ATTRIBUTE` on every entity and every query
- [ ] At least 2 entity types
- [ ] Open source GitHub repo
- [ ] Working demo link
- [ ] README with setup instructions

---

## Theme 1: AI — Agents Whose Memory You Actually Own

*"Memory you own, portable across any tool that reads Arkiv."*

Most AI agents today store their memory in a vendor-locked vector DB or a local file. Switch agents, lose context. The opportunity here is to build agents whose memory lives on Arkiv — queryable by tags and time, wallet-owned, portable across any app that knows how to read Arkiv.

### Concrete builds (just to spark ideas)

- A personal research assistant that archives every paper it reads plus your notes, queryable months later by tag
- A coding agent whose project context is a shared Arkiv DB across teammates
- An [MCP server](https://modelcontextprotocol.io/) that hands any LLM a memory backend keyed to a user's wallet
- An agent that maintains a public reputation log of its own decisions
- A multi-agent system where agents read each other's public memory entities for coordination

These are examples, not a brief. Build something else if it fits the spirit of the theme.

### Things worth thinking through

- **What entity types do you actually need?** Most builds will have at least an *agent* and *memory items*, but you might also have decisions, attestations, sessions, or shared spaces.
- **How will memory get retrieved?** By tag, by time window, by importance, by embedding similarity, or some combination? That shapes which attributes you'll want indexed (and which need to be numeric for range queries).
- **Does all memory deserve the same lifespan?** Scratchpad context, working memory, and long-term beliefs probably don't expire on the same schedule. Differentiated expiration is one of the strongest signals you understand Arkiv.
- **Who owns what?** Is the agent identity tied to the user's wallet, or does the agent itself have a wallet? Does memory follow the agent, the user, or both?
- **Is it portable?** Can a different UI, model, or agent client use the same Arkiv-stored memory? If yes, demonstrate it.

### Directions to push it further

- Memory hierarchy with differentiated expirations
- Vector embeddings stored alongside text content for semantic retrieval
- Multi-agent shared memory with `$creator`-based attribution per agent
- MCP server wrapper exposing memory to any LLM client
- Public reputation log: every decision an agent makes recorded as a separate entity
- "Memory portability" demo: same Arkiv-stored memory used by two different agent UIs

---

## Theme 2: Privacy — Confidential Data Patterns

*"Encryption and access control on a public, tamper-proof layer."*

Arkiv is public-by-default and tamper-proof. Privacy on Arkiv isn't about hiding data inside the protocol — it's about what builders put *in* the entities and how access is gated. Encrypted payloads, ZK proofs over entity contents, anonymous attestations, selective-disclosure flows.

This is the highest-skill theme of the three. Pick something tractable and ship it cleanly.

### Concrete builds (just to spark ideas)

- Encrypted medical records where a patient grants a doctor 30-day access (auto-expires)
- Anonymous DAO membership rolls with public proofs of total members
- Sealed-bid auctions that publish only the winner
- Whistleblower-style document archives with verifiable timestamps and revealed metadata only
- Anonymous credentials: prove you have a verified GitHub contribution count without revealing your identity
- Group messaging with end-to-end encryption and a public message-history audit trail

### Things worth thinking through

- **Where does the privacy boundary actually sit?** Ciphertext on Arkiv? Off-chain witnesses with on-chain proofs? Group keys? Per-recipient key envelopes? Different choices have very different implementation budgets.
- **How is access granted, and how is it revoked?** Manual revocation, auto-expiration via Arkiv's `expiresIn`, key rotation? Auto-revocation that *just works* via expiration is rare in Web3 and is a strong feature to demonstrate.
- **What's the threat model?** Eavesdropper-only, malicious owner, malicious grantee, leaked keys? Building for one is honest; pretending to defend against all is not.
- **How do you prove things without revealing them?** ZK proofs, anonymous attestations, selective disclosure — pick one approach and make it work end-to-end rather than gesturing at three.
- **What's auditable and by whom?** Encrypted payloads + tamper-proof storage = a verifiable audit trail without leaking content. If your build can demonstrate that property, surface it.

### Directions to push it further

- ZK proof of attribute (e.g., over-18, GitHub contribution-count thresholds, DAO membership)
- Anonymous attestations
- Auto-revocation purely via expiration (no manual action needed)
- Sealed-bid reveal pattern (encrypted submissions, decrypt all on event end)
- Audit log of access events as separate entities
- Selective field disclosure (encrypt some fields, leave others public)

---

## Theme 3: DePIN — A Queryable Data Layer for the Physical World

*"Time-scoped, tamper-proof telemetry — owned by the operator."*

DePIN networks generate enormous time-series data: sensor readings, location pings, bandwidth measurements, energy production, environmental telemetry. Today most of it lands in centralised databases (defeating the point) or gets summarised to L1 (too expensive, not queryable). Arkiv is the missing data layer.

### Concrete builds (just to spark ideas)

- Proof-of-coverage map where every node ping is a verifiable Arkiv entity
- Solar-panel production tracker feeding tokenised energy markets
- Air-quality sensor network with a public queryable API
- Fleet-tracking system where operators retain sovereignty over their telemetry
- Bandwidth measurement network with reward calculation pulling from Arkiv
- Decentralised weather-station network with verifiable readings

### Things worth thinking through

- **What does a "device" mean in your build?** Real hardware, simulated, or a hybrid? You don't need real sensors to ship something credible — but the demo is more compelling if the data is real.
- **How do you prove the data came from the device, not someone making it up?** Arkiv's `$creator` is immutable — readings created by a device's own wallet are tamper-proof attribution with no custom signature scheme needed. If a third party could inject readings, your data layer isn't really doing its job.
- **What's the volume?** A handful of readings per minute is a very different problem from thousands per second. Volume shapes whether you batch writes, whether you aggregate, and how aggressive your expiration tiers should be.
- **What's the data lifecycle?** Raw readings probably don't need to live forever — but aggregates might. Differentiated expiration across raw / hourly / daily / device-profile is a strong fit here.
- **Who's the consumer?** A dashboard, an API, a downstream protocol calculating rewards? Different consumers want different query shapes — which shapes which attributes you'll want indexed.

### Directions to push it further

- Real hardware integration (ESP32, Raspberry Pi, Arduino, mobile-phone sensors)
- Aggregation pipeline: raw readings rolled into hourly/daily aggregates as separate entities, raw readings expiring sooner
- Operator reward calculator that pulls from Arkiv and computes payouts
- Map / geographic visualisation
- Multi-device fleet dashboard
- Anomaly detection / quality scoring for readings

---

## Mixing themes

You're free to combine themes. The expectation is that you go *deep* — a hybrid should be more than "I added a tag." Examples:

- **DePIN + Privacy:** A DePIN network where individual operator readings are encrypted but aggregate statistics are public.
- **AI + Privacy:** An agent whose memory is encrypted and only the owner (or specific delegates) can read it. Public attestation that "agent X made decision Y" without revealing the inputs.
- **AI + DePIN:** An AI agent that ingests sensor readings from a DePIN network as memory and produces decisions queryable on Arkiv.

State your theme(s) explicitly in the README and submission form.

---

## Getting Started

1. **Pick your theme** — whichever excites you most (or mix them)
2. **Read the section for your theme** above — use the questions and directions to shape your build, not as a checklist
3. **Read the [Arkiv docs](https://docs.arkiv.network)** and the [SDK fundamentals guide](https://docs.arkiv.network/start-here/fundamentals/)
4. **Set up your project's `PROJECT_ATTRIBUTE`** before writing any entity code
5. **Connect to Kaolin testnet** — [faucet](https://kaolin.hoodi.arkiv.network/faucet/), [explorer](https://explorer.kaolin.hoodi.arkiv.network/)
6. **Get create + read + query working for one entity type first** — then add relationships, then add more types
7. **Join Discord support channel** — [Arkiv Discord](https://discord.gg/arkiv) → **#builders-challenge**

---

## Submission Requirements

| What            | Details                                                                              |
| --------------- | ------------------------------------------------------------------------------------ |
| **Theme**       | AI, Privacy, DePIN, or an explicit hybrid                                            |
| **GitHub repo** | Public, open source, includes README with setup instructions                          |
| **Demo link**   | Working deployment connected to Arkiv testnet                                         |
| **Demo video**  | Optional at submission, required for prize claim (2–3 min walkthrough)                |
| **Team info**   | Names, GitHub handles, wallet address for prize                                       |

Submission form: published in the [README](../README.md) before kickoff.

---

## Questions?

Join our [Discord](https://discord.gg/arkiv) and head to **#builders-challenge**, or post in the Network School channel. The Arkiv team is on call daily during the build window. There's also an Office Hour planned during the build window — date announced in Discord.

Don't struggle alone. If you're stuck on an Arkiv integration issue, ask. We'd rather help you ship something great than watch you debug in silence for 3 days.
