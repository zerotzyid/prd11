# PRD.md — XenoDactyl
## Product Requirements Document

**Status:** Implementation baseline  
**Project:** XenoDactyl — Centralized Multi-Panel Hosting Client Area & Modular Hosting Platform  
**Primary AI coding model:** Nemotron Super VL  
**Base project:** Official Pterodactyl Panel repository/fork (Laravel + React + TypeScript)  
**Document rule:** This PRD is a requirements source, not evidence that a feature already exists.

---

## 1. Product vision

XenoDactyl is a centralized customer and administration platform built on the Pterodactyl Panel foundation. It lets an operator connect multiple Pterodactyl panels and present their customers with one consistent Client Area. The Core must work independently as a **Client Area Only** product. Additional products such as VPS, Nokos, Domain, RDP, and other provider-backed services are optional modules installed through a controlled Plugin System.

The platform must not be implemented as a collection of disconnected mock pages. Pages are interfaces to real domain workflows, persistence, permissions, APIs, and state transitions.

### 1.1 Product modes
- **Mode A — Client Area Only:** Core, account/customer experience, centralized billing as configured, and Multi-Panel integration. No VPS/Nokos/Domain plugin is required.
- **Mode B — Full Hosting Platform:** Mode A plus optional service plugins and payment gateway plugins.
- Both modes are the same application. Mode B is enabled by installing/configuring modules, not by maintaining a separate fork.

### 1.2 Primary users
- Customer: buys, pays for, and manages services.
- Operator/Admin: manages panels, catalog, orders, invoices, plugins, payment integrations, and settings.
- Support/Staff: handles permitted customer/service support actions.
- Plugin developer: implements a documented plugin contract with restricted capabilities.

## 2. Non-negotiable product principles
1. Preserve existing Pterodactyl features, routes, APIs, authentication, authorization, server lifecycle, and business logic unless a requirement explicitly authorizes a change.
2. Audit the actual repository before implementation. Do not assume version, file path, component, schema, package, or route.
3. A visual redesign must not remove or silently alter behavior.
4. Every implemented page must be connected to real data and real actions, or clearly marked as intentionally unavailable. No fake success states, fake metrics, placeholder CRUD, dead buttons, or invented provider responses.
5. Use a consistent XenoDactyl Design System and shadcn/ui components. Choose Tailwind version from repository audit; do not upgrade it blindly.
6. Core is useful without service plugins. Plugin failure must not break unrelated Core functions.
7. Provider integration must be based on verified official documentation or repository evidence. Unknown behavior is a blocking unknown, not permission to guess.
8. Security, tenancy boundaries, permission checks, webhook validation, and auditability are first-class requirements.
9. AI-generated code must be tested and verified against evidence. A polished page is not considered complete if the system flow behind it is absent.
10. Avoid claiming Vercel support for the full Laravel/Pterodactyl stack unless deployment architecture and all runtime requirements have been proven. Vercel may be considered for a separately deployable frontend or compatible component only after technical validation.

## 3. Scope

### 3.1 Core — required
- Existing Pterodactyl base functionality preserved.
- Client Area and Admin Area.
- Authentication and account/profile management, using existing base implementation and documented additions.
- Centralized service catalog and product/order/invoice primitives where compatible with base.
- Multi-Panel Management for connecting multiple Pterodactyl panels.
- Plugin Manager and plugin lifecycle/compatibility interface.
- Billing Core independent of payment providers.
- Payment gateway abstraction and payment attempt records.
- Theme system: Light, Dark, System; responsive design.
- XenoDactyl Design System and shadcn/ui-based UI overhaul.
- API integration and configuration management.
- Notifications and audit logs for important administrative/security events.
- Landing page and SEO functionality, subject to existing framework and repository architecture.
- Documentation, tests, and safe migrations.

### 3.2 Optional service plugins
- VPS service.
- Nokos / phone-number service (provider terminology and legal/operational suitability must be verified).
- Panel Hosting / Pterodactyl provisioning beyond connected panels.
- Domain service.
- RDP service.
- Other provider-backed products.

These are optional. Do not hard-code all service domains into Core.

### 3.3 Payment gateway plugins
Target integrations:
- PixPayz
- MustikaPayment
- Tako.id
- Midtrans
- OrderKouta
- Additional gateways installed as plugins

**Important:** Do not invent API endpoints, request fields, signatures, statuses, retry rules, or credentials for any provider. Verify current official docs. If official docs cannot be found or access is unavailable, implement the provider adapter only after the operator supplies valid documentation; otherwise create a documented integration stub that cannot be enabled in production.

