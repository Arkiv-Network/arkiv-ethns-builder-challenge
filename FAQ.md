# Arkiv × ETHNS Builder Challenge — FAQ

---

### General

**What is the Arkiv × ETHNS Builder Challenge?**
An invitation to build a web3-native application using Arkiv as the data layer, hosted with Network School during Genesis Block Month (May 2026). Pick one of three open themes — AI, Privacy, or DePIN (or hybridise). Two winners each receive a **$1,500 USDC stipend** toward a one-month stay at Network School.

**Who can participate?**
Anyone 18+, anywhere in the world. Solo or team. Hybrid format — you can build IRL at Network School or fully remote.

**Do I need to be at Network School to win?**
No — one of the two prize slots (the **Open slot**) is awarded to the top-ranked submission overall, so a fully remote builder can win it. The second slot (the **Local slot**) is reserved for participants attending or affiliated with Network School. All submissions are judged together on the same rubric — the split only determines which slot each winning entry occupies. See [RULES Section 5](RULES.md#5-prizes) for details.

**Do I need to know Arkiv beforehand?**
No. The May 15 IRL Arkiv 101 Workshop and the pre-challenge X Spaces (May 1–14) cover the basics. The mental model, per-theme entity design, and getting-started steps live in the [Builder's Guide](docs/builders-guide.md).

**Where are the rules hosted?**
In this repo — see [RULES.md](RULES.md).

**Can I use AI tools (Copilot, Claude, ChatGPT)?**
Yes. We care about the result, not how you got there.

**Can I use pre-existing code, libraries, or boilerplate?**
Yes for libraries, frameworks, and boilerplate. The Arkiv integration and core application logic must be original work created during the challenge period (May 16–25).

---

### Themes

**What are the three themes?**

| Theme | What you build | The hook |
|-------|---------------|---------|
| **AI** | AI agents whose memory lives as Arkiv entities | Memory you actually own — portable across any app that reads Arkiv |
| **Privacy** | Confidential data patterns on a public, tamper-proof layer | Encrypted payloads, ZK proofs, selective disclosure |
| **DePIN** | Queryable data layer for sensor / telemetry / device data | Time-scoped, tamper-proof readings — the missing layer for physical-world networks |

Full descriptions are in the [Builder's Guide](docs/builders-guide.md).

**Do I have to pick one of the three?**
Yes — your submission must address at least one. You can also hybridise (e.g., privacy-preserving DePIN, AI agent with encrypted memory). State your theme(s) in the submission form.

**Does my theme affect my score?**
No. All themes are scored on the same rubric (40% Arkiv integration depth, 30% functionality, 20% design/UX, 10% code quality).

**Can I combine themes?**
Yes — explicitly. We'd rather you go deep than shallow, but if your project naturally bridges two themes (e.g., a DePIN sensor network with encrypted operator-data — that's DePIN + Privacy), say so in your submission form and README.

**Can multiple teams pick the same theme?**
Yes. There's no cap per theme.

---

### Teams & Submissions

**Can I participate as a team?**
Yes. The team size limit is 5 members. There's only one prize per winning team — $1,500 USDC. How you split the stipend (and who, if anyone, attends NS) is up to you.

**If my team wins, how is the prize distributed?**
All team members must complete KYC, and the $1,500 is sent to one wallet confirmed by the whole team. See [RULES Section 7](RULES.md#7-kyc--prize-disbursement) for the full disbursement process.

**Can I submit more than one project?**
No. One submission per individual or team. If you submit multiple times, only the last submission counts.

**Can I update my submission after submitting?**
Yes. The form allows edits until the deadline (May 25 23:59 SGT). After the deadline, submissions are final.

**What's required in my submission?**
Your chosen theme(s), public GitHub repo, working demo URL, README with setup instructions, and the completed submission form. Full details in [RULES.md](RULES.md).

---

### Building

**Where do I find requirements for my theme?**
The [Builder's Guide](docs/builders-guide.md) describes each theme with concrete build ideas, entity-design hints, and expiration guidance.

**Can I use any tech stack?**
Yes. Arkiv is the data layer — pick whatever you want for the frontend, styling, wallet connection, hosting, and (for the AI theme) the LLM you wrap.

**Do I need a smart contract?**
Not required. It's a nice-to-have. You can build a fully functional submission using only the Arkiv SDK.

**What chain does this run on?**
Arkiv testnet — **Braga**:

| | |
|---|---|
| **Network ID** | `60138453102` |
| **HTTP RPC** | `https://braga.hoodi.arkiv.network/rpc` |
| **WebSocket RPC** | `wss://braga.hoodi.arkiv.network/rpc/ws` |
| **Standard Bridge** | `0xB52b417A79c9dE21ffe221dF9a3821B7EaC60813` |
| **Faucet** | [braga.hoodi.arkiv.network/faucet](https://braga.hoodi.arkiv.network/faucet/) |
| **Explorer** | [explorer.braga.hoodi.arkiv.network](https://explorer.braga.hoodi.arkiv.network/) |

Use `@arkiv-network/sdk` **v0.6.8 or newer**.

**Is my data private? Can other projects see it?**
Arkiv is a **shared, public database** — every entity is publicly readable, and your project shares the store with everyone else's. Two practical consequences:

1. **PROJECT_ATTRIBUTE.** Every project must define a unique attribute and stamp it on every entity *and* every query. Without it, your queries return everyone else's data and vice versa. This is mandatory and is graded — see the [Builder's Guide](docs/builders-guide.md).
2. **Privacy theme.** If you need confidentiality, encrypt the payload bytes before storing the entity. Arkiv stores the encrypted payload; only key-holders can decrypt. There's no public/private toggle at the protocol level — the encryption layer is yours to design.

**Who can edit or delete my entities?**
Only the **`$owner`** (the wallet that currently controls the entity). Ownership can be transferred. Separately, **`$creator`** is set immutably at creation — useful for tamper-proof attribution (e.g., verifying a DePIN reading came from a specific device wallet via `.createdBy(deviceWallet)`).

**Is there an agent skill that knows Arkiv?**
Yes — `arkiv-best-practices`. Install it in your AI coding assistant (Claude Code, Cursor, Copilot, Cline, Windsurf) and your agent stops inventing SDK calls. Setup + example prompts in [docs/agent-skill.md](docs/agent-skill.md).

**How do I report an Arkiv bug or suggest a feature?**
Install the `arkiv-feedback` skill and run `/arkiv-feedback` in your AI agent. It walks you through an interactive form — what you expected, what happened, reproduction steps, SDK version — and files the result directly to the public [`Arkiv-Network/reported-issues`](https://github.com/Arkiv-Network/reported-issues) repo. Install it with:

```bash
npx skills add https://github.com/arkiv-network/skills --skill arkiv-feedback
```

The Arkiv team monitors that repo throughout the build window.

**Where do I get help if I'm stuck?**
[**#ethns-arkiv-challenge**](https://discord.com/channels/1422146278883852412/1473629252183392266) on the [Arkiv Discord](https://discord.gg/arkiv). The Arkiv team is on call daily during the build window.

---

### Prizes & Travel

**What exactly do I win?**
Each of the two winners receives **$1,500 USDC**, paid to the EVM wallet address you provide. The stipend is intended toward a one-month stay at Network School (Forest City, Malaysia), in the month of your choice.

**What's Network School?**
A one-month residency programme at the NS campus (Forest City, Malaysia). It includes housing, programming, and the NS community for the month you attend. See [ns.com](https://ns.com) for what's covered. Attendance is subject to NS application and availability; the Arkiv team can connect winners with the NS operations team to coordinate.

**What if I can't travel to Network School?**
You keep the stipend. If you can't or don't want to travel, the $1,500 is still yours.

**What if I win but can't complete KYC?**
The prize may transfer to the next-ranked submission. You have 3 days to complete KYC after being notified.

**Do I need to do KYC to enter?**
No. KYC is only required to claim a prize. Enter freely; worry about KYC only if you win.

**What KYC do I need?**
All team members must complete KYC individually. You'll need: a government-issued ID, a signed declaration form (provided to winners after notification — print it and sign by hand), and a selfie with your ID. Full details in [RULES Section 7](RULES.md#7-kyc--prize-disbursement).

**Is a demo video required?**
Optional at submission, but **required to claim your prize**. If you win, you'll need to record and submit a 2–3 minute walkthrough of your project before the prize is disbursed. Plan for it — it's not long, but don't leave it to the last minute.

---

### Judging

**How are submissions scored?**
Four criteria with published weights — same rubric for all themes:

| Criteria | Weight |
|----------|--------|
| Arkiv integration depth | 40% |
| Functionality | 30% |
| Design & UX | 20% |
| Code quality & docs | 10% |

See the [Scoring Rubric](docs/scoring-rubric.md) for detailed sub-criteria.

**How do you compare submissions across different themes?**
The rubric is theme-agnostic. "Arkiv integration depth" means the same thing whether you built an AI agent memory layer, an encrypted-payload pattern, or a DePIN sensor archive — proper entity schemas, queryable attributes, wallet ownership, entity relationships, thoughtful expiration dates.

**Who are the judges?**
Judging panel will be confirmed and announced soon. The Network School operations team does not judge.

**Can I get feedback on my submission?**
Winners get feedback through the announcement Space on May 29. Individual feedback for non-winners may happen post-challenge but isn't guaranteed.

---

### Rules

**Where are the full rules?**
[Official Rules & Terms](RULES.md)

**Who owns the code I write?**
You do. By entering, you grant Arkiv a non-exclusive license to showcase your project (marketing, docs, presentations). You can do whatever you want with your own code.

**What license do I need?**
Open source — MIT, Apache 2.0, or equivalent.

**What gets me disqualified?**
Plagiarism, not using Arkiv as the data layer, malicious code, no working demo, submitting outside the three themes, or submitting after the deadline.

---

*Don't see your question? Join our [Discord](https://discord.gg/arkiv) and ask in [**#ethns-arkiv-challenge**](https://discord.com/channels/1422146278883852412/1473629252183392266).
