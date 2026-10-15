# Health Watchers — GitHub Issues for Contributors

> 125 curated issues spanning design, frontend, backend, DevOps, documentation, testing, refactoring, and more.

---

## [Docs] Add interactive system architecture diagram

**Description:**
The project lacks a visual, interactive overview of the full system — how the Next.js frontend, Express API, Stellar service, MongoDB, and mobile app communicate. New contributors struggle to orient themselves.

**Tasks:**
- Create a C4-style diagram covering: web, API, stellar-service, mobile, MongoDB, and Stellar network
- Show data flow for patient registration, encounter creation, and payment processing
- Illustrate auth flow: JWT issuance → middleware → refresh rotation
- Export as both SVG (for README embed) and Mermaid source (for easy diffs)
- Keep diagram in `docs/ARCHITECTURE.md` and link from README
- Add a second diagram specifically for the Stellar payment lifecycle (initiate → batch → reconcile → receipt)

---

## [Docs] Write contributor onboarding guide for first-time contributors

**Description:**
`CONTRIBUTING.md` exists but is high-level. New contributors need a step-by-step guide that gets them to a working local environment with real data within 15 minutes.

**Tasks:**
- Document exact prerequisites (Node 18, Docker, etc.) with version check commands
- Write a "from clone to first test pass" walkthrough
- Cover the monorepo structure and Turbo task graph
- Document the seed script and what data it creates
- Add a "good first issues" section with links
- Add troubleshooting steps for the most common setup errors (MongoDB connection, env vars, port conflicts)

---

## [Docs] Document all environment variables with descriptions and defaults

**Description:**
`.env.example` lists variable names but lacks descriptions for many of them. Contributors are unsure which are required, which are optional, and what format each expects.

**Tasks:**
- Go through every variable in `.env.example` and add an inline comment
- Group variables by service: API, web, stellar-service
- Mark required vs. optional
- Document valid values for enum-style variables (e.g., `STELLAR_NETWORK=testnet|mainnet`)
- Add a note about AWS Secrets Manager usage in production
- Keep in sync with `packages/config/index.ts` validation logic

---

## [Docs] Add API versioning documentation

**Description:**
`docs/API_DOCUMENTATION.md` and `apps/api/docs/api-versioning-strategy.md` describe the strategy, but there is no clear guide for consumers on how to handle version negotiation, deprecation notices, and migration paths.

**Tasks:**
- Document the current versioning scheme (header vs. URL path)
- Write a migration guide template for breaking changes
- Add versioning examples to the Postman collection
- Link from the main API documentation
- Add sunset dates for any deprecated endpoints

---

## [Docs] Create a HIPAA compliance checklist for contributors

**Description:**
Health Watchers is HIPAA-compliant, but there is no checklist to help contributors understand what they must verify before merging a change that touches PHI (Protected Health Information).

**Tasks:**
- List all PHI fields in the patient, encounter, and lab-results models
- Document which fields are encrypted at rest and how to add new encrypted fields
- Add a checklist item to the PR template: "Does this change touch PHI? Have you verified encryption and audit logging?"
- Document the audit log model and when `createAuditLog` must be called
- Reference `SECURITY.md` and `docs/SECURITY_POLICY.md`

---

## [Docs] Write testing guide for each app in the monorepo

**Description:**
`docs/TESTING_GUIDE.md` exists but coverage is uneven across the apps. Developers need clear instructions for running unit, integration, contract, mutation, and E2E tests for each app independently.

**Tasks:**
- Document unit test patterns for the API modules
- Document how to run contract tests (`jest.contract.config.ts`) and what Pact providers/consumers are defined
- Document Playwright E2E setup and how to run tests against a local dev stack
- Document Stryker mutation testing and how to interpret the HTML report
- Add a section on accessibility tests (`playwright.a11y.config.ts`)

---

## [Docs] Add Stellar integration developer guide

**Description:**
The Stellar service is complex (batch payments, multi-sig, cold wallet, payment reconciliation, exchange rates, DEX) but lacks a developer guide explaining how the pieces fit together and how to test against Stellar testnet.

**Tasks:**
- Document how to fund a testnet account using Friendbot
- Explain the payment state machine and its states
- Document the batch processor and how to trigger a batch manually
- Explain the cold wallet architecture and signing flow
- Add a section on the payment reconciliation service
- Cover the exchange rate cache and refresh logic

---

## [Docs] Document database schema and index strategy

**Description:**
`docs/DATABASE_SCHEMA.md` exists but may be out of sync with the actual Mongoose models. Indexes are scattered across model files and migrations.

**Tasks:**
- Audit all Mongoose models and verify the schema doc is accurate
- Add an index strategy section: which queries each index supports
- Document the migration workflow end-to-end with an example
- Add an ERD (entity-relationship diagram) for the core entities
- Document the archive policy from `apps/api/src/modules/archive/ARCHIVAL_STRATEGY.md`

---

## [Docs] Add runbook for common operational failures

**Description:**
`monitoring/runbooks/` has runbooks for MongoDB and Stellar failures, but several common failure scenarios are missing.

**Tasks:**
- Add runbook: API pod crash loop (OOMKilled, config error)
- Add runbook: Stellar transaction submission failure
- Add runbook: Redis connection failure (affects rate limiting)
- Add runbook: JWT secret rotation
- Add runbook: Webhook delivery failure spike
- Ensure all runbooks follow the same template: symptoms → diagnosis → remediation → escalation

---

## [Docs] Publish Docusaurus documentation site

**Description:**
`apps/docs/` contains a Docusaurus setup with a skeleton of docs, but it is not deployed anywhere. The CI/CD pipeline does not build or deploy it.

**Tasks:**
- Verify `apps/docs` builds without errors
- Add a GitHub Actions job to build and deploy to GitHub Pages on merge to `main`
- Migrate key docs from the `docs/` folder into the Docusaurus structure
- Add a search plugin (Algolia or local)
- Link the deployed site from the README

---

## [CI/CD] Add environment-specific deployment gates to deploy workflow

**Description:**
`deploy.yml` deploys to environments but lacks explicit approval gates between staging and production.

**Tasks:**
- Add a `staging` environment with auto-deploy on merge to `main`
- Add a `production` environment requiring manual approval from a CODEOWNER
- Integrate smoke test execution between staging deploy and production approval
- Notify on Slack/Teams when production approval is pending
- Ensure rollback step is available as a manual workflow trigger

---

## [CI/CD] Add Lighthouse CI to measure web performance on every PR

**Description:**
There is no automated check for web performance regressions. A PR could silently degrade Lighthouse scores.

**Tasks:**
- Add `@lhci/cli` to the web app dev dependencies
- Create `.lighthouserc.js` with thresholds: performance ≥ 80, accessibility ≥ 95, best-practices ≥ 90
- Add a `lighthouse` job to `ci.yml` that builds the Next.js app and runs LHCI against it
- Post results as a PR comment with score deltas
- Fail the job if any score drops below the threshold

---

## [CI/CD] Add end-to-end test job to CI pipeline

**Description:**
Playwright E2E tests exist (`apps/web/e2e/`) but are not run in CI. Regressions in critical flows (auth, patient registration, payment) go undetected.

**Tasks:**
- Add an `e2e` job to `ci.yml` that runs after the `build` job
- Start the full stack using `docker-compose -f docker-compose.dev.yml`
- Wait for health endpoints to return 200 before running tests
- Run `npx playwright test` against all spec files in `apps/web/e2e/`
- Upload Playwright HTML report as a CI artifact
- Only run on PRs that touch `apps/web/` or `apps/api/`

---

## [CI/CD] Add Snyk or OWASP Dependency-Check to CI

**Description:**
`dependency-vulnerability-tracking.yml` exists but relies on GitHub Dependabot. A dedicated SCA (Software Composition Analysis) scan step with pass/fail gates is needed.

**Tasks:**
- Add Snyk or OWASP Dependency-Check as a CI step
- Fail the build if any critical or high CVE is introduced by a PR
- Allow suppression of false positives via a checked-in suppression file
- Post a summary comment on the PR listing new vulnerabilities
- Configure weekly full scans and report to the security team

---

## [CI/CD] Add database migration check to CI pipeline

**Description:**
Migrations run automatically before tests in CI, but there is no check to verify that every migration has a `down` function or that `migrate:status` is clean before merging.

**Tasks:**
- Add a `migrations` job that runs `migrate:status` and fails if any migration is in an unexpected state
- Lint migration files to ensure both `up` and `down` are exported
- Add a test that runs `up` followed by `down` for every new migration in a PR
- Document the migration CI process in the README

---

## [CI/CD] Publish Postman collection to public workspace on merge

**Description:**
The CI pipeline has a `postman-sync` job stub, but it requires `POSTMAN_API_KEY` and `POSTMAN_COLLECTION_UID` secrets to be configured and the job is not fully wired up.

**Tasks:**
- Document how to set the required secrets in the repository settings
- Complete the `postman-sync` job in `ci.yml`
- Run `scripts/generate-postman.js` to regenerate the collection from source before syncing
- Add a validation step that checks the collection against the OpenAPI spec
- Post a link to the updated workspace in the PR comment

---

