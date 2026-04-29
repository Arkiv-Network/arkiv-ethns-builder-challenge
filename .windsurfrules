# Arkiv × ETHNS Builder Challenge — AI Agent Context

This file provides context for AI coding assistants (Claude Code, Cursor, Copilot, Windsurf, Cline, etc.) working with builders participating in the Arkiv × ETHNS Builder Challenge at Network School.

## What is this repo?

The official rules, guides, and resources for the **Arkiv × ETHNS Builder Challenge** — a 9-day challenge to build a web3-native application using [Arkiv](https://arkiv.network) as the data layer, hosted with [Network School](https://ns.com) during Genesis Block Month (May 2026). Two winners each receive a **$1,500 USDC stipend + 1-month Network School membership**.

## Doc Map

| Question | File |
|----------|------|
| What is the challenge? What's the pitch? | [README.md](README.md) |
| What should I build? Requirements? Getting started? | [docs/builders-guide.md](docs/builders-guide.md) |
| Reference patterns for each theme | [docs/reference-patterns/](docs/reference-patterns/) |
| Official rules, eligibility, prizes, legal? | [RULES.md](RULES.md) |
| How is my submission scored? | [docs/scoring-rubric.md](docs/scoring-rubric.md) |
| May 15 Arkiv 101 workshop material | [docs/workshop-arkiv-101.md](docs/workshop-arkiv-101.md) |
| Common questions? | [FAQ.md](FAQ.md) |

## Key Facts

- **Prize:** $1,500 USDC + 1-month NS membership per winner (2 winners).
- **Themes:** AI, Privacy, or DePIN. Pick one or hybridise. Single rubric across all three.
- **Scoring weights:** Arkiv integration depth (40%), Functionality (30%), Design & UX (20%), Code quality & docs (10%).
- **All entities expire.** Expiration dates are core to Arkiv, not optional. Shorter-lived = cheaper on mainnet. Builders should choose expiration dates thoughtfully.
- **Testnet only.** All building happens on Arkiv testnet.
- **AI tools allowed.** Copilot, Claude, ChatGPT, etc. are all encouraged.
- **Demo video:** Optional at submission, required for prize claim (2–3 min).
- **Open source required.** MIT, Apache 2.0, or equivalent.
- **Build window:** May 16–25, 2026 (9 days). Submissions close May 25 23:59 SGT.

## Arkiv Resources

- **Documentation:** https://arkiv.network/docs
- **Developer Portal:** https://arkiv.network/dev
- **TypeScript SDK:** https://arkiv.network/getting-started/typescript
- **Python SDK:** https://arkiv.network/getting-started/python

### Current Testnet: Kaolin

Use Kaolin for all building:

| | |
|---|---|
| **Network ID** | `60138453025` |
| **HTTP RPC** | `https://kaolin.hoodi.arkiv.network/rpc` |
| **WebSocket RPC** | `wss://kaolin.hoodi.arkiv.network/rpc/ws` |
| **Standard Bridge** | `0x6db217C596Cd203256058dBbFcA37d5A62161b78` |
| **TS/JS SDK version** | `@arkiv-network/sdk` v0.6.0 or newer |

## Minimum Technical Requirements

Every submission must:
- Store all core data as Arkiv entities (not a traditional database). For the Privacy theme, encrypted payloads stored as entities count.
- Use wallet-based ownership (creators own their data)
- Have at least 2 entity types with a relationship between them
- Use queryable attributes for filtering or search
- Set rational expiration dates on entities
- Allow public read access for non-encrypted entities (no wallet needed to browse)
- Be open source on GitHub with a working demo link and README

## When Helping Builders

- **Theme selection:** Three open themes (AI / Privacy / DePIN). Builders pick one OR hybridise (e.g., privacy-preserving DePIN data, AI agent with encrypted memory). Point to `docs/builders-guide.md` for entry points and `docs/reference-patterns/` for starter patterns.
- **Entity design:** Don't invent data models — guide builders to think about entity types, relationships, queryable attributes, and expiration. The reference patterns in `docs/reference-patterns/` are the on-ramp.
- **Expiration:** All Arkiv entities expire. Don't use the term "TTL" — Arkiv calls it "expiration dates." Help builders choose different expiration durations for different entity types.
- **Privacy theme nuance:** Arkiv is public-by-default and tamper-proof. Privacy comes from what's *in* the entities (encrypted payloads, ZK proofs over contents) and how access is gated, not from hiding inside the protocol. See `docs/reference-patterns/privacy-encrypted-payload.md` and `docs/reference-patterns/privacy-zk-attribute.md`.
- **Language:** "Arkiv" (never "Golem Base"). "Tamper-proof" (never "verifiable" as shorthand). Avoid "trustless" and "fully decentralised" — Arkiv launches with centralised sequencers.
- **Scoring:** The rubric in `docs/scoring-rubric.md` is published and transparent. If a builder asks how to score well, point them to the specific sub-criteria.
- **Rules questions:** Always reference `RULES.md` for anything about eligibility, prizes, deadlines, or legal terms. Don't paraphrase legal language.
- **Tech stack:** Arkiv is the required data layer. Everything else (frontend, styling, wallet lib, hosting) is the builder's choice. Suggest what fits their experience.
- **Support:** Direct builders to **#builders-challenge** on the [Arkiv Discord](https://discord.gg/arkiv) and the relevant Network School channel for Arkiv-specific issues.
