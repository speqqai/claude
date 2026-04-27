# Technical Design Document Template

## 1. Document Control
- TDD Title:
- Feature/Initiative:
- PRD Reference: [Link to PRD file]
- Tech Lead:
- Status: Draft | In Review | Ready for Implementation | Draft - Blocked
- Last Updated:

## 2. PRD Reference

### 2.1 Requirements Being Designed

List every PRD requirement this TDD addresses:

| REQ ID | Requirement (from PRD) | Who (roles/plans) | Priority |
|---|---|---|---|
| REQ-001 | [Copy from PRD summary table] | [Copy from PRD] | P0/P1/P2 |
| REQ-002 | | | |

### 2.2 Technical Considerations (from PRD)
Copy the Technical Considerations section from the PRD here for reference:
- Platforms:
- Surfaces touched:
- Data model notes:
- Integrations:
- Business rules:

## 3. Research References

### 3.1 SDK/Framework Documentation Consulted

For every framework/SDK the design uses, list the specific documentation referenced:

| Technology | Version | Documentation Referenced | Specific APIs/Features Used |
|---|---|---|---|
| [e.g., Next.js] | [e.g., 16] | [Doc URL or section name] | [e.g., Route Handlers, Server Actions] |
| [e.g., Supabase] | | [Doc URL or section name] | [e.g., RLS policies, auth.getUser()] |

### 3.2 Key Findings from Research
- [Finding 1: What the docs say about how to accomplish X]
- [Finding 2: Built-in capability that eliminates need for custom code]
- [Finding 3: Version-specific behavior or limitation to be aware of]

## 4. Existing Codebase Capabilities

### 4.1 Components/Patterns to Reuse

| Existing Capability | Location | How It's Reused in This Design |
|---|---|---|
| [e.g., auth middleware] | [e.g., lib/auth.ts] | [e.g., Used for session verification on new routes] |
| [e.g., SWR data hook pattern] | [e.g., features/workspace/hooks/] | [e.g., Follow same pattern for new data fetching] |

### 4.2 Patterns to Follow
- [Pattern 1: How existing features handle X — follow the same approach]
- [Pattern 2: Existing database table structure to extend]

## 5. PRD-to-Design Traceability

| PRD Requirement | Design Approach | SDK/Framework | Doc Reference | Existing Code to Reuse |
|---|---|---|---|---|
| REQ-001: [title] | [How this requirement is implemented] | [Which SDK/framework] | [Doc URL/section] | [Existing code path] |
| REQ-002: [title] | | | | |

## 6. Architecture

### 6.1 Architecture Diagram

```
┌─────────────────────────────────────────────────────┐
│                    [Title]                            │
├─────────────────────────────────────────────────────┤
│                                                      │
│   [ASCII diagram showing component relationships,    │
│    data flow, and system boundaries]                 │
│                                                      │
│   Client ──► Server ──► Database                     │
│                │                                     │
│                ▼                                     │
│          External Service                            │
│                                                      │
└─────────────────────────────────────────────────────┘
```

### 6.2 Component Responsibilities
- [Component 1]: [What it does, why it exists]
- [Component 2]: [What it does, why it exists]

### 6.3 Data Flow
1. [Step 1: User action triggers...]
2. [Step 2: Server receives and...]
3. [Step 3: Database query...]
4. [Step 4: Response returned...]

## 7. Technical Solution

### 7.1 API Contracts

#### [Route 1: e.g., POST /api/feature/action]
- **Method:** POST | GET | PATCH | DELETE
- **Auth:** Required (roles: [from PRD Who column])
- **Request:**
```typescript
{
  field: type // description
}
```
- **Response (success):**
```typescript
{
  field: type // description
}
```
- **Response (error):**
```typescript
{
  error: string // error message
}
```
- **PRD Requirement:** REQ-xxx

### 7.2 Database Changes