## [CI/CD] Add Helm chart linting and dry-run to CI

**Description:**
`helm/health-watchers/` contains a full Helm chart but there is no CI step that lints it or performs a dry-run against a test cluster.

**Tasks:**
- Add a `helm-lint` job to `ci.yml` running `helm lint helm/health-watchers/`
- Add a `helm-template` step to validate templating with `helm template`
- Run `helm upgrade --dry-run` against a kind (Kubernetes in Docker) cluster
- Validate the generated YAML with `kubectl apply --dry-run=server`
- Only trigger on changes to `helm/**` or `.github/workflows/ci.yml`

---

## [CI/CD] Add branch protection rules documentation

**Description:**
The repository's branch protection rules are not documented, causing confusion about what is required before a PR can merge.

**Tasks:**
- Document required status checks for the `main` branch
- Document required reviewers (see `CODEOWNERS`)
- Add a `BRANCH_PROTECTION.md` or update `CONTRIBUTING.md`
- Add a GitHub Actions step that verifies PRs reference a GitHub issue
- Document the commitlint convention (`commitlintrc.json`)

---

## [Backend] Standardize API pagination across all list endpoints

**Description:**
`.kiro/specs/api-pagination-standardization/` has a detailed spec for cursor-based and offset-based pagination, but implementation is inconsistent across modules. Some endpoints return arrays with no metadata; others use different field names.

**Tasks:**
- Implement a shared `paginate()` utility in `apps/api/src/shared/`
- Apply it to: patients, encounters, appointments, payments, invoices, lab-results, medications, referrals, and notifications
- Return consistent shape: `{ data: T[], pagination: { total, page, limit, hasMore, nextCursor? } }`
- Add integration tests covering boundary conditions (empty, single page, last page)
- Update OpenAPI spec and Postman collection to reflect the new schema
- Write a migration guide for API consumers

---

## [Backend] Implement Stellar Soroban smart contract integration

**Description:**
`.kiro/specs/stellar-soroban-integration/` has a complete requirements and design doc. The implementation tasks are defined but not started.

**Tasks:**
- Scaffold the Soroban contract project structure under `apps/stellar-service/src/soroban/`
- Implement the payment escrow contract in Rust
- Add contract compilation and deployment scripts
- Integrate contract invocation into the existing payment flow
- Add unit tests for the contract using the Stellar test framework
- Update the stellar-service README with contract deployment instructions

---

## [Backend] Implement encounter attachment upload and retrieval

**Description:**
`.kiro/specs/encounter-attachments/` defines requirements for uploading files (images, PDFs, lab reports) to encounters. The controller stub exists in `encounters/attachments.controller.ts` but functionality is incomplete.

**Tasks:**
- Implement multipart upload endpoint with file type and size validation
- Store files in S3 (or local filesystem for dev) via the documents storage service
- Associate attachments with encounters in the database
- Implement attachment retrieval with signed URL generation
- Add virus scanning stub (ClamAV or similar) before storing
- Write unit and integration tests

---

## [Backend] Implement patient communication log module

**Description:**
`.kiro/specs/patient-communication-log/` specifies a full communication history feature (SMS, email, in-app messages). The model exists but the service and routes are incomplete.

**Tasks:**
- Complete `communication.service.ts` with send, list, and mark-as-read operations
- Integrate with the notification service for delivery tracking
- Add communication log entries when SMS OTP or appointment reminders are sent
- Expose a paginated GET endpoint at `/patients/:id/communications`
- Add RBAC: only clinic staff with appropriate roles can view communication logs
- Write tests covering the log creation and retrieval flows

---

## [Backend] Implement encounter outcome tracking

**Description:**
`.kiro/specs/encounter-outcome-tracking/` defines requirements for tracking clinical outcomes (resolved, follow-up needed, referred, etc.) on encounters. The encounter model has fields but the logic is not implemented.

**Tasks:**
- Add outcome status transitions to the encounter state machine
- Implement the follow-up reminder job (`follow-up-reminder-job.ts`) to send notifications
- Add an outcomes summary endpoint for the dashboard
- Integrate outcome data into the AI risk stratification service
- Write tests for state transitions and reminder scheduling

---

## [Backend] Implement AI voice transcription for encounters

**Description:**
`.kiro/specs/ai-voice-transcription/` specifies a workflow where audio recordings of consultations are transcribed and auto-populated into SOAP notes. The spec is fully defined but not implemented.

**Tasks:**
- Integrate Google Speech-to-Text or Whisper API into `apps/api/src/modules/ai/`
- Add an audio upload endpoint on the encounters module
- Implement transcription queuing and async processing
- Map transcription output to SOAP note fields (Subjective, Objective, Assessment, Plan)
- Add a review/edit UI step before saving
- Ensure audio files are deleted after transcription (PHI minimization)

---

## [Backend] Implement data anonymization package

**Description:**
`packages/anonymize/src/index.ts` implements anonymization functions, but `.kiro/specs/data-anonymization/` indicates the design doc is empty and full integration into the export and research modules is incomplete.

**Tasks:**
- Complete the design doc in `.kiro/specs/data-anonymization/design.md`
- Ensure k-anonymity for research data exports (`research.controller.ts`)
- Add differential privacy noise to aggregate reports
- Integrate anonymization into the FHIR export pipeline
- Add a test dataset that verifies re-identification risk is below threshold
- Document the anonymization algorithm and parameters

---

## [Backend] Complete Stellar multi-signature payment implementation

**Description:**
`.kiro/specs/stellar-multisig-payments/design.md` is empty despite having complete requirements. Multi-sig is critical for large payment approvals.

**Tasks:**
- Design the multi-sig workflow: who are the signers, what thresholds are required
- Implement the multi-sig transaction building in `stellar.ts`
- Add an approval queue endpoint: `POST /payments/:id/approve`
- Notify pending signers via the notification service
- Handle timeout: auto-void transactions that do not gather signatures within 24 hours
- Add tests for 2-of-3 and 3-of-5 signature scenarios

---

## [Backend] Implement Stellar batch payment processor

**Description:**
`batch-processor.ts` exists in the stellar-service but `.kiro/specs/stellar-batch-payments/tasks.md` shows many tasks remain. The API-side batch endpoint is also incomplete.

**Tasks:**
- Complete the batch payment creation endpoint in `payments/batch-payment.controller.ts`
- Implement the retry logic for failed transactions within a batch
- Add partial success handling: report which payments succeeded and which failed
- Implement batch cancellation before submission
- Add metrics for batch throughput and failure rate
- Write end-to-end tests simulating a 50-payment batch on testnet

---

## [Backend] Implement request ID propagation across all services

**Description:**
`.kiro/specs/request-id-propagation/` defines requirements for propagating a unique `X-Request-ID` header from the web app through the API to the stellar-service for end-to-end tracing.

**Tasks:**
- Add request ID middleware to the API that generates or passes through `X-Request-ID`
- Attach the request ID to all outbound HTTP calls to stellar-service
- Include the request ID in all structured log entries
- Add the request ID to error responses so users can reference it when reporting issues
- Verify OpenTelemetry trace IDs are correlated with request IDs
- Write integration tests verifying the header is present throughout

---

## [Backend] Add rate limiting per-endpoint with Redis

**Description:**
`docs/API_RATE_LIMITING.md` documents a rate limiting strategy, but the implementation may not cover all sensitive endpoints (auth, PHI export, AI).

**Tasks:**
- Audit all endpoints and classify by sensitivity level
- Add stricter limits to: `/auth/login`, `/auth/register`, `/export/*`, `/ai/*`, `/portal/mfa/*`
- Implement sliding window rate limiting backed by Redis
- Return `Retry-After` header on 429 responses
- Add rate limit metrics to Prometheus
- Write tests simulating burst traffic

---

## [Backend] Improve error handling consistency across all controllers

**Description:**
Error handling is inconsistent — some controllers return plain strings, others return structured objects, and some do not forward errors to the central error handler.

**Tasks:**
- Audit all controllers for `try/catch` blocks that swallow errors
- Create a shared `AppError` class with `statusCode`, `code`, and `message`
- Replace ad-hoc error objects with `AppError` instances
- Ensure all async route handlers use the `asyncHandler` wrapper
- Add an integration test that verifies all error responses conform to the standard shape
- Update the OpenAPI spec with error response schemas

---

## [Backend] Add input validation to all POST/PATCH endpoints using Zod

**Description:**
Validation is inconsistent — some endpoints use Joi, some use custom validation, and some have no validation at all. The project should standardize on Zod.

**Tasks:**
- Audit all `*.validation.ts` files and identify non-Zod implementations
- Create a `validate` middleware factory that accepts a Zod schema
- Migrate the following to Zod: patients, encounters, appointments, payments, invoices
- Add validation for query parameters (pagination, filters) in addition to bodies
- Return 400 with a structured list of field errors on validation failure
- Update OpenAPI generation to derive schemas from Zod

---

## [Backend] Add idempotency key support to payment endpoints

**Description:**
Network retries can cause duplicate payment submissions. Payment creation endpoints must support idempotency keys.

