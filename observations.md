# Observation

## Claim 1 — MERN Stack Architecture

Claim: Accountill is a MERN-stack invoicing application, using a Node.js/Express backend, a React/Redux frontend, and MongoDB with Mongoose.
Evidence: README.md:1-12
Confidence: Confirmed

## Claim 2 — Unauthenticated Invoice API Routes

**Claim:** The `/invoices` API router exposes invoice creation, retrieval, mutation, and deletion without attaching any authentication middleware or user-ownership verification, leaving endpoints like `GET /invoices/:id`, `PATCH /invoices/:id`, and `DELETE /invoices/:id` publicly accessible.
**Evidence:** `server/routes/invoices.js:1-14`
**Confidence:** Confirmed

## Claim 3 — Denormalized Embedded Client Data Model

**Claim:** In `InvoiceModel`, customer data is not stored as a relational reference or foreign key to `ClientModel._id`; instead, customer contact fields (`name`, `email`, `phone`, `address`) are snapshot-copied into an embedded object (`client`), while the owning user identity is stored as an array of strings (`creator: [String]`).
**Evidence:** `server/models/InvoiceModel.js:15-17`
**Confidence:** Confirmed

## Claim 4 — Shared PDF File Overwrite Race Condition

**Claim:** PDF generation routes write output to a single static file path (`'invoice.pdf'`) on the server filesystem, creating a race condition where concurrent generation requests overwrite each other and unauthenticated calls to `GET /fetch-pdf` serve whichever user's invoice was generated most recently.
**Evidence:** `server/index.js:57, 88, 97-99`
**Confidence:** Confirmed

## Claim 5 — AI Correction

**Original AI claim:** `GET /clients/:id` is an active server route handled by the `getClient` controller to retrieve a single customer record.

**Why it was wrong:** While `getClient` is implemented in `server/controllers/clients.js:24-34` and referenced by the frontend API client in `client/src/api/index.js:21`, `server/routes/clients.js:1-12` never imports or registers `getClient` on the Express router.

**Corrected claim:** `GET /clients/:id` is not registered on the Express router, leaving `getClient` as unmounted dead code in the controller and causing requests to fetch a single client by ID to return an HTTP 404 response.

**Evidence:** `server/routes/clients.js:1-12`

**Confidence:** Confirmed
