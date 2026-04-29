# Arkiv × ETHNS Builder Challenge — Builder's Guide

---

## What you're building

A **web3-native application** where all data lives on Arkiv. Users own their data — not the platform.

Pick one of three open themes:

| Theme | The pitch | Concrete examples |
|-------|-----------|------------------|
| **AI** | Agents whose memory you actually own | Personal research assistant, coding-agent project context, MCP memory backend, agent reputation log |
| **Privacy** | Confidential data patterns on a public, tamper-proof layer | Encrypted records with access control, anonymous attestations, sealed-bid auctions, selective disclosure |
| **DePIN** | A queryable data layer for sensor / telemetry / device data | Proof-of-coverage map, solar tracker, air-quality network, fleet telemetry |

**Pick the one that excites you, or hybridise.** All themes are scored on the same rubric. There's no advantage to choosing one over another.

The [reference patterns](reference-patterns/) folder gives you a starting point for each theme — entity schemas, key code paths, and the minimum viable structure.

---

## Minimum Requirements (all themes)

Regardless of theme, your submission must:

### Technical baseline
- [ ] All core data stored as Arkiv entities (not in a traditional database). For the Privacy theme, encrypted payloads stored as entities count.
- [ ] Wallet-based ownership (creators own their data)
- [ ] At least 2 entity types with a relationship between them
- [ ] Queryable attributes used for filtering or search
- [ ] Rational expiration dates on entities (see [A note on expiration](#a-note-on-expiration))
- [ ] Public read access on non-encrypted entities (no wallet needed to browse)
- [ ] Open source GitHub repo
- [ ] Working demo link
- [ ] README with setup instructions

---

## Theme 1: AI — Agents Whose Memory You Actually Own

*"Memory you own, portable across any tool that reads Arkiv."*

Most AI agents today store their memory in a vendor-locked vector DB or a local file. Switch agents, lose context. Build agents whose memory lives as Arkiv entities — queryable by tags and time, wallet-owned, portable across any app that knows how to read Arkiv.

### Concrete builds

- A personal research assistant that archives every paper it reads plus your notes, queryable months later by tag
- A coding agent whose project context is a shared Arkiv DB across teammates
- An [MCP server](https://modelcontextprotocol.io/) that hands any LLM a memory backend keyed to a user's wallet
- An agent that maintains a public reputation log of its own decisions
- A multi-agent system where agents read each other's public memory entities for coordination

### Minimum features

**For agent owners (wallet required):**
- [ ] Create / register an agent identity (one entity per wallet, or per agent if multi-agent)
- [ ] Agent writes memory items as entities with content, tags, and expiration dates
- [ ] Agent retrieves memory entries by query (e.g., by tag, time range, or relevance)
- [ ] Demo flow: agent uses stored memory across at least 2 sessions (proves persistence + retrieval)

**For other users / observers (no wallet needed):**
- [ ] Browse public memory entries by tag or time window (where the agent owner has chosen public visibility)
- [ ] View agent profile and recent activity

**Data lifecycle:**
- [ ] Differentiated expiration dates on memory items (short-lived scratchpad vs. long-term beliefs — see [A note on expiration](#a-note-on-expiration))

### Nice-to-haves
- Memory hierarchy with differentiated expirations (e.g., scratchpad: hours, working memory: days, long-term: months)
- Vector embeddings stored alongside text content for semantic retrieval
- Multi-agent shared memory with attribution per agent
- MCP server wrapper exposing memory to any LLM client
- Public reputation log: every decision an agent makes recorded as an entity
- "Memory portability" demo: same memory used by two different agent UIs

### Suggested entity design

Think in terms of **two core entity types**: an Agent profile (one per wallet, or per agent in a multi-agent system) and Memory items (owned by the agent, referencing their parent agent). Use queryable attributes for tags, content type, importance, and a coarse time bucket. Each memory item should have an expiration date that reflects its role in the memory hierarchy.

A starter schema and minimal code path are in [`reference-patterns/ai-agent-memory.md`](reference-patterns/ai-agent-memory.md).

---

## Theme 2: Privacy — Confidential Data Patterns

*"Encryption and access control on a public, tamper-proof layer."*

Arkiv is public-by-default and tamper-proof. Privacy on Arkiv isn't about hiding data inside the protocol — it's about what builders put *in* the entities and how access is gated. Encrypted payloads, ZK proofs over entity contents, anonymous attestations, selective-disclosure flows.

This is the highest-skill theme of the three. We've published [reference patterns](reference-patterns/) for the two most common starting points (encrypted-payload + access-list, ZK-proof-of-attribute) so you don't begin from scratch.

### Concrete builds

- Encrypted medical records where a patient grants a doctor 30-day access (auto-expires)
- Anonymous DAO membership rolls with public proofs of total members
- Sealed-bid auctions that publish only the winner
- Whistleblower-style document archives with verifiable timestamps and revealed metadata only
- Anonymous credentials: prove you have a verified GitHub contribution count without revealing your identity
- Group messaging with end-to-end encryption and on-chain message-history audit trail

### Minimum features

**For data owners (wallet required):**
- [ ] Create at least one entity type with an encrypted payload
- [ ] Define an access-control entity (which wallet addresses can decrypt) with wallet-bound write permissions
- [ ] Grant access to another wallet
- [ ] Revoke access (by deleting / expiring the access entry, or by rotating keys)

**For data consumers (wallet required to access):**
- [ ] Decrypt an entity they have been granted access to
- [ ] Cannot decrypt an entity they have not been granted access to (demonstrate this in the demo!)

**Data lifecycle:**
- [ ] Time-bounded access using expiration dates on access-control entities
- [ ] Demo: data owner stores → grants access → granted user decrypts → access revoked → access entity expires → revoked user can no longer decrypt

### Nice-to-haves
- ZK proof of attribute (e.g., prove a user is over 18 without revealing date of birth)
- Anonymous attestations (prove a credential without revealing which one backs it)
- Auto-revocation via expiration dates only (no manual action needed)
- Sealed-bid reveal pattern (encrypted submissions, decrypt all on event end)
- Audit log of access events as entities (every decryption recorded)
- Selective field disclosure (encrypt some fields of an entity, leave others public)

### Suggested entity design

Think in terms of **two or three core entity types**: a Record (encrypted payload + public metadata + owner wallet), an Access Grant (record reference + grantee wallet + expiration date), and optionally an Audit Event (who accessed what, when). The Record's payload is the ciphertext; the metadata is anything you're willing to expose publicly (e.g., record type, creation time).

Starter schemas and code paths:
- [`reference-patterns/privacy-encrypted-payload.md`](reference-patterns/privacy-encrypted-payload.md) — encrypted record + access-list
- [`reference-patterns/privacy-zk-attribute.md`](reference-patterns/privacy-zk-attribute.md) — ZK-proof-of-attribute

---

## Theme 3: DePIN — A Queryable Data Layer for the Physical World

*"Time-scoped, tamper-proof telemetry — owned by the operator."*

DePIN networks generate enormous time-series data: sensor readings, location pings, bandwidth measurements, energy production, environmental telemetry. Today most of it lands in centralised databases (defeating the point) or gets summarised to L1 (too expensive, not queryable). Arkiv is the missing data layer.

### Concrete builds

- Proof-of-coverage map where every node ping is a verifiable Arkiv entity (a Helium-style network with the data on Arkiv)
- Solar-panel production tracker feeding tokenised energy markets
- Air-quality sensor network with a public queryable API
- Fleet-tracking system where operators retain sovereignty over their telemetry
- Bandwidth measurement network with on-chain reward calculation pulling from Arkiv
- Decentralised weather-station network with verifiable readings

### Minimum features

**For device operators (wallet required):**
- [ ] Register a Device as an entity (wallet-owned, with metadata like device type, location, capabilities)
- [ ] Submit Readings as entities — time-stamped, owner-signed, referenced to the parent Device, queryable by attribute
- [ ] Demo: at least 1 device (real or simulated) emitting readings to Arkiv

**For consumers / observers (no wallet needed):**
- [ ] Browse readings by device, by time window, and by at least one other attribute (e.g., location, type)
- [ ] View device profiles with their reading history
- [ ] Public dashboard or API surfacing the data

**Data lifecycle:**
- [ ] Tiered / differentiated expiration dates on readings vs. devices (e.g., raw readings: 30 days, hourly aggregates: 1 year, devices: long-lived — see [A note on expiration](#a-note-on-expiration))

### Nice-to-haves
- Real hardware integration (ESP32, Raspberry Pi, Arduino, mobile-phone sensors)
- Reading aggregation: raw readings get summarised into hourly/daily aggregates as separate entities, with raw readings expiring sooner
- Operator reward calculator that pulls reading data from Arkiv and computes payouts
- Map / geographic visualization
- Multi-device fleet dashboard
- Anomaly detection / quality scoring for readings

### Suggested entity design

Think in terms of **two core entity types** (with an optional third): a Device (wallet-owned, one per physical device or operator), Readings (time-stamped, signed, referenced to a parent Device), and optionally Aggregates (hourly / daily roll-ups computed from raw readings, with longer expiration). Use queryable attributes for time bucket, location, reading type, and device class.

A starter schema and minimal code path are in [`reference-patterns/depin-sensor-archive.md`](reference-patterns/depin-sensor-archive.md).

---

## Hybridising themes

You're free to combine themes. The expectation is that you go *deep* — a hybrid should be more than "I added a tag." Examples:

- **DePIN + Privacy:** A DePIN network where individual operator readings are encrypted but aggregate statistics are public.
- **AI + Privacy:** An agent whose memory is encrypted and only the owner (or specific delegates) can read it. Public attestation that "agent X made decision Y" without revealing the inputs.
- **AI + DePIN:** An AI agent that ingests sensor readings from a DePIN network as memory and produces decisions queryable on Arkiv.

State your theme(s) explicitly in the README and submission form.

---

## What "Arkiv integration depth" means

This is 40% of your score. Same rubric regardless of theme.

**Shallow (low score):**
- Using Arkiv as a simple key-value store
- Storing a JSON blob and reading it back
- Minimal use of entity attributes or querying

**Medium (decent score):**
- Proper entity schema with queryable attributes
- Using Arkiv's query capabilities for filtering and search
- Wallet-based ownership

**Deep (high score):**
- Entity relationships maintained properly (parent → children via references)
- Rational expiration dates — thoughtful choices about how long each entity type should live (see [A note on expiration](#a-note-on-expiration))
- Multiple entity types working together
- For Privacy: encrypted payloads with proper access-control patterns, or ZK proofs over entity contents
- For DePIN: tiered expiration (raw vs aggregated), high-volume signed readings
- For AI: differentiated memory types with role-appropriate expiration, multi-agent or shared-memory patterns
- Creative use of Arkiv features we haven't thought of

---

## A note on expiration

All Arkiv entities have an expiration date — this is core to how Arkiv works, not an optional feature. Memory items that expire after their relevance window, sensor readings that expire after the aggregate is computed, access grants that auto-revoke after 30 days — that's the expected behaviour.

**Choose expiration dates thoughtfully.** On testnet (which is what you'll be building on), expiration has no cost implications. But on mainnet, expiration directly impacts pricing — shorter-lived entities are cheaper, longer-lived entities cost more. If you're designing your data model, think about it the way a real product would:

- **AI:** scratchpad memory probably expires in hours, working memory in days, long-term beliefs in months/years.
- **Privacy:** access grants typically expire (that's the *point*); encrypted records may be longer-lived if the data needs to persist.
- **DePIN:** raw readings probably expire in 30–90 days, hourly aggregates in 1+ years, devices long-lived.

That kind of intentionality is what scores well on Arkiv integration depth.

---

## Getting Started

1. **Pick your theme** — whichever excites you most (or pick a hybrid)
2. **Read the [Builder's Guide](builders-guide.md) section for your theme**
3. **Open the [reference pattern](reference-patterns/) for your theme** — copy the schema, run the snippet, prove the round-trip works
4. **Read the Arkiv docs** — [arkiv.network/docs](https://arkiv.network/docs)
5. **Pick your SDK** — [TypeScript](https://arkiv.network/getting-started/typescript) or [Python](https://arkiv.network/getting-started/python)
6. **Connect to Kaolin testnet** — [instructions in the getting-started guides]
7. **Join Discord support channel** — [Arkiv Discord](https://discord.gg/arkiv) → **#builders-challenge**
8. **Start with your primary entity** — get create + read working first, then build relationships on top

---

## Submission Requirements

| What | Details |
|------|---------|
| **Theme** | AI, Privacy, DePIN, or an explicit hybrid |
| **GitHub repo** | Public, open source, includes README with setup instructions |
| **Demo link** | Working deployment connected to Arkiv testnet |
| **Demo video** | Optional at submission, required for prize claim (2–3 min walkthrough) |
| **Team info** | Names, GitHub handles, wallet address for prize |

Submission form: published in the [README](../README.md) before kickoff.

---

## Questions?

Join our [Discord](https://discord.gg/arkiv) and head to **#builders-challenge**, or post in the Network School channel. The Arkiv team is on call daily during the build window. There's also an Office Hour planned during the build window — date announced in Discord.

Don't struggle alone. If you're stuck on an Arkiv integration issue, ask. We'd rather help you ship something great than watch you debug in silence for 3 days.