**Tasks:**
- Accept `Idempotency-Key` header on `POST /payments`, `POST /payments/batch`, `POST /payments/recurring`
- Store idempotency key → response mapping in Redis with a 24-hour TTL
- Return the cached response if the same key is received again
- Add a unique index on the idempotency key in MongoDB as a fallback
- Document the idempotency behavior in the API docs
- Write tests verifying duplicate requests return identical responses

---

## [Backend] Implement webhook signature verification

**Description:**
`webhooks.controller.ts` handles outbound webhooks, but incoming webhooks from external systems (e.g., Stellar payment events) may lack signature verification.

**Tasks:**
- Add HMAC-SHA256 signature verification to the incoming webhook endpoint
- Verify `X-Webhook-Signature` header on all incoming webhook requests
- Return 401 if signature is missing or invalid
- Add a secret rotation mechanism for webhook signing keys
- Document how to configure webhook receivers
- Write tests covering valid, invalid, and missing signatures

---

## [Backend] Add cursor-based pagination to the audit log endpoint

**Description:**
The audit log can grow very large. The current implementation uses offset pagination, which degrades performance on large collections.

**Tasks:**
- Add a compound index on `(clinicId, createdAt, _id)` to the audit log model
- Implement cursor-based pagination using `_id` as the cursor
- Accept `after` and `before` cursor parameters
- Return `nextCursor` and `prevCursor` in the response
- Add a test with 10,000+ audit entries verifying response time under 100ms
- Update the OpenAPI spec and Postman collection

---

## [Backend] Implement clinic onboarding step validation

**Description:**
`clinics/onboarding.routes.ts` defines a multi-step onboarding flow, but the steps lack proper validation and progress tracking.

**Tasks:**
- Add a `onboardingStep` field to the clinic model tracking current step
- Validate that steps are completed in order
- Implement rollback: allow a clinic to return to a previous step
- Send a welcome email when onboarding is complete
- Add an admin endpoint to reset onboarding state for support purposes
- Write tests for each onboarding step transition

---

## [Backend] Add full-text search to patient and encounter records

**Description:**
Clinics need to search patients by name, phone, or record number, and search encounter notes. MongoDB text indexes can provide this without an external search engine.

**Tasks:**
- Add a compound text index to the patient model: `name`, `phone`, `nationalId`
- Add a text index to the encounter model: `soapNotes`, `diagnosis`, `chiefComplaint`
- Implement `GET /patients?q=<search>` and `GET /encounters?q=<search>` endpoints
- Add relevance scoring and sort by score descending
- Add tests for multi-word queries, partial matches, and empty results
- Document search behavior and limitations in the API docs

---

## [Backend] Implement medication drug interaction check in AI module

**Description:**
`apps/api/src/modules/ai/ai.drug-interactions.test.ts` has tests but the implementation of real-time drug interaction checking is incomplete.

**Tasks:**
- Integrate with an open drug database (OpenFDA or RxNorm) or the Gemini API
- Check interactions when a new medication is added to an encounter
- Return severity levels: mild, moderate, severe, contraindicated
- Trigger a CDS alert if a severe interaction is detected
- Cache interaction responses for common drug pairs
- Write unit tests for severity classification

---

## [Backend] Implement pre-authorization request workflow

**Description:**
`apps/api/src/modules/pre-auth/pre-auth.controller.ts` manages insurance pre-authorization but the approval/denial workflow and notification integration are incomplete.

**Tasks:**
- Implement state machine: `pending → submitted → approved/denied/more-info-needed`
- Send notifications to clinic staff on status changes
- Add a bulk submission endpoint for submitting multiple pre-auths at once
- Integrate with the billing module to block claim submission without approval
- Add an expiry date check (pre-auth often expires after 90 days)
- Write integration tests for the full approval and denial flows

---

## [Backend] Implement FHIR R4 export validation

**Description:**
`fhir-mapper.ts` maps internal models to FHIR resources, but there is no validation that the output conforms to the FHIR R4 specification.

**Tasks:**
- Add the `fhir-validator` or `fhir.js` library for schema validation
- Validate all mapped resources before returning them in the export
- Add a CI test that validates FHIR output against the R4 spec
- Support FHIR Patient, Observation, Encounter, MedicationRequest, and Immunization resources
- Document which FHIR profiles are supported
- Log validation errors without exposing them to end users

---

## [Backend] Add telemetry for Stellar transaction success and failure rates

**Description:**
`apps/stellar-service/src/metrics.ts` has a basic Prometheus client, but transaction-level metrics are missing.

**Tasks:**
- Add counters: `stellar_transactions_submitted_total`, `stellar_transactions_confirmed_total`, `stellar_transactions_failed_total`
- Add histograms: `stellar_transaction_confirmation_duration_seconds`
- Add gauges: `stellar_account_balance_xlm`, `stellar_network_fee_stroops`
- Expose metrics at `/metrics` on the stellar-service
- Add Grafana dashboard panels for the new metrics
- Set up alert rules for high failure rates

---

## [Frontend] Implement real-time dashboard with WebSocket updates

**Description:**
The dashboard page (`(dashboard)/page.tsx`) uses React Query for polling, but the `RealtimeProvider` and WebSocket infrastructure exist to push updates. The dashboard widgets should reflect live data.

**Tasks:**
- Connect the dashboard to the WebSocket room for the current clinic
- Subscribe to: new patient registrations, appointment status changes, payment confirmations
- Update `StatCard`, `AppointmentWidget`, and `RecentTable` in real time without page refresh
- Add a connection status indicator using `LazyRealtimeProvider`
- Gracefully fall back to polling when WebSocket is unavailable
- Write a Playwright test verifying live updates

---

## [Frontend] Build patient detail page with all clinical tabs

**Description:**
`apps/web/src/app/patients/[id]` exists as a directory but the tabbed patient detail view (encounters, lab results, medications, care plans, immunizations, consent, insurance) may have incomplete tabs.

**Tasks:**
- Audit each tab component for completeness
- Complete `LabResultsTab.tsx`: add trend charts and critical value highlighting
- Complete `CarePlanTab.tsx`: add goal tracking and task completion UI
- Add `MedicationsTab.tsx` if missing, displaying current medications with interaction warnings
- Add `ImmunizationsTab.tsx` pulling from the immunizations module
- Ensure all tabs are lazy-loaded for performance

---

## [Frontend] Implement dark mode across all pages

**Description:**
`ThemeToggle.tsx` and `ThemeSync.tsx` exist, and `e2e/dark-mode.spec.ts` has tests, but dark mode styles may be incomplete for many components.

**Tasks:**
- Audit all Tailwind classes for hardcoded colors that don't use the `dark:` variant
- Add `dark:` variants to: payment tables, patient cards, CDS panels, encounter forms
- Verify dark mode in the portal subsection
- Test dark mode on charts (`VitalSignsChart`, `PaymentAnalyticsCharts`)
- Add a dark mode screenshot to the README
- Ensure the user's preference persists across sessions via `localStorage`

---

## [Frontend] Add skeleton loading states to all data-fetching components

**Description:**
Some components show a blank screen or spinner while loading. Skeleton loaders improve perceived performance and reduce layout shift.

**Tasks:**
- Audit all `Client` components for loading state handling
- Replace generic spinners in `PatientsClient`, `EncountersClient`, `PaymentsClient`, `AppointmentsClient` with skeleton loaders using `Skeleton.tsx`
- Add skeleton states to the dashboard stat cards and recent tables
- Add skeleton to the wallet balance display
- Write Storybook stories (or inline visual tests) for skeleton states
- Verify there is no CLS (cumulative layout shift) when data loads

---

## [Frontend] Implement patient import from CSV

**Description:**
`PatientImport.tsx` exists but the actual CSV parsing, field mapping, and validation UI may be incomplete.

**Tasks:**
- Implement CSV parsing with preview (show first 5 rows before import)
- Add a column mapping step: map CSV columns to patient fields
- Validate required fields and flag duplicates before submission
- Show a progress bar and per-row result after import
- Add support for both comma and semicolon delimiters
- Write tests for the parsing and validation logic

---

## [Frontend] Complete the disputes management page

**Description:**
`apps/web/src/app/disputes/page.tsx` exists but may lack full CRUD operations and status tracking for payment disputes.

**Tasks:**
- Implement dispute list view with filtering by status and date range
- Add dispute filing form using `DisputeFilingForm.tsx`
- Show dispute timeline and document attachments
- Implement resolution actions: accept, reject, counter-offer
- Add export to CSV/PDF for dispute records
- Write Playwright E2E test for the full dispute lifecycle

---

## [Frontend] Implement offline mode with service worker sync

**Description:**
`apps/web/public/sw.js` and `apps/web/e2e/offline.spec.ts` exist, and `PWA_OFFLINE_POLICY.md` documents the strategy. The full sync queue may not be implemented.

**Tasks:**
- Implement `SyncQueueStatus.tsx` to show pending sync items to the user
- Queue patient registration and encounter creation when offline
- Sync queued operations when connectivity is restored
- Handle merge conflicts (server state changed while offline)
- Test offline behavior with Playwright's `page.route()` network interception
- Add a visual indicator when data may be stale due to offline period

---

## [Frontend] Add comprehensive form validation with inline error messages