#### New Tables
| Table | Column | Type | Constraints | Purpose |
|---|---|---|---|---|
| [table_name] | [column] | [type] | [NOT NULL, FK, etc.] | [Why this column exists] |

#### Modified Tables
| Table | Change | Reason | PRD Requirement |
|---|---|---|---|
| [table_name] | [Add column X] | [Why] | REQ-xxx |

#### RLS Policies
| Table | Policy Name | Operation | Check | Using | PRD Requirement |
|---|---|---|---|---|---|
| [table] | [name] | SELECT/INSERT/UPDATE/DELETE | [condition] | [condition] | REQ-xxx |

### 7.3 Server-Side Logic

For each piece of server logic, document:
- What it does
- Which SDK/framework API it uses (with doc reference)
- Where it lives in the codebase
- Which PRD requirement it fulfills

### 7.4 Client-Side Logic (only if justified)

For any client-side logic, document:
- What it does
- Why it MUST be client-side (specific justification)
- Which SDK/framework API it uses (with doc reference)

## 8. Error Handling and Logging

### 8.1 Error Points

| Error Point | Trigger | Error Response (user-visible) | Log Level | Log Content | PRD Requirement |
|---|---|---|---|---|---|
| [e.g., Auth failure] | [When/how it occurs] | [What user sees] | ERROR/WARN | [What gets logged] | REQ-xxx |
| [e.g., DB constraint violation] | | | | | |

### 8.2 Error Handling Rules
- No silent failures. Every error must be logged AND surfaced to the user.
- No fallback logic. If the primary path fails, log the error and return it.
- No try/catch that swallows errors.
- Use existing error patterns in the codebase (reference specific examples).

## 9. Security Considerations

### 9.1 Authentication
- How is the user verified for this feature?
- Which existing auth flow is used?

### 9.2 Authorization
- How are roles enforced? (Map from PRD Who column)
- How are plan tiers enforced? (Map from PRD Who column)
- Which RLS policies apply?

### 9.3 Input Validation
- Where is input validated? (Server-side)
- What validation rules apply?
- Which validation library/framework is used? (with doc reference)

### 9.4 Sensitive Data
- What PII or secrets does this feature handle?
- How are they protected in transit and at rest?
- Are there compliance requirements?

## 10. Custom Code Audit

### 10.1 Custom Code Inventory

| Custom Code | Purpose | SDK Alternative Checked? | Justification |
|---|---|---|---|
| [Description] | [Why it exists] | [Yes — no SDK capability exists for X because...] | [Why glue code is necessary] |

### 10.2 Verdict
- [ ] Zero custom code — all logic uses SDK/framework built-in capabilities
- [ ] Minimal glue code only — each instance justified above
- [ ] Custom code present — requires review (list blockers)

## 11. Implementation Plan

### 11.1 Phases

#### Phase 1: [Name]
- Deliverables:
- Dependencies:
- PRD Requirements covered: REQ-xxx, REQ-xxx

#### Phase 2: [Name]
- Deliverables:
- Dependencies on Phase 1:
- PRD Requirements covered: REQ-xxx, REQ-xxx

### 11.2 Parallel Work
- [What can be built simultaneously]

## 12. Open Questions
- Question:
- Impact on design:
- Owner:
- Due date (YYYY-MM-DD):

## 13. Implementation Readiness Verdict
- Verdict: Ready for Implementation | Draft - Needs Revision | Draft - Blocked
- Blockers (if any):
- Unresolved open questions:

## 14. Validation Report
- Check 1 (PRD Traceability): Pass | Fail
- Check 2 (Research References): Pass | Fail
- Check 3 (Existing Code Reuse): Pass | Fail
- Check 4 (Zero Custom Code): Pass | Fail
- Check 5 (Architecture Diagram): Pass | Fail
- Check 6 (Error Logging): Pass | Fail
- Check 7 (Security): Pass | Fail
- Check 8 (Server-Side Default): Pass | Fail
- Rubric score (0-16):
- Weak areas + fixes:
