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
| **Entity schema design** | Single blob entity, no structure | Separate entity types with typed fields (e.g., agent + memory items, sensor + reading, identity + attribute) | Well-designed schema with queryable attributes, proper field types, and clear separation of concerns |
| **Query usage** | Only reads by ID / key | Filters by 1–2 attributes (e.g., tag, time window, owner) | Uses multiple query filters, attribute-based sorting, demonstrates understanding of Arkiv's query model |
| **Ownership model** | No wallet association | Project wallet owns entities and references end-user | End-user wallet-bound ownership; only the owner can edit/delete their entities. For Privacy: clear access-control logic on top of wallet ownership. |
| **Entity relationships** | No relationships | Parent → child references exist loosely (e.g., agent → memories, device → readings, identity → attestations) | Clear entity references, maintained on create/delete, relationships used for navigation and data integrity |
| **Expiration dates** | No expiration set, or same expiration on everything | Expiration dates present and reasonable for the domain (e.g., scratchpad memory expires fast, telemetry readings rolled up before expiry, encrypted records expire when access is revoked) | Thoughtful, differentiated expiration — different entity types have different lifespans reflecting real-world product logic |
| **Advanced features** | None | Entity lifecycle transitions based on business logic (e.g., draft → published, raw → aggregated, granted → revoked) | Multiple: entity updates based on business logic, encrypted-payload patterns, ZK-proof attestations, or creative use of Arkiv features we haven't thought of |

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