**Description:**
Forms like `CreatePatientForm.tsx`, `EncounterForm.tsx`, and `PaymentIntentForm.tsx` may lack real-time inline validation feedback.

**Tasks:**
- Add `react-hook-form` + Zod resolver to all major forms if not already present
- Show field-level error messages below each input on blur
- Add a summary error banner at the top of the form for submit failures
- Highlight required fields clearly
- Disable the submit button while submission is in progress
- Add ARIA `aria-invalid` and `aria-describedby` attributes for accessibility

---

## [Frontend] Add confirm dialog before destructive actions

**Description:**
Destructive actions (delete patient, void payment, cancel appointment) currently execute without a confirmation step, risking accidental data loss.

**Tasks:**
- Create a reusable `ConfirmDialog` component that accepts a message and destructive label
- Add confirmation to: delete patient, void/cancel payment, remove encounter, cancel appointment
- Require typing the item name (e.g., patient full name) for permanent deletions
- Animate the dialog using CSS transitions
- Write unit tests for the dialog component
- Add Playwright tests verifying the dialogs block accidental deletion

---

## [Frontend] Improve the encounter SOAP notes editor

**Description:**
`SoapNotesEditor.tsx` is a basic textarea-based editor. Clinical staff need richer formatting options and auto-save.

**Tasks:**
- Add auto-save every 30 seconds with a "Saved" indicator
- Add basic markdown/rich text formatting (bold, lists, headers)
- Add `@mention` support to reference other encounters or patients
- Add keyboard shortcuts (Ctrl+S to save, Ctrl+B for bold)
- Integrate with the AI transcription feature to populate fields from voice
- Add conflict detection if another user edits the same encounter

---

## [Frontend] Build a comprehensive payment analytics dashboard

**Description:**
`apps/web/src/app/payments/analytics/` directory exists but the analytics UI may be minimal. Clinics need detailed financial insights.

**Tasks:**
- Add charts: revenue over time, payment method breakdown, top diagnoses by revenue
- Add a KPI row: total collected, outstanding, refunded, disputed
- Implement date range filtering (7d, 30d, 90d, custom)
- Add a table showing payment-to-encounter linkage
- Export analytics data to CSV and PDF
- Ensure all charts use accessible color palettes and have legends

---

## [Frontend] Implement notifications center with read/unread state

**Description:**
`apps/web/src/app/notifications/page.tsx` and `NotificationBell.tsx` exist, but the full notifications center with categorization and bulk actions may be incomplete.

**Tasks:**
- Show notifications grouped by type: appointment, payment, lab result, system alert
- Add mark all as read, mark single as read, and delete actions
- Add real-time delivery via WebSocket
- Add a badge count on the navbar bell icon
- Add deep links: clicking a notification navigates to the relevant resource
- Write tests for the unread count logic

---

## [Frontend] Add print-friendly layout for encounter notes and invoices

**Description:**
Clinical staff need to print encounter notes and invoices for patients. There are no print-specific styles or print preview pages.

**Tasks:**
- Add `@media print` styles to encounter detail and invoice pages
- Hide navigation, sidebar, and interactive elements in print view
- Add a "Print" button that triggers `window.print()`
- For invoices, generate a PDF using the API's `pdf-generator.service.ts`
- Ensure the printed layout conforms to standard A4 paper
- Test in Chrome and Firefox print preview

---

## [Frontend] Add patient photo upload and display

**Description:**
`apps/web/src/components/patients/PhotoUpload.tsx` exists, but the integration into the patient detail header and list views may be incomplete.

**Tasks:**
- Show patient photo in the patient detail page header
- Show patient thumbnail in the patients list (`PatientThumbnail.tsx`)
- Implement crop-and-resize before upload (client-side, using `canvas`)
- Add a fallback avatar with initials when no photo is set
- Respect the 2MB size limit and show a friendly error if exceeded
- Write a test for the upload flow including error states

---

## [Frontend] Implement ICD-10 favorites and recent codes

**Description:**
`ICD10Search.tsx` exists but the favorites and recent codes feature (`icd10-favorites.service.ts`) may not be fully wired to the frontend.

**Tasks:**
- Show a "Favorites" and "Recent" tab in the ICD-10 search dropdown
- Add a star/bookmark icon to favorite a code
- Sync favorites with the backend (`/icd10/favorites`)
- Show up to 10 recent codes in the search dropdown before typing
- Add keyboard navigation through search results
- Write unit tests for the favorites toggle logic

---

## [Frontend] Add accessibility audit and fix violations

**Description:**
`apps/web/tests/accessibility.spec.ts` has Playwright accessibility tests. The audit likely reveals violations that need fixing.

**Tasks:**
- Run the existing accessibility tests and document all violations
- Fix all critical (WCAG 2.1 AA) violations first
- Ensure all form inputs have associated labels
- Ensure all images have descriptive `alt` text
- Verify keyboard-only navigation works on all modals and dropdowns
- Add `aria-live` regions for dynamic content updates (notifications, toasts)

---

## [Frontend] Implement schedule/calendar management for clinic staff

**Description:**
`ScheduleCalendar.tsx` and `SchedulingCalendar.tsx` exist, but the calendar may lack drag-and-drop, slot management, and conflict detection.

**Tasks:**
- Implement a weekly view showing all staff schedules
- Add drag-and-drop to reschedule appointments
- Show slot availability: free, booked, blocked
- Allow staff to block time off directly on the calendar
- Highlight double-booking conflicts in red
- Sync calendar state with the backend via React Query mutations

---

## [Frontend] Add deep link support for portal patients

**Description:**
The patient portal (`apps/web/src/app/portal/`) has dedicated pages, but sharing a direct link to a specific document, appointment, or encounter may not work correctly due to auth redirects.

**Tasks:**
- Store the intended URL before redirecting to portal login
- Redirect to the original URL after successful portal authentication
- Add Open Graph meta tags for shareable portal pages
- Test deep links for: appointments, encounters, payments, and documents
- Ensure portal deep links do not expose admin routes

---

## [Frontend] Add language switcher to portal pages

**Description:**
`LanguageSwitcher.tsx` exists but may not be present on all portal pages. The portal serves patients who may prefer Yoruba, French, Hausa, or Portuguese.

**Tasks:**
- Add the language switcher to the portal layout (`portal/layout.tsx`)
- Ensure all portal pages have complete translations in all 5 languages (en, fr, yo, ha, pt)
- Run `scripts/check-translations.ts` and fix any missing keys
- Add a language preference to the patient portal profile
- Test right-to-left support if Arabic is planned in the future

---

## [Frontend] Implement settings and profile page

**Description:**
`.kiro/specs/settings-profile/` is defined. The `ClinicSettingsClient.tsx` and settings components exist, but the full profile management (avatar, display name, timezone, notifications preferences) may be incomplete.

**Tasks:**
- Complete `ProfileSection.tsx`: add avatar upload, display name, timezone
- Complete `PreferencesSection.tsx`: notification preferences, default view, language
- Add an "Active sessions" panel using `SessionManagement.tsx`
- Add a "Danger zone" section: deactivate account
- Ensure all changes persist via the API and reflect immediately in the UI
- Write Playwright tests for each settings section

---

## [Frontend] Add visual regression tests using Playwright screenshots

**Description:**
`apps/web/e2e/visual-regression.spec.ts` exists but may have limited coverage. Visual regressions in the UI design system go undetected.

**Tasks:**
- Add screenshot tests for: login page, dashboard, patient detail, encounter form, payment table
- Use Playwright's `toMatchSnapshot` with a tolerance threshold
- Store baseline screenshots in version control
- Add a CI step to fail on visual regressions
- Document how to update baselines when intentional UI changes are made

---

## [Design] Create and publish a design system documentation page

**Description:**
`UI Designs/` contains Figma links. The component library (`apps/web/src/components/ui/`) has many components but no design system documentation page for contributors.

**Tasks:**
- Create a `design-system` page or Storybook showing all UI components with variants
- Document `Button` (all variants), `Input`, `Modal`, `Toast`, `Badge`, `Card`, `Table`
- Show the color palette, typography scale, and spacing tokens
- Document WCAG color contrast compliance for each color pair
- Publish to GitHub Pages as part of the docs workflow

---

## [Design] Implement design tokens as CSS custom properties

**Description:**
Tailwind configuration defines colors and spacing, but design tokens are not available as CSS custom properties, making it harder to implement custom themes (e.g., white-label clinic branding).

**Tasks:**
- Extract core design tokens from `tailwind.config.ts` into `globals.css` as CSS custom properties
- Support: color palette, spacing, border radius, font sizes, shadows
- Allow clinics to override tokens via their settings (custom primary color)
- Test that overriding tokens propagates correctly to all components
- Document the theming system for contributors

---

## [Design] Redesign the mobile navigation for small screens

**Description:**
`MobileNavigation.tsx` and `MobileNavigationDrawer.tsx` exist, but the mobile UX may not be optimized per the `MOBILE_RESPONSIVE_GUIDE.md` guidelines.

**Tasks:**
- Audit the navigation on screens 320px–768px wide
- Implement a bottom tab bar for the most common actions (Dashboard, Patients, Appointments, Payments)
- Add a slide-over drawer for secondary navigation
- Ensure touch targets are at least 44×44px
- Add swipe gestures to open/close the drawer
- Test on iOS Safari and Android Chrome using BrowserStack or local device

