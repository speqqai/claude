# TDD Writing Standard (Production Grade)

## Purpose

Use this guide to produce Technical Design Documents (TDDs) that translate PRD requirements into implementation-ready architecture using existing frameworks, SDKs, and built product capabilities — with zero custom code.

This document defines exactly how an LLM agent must work, what it must output, and the validation loop it must run before calling a TDD ready.

## Your Audience

Engineers who will implement the feature. They need:

- Clear architecture decisions with rationale
- Exact SDK/framework APIs to use, with documentation references
- What already exists in the codebase that they should reuse
- How each design decision maps to a PRD requirement
- ASCII architecture diagram showing component relationships

## Your Job

Design HOW to build what the PRD defines. You are not deciding WHAT to build — that was decided in the PRD. Your job is:

- **How** to implement each PRD requirement using existing tools 
-**How** to keep the codebase healthy for maintainablity
- **Where** in the codebase each piece lives
- **Which** SDK/framework APIs to use (with doc references)
- **What** already exists that can be reused
- **Why** each architectural decision was made

## When to Use This Standard

Use this standard when asked to:

- Create a new TDD for a feature with an approved PRD.
- Review or improve an existing TDD.
- Evaluate whether a TDD is implementation-ready.

## Input Contract (Required Before Designing)

### Mandatory inputs:

- Approved PRD with requirement IDs (REQ-xxx). **A PRD is required. If no PRD exists, the agent MUST create one first using the `prd` skill before proceeding with the TDD.** A TDD without a PRD is not valid.
- Access to the codebase to identify existing capabilities.

### Research phase (agent must complete before designing):

For every framework/SDK the design will use, the agent MUST:

1. Fetch or read the official documentation for the relevant APIs.
2. Identify the specific built-in capabilities that fulfill each requirement.
3. Record the documentation reference (URL or section name) for each capability used.
4. Verify the API exists in the version currently used by the project.

Example: If designing with Next.js App Router, fetch the Next.js docs for route handlers, server actions, middleware — whatever the design touches. If using Supabase, fetch the Supabase docs for RLS, auth, realtime — whatever applies. If using Graphiti, fetch the Graphiti docs. If using Python, fetch the relevant Python library docs.

The research is not optional. The agent must be able to cite where every design decision comes from.

## Core Principles

1. **Zero custom code:** Use SDK/framework built-in capabilities for everything. Glue code to connect framework features is acceptable. Reimplementing behavior that exists in an approved dependency is not.
2. **Reuse what exists:** Before designing anything new, check what the product already has. If a component, utility, pattern, or API already exists in the codebase, use it.
3. **PRD traceability:** Every design decision must map to a PRD requirement. If a design element doesn't trace to a requirement, it shouldn't exist.
4. **Research-backed:** Every SDK/framework usage must reference official documentation. An engineer should be able to follow the reference to verify the approach.
5. **Server-side by default:** All logic runs on the server unless there is a specific, documented reason it must run on the client.
6. **No fallbacks:** Design for the correct path. Do not add defensive workarounds, feature flags for "just in case," or fallback implementations. If the primary approach fails, that's a bug to fix — not a fallback to catch.
7. **Error logging everywhere:** Every failure point must have explicit error logging. Errors must be visible and actionable, not swallowed.

## Non-Negotiable Rules

1. A PRD is required. If no PRD exists, create one first using the `prd` skill. Do not design without requirements.
2. Complete the research phase before designing. Do not design without documentation.
3. Do not introduce custom code where an SDK/framework capability exists.
4. Do not design features not in the PRD.
5. Do not add fallback logic or defensive workarounds.
6. Do not add monitoring, observability, rollback plans, or test strategy (those are handled by separate processes).
7. Every failure point must have error logging.
8. Security considerations are mandatory for every TDD.
9. Include an ASCII architecture diagram.
10. Reference official documentation for every framework/SDK decision.
11. Identify and list existing codebase capabilities that will be reused.
12. All logic is server-side unless explicitly justified otherwise.

## Mandatory Workflow

### Step 0: Read the PRD

- Load and read the PRD in full.
- Extract all requirement IDs (REQ-xxx) and their acceptance criteria.
- Note the Who column (roles/plans) — this informs access control design.
- Note Technical Considerations section if present.

Gate:

- PRD is read and all requirement IDs are captured.

### Step 1: Research Phase — Gather Framework/SDK Documentation

For every technology the design will touch:

- Fetch or read the official documentation.
- Identify specific APIs, methods, configurations that apply.
- Record documentation references (URLs or section names).
- Verify version compatibility with the project.

What to research depends on the feature:

- **Next.js**: Route handlers, server actions, middleware, caching, App Router patterns
- **Supabase**: RLS policies, auth helpers, realtime, edge functions, database functions
- **Vercel AI SDK**: Streaming, tool calling, message format, provider configuration
- **React**: Server components, client components, hooks, context
- **TypeScript**: Type patterns, generics, utility types relevant to the design
- **Any other SDK/framework**: The specific APIs the design will use

Gate:

- Documentation references exist for every framework/SDK the design touches.

### Step 2: Audit Existing Codebase

Before designing anything new, identify:

- Existing components that can be reused.
- Existing patterns (data fetching, state management, API routes) to follow.
- Existing utilities, hooks, or helpers that apply.
- Existing database tables, RLS policies, or auth flows to extend.

Document what exists and how the new design will integrate with it.

Gate:

- Existing capabilities are documented. No design reinvents what already exists.

### Step 3: Map PRD Requirements to Design Decisions

For each PRD requirement (REQ-xxx):

- Identify which SDK/framework capabilities fulfill it.
- Reference the documentation for those capabilities.
- Identify which existing codebase features support it.
- Determine if it requires server-side or client-side execution (prefer server).

Present this as a traceability table:


| PRD Requirement | Design Approach        | SDK/Framework | Doc Reference     | Existing Code to Reuse |
| --------------- | ---------------------- | ------------- | ----------------- | ---------------------- |
| REQ-001         | [How it's implemented] | [Which SDK]   | [Doc URL/section] | [What exists]          |


Gate:

- Every REQ-xxx has a row in the traceability table.

### Step 4: Design the Architecture

- Create an ASCII architecture diagram showing component relationships.
- Define data flow between components.
- Define API contracts (routes, request/response shapes).
- Define database changes (tables, columns, RLS policies) if applicable.
- Specify error handling and logging at every failure point.

Gate:

- Architecture diagram exists.
- Data flow is clear.
- Error logging is defined for every failure point.

### Step 5: Security Considerations

For every TDD, document:

- Authentication: How is the user verified?
- Authorization: How are roles/plans enforced? (Map from PRD Who column)
- Data access: What RLS policies apply?
- Input validation: Where and how is input validated?
- Sensitive data: What PII or secrets are involved? How are they protected?

Gate:

- Security is explicitly addressed. No "N/A" without justification.

### Step 6: Custom Code Audit

Review the entire design and flag any custom code:

- For each piece of custom logic, ask: does an SDK/framework capability already do this?
- If yes: replace with the SDK/framework capability and reference the docs.
- If no: document why custom code is necessary, what SDK gap it fills, and keep it minimal.
- Glue code (connecting two framework features) is acceptable — document what it connects and why.

Gate:

- Zero custom code, or every instance of custom code has explicit justification with SDK gap documentation.

### Step 7: Implementation Plan

Define the implementation order:

- Phases with clear deliverables.
- Dependencies between phases.
- What can be built in parallel.

Do NOT include: time estimates, monitoring setup, rollback plans, or test strategy.

Gate:

- Implementation order is clear and dependencies are explicit.

## Output Contract (Required Sections)

Use this exact order:

1. Document Control
2. PRD Reference
3. Research References (SDK/Framework Documentation)
4. Existing Codebase Capabilities
5. PRD-to-Design Traceability
6. Architecture (with ASCII diagram)
7. Technical Solution (API contracts, data flow, database changes)
8. Error Handling and Logging
9. Security Considerations
10. Custom Code Audit
11. Implementation Plan
12. Open Questions
13. Implementation Readiness Verdict
14. Validation Report

Use `references/tdd-template.md` when drafting.

## Validation

Score using `references/validation-rubric.md`.

Readiness threshold:

- All critical checks must pass.
- Rubric score must be >=14/16.

## Definition of Done

A TDD is implementation-ready only when:

- PRD is read and all requirements are traced.
- Research phase is complete with documentation references.
- Existing codebase capabilities are documented.
- Zero custom code (or justified exceptions).
- ASCII architecture diagram exists.
- Error logging is defined for every failure point.
- Security is explicitly addressed.
- No fallback logic exists.
- All logic is server-side unless justified.
- Validation rubric passes.
- Founder approval is recorded.

