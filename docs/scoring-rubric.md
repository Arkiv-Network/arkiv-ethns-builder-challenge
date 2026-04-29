# Arkiv × ETHNS Builder Challenge — Scoring Rubric

---

## Scoring Scale

Each sub-criteria is scored **1–5**:

| Score | Meaning |
|-------|---------|
| 1 | Missing or broken |
| 2 | Minimal effort, barely functional |
| 3 | Works, meets expectations |
| 4 | Good — thoughtful implementation, above average |
| 5 | Excellent — impressive, creative, or production-quality |

---

## Criteria 1: Arkiv Integration Depth (40%)

This is the core of the challenge. We're evaluating how meaningfully Arkiv is used as the data layer — not just whether it's present.

| Sub-criteria | 1 (Weak) | 3 (Solid) | 5 (Excellent) |
|-------------|----------|-----------|----------------|
| **Entity schema design** | Single blob entity, no structure. Missing or generic `PROJECT_ATTRIBUTE`. | Separate entity types stamped with a unique `PROJECT_ATTRIBUTE`, with typed attributes (numerics for range queries, strings for eq/glob) and clear separation of concerns | Well-designed schema with right-typed attributes, payload structured for the use case, project-namespaced cleanly, no array-as-attribute anti-patterns |
| **Query usage** | Only reads by entity key | Filters by `PROJECT_ATTRIBUTE` plus 1–2 theme attributes (e.g., tag, time window, type) | Uses multiple query filters, range queries on numeric attributes, paginates large result sets, demonstrates understanding of Arkiv's query model |
| **Ownership model** | No wallet association | Uses `$owner` correctly (only the owner can update/delete); writes wallet-gated | End-user `$owner` for write/update/delete control, plus `$creator` used intentionally for tamper-proof attribution where it matters (e.g., DePIN readings filtered by `.createdBy(deviceWallet)`, trusted-source backend reads). For Privacy: layered access-control via wrapped-key entities on top of `$owner`. |
| **Entity relationships** | No relationships | Parent → child links via shared-attribute foreign keys (e.g., `{ key: "agentKey", value: parentEntityKey }`); relationships exist loosely | Foreign-key attributes used consistently, relationships maintained on create/delete, used for navigation and data integrity. Many-to-many or list relationships modeled correctly (no array-attribute hacks). |
| **Expiration dates** | No expiration set, or same expiration on everything. Hardcoded raw seconds instead of `ExpirationTime` helpers. | `expiresIn` durations present and reasonable for the domain (e.g., scratchpad memory expires fast, telemetry rolled up before expiry, AccessGrants auto-revoke) | Thoughtful, differentiated `expiresIn` per entity type reflecting real product logic. Use of `extendEntity()` where appropriate. |
| **Advanced features** | None | Entity lifecycle transitions based on business logic (e.g., draft → published, raw → aggregated, granted → revoked) | Multiple: batch creates via `mutateEntities`, encrypted-payload patterns with envelope encryption + auto-revoking AccessGrants, ZK-proof attestations anchored on Arkiv, or creative use of Arkiv features we haven't thought of |

**Section score** = average of 6 sub-criteria, weighted at 40%

---

## Criteria 2: Functionality (30%)

Does it work? Can a real user complete the core flows for the chosen theme?

| Sub-criteria | 1 (Weak) | 3 (Solid) | 5 (Excellent) |
|-------------|----------|-----------|----------------|
| **Core flows work** | Can't complete basic create or browse flow | Create + browse + view details all work end-to-end for the chosen theme | All flows work reliably: create, browse, filter, view, interact, edit, manage, etc. |
| **Filtering & search** | No filtering | 1–2 filters work (e.g., tag, time window, owner, status) | Multiple filters, keyword search, filters combinable, results update correctly |
| **Wallet integration** | Wallet connects but nothing happens | Wallet-gated features work (create, edit, manage, grant access) | Smooth wallet flow: connect, chain check, error states, disconnect. Blockchain complexity abstracted away. |
| **Error handling** | Crashes or silent failures | Basic error messages shown to user | Graceful error states: network issues, failed transactions, validation errors. User always knows what's happening. |
| **Data integrity** | Data inconsistencies, broken references | Data is consistent within the app | Entity status transitions are reliable, no orphaned data, encrypted payloads stay decrypt-able to authorised parties |