---

## [Design] Improve empty state designs across the application

**Description:**
`EmptyState.tsx` is a generic component, but many empty states across the app use plain text or minimal UI, missing opportunities to guide users to take action.

**Tasks:**
- Audit all empty states: patients list, encounters, appointments, payments, notifications
- Add descriptive illustration or icon to each empty state
- Add a call-to-action button ("Add patient", "Schedule appointment", etc.)
- Ensure empty states are distinct from loading states and error states
- Add i18n strings for all empty state messages
- Document the empty state design pattern in the design system

---

## [Design] Add loading and error states to all page-level components

**Description:**
Next.js 14 supports `loading.tsx` and `error.tsx` at the route level, but many routes do not have these files.

**Tasks:**
- Audit all route directories in `apps/web/src/app/` for missing `loading.tsx` and `error.tsx`
- Add `loading.tsx` with a skeleton matching the page layout for: encounters, appointments, patients, payments, reports
- Add `error.tsx` with a clear message and a retry button for all major routes
- Ensure the global `error.tsx` and `global-error.tsx` show user-friendly messages
- Write tests verifying error boundaries catch thrown errors

---

## [Design] Add data visualization to the reports page

**Description:**
`apps/web/src/app/reports/page.tsx` is a large file (13KB) but may lack rich charts and interactive filtering.

**Tasks:**
- Add a chart for: encounters per day/week/month, revenue trend, top diagnoses
- Add interactive filters: date range, provider, clinic branch
- Use `recharts` or `chart.js` (check existing usage in `charts/`)
- Add a "download as image" button for each chart
- Ensure charts are keyboard-accessible and have text alternatives
- Add export to PDF for the full report

---

## [Refactoring] Split large controller files into service + controller

**Description:**
Several controller files are extremely large and contain business logic that should live in services: `payments.controller.ts` (63KB), `patients.controller.ts` (57KB), `ai.routes.ts` (32KB), `portal.controller.ts` (27KB).

**Tasks:**
- Extract business logic from `payments.controller.ts` into `payments.service.ts`
- Extract business logic from `patients.controller.ts` into `patients.service.ts`
- Extract AI route handlers into an `ai.controller.ts` and `ai.service.ts`
- Ensure controller methods delegate to services and handle only HTTP concerns
- Verify all existing tests still pass after the split
- Add JSDoc comments to all exported service methods

---

## [Refactoring] Consolidate duplicate toast/notification utilities

**Description:**
`apps/web/src/components/ui/Toast.tsx` and `apps/web/src/components/ui/toast.tsx` (note the casing) appear to be duplicates. Inconsistent imports cause confusion.

**Tasks:**
- Identify which file is the canonical implementation
- Migrate all imports to the canonical file
- Delete the duplicate
- Add a lint rule preventing duplicate component names
- Write tests to verify toast appears and disappears correctly
- Check for similar duplication in other UI components

---

## [Refactoring] Replace inline styles with Tailwind utility classes

**Description:**
Some components use inline `style={{}}` props for layout, breaking the consistent Tailwind approach and making responsive design harder.

**Tasks:**
- Run a grep for `style={{` across `apps/web/src/`
- Replace inline styles with equivalent Tailwind classes where possible
- For dynamic values (e.g., variable widths), use CSS custom properties within Tailwind
- Remove any `!important` overrides that were added as workarounds
- Run visual regression tests after changes to verify nothing broke

---

## [Refactoring] Migrate remaining CommonJS `require()` to ES module imports

**Description:**
Several configuration files use `require()` (e.g., `jest.config.js`, `migrate-mongo-config.js`). The project is TypeScript-first and should use ES module syntax consistently.

**Tasks:**
- Identify all `.js` files using `require()`
- Migrate `jest.config.js` files to `jest.config.ts` where possible
- Migrate `eslint.config.js` to use `import` syntax
- Ensure the build pipeline handles the change correctly
- Update the contributing guide to note ES modules as the standard

---

## [Refactoring] Extract shared date/time utilities into `packages/utils`

**Description:**
Date formatting and manipulation logic is duplicated across the API, web, and stellar-service. A shared `packages/utils` package would eliminate this duplication.

**Tasks:**
- Create `packages/utils/` with a `package.json` and TypeScript config
- Move date formatting, timezone conversion, and relative time functions from each app into the package
- Update all import sites to use the shared package
- Add unit tests to the `utils` package
- Export from `packages/types/src/index.ts` if relevant types are involved

---

## [Refactoring] Remove dead code and unused exports from API modules

**Description:**
TypeScript's strict mode and ESLint may reveal unused exports and dead code paths in the API modules.

**Tasks:**
- Run `ts-prune` or `knip` to find unused exports
- Remove identified dead code after confirming it is not used
- Enable `no-unused-vars` and `@typescript-eslint/no-unused-vars` in ESLint config
- Remove unused imports in all module files
- Audit `__mocks__` directory for stale mock implementations

---

## [Refactoring] Standardize MongoDB model definitions with shared base schema

**Description:**
Every Mongoose model re-declares `createdAt`, `updatedAt`, `clinicId`, `isActive`, and `createdBy`. A shared base schema would reduce duplication and enforce consistency.

**Tasks:**
- Create a `baseSchema` in `apps/api/src/shared/` with common fields
- Apply `baseSchema` to all models using Mongoose schema composition
- Ensure soft-delete (`isActive`) and audit fields are consistently present
- Update migration scripts to handle schema changes
- Add a unit test verifying all models have the required base fields

---

## [Refactoring] Replace magic strings with typed enums and constants

**Description:**
Many files use magic strings for status values, roles, and event names (e.g., `"pending"`, `"admin"`, `"payment.created"`). These should be typed enums or constants.

**Tasks:**
- Identify magic strings in models, controllers, and validation schemas
- Create enums/constants in `packages/types/src/index.ts` for: user roles, appointment statuses, payment statuses, encounter statuses, notification types
- Replace magic strings with the typed values
- Update `packages/anonymize` and other packages to use shared types
- Run tests to verify no broken comparisons after the change

---

## [Refactoring] Improve TypeScript strictness settings

**Description:**
`tsconfig.base.json` may not have all strict TypeScript checks enabled. Gradually increasing strictness improves code quality and catches bugs.

**Tasks:**
- Enable `strict: true` in `tsconfig.base.json` if not already set
- Fix all resulting TypeScript errors app by app, starting with `packages/`
- Enable `noUncheckedIndexedAccess` and `exactOptionalPropertyTypes`
- Add `@typescript-eslint/recommended-type-checked` to the ESLint config
- Document the strictness progression plan in `CONTRIBUTING.md`

---

## [Testing] Add unit tests for the AI risk calculator

**Description:**
`risk-calculator.test.ts` exists but may not cover edge cases: patients with no history, patients with chronic conditions, risk score boundaries.

**Tasks:**
- Audit existing test coverage using Istanbul/nyc
- Add tests for: zero-history patient, maximum risk patient, age boundary conditions
- Add tests for the risk factor weighting algorithm
- Add a property-based test using `fast-check` for fuzz testing
- Ensure test data does not use real patient identifiers
- Achieve at least 90% branch coverage for `risk-calculator.ts`

---

## [Testing] Add contract tests for API ↔ Web consumer/provider

**Description:**
`apps/api/src/contracts/` and `apps/web/src/contracts/` exist for Pact contract testing, but the number of covered interactions may be low.

**Tasks:**
- Add consumer contract tests for all major API calls from the web app: login, patients CRUD, encounters CRUD, payments
- Run Pact provider verification on the API side in CI
- Publish Pact files to a Pact Broker or GitHub artifact store
- Add contract tests for the web app ↔ stellar-service interactions
- Document the Pact workflow in `docs/TESTING_GUIDE.md`

---

## [Testing] Add integration tests for the Stellar payment reconciliation service

**Description:**
`payment-reconciliation.ts` is a critical service but may lack integration tests that simulate the Stellar testnet responses.

**Tasks:**
- Mock the Horizon API client for deterministic tests
- Test successful reconciliation of a single payment
- Test reconciliation when a payment is not found on the ledger
- Test reconciliation of a batch with partial failures
- Test the retry logic and idempotency
- Test the reconciliation job's scheduled execution

---

## [Testing] Add mutation testing baseline for the auth module

**Description:**
`apps/api/stryker.config.json` configures Stryker mutation testing. The auth module is the most security-critical and should have a mutation score above 80%.

**Tasks:**
- Run Stryker on `apps/api/src/modules/auth/`
- Identify and fix surviving mutants in: JWT verification, password hashing, MFA validation, token rotation
- Document the mutation testing process and current score in `docs/TESTING_GUIDE.md`
- Add a CI gate that fails if the auth module mutation score drops below 80%
- Add Stryker to the weekly CI schedule for the full API

---

## [Testing] Add performance regression tests with k6 baselines

**Description:**
`k6/` contains load test scripts and `performance/baselines/baselines.json` stores baseline metrics, but the baselines may be outdated.

