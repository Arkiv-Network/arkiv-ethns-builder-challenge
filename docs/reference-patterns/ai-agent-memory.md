# Reference Pattern — AI Agent Memory on Arkiv

A starter pattern for the **AI** theme. Use it as the on-ramp; depart from it as your build demands.

The goal: an agent that writes new memories as Arkiv entities, queries them later by tag and time, and lets the data follow the user across tools.

> Code samples are illustrative. Confirm exact SDK API against the [Arkiv TypeScript SDK docs](https://arkiv.network/getting-started/typescript) — version `@arkiv-network/sdk` v0.6.0 or newer.

---

## Entity schema

Two core entity types.

### `Agent`

One per wallet, or per agent in a multi-agent system.

| Field | Type | Notes |
|------|------|-------|
| `name` | string | Human-readable agent label |
| `description` | string | What the agent does |
| `model` | string | LLM model identifier (optional) |
| `owner` | wallet address | Set automatically; ownership-bound |
| `expiration` | timestamp | Long (e.g., 1 year+) — agents persist |

### `MemoryItem`

Owned by the agent (referencing `Agent.id`). Many of these per agent.

| Field | Type | Notes |
|------|------|-------|
| `agentId` | reference → Agent | Parent agent |
| `content` | string (or bytes) | The memory itself |
| `contentType` | string | `note`, `decision`, `observation`, `belief`, ... |
| `tags` | string[] | Queryable tags |
| `importance` | int (1-5) | Drives expiration tier |
| `embedding` | bytes | Optional — vector embedding for semantic retrieval |
| `expiration` | timestamp | **Differentiated per content type / importance** |

**Differentiated expiration** is what scores points on Arkiv integration depth. Suggested ladder:

| Memory type | Expiration |
|-------------|-----------|
| `scratchpad` (transient context) | 6–24 hours |
| `working` (current task) | 7–14 days |
| `note` / `observation` | 30–90 days |
| `decision` / `belief` (long-term) | 6–12 months |

---

## Minimal code outline (TypeScript)

```typescript
import { Arkiv } from "@arkiv-network/sdk";

const arkiv = new Arkiv({ rpc: "https://kaolin.hoodi.arkiv.network/rpc" });

// 1. Register agent
async function registerAgent(name: string, description: string) {
  return arkiv.entities.create({
    type: "Agent",
    payload: { name, description, model: "claude-opus-4-7" },
    expiration: addMonths(new Date(), 12),
  });
}

// 2. Write memory
async function remember(agentId: string, item: {
  content: string;
  contentType: "scratchpad" | "working" | "note" | "decision";
  tags: string[];
  importance?: number;
}) {
  const expirationMap = {
    scratchpad: hours(12),
    working: days(7),
    note: days(60),
    decision: months(9),
  };
  return arkiv.entities.create({
    type: "MemoryItem",
    payload: item,
    references: { agentId },
    expiration: expirationMap[item.contentType],
  });
}

// 3. Recall memory by tag and time window
async function recall(agentId: string, opts: { tags?: string[]; since?: Date }) {
  return arkiv.entities.query({
    type: "MemoryItem",
    filters: {
      "references.agentId": agentId,
      "payload.tags": opts.tags ? { contains: opts.tags } : undefined,
      "createdAt": opts.since ? { gte: opts.since } : undefined,
    },
    orderBy: { createdAt: "desc" },
    limit: 50,
  });
}
```

---

## Demo flow

To prove persistence + retrieval (a minimum requirement):

1. Register the agent. Note the agent ID.
2. In **Session A**: agent writes 5 memories of mixed types (scratchpad / note / decision).
3. End Session A. Restart the process / reload the page.
4. In **Session B**: agent calls `recall()` and demonstrates it pulls the items written in Session A.
5. Wait long enough (or set short expirations on `scratchpad`) to show that scratchpad items have expired while `decision` items persist.

---

## Where to go from here

- **MCP server wrapper:** Expose `remember()` and `recall()` as MCP tools so any LLM client can use the same memory backend.
- **Vector search:** Store embeddings alongside `content`. Query by cosine similarity in the client; use Arkiv tags + time as a coarse pre-filter.
- **Multi-agent shared memory:** Memories tagged `shared:true` are visible across agents; demonstrate two agents reading each other's memory.
- **Public reputation log:** Every `decision` is also written as a separate `Decision` entity with public read access — your agent's track record becomes verifiable.

---

## Scoring tips for this theme

- Differentiated expirations on memory types = high score on the **Expiration dates** sub-criterion.
- Multiple entity types with proper references (Agent ↔ MemoryItem, optionally Decision) = high score on **Entity relationships**.
- Querying by tag + time + content type = high score on **Query usage**.
- Demonstrating memory portability (two different UIs reading the same Arkiv memory) = high score on **Advanced features**.