**Section score** = average of 5 sub-criteria, weighted at 30%

---

## Criteria 3: Design & UX (20%)

Would someone actually use this? Does it feel like a product, not a demo?

| Sub-criteria | 1 (Weak) | 3 (Solid) | 5 (Excellent) |
|-------------|----------|-----------|----------------|
| **Visual design** | Default/unstyled, no design effort | Clean, consistent styling. Looks intentional. | Distinctive visual identity, good typography, cohesive color palette, feels professional |
| **User experience** | Confusing navigation, unclear what to do next | Clear information hierarchy, obvious CTAs, reasonable flow | Intuitive from first visit, good empty states, loading states, progressive disclosure. Feels like a real product. |
| **Responsive** | Broken on mobile | Usable on mobile, basic responsive layout | Looks and works well across screen sizes |
| **Blockchain abstraction** | User needs to understand Arkiv/blockchain to use the app | Blockchain details present but not blocking | User doesn't need to know about Arkiv or blockchain to browse and use core flows. Web3 complexity is behind the scenes. |

**Section score** = average of 4 sub-criteria, weighted at 20%

---

## Criteria 4: Code Quality & Documentation (10%)

Can someone else understand and run your project?

| Sub-criteria | 1 (Weak) | 3 (Solid) | 5 (Excellent) |
|-------------|----------|-----------|----------------|
| **README** | Missing or "TODO" | Setup instructions that work, basic description of the project | Clear README with architecture overview, setup steps, screenshots/demo GIF, and explanation of Arkiv integration approach |
| **Code organization** | Single file or spaghetti | Reasonable file structure, components separated | Clean architecture, separation of concerns, readable naming |
| **Code quality** | Unreadable, no error handling | Consistent style, basic error handling | Clean, consistent, well-structured. Types where appropriate. No obvious security issues. |

**Section score** = average of 3 sub-criteria, weighted at 10%

---

## Final Score Calculation

```
Final Score = (Arkiv Integration × 0.40) + (Functionality × 0.30) + (Design & UX × 0.20) + (Code Quality × 0.10)
```

Each section score is the average of its sub-criteria (all on 1–5 scale), so the final score is also on a 1–5 scale.

**Example:**
- Arkiv Integration: avg 4.2 → × 0.40 = 1.68
- Functionality: avg 3.8 → × 0.30 = 1.14
- Design & UX: avg 3.5 → × 0.20 = 0.70
- Code Quality: avg 4.0 → × 0.10 = 0.40
- **Final: 3.92 / 5.00**

---

## Judge Scorecard Template

Each judge fills out one per submission:

```
Submission: [Team name]
Theme(s): [AI / Privacy / DePIN / hybrid]
Judge: [Name]
Date: [Date]

ARKIV INTEGRATION (40%)
  Entity schema design:     _/5
  Query usage:              _/5
  Ownership model:          _/5
  Entity relationships:     _/5
  Expiration dates:         _/5
  Advanced features:        _/5
  Section avg:              _/5

FUNCTIONALITY (30%)
  Core flows work:          _/5
  Filtering & search:       _/5
  Wallet integration:       _/5
  Error handling:           _/5
  Data integrity:           _/5
  Section avg:              _/5

DESIGN & UX (20%)
  Visual design:            _/5
  User experience:          _/5
  Responsive:               _/5
  Blockchain abstraction:   _/5
  Section avg:              _/5

CODE QUALITY (10%)
  README:                   _/5
  Code organization:        _/5
  Code quality:             _/5
  Section avg:              _/5

WEIGHTED FINAL:             _/5

Notes:
[Free-form observations, standout features, concerns]
```

---

## Tiebreaker

If two submissions have the same final score (within 0.1):
1. Arkiv Integration score is the tiebreaker (highest wins)
2. If still tied, the judge panel discusses and reaches consensus