### 3.4 Out of scope unless explicitly approved
- Replacing Pterodactyl's daemon/Wings architecture.
- Rewriting the application into another framework.
- Removing existing base features to simplify the UI.
- Unverified automatic payment settlement or fulfillment.
- Arbitrary third-party plugin code running with unrestricted application/database access.
- Claiming support for a deployment target without testing it.

## 4. Architecture overview

### 4.1 Core domains
- Identity and authorization
- Customer area
- Admin area
- Multi-Panel manager
- Product/catalog and order orchestration
- Billing Core
- Plugin Manager and plugin contracts
- Notifications/audit
- UI Design System

### 4.2 Conceptual workflow
Customer -> Catalog -> Order -> Invoice -> Payment attempt -> Verified payment event -> Billing Core marks paid -> Core/Plugin fulfillment -> Service record -> Customer service view.

Payment status and fulfillment status must be separate:
- Payment: draft, unpaid, pending, paid, expired, cancelled, failed, refunded (only if supported and verified).
- Fulfillment: queued, provisioning, active, suspended, failed, terminating, terminated.
These are proposed domain states; reconcile with actual base schema and provider semantics before migration.

### 4.3 Multi-Panel
Admin can configure a panel connection with:
- Display name
- Base URL
- Credential reference (secret stored securely; never rendered back in full)
- Connection status and validation timestamp
- Enabled/disabled state
- Optional routing/availability metadata

Requirements:
- Validate URL and connection before marking healthy.
- Map customer identity and server ownership explicitly.
- Present connected servers in a unified Client Area.
- Enforce access checks at XenoDactyl and the source panel.
- Handle pagination, API rate limits, timeouts, and panel unavailability.
- Never expose panel admin credentials to browser/client.
- Do not imply that adding a URL alone is sufficient; required API credential scopes and identity mapping must be documented.

### 4.4 Plugin System
Plugin capabilities are defined by a versioned contract and permissions:
- Plugin ID, name, version, vendor, compatibility range, entry points
- Declared permissions/capabilities
- Settings schema
- Admin navigation and customer-facing product/service extension points
- Product provisioning, lifecycle, status retrieval, optional suspend/renew/terminate
- API/webhook registration only through approved internal interfaces

Plugin Manager requirements:
- Install/enable/disable/update/remove with checks and audit events.
- Validate manifest, version compatibility, dependencies, and declared permissions.
- Keep secrets in protected configuration.
- Plugin errors isolated from Core.
- Provide an uninstall plan for plugin-owned data.
- Do not support unsafe ZIP upload execution without a trust/security model, signature or trusted-source policy, sandbox/restricted capabilities, and rollback strategy. If this cannot be safely implemented in the current stack, provide an admin-only vetted installation workflow.

## 5. Billing and payment

### 5.1 Billing Core
- Product plan pricing and payment gateway choice are separate concepts.
- Create order and invoice with immutable amount/currency snapshots.
- Track payment attempts separately; one invoice may have multiple attempts.
- Validate amount, currency, invoice reference, provider event identity, and authenticity before marking paid.
- Idempotent webhook processing; repeated event cannot double-fulfill.
- Record webhook and reconciliation outcomes.
- Do not treat browser redirect/return URL as proof of payment.
- Fulfillment runs through a queue/retry-safe mechanism where supported.
- Failed/expired attempts can be retried with a new attempt; do not switch provider on an in-flight transaction without a defined safe state.
- Admin can inspect invoice, attempts, events, and fulfillment result with permission checks.

### 5.2 Gateway adapter contract
Proposed conceptual methods, to be reconciled with coding conventions:
- identifier/name/capabilities
- create payment attempt
- query payment status
- verify incoming callback/webhook
- normalize provider event
- optional refund only when officially supported

Do not copy this interface verbatim before inspecting repository conventions.

### 5.3 Provider verification matrix
For each gateway, document:
- Official documentation URL and date accessed
- API authentication
- Create-payment flow
- Callback/webhook signature requirements
- Status mapping
- Retry/replay behavior
- Amount/currency format
- Sandbox/test mode
- Error handling
- Tested cases

Unknown facts must be marked **UNVERIFIED** and must not be treated as implemented.

## 6. UI/UX requirements

