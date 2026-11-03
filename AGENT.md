# AGENT.md — XenoDactyl AI Engineering Rules
## Strict instructions for Nemotron Super VL and any coding agent

This file is a binding engineering contract. Follow it before editing code. **Repository evidence outranks assumptions, model memory, and plausible guesses.**

---

## 1. Anti-hallucination policy — STOP if unsure

You MUST NOT:
- Invent files, paths, classes, methods, routes, tables, columns, environment variables, API endpoints, provider payloads, credentials, status values, or package APIs.
- Claim a command, build, test, migration, endpoint, UI action, or integration works unless you actually ran it or have concrete repository evidence.
- Build fake data-driven dashboards, placeholder CRUD, nonfunctional buttons, fake payment success, pretend API responses, or simulated provisioning as production behavior.
- Assume Pterodactyl version, Laravel conventions, React setup, Tailwind version, shadcn configuration, or deployment capabilities.
- replace working base logic with an approximate rewrite.
- Mark work complete merely because the page looks attractive.

When evidence is missing:
1. Stop the affected implementation.
2. Search the repository and official documentation available to you.
3. Record the exact unknown and why it blocks progress.
4. Ask the user for clarification or provider docs when required.
5. If permitted to continue, implement only a safe interface/stub that is explicitly non-production and cannot report success.

Never fill unknowns with a guess.

## 2. Mandatory repository audit before coding
First inspect and report:
- Repository root and branch/status; do not discard uncommitted user work.
- Base repository identity and exact commit/release, if available.
- Backend framework and versions from manifests.
- Frontend framework and versions.
- Package manager and lockfile.
- Tailwind version/configuration.
- Existing UI component libraries and shadcn setup.
- Route definitions, API patterns, auth/permission conventions.
- Database schema/migrations and relevant models.
- Existing server lifecycle, queues, jobs, events, and tests.
- Build/test/lint commands discovered from package/composer scripts.
- Deployment/runtime constraints.

Use actual file contents. Do not report inferred paths as inspected. Establish a feature inventory and baseline checks. If repository is unavailable, request it; do not pretend audit was completed.

## 3. Preserve the Pterodactyl foundation
- Treat the base as a production application with established behavior.
- Preserve every existing feature, route, API contract, authorization check, server workflow, and integration unless explicitly scoped for change.
- Redesign UI in place where possible; do not rewrite business logic for visual convenience.
- Avoid unrelated refactors.
- For behavior changes, state old behavior, desired behavior, migration impact, and test evidence.
- Before removing/renaming anything, search all usages and confirm compatibility.

## 4. Requirements traceability
For each task, maintain a compact matrix:
| Requirement | Existing evidence | Files affected | Implementation | Test/verification | Status |
Use statuses: `NOT STARTED`, `IN PROGRESS`, `BLOCKED`, `VERIFIED`.
A requirement is `VERIFIED` only with real evidence.
Link every UI control to a requirement and its backend/domain action.
No feature may be marked complete without verification.

## 5. System-first workflow (not page-first)
For every feature:
1. Understand user story and acceptance criteria.
2. Inspect existing domain model, route, API, authz, database, jobs/events, and UI.
3. Define flow and state transitions.
4. Identify data ownership and permissions.
5. Design/implement persistence and backend behavior.
6. Implement API/controller/service following established conventions.
7. Implement frontend states and actions.
8. Add tests for success, invalid input, unauthorized access, errors, retries/idempotency where relevant.
9. Verify visual consistency and responsive behavior.
10. Report evidence, limitations, and remaining work.

Do not create a page first and retrofit the system afterward.

## 6. Visual quality protocol
Before implementing UI:
- Inspect current screens and shared layouts.
- Define or reuse XenoDactyl tokens and components.
- Review existing shadcn/ui setup and Tailwind compatibility; never blindly install latest or upgrade major dependencies.
- Build a coherent hierarchy: page title/description, navigation, primary action, content grouping, data density, status semantics.
- Provide mobile/tablet/desktop behavior.
- Implement skeleton/loading, empty, error, validation, and success states.
- Use semantic elements, keyboard interactions, accessible labels, focus, contrast.
- Avoid “generic AI page” patterns: arbitrary gradient hero on every page, excessive cards, giant empty spaces, inconsistent icons, decorative charts with no real data, repeated placeholder tables, and random accent colors.
- Match design tokens across Client Area and Admin Area.
- Do not use shadcn/ui as a name-only dependency; use its components coherently with project conventions.

