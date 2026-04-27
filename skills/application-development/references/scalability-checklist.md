# Scalability Checklist

Use this checklist when implementing or reviewing app architecture.

## 1. Structure and boundaries

- [ ] Feature modules own their UI, state, and domain actions.
- [ ] Shared libs are intentional and do not become a dumping ground.
- [ ] API contracts are typed and versionable.

## 2. Data and rendering strategy

- [ ] Rendering mode (static, dynamic, streaming) is chosen intentionally per route.
- [ ] Data fetching occurs as close to the server boundary as possible.
- [ ] Caching/revalidation strategy is explicit and documented.

## 3. Reliability and observability

- [ ] Error boundaries and not-found states exist for critical routes.
- [ ] Logs include enough context to debug production issues.
- [ ] Important user flows are covered by integration or E2E tests.

## 4. Performance

- [ ] Large components are split to reduce initial bundle size.
- [ ] Heavy client-side work is deferred or moved server-side.
- [ ] Core Web Vitals are measured and tracked for regressions.

## 5. Security and configuration

- [ ] Secrets never ship to the client bundle.
- [ ] Server/client environment variables are correctly separated.
- [ ] AuthZ/AuthN checks happen on trusted server boundaries.

## 6. Change management

- [ ] Major tradeoffs are documented near the code.
- [ ] New dependencies have size/maintenance justification.
- [ ] Rollback strategy exists for risky releases.