**Tasks:**
- Update baselines by running `k6 run k6/regression-baseline.js` against the current staging environment
- Add a CI job (`performance.yml`) that runs the smoke test on every PR to detect regressions
- Alert when p95 latency for login, patient list, or payment creation exceeds baseline by 20%
- Document how to update baselines in `docs/LOAD_TESTING.md`
- Add a patient-import load test scenario

---

## [Testing] Add test coverage reporting to SonarCloud

**Description:**
`sonar-project.properties` is configured but test coverage may not be uploading correctly, leading to inaccurate SonarCloud coverage metrics.

**Tasks:**
- Verify Jest is configured to output LCOV coverage in all apps
- Add coverage upload steps to `ci.yml` for API, web, and stellar-service
- Configure SonarCloud to use the LCOV reports
- Set a minimum coverage gate of 70% in SonarCloud
- Fix the badge URL in the README with the correct project key
- Ensure coverage excludes test files, mocks, and migration files

---

## [Testing] Add mobile app integration tests

**Description:**
`apps/mobile/` has screens and services but no test files. The mobile app is a contributor target with no safety net.

**Tasks:**
- Set up Jest with React Native Testing Library for the mobile app
- Add unit tests for `auth.service.ts`, `offline-cache.service.ts`, `health-workflows.service.ts`
- Add render tests for `DashboardScreen`, `PaymentScreen`, `ImmunizationsScreen`
- Mock the Expo modules for navigation and notifications
- Add a `test` script to `apps/mobile/package.json`
- Document how to run mobile tests in `apps/mobile/README.md`

---

## [Testing] Add security headers E2E test coverage

**Description:**
`apps/web/e2e/security-headers.spec.ts` exists but may only check a few headers. HIPAA-compliant apps need comprehensive security header verification.

**Tasks:**
- Add tests for: `Content-Security-Policy`, `X-Frame-Options`, `Strict-Transport-Security`, `X-Content-Type-Options`, `Permissions-Policy`, `Referrer-Policy`
- Verify CSP blocks inline scripts in production builds
- Test that the API returns `X-Request-ID` on all responses
- Verify `CORS` headers restrict origins to the expected frontend URL
- Add the test to the nightly CI schedule

---

## [Testing] Improve test isolation using test database fixtures

**Description:**
API integration tests may share state through a shared MongoDB connection, causing flaky tests when run in parallel.

**Tasks:**
- Implement per-test database isolation using `mongomock-server` or `mongodb-memory-server`
- Create fixture factories for: `Patient`, `User`, `Encounter`, `Payment`, `Clinic`
- Replace ad-hoc `beforeEach` data setup with fixture factories
- Enable Jest's `--runInBand` for integration tests to ensure isolation
- Document the fixture pattern in `docs/TESTING_GUIDE.md`

---

## [DevOps] Add Redis to the local development Docker Compose

**Description:**
Rate limiting, caching, and idempotency keys depend on Redis, but `docker-compose.dev.yml` only includes MongoDB. Local development diverges from production.

**Tasks:**
- Add a `redis` service to `docker-compose.dev.yml`
- Configure the API to use Redis for rate limiting and session caching
- Add Redis connection health check to the API startup
- Document Redis usage in the README quick-start guide
- Add a `redis` service to `docker-compose.yml` (full stack) if not already present

---

## [DevOps] Add automated secrets rotation to the Kubernetes setup

**Description:**
`k8s/secret-rotation-lambda.py` and `k8s/secret-rotation-config.yaml` exist, but the end-to-end rotation workflow may not be tested or documented.

**Tasks:**
- Document the secret rotation process for: JWT secrets, Stellar keypairs, MongoDB credentials
- Add a GitHub Actions workflow that can trigger a manual rotation with approval
- Test the Lambda function locally using `aws lambda invoke`
- Add Prometheus alerts for secrets nearing their rotation date
- Verify the API handles secret rotation without downtime (zero-downtime rotation)

---

## [DevOps] Add Kubernetes readiness and liveness probe documentation

**Description:**
The Kubernetes deployment files define probes, but developers adding new services need guidance on how to configure them correctly.

**Tasks:**
- Document the difference between liveness and readiness probes in `k8s/README.md`
- Verify the API health endpoint (`/health`) is suitable as a liveness probe
- Add a separate readiness endpoint (`/ready`) that checks database connectivity
- Add startup probes for services with slow initialization (stellar-service)
- Test probe behavior during rolling deployments

---

## [DevOps] Add cost estimation for AWS resources in production

**Description:**
The deployment uses EKS, S3, SES, and AWS Secrets Manager. There is no cost estimation or budget alert setup.

**Tasks:**
- Add an AWS Cost and Usage Report setup guide to `docs/DEPLOYMENT_GUIDE.md`
- Define a CloudWatch billing alarm for the expected monthly budget
- Add Terraform or CloudFormation cost estimation using `infracost`
- Document approximate costs for: 3-node EKS cluster, S3 storage, SES email volume
- Add a cost optimization review checklist to the deployment guide

---

## [DevOps] Add multi-region deployment documentation

**Description:**
The current deployment guide targets a single AWS region. For HIPAA compliance and availability, a multi-region strategy may be required.

**Tasks:**
- Document the data residency requirements for patient data
- Add a multi-region architecture diagram showing primary/secondary regions
- Document MongoDB Atlas global clusters vs. self-managed replica sets across regions
- Add a cross-region failover runbook
- Update `docs/DISASTER_RECOVERY_PLAN.md` with regional failover steps

---

## [DevOps] Implement automated certificate renewal with cert-manager

**Description:**
`k8s/cert-manager-setup.yaml` exists but certificate renewal automation and monitoring may not be fully configured.

**Tasks:**
- Verify cert-manager is configured with Let's Encrypt or AWS ACM
- Add a Prometheus alert for certificates expiring within 30 days
- Add a runbook for certificate renewal failure
- Test renewal by shortening the certificate validity in a staging environment
- Document the cert-manager setup in `docs/KUBERNETES_DEPLOYMENT.md`

---

## [DevOps] Add ELK stack integration test

**Description:**
`logging/` contains full ELK configuration, but there is no automated test verifying that logs from the API and stellar-service actually appear in Elasticsearch.

**Tasks:**
- Add a `docker-compose.elk.yml`-based integration test in CI
- Start the ELK stack and the API, generate some test requests
- Query Elasticsearch to verify the log entries were indexed
- Verify the Logstash pipeline (`health-watchers.conf`) parses JSON logs correctly
- Add a Kibana dashboard import test to verify the dashboard loads

---

## [DevOps] Add GitHub Dependabot auto-merge for minor and patch updates

**Description:**
`dependabot-auto-merge.yml` exists but may not be fully configured for safe auto-merging of minor/patch updates.

**Tasks:**
- Configure auto-merge for: minor and patch updates to non-security-critical packages
- Always require manual review for: major version bumps, security-related packages, AWS SDK, and Stellar SDK
- Add a comment to auto-merged PRs with the change summary
- Verify the auto-merge respects branch protection rules
- Test the workflow by manually triggering a Dependabot PR

---

## [DevOps] Add container image signing with cosign

**Description:**
Container images are built in CI but not signed. In a HIPAA environment, image provenance should be verifiable.

**Tasks:**
- Add `sigstore/cosign` to the Docker build workflow
- Sign images with a GitHub OIDC-backed keyless signature
- Add an admission controller (Kyverno or OPA Gatekeeper) policy that rejects unsigned images
- Store the public key in `k8s/` for verification
- Document the image signing process in `docs/DEPLOYMENT_GUIDE.md`

---

## [DevOps] Add MongoDB backup encryption and integrity verification

**Description:**
`ops/backup-verification.sh` and `scripts/backup-mongodb.sh` exist, but backups may not be encrypted or integrity-verified after transfer.

**Tasks:**
- Encrypt backups at rest using AES-256 before uploading to S3
- Add a checksum (SHA-256) file alongside each backup
- Verify the checksum after download during restoration
- Test the full backup → transfer → verify → restore cycle in CI
- Add a Prometheus alert if the daily backup job fails
- Document the backup encryption key management

---

## [Security] Implement Content Security Policy for the Next.js app

**Description:**
`next.config.js` may have a CSP, but it might use `unsafe-inline` or `unsafe-eval` which weakens the policy. A strict CSP is important for HIPAA compliance.

**Tasks:**
- Audit the current CSP in `next.config.js`
- Remove `unsafe-inline` by using nonces for inline scripts
- Add `report-uri` to collect CSP violations via `csp-report.controller.ts`
- Test the CSP in Chrome's CSP evaluator tool
- Add a CSP E2E test using `e2e/security-headers.spec.ts`
- Document the CSP policy in `docs/SECURITY_POLICY.md`

---

## [Security] Implement PHI field-level encryption in MongoDB

**Description:**
`apps/api/src/modules/patients/phi-encryption.test.ts` has tests, but the encryption implementation may not cover all PHI fields or may use a weak algorithm.

**Tasks:**
- Audit which patient fields are classified as PHI (per HIPAA Safe Harbor or Expert Determination)
- Verify AES-256-GCM is used for field-level encryption
- Ensure the encryption key is loaded from AWS Secrets Manager, not the environment
- Add encrypted fields for: `nationalId`, `phone`, `address`, `insuranceNumber`
- Verify encrypted fields cannot be queried directly (add a searchable hash index instead)
- Write tests verifying data is stored encrypted and decrypted on read

