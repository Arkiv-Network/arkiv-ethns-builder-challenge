# Arkiv × ETHNS Builder Challenge — AI Agent Context

This file provides context for AI coding assistants (Claude Code, Cursor, Copilot, Windsurf, Cline, etc.) working with builders participating in the Arkiv × ETHNS Builder Challenge at Network School.

## What is this repo?

The official rules, guides, and resources for the **Arkiv × ETHNS Builder Challenge** — a 9-day challenge to build a web3-native application using [Arkiv](https://arkiv.network) as the data layer, hosted with [Network School](https://ns.com) during Genesis Block Month (May 2026). Two winners each receive a **$1,500 USDC stipend toward a one-month stay at Network School**.

## Doc Map

| Question | File |
|----------|------|
| What is the challenge? What's the pitch? | [README.md](README.md) |
| What should I build? Requirements? Getting started? | [docs/builders-guide.md](docs/builders-guide.md) |
| Install the official Arkiv agent skill | [docs/agent-skill.md](docs/agent-skill.md) |
| Official rules, eligibility, prizes, legal? | [RULES.md](RULES.md) |
| How is my submission scored? | [docs/scoring-rubric.md](docs/scoring-rubric.md) |
| Common questions? | [FAQ.md](FAQ.md) |

## Key Facts

- **Prize:** $1,500 USDC stipend toward a one-month stay at NS, per winner (2 winners). The stipend is purpose-bound to the NS stay, not a no-strings cash prize.
- **Themes:** AI, Privacy, or DePIN. Pick one or hybridise. Single rubric across all three.
- **Scoring weights:** Arkiv integration depth (40%), Functionality (30%), Design & UX (20%), Code quality & docs (10%).
- **All entities expire.** Expiration dates are core to Arkiv, not optional. Shorter-lived = cheaper on mainnet. Builders should choose expiration dates thoughtfully.
- **Testnet only.** All building happens on Arkiv testnet.
- **AI tools allowed.** Copilot, Claude, ChatGPT, etc. are all encouraged.
- **Demo video:** Optional at submission, required for prize claim (2–3 min).
- **Open source required.** MIT, Apache 2.0, or equivalent.
- **Build window:** May 16–25, 2026 (9 days). Submissions close May 25 23:59 SGT.

## Arkiv Resources

- **Documentation:** https://docs.arkiv.network
- **Developer Portal:** https://arkiv.network/dev
- **TypeScript SDK — Start here:** https://docs.arkiv.network/start-here/fundamentals/
- **Python SDK — Start here:** https://docs.arkiv.network/start-here/fundamentals/

### Current Testnet: Kaolin

Use Kaolin for all building:

| | |
|---|---|
| **Network ID** | `60138453025` |
| **HTTP RPC** | `https://kaolin.hoodi.arkiv.network/rpc` |
| **WebSocket RPC** | `wss://kaolin.hoodi.arkiv.network/rpc/ws` |
| **Standard Bridge** | `0x6db217C596Cd203256058dBbFcA37d5A62161b78` |
| **Faucet** | `https://kaolin.hoodi.arkiv.network/faucet/` |
| **Explorer** | `https://explorer.kaolin.hoodi.arkiv.network/` |
| **TS/JS SDK version** | `@arkiv-network/sdk` v0.6.0 or newer |

## Minimum Technical Requirements

Every submission must:
- Use the official `@arkiv-network/sdk` (TypeScript v0.6.0+)
- Define a unique `PROJECT_ATTRIBUTE` and stamp it on every entity / every query — Arkiv is a shared DB; without a project namespace, queries leak data from other projects
- Store core data as Arkiv entities (not a traditional database). For the Privacy theme, encrypted payloads stored as entities count
- At least 2 entity types, related via shared-attribute foreign-key (e.g., `{ key: "agentKey", value: agentEntityKey }` — Arkiv has no built-in `references` field)
- Numeric values stored as numeric attributes (so `gt`/`lt` range queries work); strings as string attributes
- Right-sized `expiresIn` durations using `ExpirationTime` helpers — different per entity type
- Use `$owner` (mutable, controls write/update/delete) and `$creator` (immutable, tamper-proof attribution) correctly. For trusted-source reads, filter by `.createdBy(wallet)`.
- Open source on GitHub with working demo and README

**Note:** all Arkiv entities are publicly readable. Privacy = encrypted payload, not a public/private toggle.

## When Helping Builders

- **PROJECT_ATTRIBUTE first.** Every Arkiv project must define a unique attribute and include it on every create/query. Without it, queries leak data from other projects in the shared DB. Flag it early — builders who skip this lose points on Arkiv integration depth.
- **Theme selection:** Three open themes (AI / Privacy / DePIN). Builders pick one OR hybridise (e.g., privacy-preserving DePIN data, AI agent with encrypted memory). Point to `docs/builders-guide.md` for the mental model and per-theme entity-model on-ramps.
- **Entity model:** Don't invent data shapes. An Arkiv entity is **payload + typed attributes + `expiresIn` (seconds)**. Relationships are modeled with shared attribute keys (e.g., `{ key: "agentKey", value: parentEntityKey }`) — there's no built-in `references` field. Attributes have no array type — model lists with one-attribute-per-value or with separate relationship entities.
- **Owner vs creator:** Arkiv has two metadata fields. `$owner` is mutable (transferable, controls write/update/delete). `$creator` is immutable (tamper-proof attribution, set at creation). Filter via `.ownedBy(wallet)` or `.createdBy(wallet)`. Use `$creator` for trusted-source reads (e.g., DePIN devices submitting their own readings — verify by `.createdBy(deviceWallet)`).
- **Attribute typing:** Numeric values stored as **numbers** support `gt`/`lt`/`gte`/`lte` range queries; strings only support `eq` and glob (`~`). Always store timestamps, scores, counts as numbers.
- **Expiration:** `expiresIn` is a **duration in seconds** set at creation. Use `ExpirationTime.fromHours(...)` / `.fromDays(...)` from `@arkiv-network/sdk/utils`. Don't say "TTL" — Arkiv calls it "expiration dates." Different entity types should have different durations. `extendEntity()` lengthens lifetimes if needed.
- **Privacy theme nuance:** All Arkiv entities are publicly readable — there's no public/private toggle. Privacy comes from encrypting the payload bytes before storing the entity, then gating decryption via wrapped-key entities (envelope encryption) or via ZK proofs over off-chain witnesses. See the Privacy section in `docs/builders-guide.md`.
- **Batch + paginate:** Use `walletClient.mutateEntities({ creates: [...] })` for batch creates (high-volume DePIN especially). Read queries paginate via `result.hasNextPage()` + `result.next()` — don't try to fetch everything at once.
- **Language:** "Arkiv" (never "Golem Base"). "Tamper-proof" (never "verifiable" as shorthand). Avoid "trustless" and "fully decentralised" — Arkiv launches with centralised sequencers.
- **Scoring:** The rubric in `docs/scoring-rubric.md` is published and transparent. If a builder asks how to score well, point them to the specific sub-criteria.
- **Rules questions:** Always reference `RULES.md` for anything about eligibility, prizes, deadlines, or legal terms. Don't paraphrase legal language.
- **Tech stack:** Arkiv is the required data layer. Everything else (frontend, styling, wallet lib, hosting) is the builder's choice. Suggest what fits their experience.
- **Support:** Direct builders to **#builders-challenge** on the [Arkiv Discord](https://discord.gg/arkiv) and the relevant Network School channel for Arkiv-specific issues.
