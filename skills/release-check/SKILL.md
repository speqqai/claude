# LLM Agent Release Gate Checklist

---

## Section 1 — Custom Code

### Definition: What Is Custom Code?

Custom code is **any logic you write yourself** that replicates behavior already available in an installed dependency.

**Examples of custom code** (this list is not exhaustive):
- Utility functions: formatters, parsers, transformers
- HTTP or fetch wrappers
- Retry, timeout, or backoff logic
- Auth, session, or token handling
- Error classes or error handling abstractions
- Logging or telemetry wrappers
- Schema validators or type guards
- Queue, rate limiter, or concurrency helpers
- Any abstraction placed on top of an existing SDK or library

**The test:** If the logic is yours and a dependency could have done it — it is custom code. It must be justified before it is written.

---

### The 3-Tier Priority Rule

You must work through all three tiers **in order**. Moving to the next tier requires documented proof that the current tier was exhausted. There are no exceptions.

---

#### Tier 1 — First-Party SDKs ✅ Check this first, always

A first-party SDK is the official client library published by a vendor whose service is already integrated into the project (e.g. Anthropic SDK, Supabase client, Stripe SDK, Clerk).

- [ ] I have listed every first-party SDK currently installed in this project
- [ ] I have reviewed the **full API surface** of each relevant SDK — not just the parts I have used before
- [ ] The capability does **not** exist in any currently installed first-party SDK
- [ ] The capability **cannot** be achieved by composing two or more existing first-party SDK calls
- [ ] I have written this statement in the PR: *"I checked [SDK name] and it does not cover this because [specific reason]"*

> ⛔ Do not proceed to Tier 2 until all Tier 1 boxes are checked and documented.

---

#### Tier 2 — Established OSS Libraries ✅ Check this second

An OSS library is a well-adopted open source package that is not first-party but is widely trusted in the ecosystem.

- [ ] I have searched for an OSS library that covers the exact capability needed
- [ ] The library is scoped to the **narrowest possible use** — I am not importing it wholesale when only one function is needed
- [ ] The library meets **all three** adoption criteria:
  - [ ] Actively maintained: at least one commit within the last 90 days
  - [ ] Community trust: 1,000+ GitHub stars
  - [ ] Security: no critical or high-severity open CVEs
- [ ] The library does **not** duplicate functionality already provided by a Tier 1 SDK in the project
- [ ] I have written this statement in the PR: *"No Tier 1 SDK covers this. I am proposing [library name] because [specific reason]. It meets all three adoption criteria."*
- [ ] A second engineer has reviewed and approved this new dependency before it was added

> ⛔ Do not proceed to Tier 3 until all Tier 2 boxes are checked and documented.

---

#### Tier 3 — Custom Code ✅ Last resort only

Custom code is permitted only when Tier 1 and Tier 2 are both fully exhausted with written evidence in the PR.

- [ ] Written Tier 1 exhaustion statement exists in the PR
- [ ] Written Tier 2 exhaustion statement exists in the PR
- [ ] The custom code is **scoped to the minimum logic required** — no speculative helpers, no "we might need this later" abstractions
- [ ] Every custom file or function has this comment block at the top:

```
// CUSTOM CODE — TIER 3 APPROVED
// Capability: [what this does in one sentence]
// Tier 1 checked: [SDK name(s)] — not covered because [reason]
// Tier 2 checked: [library name(s) evaluated] — not used because [reason]
// Approved by: [engineer name] on [date]
```

- [ ] The custom code has a corresponding unit test
- [ ] A second engineer has reviewed and approved this code before merge
- [ ] A ticket exists to revisit and replace this code if a Tier 1 or Tier 2 solution becomes available

---

## Section 2 — Dead and Obsolete Code

- [ ] No files exist that are not imported or referenced anywhere in the codebase
- [ ] No commented-out code blocks exist in any modified file
- [ ] No `TODO`, `FIXME`, or `HACK` comments remain for work that was scoped to this PR
- [ ] No unused variables, functions, or exports exist in any modified file
- [ ] No unused `import` or `require` statements exist in any modified file
- [ ] No feature flags or conditional branches reference deprecated or removed features
- [ ] No migration scripts, seed files, or one-time utilities from previous releases remain
- [ ] No stale types, interfaces, or enums exist that no longer map to any active data structure
- [ ] No orphaned test files exist for deleted modules or functions
- [ ] `package.json` has no unused dependencies (verified via `depcheck` or equivalent)

---

## Section 3 — No ESLint Ignores

- [ ] Zero `// eslint-disable` comments in the diff
- [ ] Zero `// eslint-disable-next-line` comments in the diff
- [ ] Zero `/* eslint-disable */` block comments in the diff
- [ ] Zero new entries added to `.eslintignore` in the diff
- [ ] Zero `eslintIgnore` additions to `package.json` in the diff
- [ ] Any pre-existing ESLint ignore that was touched in this PR has been resolved and removed

---

## Section 4 — Type Safety

- [ ] No `any` types introduced — use `unknown` with a type guard, or a typed interface
- [ ] No non-null assertions (`!`) without an inline comment explaining why null is impossible at that callsite
- [ ] No `@ts-ignore` or `@ts-expect-error` without a written explanation and a linked ticket to remove it

---

## Section 5 — Tests

- [ ] Every new function or module has at least one unit test
- [ ] Every new API route or agent tool has at least one integration test: one for the happy path, one for a failure case
- [ ] No test file uses `skip`, `xtest`, or `xit` without a linked ticket explaining why
- [ ] Test coverage has not decreased from the baseline on the main branch

---

## Section 6 — Environment and Secrets

- [ ] No secrets, API keys, or tokens are hardcoded anywhere in the diff
- [ ] All new environment variables are documented in `.env.example` with a plain-language description
- [ ] All new environment variables are validated at startup using a schema (e.g. Zod `process.env` parse)

---

## Section 7 — Database and State

- [ ] All new queries use parameterized inputs — no string interpolation into SQL or query builders
- [ ] All new migrations are reversible: a `down` migration exists for every `up`
- [ ] No migration alters or drops a column without a confirmed backfill strategy documented in the PR

---

## Section 8 — Agent-Specific

- [ ] Every tool the agent can call has explicit input validation before execution
- [ ] Every agent action that touches user data is logged with enough context to replay or audit
- [ ] All agent error paths return a structured, typed error response — no raw exception propagation to the user