---

## [Security] Add brute-force protection to the patient portal login

**Description:**
The portal has SMS OTP and MFA (`portal-mfa.service.ts`), but the portal login endpoint may lack brute-force protection (account lockout after N failures).

**Tasks:**
- Add progressive delays after failed portal login attempts
- Lock the portal account after 10 failed attempts and require email/SMS verification to unlock
- Log failed attempts to the audit log
- Add a CAPTCHA challenge after 5 failed attempts
- Write tests simulating brute-force attacks
- Document the lockout policy in `docs/SECURITY_POLICY.md`

---

## [Security] Implement API key scoping and least-privilege access

**Description:**
`apps/api/src/modules/api-keys/` manages API keys but keys may grant broad access. Scoped keys allow third-party integrators to access only the data they need.

**Tasks:**
- Add a `scopes` array to the API key model (e.g., `patients:read`, `payments:write`)
- Enforce scope checks in the `api-key.middleware.ts`
- Display available scopes in the API key creation UI
- Add a test verifying a key with `patients:read` scope cannot write
- Rotate all existing keys without scopes on next login (warn users)
- Document the available scopes in the API docs

---

## [Security] Add SSRF protection to outbound HTTP requests

**Description:**
The API makes outbound HTTP requests to Stellar Horizon, exchange rate APIs, and webhooks. Without SSRF protection, an attacker could redirect requests to internal services.

**Tasks:**
- Add an allowlist of permitted outbound hostnames in `packages/config/`
- Block requests to RFC 1918 private IP ranges (10.x.x.x, 172.16.x.x, 192.168.x.x) and localhost
- Add SSRF protection to the webhook delivery service
- Log blocked outbound requests to the audit log
- Write tests verifying requests to private IPs are blocked

---

## [Security] Implement TOTP-based MFA for clinic admin accounts

**Description:**
`apps/api/src/modules/auth/totp.service.ts` exists and MFA toggle is in the settings UI, but the enforcement for admin and doctor roles may be incomplete.

**Tasks:**
- Enforce MFA for users with `admin`, `doctor`, and `nurse` roles
- Implement a grace period for newly enrolled users (`mfa-grace-period-job.ts`)
- Add a recovery flow using backup codes (`backup-codes.test.ts` tests this)
- Add MFA audit events: enrolled, verified, recovery code used
- Write a test verifying admin login fails without MFA
- Update the MFA documentation in `SECURITY.md`

---

## [Mobile] Add biometric authentication to the mobile app

**Description:**
`apps/mobile/src/services/auth.service.ts` handles login but the mobile app may not support biometric authentication (Face ID, fingerprint) for returning users.

**Tasks:**
- Add `expo-local-authentication` dependency
- Check for biometric support on device startup
- Offer biometric enrollment after successful password login
- Store the auth token securely in the OS keychain using `expo-secure-store`
- Add a fallback to PIN/password when biometric fails
- Write tests mocking the biometric API

---

## [Mobile] Implement offline data caching for patient records

**Description:**
`apps/mobile/src/services/offline-cache.service.ts` stubs exist but the full implementation of caching patient records and encounters for offline use is incomplete.

**Tasks:**
- Cache the last 50 viewed patients and their encounters using AsyncStorage
- Show a "cached" badge on data that may be stale
- Implement a sync strategy when connectivity is restored
- Add conflict resolution: show a diff when server data differs from cached data
- Test offline behavior by disabling the network in the test environment

---

## [Mobile] Add push notifications for appointment reminders

**Description:**
`apps/mobile/src/services/notification.service.ts` stubs exist, but the full Expo push notification integration for appointment reminders is incomplete.

**Tasks:**
- Register the device for Expo push notifications on login
- Store the Expo push token on the API (`users.controller.ts`)
- Send appointment reminders 24 hours and 2 hours before the appointment
- Handle notification taps by deep-linking to the appointment detail
- Test notification delivery using the Expo push notification tool
- Add a notification preferences screen in the mobile app

---

## [Mobile] Add immunization tracking screen with due date alerts

**Description:**
`apps/mobile/src/screens/ImmunizationsScreen.tsx` exists but may lack due date visualization and alert integration.

**Tasks:**
- Show upcoming due immunizations in a card at the top of the screen
- Add a visual timeline of past and upcoming immunizations
- Send a push notification 7 days before an immunization is due
- Allow caregivers to mark immunizations as administered from the mobile app
- Add a PDF export of the immunization certificate
- Write tests for the due date calculation logic

---

## [Mobile] Implement care plans progress tracking in mobile

**Description:**
`apps/mobile/src/screens/CarePlansScreen.tsx` lists care plans but may lack progress tracking, goal completion, and reminders.

**Tasks:**
- Add a progress bar for each care plan goal
- Allow patients to mark daily/weekly tasks as complete from the mobile app
- Send a daily reminder push notification for outstanding tasks
- Show a streak counter for consistent task completion
- Sync completion status with the backend API
- Write tests for the progress calculation logic

---

## [Refactoring] Migrate encounters module to service-repository pattern

**Description:**
`encounters.controller.ts` (30KB) is too large and directly queries MongoDB. Extracting a repository layer improves testability and separation of concerns.

**Tasks:**
- Create `encounter.repository.ts` with typed methods: `findById`, `findByPatient`, `findByClinic`, `create`, `update`, `softDelete`
- Move all Mongoose queries from the controller into the repository
- Create `encounter.service.ts` that calls the repository and applies business logic
- Keep the controller thin: only handle HTTP request/response
- Update all tests to mock the repository instead of the database
- Document the pattern in `CONTRIBUTING.md`

---

## [Refactoring] Centralize clinic-scoping logic in a shared middleware

**Description:**
Nearly every controller extracts `clinicId` from the JWT and uses it to scope database queries. This pattern is duplicated hundreds of times and should be centralized.

**Tasks:**
- Create a `clinicScope` middleware in `apps/api/src/middlewares/`
- The middleware should attach `req.clinicId` after verifying the JWT
- Update all controllers to use `req.clinicId` instead of extracting it manually
- Add a test verifying that a request without a valid clinic scope returns 403
- Document the middleware in the API module development guide

---

## [Refactoring] Consolidate multiple ESLint configuration files

**Description:**
The project has `eslint.config.js` at the root and in each app (`apps/api/eslint.config.js`, `apps/web/.eslintrc.json`, `apps/stellar-service/eslint.config.js`). This duplication leads to inconsistent rules.

**Tasks:**
- Create a canonical ESLint config in `packages/config/eslint-config/`
- Extend the shared config in each app's ESLint file
- Add app-specific overrides only where necessary
- Run `eslint --fix` across the codebase to apply the new rules
- Ensure the `pre-commit` hook runs lint-staged correctly

---

## [Refactoring] Improve structured logging across all services

**Description:**
`apps/stellar-service/src/logger.ts` and the API use a logger, but log levels, fields, and formats may be inconsistent across services, making Logstash parsing unreliable.

**Tasks:**
- Standardize on a JSON log format with mandatory fields: `timestamp`, `level`, `service`, `requestId`, `message`
- Add `clinicId` and `userId` to all request-scoped log entries
- Remove `console.log` and `console.error` calls (replace with the logger)
- Add a lint rule banning `console.log` in production code
- Test that Logstash parses the standardized log format correctly

---

## [Refactoring] Reduce bundle size of the Next.js web app

**Description:**
Large bundle sizes hurt initial load performance. The web app likely has opportunities for code splitting, tree shaking, and dependency removal.

**Tasks:**
- Run `@next/bundle-analyzer` and generate a bundle visualization
- Identify the top 5 largest dependencies and evaluate alternatives
- Add dynamic `import()` for heavy components (charts, PDF generator, signature pad)
- Remove unused CSS with PurgeCSS (should be handled by Tailwind, verify config)
- Set a bundle size budget in Webpack/Next.js config to fail builds that exceed it
- Document the bundle optimization strategy

---

## [Refactoring] Move hardcoded configuration values to environment variables

**Description:**
Some values that should be configurable are hardcoded: retry counts, timeout values, batch sizes, rate limit thresholds.

**Tasks:**
- Grep for hardcoded timeout values (`5000`, `30000`, etc.) in controller and service files
- Move retry counts and timeouts to `packages/config/index.ts` with env var overrides
- Move batch size limits to config with sensible defaults
- Validate configuration at startup and fail fast if required values are missing
- Update `.env.example` with the new configuration variables

---

## [Feature] Implement telemedicine video call integration

**Description:**
`appointments/telemedicine.service.ts` and `telemedicine.test.ts` exist, but the video call integration (e.g., Twilio Video, Daily.co, or WebRTC) is not complete.

**Tasks:**
- Choose a video call provider (evaluate Twilio Video, Daily.co, Jitsi self-hosted)
- Implement room creation and token generation in `telemedicine.service.ts`
- Add a "Start Video Call" button to the appointment detail page
- Implement the in-browser video call UI using the provider's SDK
- Add waiting room functionality for patients
- Record session metadata (start time, duration, participants) for billing purposes