For substantial screens, perform visual review using screenshots/browser if available. If visual review tools are unavailable, say so and provide code-based verification; do not claim screenshot review.

## 7. Data integrity and functional UI
- Each action must invoke a real, authorized backend operation.
- Use persisted records and real API results.
- Every form must validate on server and present errors.
- Destructive actions require confirmation and permission.
- Loading/empty/error states must reflect actual API states.
- Never hardcode production metrics or pretend rows.
- Avoid dead links and inert buttons.
- A temporary placeholder must be conspicuously labeled and not present as finished functionality.
- Handle pagination and API errors where data can be large or remote.

## 8. Multi-Panel safety
- Panel connection configuration requires verified API approach and explicit credential scopes.
- Never expose admin API secrets to frontend.
- Validate URLs and prevent SSRF/outbound access to prohibited hosts according to policy.
- Map customer accounts and servers explicitly; do not assume matching email means ownership without verified mapping logic.
- Check authorization on every server action.
- Handle source panel outage, rate limit, timeout, and partial results.
- Document panel-specific capabilities/limitations.
- Do not state that only a panel URL is enough if API credentials and identity mapping are required.

## 9. Plugin architecture safety
- Follow a versioned plugin contract discovered/designed with repository conventions.
- Plugins declare identity, version, compatibility, permissions, config schema, and capability hooks.
- Plugin-owned data must be isolated and migrations reversible/documented.
- Core should not import every optional service implementation directly.
- Plugin errors must not take down unrelated Core functionality.
- No arbitrary unrestricted PHP execution from untrusted ZIP uploads.
- If sandboxing/trust/signature/rollback cannot be demonstrated, do not implement open plugin ZIP execution. Use vetted server-side installation or allowlisted plugins and explain limitation.
- Never give plugins unrestricted database access or secrets by default.
- Test install/enable/disable/update/uninstall and compatibility failure paths.

## 10. Billing/payment security
- Provider docs must be verified. Cite official documentation in integration notes (URL and access date).
- Do not fabricate API endpoints, fields, signature schemes, callback retries, or payment statuses.
- Separate order, invoice, payment attempt, provider event, and fulfillment.
- Amount/currency/reference must be validated server-side.
- Webhook authenticity and idempotency are mandatory.
- Redirect query parameters are not payment proof.
- Repeated webhooks must not double-pay or double-provision.
- Keep raw webhook payloads only under privacy/security policy; redact sensitive data.
- Payment and fulfillment statuses are separate.
- No production-ready claim for unverified gateway integration.

## 11. Coding standards and dependency discipline
- Follow actual repository style, lint, formatter, naming, and architecture.
- Use the installed package manager and lockfile.
- Do not add dependencies without explaining necessity, version compatibility, security impact, and alternative.
- Avoid broad dependency upgrades.
- Keep changes focused and small enough to review.
- Never delete lockfiles or user modifications without approval.
- Add comments for non-obvious safety/compatibility decisions, not obvious code.
- Secrets must never be committed.

## 12. Verification commands and honesty
Discover commands from repository manifests/scripts first. Run appropriate:
- backend unit/feature tests
- frontend type check/lint
- build
- migrations on safe test database if available
- integration tests/mocks for external providers
- relevant manual verification

If a command fails:
- Show exact command and summarized failure.
- Do not conceal or claim success.
If not run:
- Mark `NOT RUN` and state why.
Do not run destructive commands against user data.

## 13. Vercel / deployment claims
The base Pterodactyl/Laravel system may require persistent PHP runtime, queues, storage, WebSockets, and network access to Wings/panel infrastructure. Do not assume it is compatible with Vercel serverless.
- Audit actual deployment needs.
- Record architecture decision and tested limits.
- Only claim Vercel support for components proven compatible.
- Preserve XAMPP Windows local-development guidance and Linux VPS production guidance if those environments are part of project requirements.

## 14. Completion report format
At end of each task, report:
1. What changed (specific files/modules).
2. Requirement(s) fulfilled.
3. System behavior implemented.
4. Tests/build/typecheck actually run and results.
5. UI review performed (and viewport/tool limitations).
6. Unverified items or blockers.
7. Any migrations/configuration required.
8. Explicit status: VERIFIED / PARTIALLY VERIFIED / BLOCKED.

Never say “done, everything works” without evidence.
