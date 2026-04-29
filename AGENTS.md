# AGENTS.md — Arkiv × ETHNS Builder Challenge

Apply the root `AGENTS.md` first. This file adds challenge-specific context and constraints.

## Purpose

This repo contains the official rules, guides, and resources for the Arkiv × ETHNS Builder Challenge: a 9-day challenge to build a web3-native application on Arkiv, hosted with Network School during Genesis Block Month (May 2026). Two winners each receive a $1,500 USDC stipend toward a one-month stay at Network School. If a winner can't attend, they keep the stipend.

## Use these files

| Question | File |
|----------|------|
| What is the challenge? | `README.md` |
| What should builders create? | `docs/builders-guide.md` |
| Official Arkiv agent skill (install for SDK + best-practice context) | `docs/agent-skill.md` |
| Rules, eligibility, prizes, legal terms | `RULES.md` |
| How submissions are scored | `docs/scoring-rubric.md` |
| Common questions | `FAQ.md` |

## Key facts

- Prize: $1,500 USDC per winner (2 winners), intended toward a one-month stay at Network School. If a winner can't or doesn't want to attend NS, they keep the stipend.
- Themes: AI, Privacy, or DePIN. Pick one or hybridise.
- Scoring: Arkiv integration depth 40%, functionality 30%, design and UX 20%, code quality and docs 10%.
- All entities expire. Use `expiration dates`, never `TTL`.
- Testnet only.
- AI tools are allowed.
- Demo video is optional at submission and required for prize claim.
- Open source is required.
- Build window: May 16–25, 2026.

## Current testnet

Use Kaolin for all building.

| Item | Value |
|------|-------|
| Network ID | `60138453025` |
| HTTP RPC | `https://kaolin.hoodi.arkiv.network/rpc` |
| WebSocket RPC | `wss://kaolin.hoodi.arkiv.network/rpc/ws` |
| Standard Bridge | `0x6db217C596Cd203256058dBbFcA37d5A62161b78` |
| Faucet | `https://kaolin.hoodi.arkiv.network/faucet/` |
| Explorer | `https://explorer.kaolin.hoodi.arkiv.network/` |
| TS/JS SDK | `@arkiv-network/sdk` v0.6.0 or newer |

## Minimum technical requirements

Every submission must:
- Use the official `@arkiv-network/sdk` (TypeScript v0.6.0+).
- Define a unique `PROJECT_ATTRIBUTE` and stamp it on every entity / every query — Arkiv is a shared DB.
- Store core data as Arkiv entities, not a traditional database. Encrypted payloads stored as entities count for the Privacy theme.
- At least 2 entity types, related via shared-attribute foreign-key (no built-in `references` field).
- Numeric values stored as numeric attributes (range queries work); strings as string attributes (eq / glob only).
- Right-sized `expiresIn` durations using `ExpirationTime` helpers — different per entity type.
- Use `$owner` (mutable, controls writes) and `$creator` (immutable, tamper-proof attribution) correctly.
- Open source on GitHub with a working demo and README.

All Arkiv entities are publicly readable. Privacy = encrypted payload, not a public/private toggle.

## Operating boundaries

### Always do

- Point builders to the published docs before inventing explanations.
- Use `docs/builders-guide.md` for theme guidance and per-theme entity models.
- Use `docs/scoring-rubric.md` when asked how to score well.
- Use `RULES.md` for eligibility, prizes, deadlines, or legal terms.
- Preserve the challenge framing, published rubric, and official requirements.
- Respect Arkiv's language conventions: "tamper-proof" (not "verifiable"), "expiration dates" (not "TTL"), "Arkiv" (never "Golem Base").
- Stamp `PROJECT_ATTRIBUTE` on every entity create and every query. Without it, queries leak data from other projects in the shared DB.
- Default to `walletClient.mutateEntities({ creates: [...] })` for batch creates and paginate reads via `result.hasNextPage()` / `result.next()`.

### Ask before

- Changing official challenge rules, prize details, deadlines, or scoring language.
- Inventing new technical requirements not present in the published materials.
- Editing submission requirements in ways that could change builder expectations.

### Never do

- Paraphrase legal terms from `RULES.md` when exact wording matters.
- Use `TTL` instead of `expiration dates`.
- Present Arkiv as optional. It is the required data layer.
- Encourage builders to use Mendoza when Kaolin is the active testnet.
- Use deprecated terms like "Golem Base" or describe Arkiv as "trustless" or "fully decentralised" — Arkiv launches with centralised sequencers.

## Builder guidance

- Help builders think in **entities** (payload + typed attributes + `expiresIn` in seconds), **shared-attribute foreign keys** for relationships, and **`$owner` / `$creator`** for ownership and tamper-proof attribution.
- Numeric attributes for range queries; string attributes for equality + glob. Attributes have no array type — model lists via one-attribute-per-value or via separate relationship entities.
- Three themes are open and hybridable. AI: agent memory on Arkiv. Privacy: encrypted payloads + envelope encryption + access-grant entities, or ZK proofs anchored on Arkiv. DePIN: time-scoped sensor and telemetry data with batch creates and `$creator`-based provenance.
- Suggest stack choices that fit the builder's experience, but keep Arkiv as the required data layer.
- If Arkiv-specific issues come up, direct builders to `#builders-challenge` on the Arkiv Discord and the relevant Network School channel.