### 6.1 Design system
- Full coherent redesign using shadcn/ui primitives, with compatibility to installed framework and Tailwind version.
- Establish design tokens for color, typography, spacing, radius, border, elevation, focus, and semantic states.
- Light/Dark/System theme.
- Responsive layouts for mobile, tablet, desktop.
- Accessibility: keyboard navigation, visible focus, semantic labels, sufficient contrast, accessible dialogs and form errors.
- Consistent navigation, page headers, breadcrumbs where useful, loading/empty/error/success states.
- Avoid excessive decoration, inconsistent cards, and arbitrary styling per page.

### 6.2 Required page quality
Every page must have:
- Purpose and user role
- Real data source or explicitly identified no-data state
- Actual primary and secondary actions
- Loading, empty, error, and success feedback as applicable
- Validation and authorization
- Responsive behavior
- Consistent visual hierarchy
- Tests or verification steps

### 6.3 UI is not the system
A page is not done because it renders. Any UI action must connect to a working endpoint/domain service, persisted state, permission check, and user feedback. If a backend action is not yet implemented, show disabled/unavailable state with explanation; do not simulate completion.

## 7. Technical baseline and environment
- Base: official Pterodactyl Panel codebase; exact release/commit must be recorded before work.
- Backend: Laravel/PHP as found in repository.
- Frontend: React/TypeScript as found in repository.
- Database: MySQL/MariaDB; local Windows development via XAMPP, production on Linux VPS.
- Use the package manager and versions already present unless a justified, reviewed change is approved.
- Tailwind and build tooling must be audited before selecting shadcn/ui setup.
- Preserve supported Pterodactyl deployment requirements.
- Vercel: full stack support is not assumed. Any Vercel requirement needs an architecture decision and proof against PHP runtime, queues, persistent processes, WebSockets, storage, and Wings connectivity. Do not silently claim compatibility.

## 8. SEO and landing page
- Landing page metadata, titles, descriptions, canonical URL where appropriate, Open Graph tags, sitemap/robots where framework supports them.
- Product pages have meaningful indexable content when public.
- Admin/customer private pages must not be indexed.
- No fabricated SEO statistics or fake content.
- Ensure performance and semantic markup.

## 9. Security requirements
- Use existing Laravel security mechanisms and authorization patterns.
- Validate all inputs server-side.
- Protect secrets; redact logs; do not return secrets to UI.
- CSRF/session protections consistent with base.
- Rate limit sensitive endpoints.
- Verify webhooks; timestamp/replay protection where provider supports it.
- Idempotency for payment and fulfillment.
- Audit admin actions.
- Secure plugin permissions and dependency handling.
- Prevent SSRF through panel URL configuration (scheme/host validation, private network policy, DNS/rebinding considerations, controlled outbound requests).
- Avoid arbitrary file upload and code execution.
- Keep tenant/customer boundaries explicit.

## 10. Acceptance criteria
A release is acceptable only when:
1. Base Pterodactyl features and routes have been inventoried and regression-tested.
2. Multi-Panel connection can be configured, validated, and safely displayed.
3. Customer only sees services authorized to their account.
4. Core runs without VPS/Nokos/Domain plugins.
5. Plugin contracts and lifecycle checks are documented and tested.
6. Billing creates orders/invoices/payment attempts without fake settlement.
7. Payment can be marked paid only after verified provider event or verified status query.
8. Duplicate callbacks do not duplicate fulfillment.
9. UI pages are consistent, responsive, accessible, and connected to working flows.
10. Light/Dark/System themes work across all major screens.
11. Loading/empty/error states exist for data-backed pages.
12. Migrations, tests, build, and static checks pass; failures are reported honestly.
13. Setup and deployment documentation is accurate for tested environments.
14. Unknown/unavailable integrations are explicitly documented rather than fabricated.

## 11. Delivery phases
- Phase 0: Repository audit, requirements traceability, architecture decisions, baseline tests.
- Phase 1: Design system and safe Pterodactyl UI redesign (preserve behaviors).
- Phase 2: Multi-Panel connection and unified service view.
- Phase 3: Billing Core and payment gateway abstraction.
- Phase 4: Plugin Manager contracts/lifecycle and first verified plugin(s).
- Phase 5: VPS/Nokos/other service plugins based on provider docs.
- Phase 6: SEO, hardening, accessibility, performance, deployment verification.

Do not start later phases while blocking foundations are unverified.

## 12. Required documentation per integration
Each integration must have: overview, official sources, configuration fields, secret handling, workflow diagram, status mapping, permission requirements, error/retry policy, tests, limitations, and setup steps.