---

## [Feature] Implement subscription and billing management

**Description:**
`apps/api/src/modules/subscriptions/` and `SubscriptionSection.tsx` exist, but the subscription lifecycle (trial, upgrade, downgrade, cancellation) may be incomplete.

**Tasks:**
- Integrate with Stripe or a payment processor for subscription billing
- Implement trial expiry notifications
- Add plan upgrade/downgrade with proration
- Implement feature flags based on subscription tier (`subscription.tiers.ts`)
- Add a subscription analytics dashboard for admins
- Write tests for plan change scenarios

---

## [Feature] Implement patient portal surveys

**Description:**
`apps/api/src/modules/surveys/` and the survey model exist, but the patient portal does not expose surveys for patients to complete.

**Tasks:**
- Add a `/portal/surveys` page listing pending surveys
- Implement the survey completion flow with progress indicator
- Support question types: multiple choice, Likert scale, free text, date
- Send a notification when a new survey is assigned
- Show survey results to clinic staff in the encounter detail
- Add analytics: response rate, average scores per question

---

## [Feature] Implement automated billing aging report

**Description:**
`apps/api/src/modules/billing/billing-aging.ts` stubs an aging report, but the full implementation with 30/60/90/120+ day buckets and automated reminder emails is not complete.

**Tasks:**
- Calculate aging buckets for all outstanding invoices
- Implement automated email reminders at 30, 60, and 90 days overdue
- Add an aging summary to the finance dashboard
- Allow manual overrides: mark as "in dispute", "payment plan", "written off"
- Export the aging report to CSV and PDF
- Write tests for the aging bucket calculation

---

## [Feature] Implement Stellar DEX trading for currency conversion

**Description:**
`apps/api/src/modules/payments/dex/` directory exists, suggesting DEX (decentralized exchange) features are planned for converting between currencies using Stellar's DEX.

**Tasks:**
- Implement path payment discovery using the Stellar Horizon `/paths` API
- Show available conversion paths and exchange rates before a payment
- Allow patients to pay in local currency (converted to XLM via DEX)
- Add slippage protection: cancel if rate moves more than X% during submission
- Show DEX conversion fees in the payment breakdown
- Write integration tests using the Stellar testnet DEX

---

## [Feature] Implement population health management dashboard

**Description:**
`apps/api/src/modules/ai/population-health.service.ts` and `population-health.controller.ts` exist, but there is no frontend for population health analytics.

**Tasks:**
- Create a `/reports/population-health` page
- Show: prevalence rates for top diagnoses, vaccination coverage, chronic disease distribution
- Add cohort filtering: age group, gender, clinic branch
- Integrate with the AI risk stratification service to show high-risk cohorts
- Add an export to CSV for research data (with anonymization)
- Visualize trends over time using line charts

---

## [Feature] Add differential diagnosis suggestion to encounter form

**Description:**
`DifferentialDiagnosisPanel.tsx` exists, but the integration with the AI service (`differential-diagnosis.service.ts`) and encounter form may be incomplete.

**Tasks:**
- Show AI-suggested diagnoses in real time as the clinician enters SOAP notes
- Display confidence scores and supporting evidence for each suggestion
- Allow the clinician to accept, reject, or modify suggestions
- Log accepted/rejected suggestions for AI model improvement
- Add a feedback button to flag incorrect suggestions
- Write tests for the suggestion display and accept/reject flow

---

## [Feature] Implement peer review workflow for encounters

**Description:**
`apps/api/src/modules/peer-reviews/peer-reviews.router.ts` and `peer-reviews.test.ts` exist, but the frontend for peer review management is missing.

**Tasks:**
- Create a peer review request form in the encounter detail page
- Show pending peer reviews on the reviewer's dashboard
- Implement the review completion form: rating, comments, recommendations
- Send notifications to the author when a review is completed
- Add peer review analytics: average rating, common feedback themes
- Write E2E tests for the full peer review workflow

---

## [Feature] Implement claim submission and insurance integration

**Description:**
`apps/api/src/modules/billing/claim-builder.ts` and `claims.controller.ts` exist in payments, but the end-to-end insurance claim submission workflow may be incomplete.

**Tasks:**
- Implement CMS-1500 (HCFA) claim form generation as a PDF
- Add a clearinghouse integration stub (Availity, Change Healthcare)
- Implement claim status tracking: submitted, accepted, rejected, paid, denied
- Add a claims dashboard showing ERA (Electronic Remittance Advice) status
- Implement claim correction and resubmission workflow
- Write tests for claim form field validation and status transitions

---

## [Feature] Add recurring payment management to patient portal

**Description:**
`recurring-payment.controller.ts` and `recurring-payment.service.ts` exist on the API side. The portal should allow patients to manage their recurring payments.

**Tasks:**
- Add a `/portal/payments/recurring` page listing active recurring payments
- Allow patients to pause, resume, or cancel a recurring payment
- Show payment history for each recurring plan
- Send email confirmation when a recurring payment is modified
- Add a portal payment method management page (add/remove cards)
- Write Playwright tests for the recurring payment management flow

---

## [Feature] Implement clinical decision support (CDS) rule management

**Description:**
`CDSRuleForm.tsx`, `CDSRulesList.tsx`, and `CDSRuleTester.tsx` exist in the frontend, and the CDS rules engine is implemented on the backend. The management UI may be incomplete.

**Tasks:**
- Complete the rule creation form with a visual condition builder
- Add a rule testing sandbox that runs rules against sample patient data
- Implement rule versioning: show history of changes to a rule
- Add role-based access: only `admin` and `medical_director` can create rules
- Add rule conflict detection: warn when two rules fire for the same condition
- Write tests for the rule creation and testing flows

---

## [Feature] Implement HIPAA data export for patients

**Description:**
`.changeset/feat-hipaa-data-export.md` indicates this feature is planned. Patients have the right to access and export their health data under HIPAA.

**Tasks:**
- Add a "Download my data" button to the patient portal
- Export all data: demographics, encounters, lab results, medications, immunizations, payments
- Support formats: JSON (FHIR R4), PDF, and CSV
- Implement async export for large records (notify by email when ready)
- Add a 30-day download link expiry for security
- Log all data exports to the audit trail
- Write tests verifying the export contains complete and anonymized-where-appropriate data

---

## [Feature] Implement referral tracking with external provider integration

**Description:**
`referrals.controller.ts` manages internal referrals, but tracking referrals to external providers and receiving status updates from them is not implemented.

**Tasks:**
- Add an "external provider" type to the referral model
- Implement a referral status webhook endpoint for external providers
- Add a referral status timeline in the patient detail page
- Send automated follow-up notifications to the referring clinic if no status update is received within 30 days
- Add a referral analytics report: referral volume by specialty, average time to first appointment
- Write tests for the external referral webhook handler

---

## [Infrastructure] Add Terraform modules for AWS infrastructure

**Description:**
The project has Kubernetes manifests and Helm charts but no Infrastructure-as-Code for the underlying AWS resources (EKS, VPC, RDS, S3, SES).

**Tasks:**
- Create a `terraform/` directory with modules for: VPC, EKS cluster, S3 buckets, SES configuration, IAM roles
- Add a `terraform/environments/` directory for staging and production configs
- Document how to bootstrap the infrastructure in `docs/DEPLOYMENT_GUIDE.md`
- Add `terraform plan` to CI for pull requests that touch `terraform/`
- Use remote state (S3 + DynamoDB) for state management

---

## [Infrastructure] Implement multi-tenant clinic isolation at the infrastructure level

**Description:**
Clinic data is scoped via `clinicId` in the application layer, but there is no infrastructure-level isolation (separate namespaces, network policies, or database users per tenant).

**Tasks:**
- Add Kubernetes `NetworkPolicy` resources to isolate clinic namespaces (if multi-namespace is adopted)
- Document the current multi-tenancy model and its security assumptions
- Add database-level read/write users with scoped permissions per clinic tier
- Add a penetration test scenario: clinic A should not access clinic B's data
- Document multi-tenant compliance considerations in `SECURITY.md`

---

## [Infrastructure] Add CDN configuration for static assets

**Description:**
`.env.cdn.example` references CDN configuration, but the CDN setup for the Next.js app's static assets is not fully documented or automated.

**Tasks:**
- Configure CloudFront (or similar CDN) to serve `/_next/static/` assets
- Add cache-busting via content hashes (Next.js does this by default — verify)
- Configure CDN to set long `Cache-Control` headers for static assets
- Add CDN cache invalidation to the deployment pipeline
- Document CDN setup and configuration in `docs/DEPLOYMENT_GUIDE.md`

---

## [Infrastructure] Implement MongoDB connection pooling optimization

**Description:**
`docs/REPLICATION_OPTIMIZATION.md` documents replication, but connection pooling settings for high-traffic scenarios are not explicitly configured.

**Tasks:**
- Set optimal `maxPoolSize`, `minPoolSize`, and `waitQueueTimeoutMS` in the MongoDB connection config
- Add connection pool metrics to Prometheus
- Add an alert when the wait queue exceeds a threshold
- Test pool exhaustion behavior under load
- Document the connection pool configuration in `docs/DATABASE_SCHEMA.md`

---
