Stage 8: Verification & Audit Report

This stage conducts an evidence-by-evidence audit of the findings from Stages 0, 3, 4, and 7 against the raw codebase in accountill. Every claim has been re-verified by inspecting source files line by line.

1. Explicit List of Corrections & Clarifications

Before presenting the verified findings, the following corrections and precision adjustments to earlier stages are noted:

Unmounted Controller Handlers Identified (getClient and getInvoices) [Confirmed]:

In server/controllers/clients.js:24–34, getClient (GET /clients/:id) is implemented. However, server/routes/clients.js:1–12 only imports and mounts getClients, createClient, updateClient, deleteClient, and getClientsByUser. getClient is not mounted. Attempting GET /clients/:id yields an Express 404.
In server/controllers/invoices.js:36–48, getInvoices is implemented, but server/routes/invoices.js:2, 8 mounts getInvoicesByUser at GET /invoices. getInvoices is unmounted dead code.

Route Collision & Commented-out Route in profile.js [Confirmed]:

In server/routes/profile.js:7, // router.get('/', getProfiles) is explicitly commented out, and line 8 mounts router.get('/', getProfilesByUser).
getProfilesBySearch (server/controllers/profile.js:85–98) is implemented in the controller, but is never imported or registered on the router in server/routes/profile.js:2.

docker-compose.prod.yml vs README.md Database URI Clarification [Confirmed]:

The database URI mongodb://mongo:27017/arch appears in the documentation's Docker instructions at README.md:141. docker-compose.prod.yml:1–33 itself does not hardcode environment variables and expects them via external environment files.
2. Verified Technical Findings
2.1 Languages, Frameworks & Tooling
Component	Claim	Classification	Supporting Source Evidence
Backend Runtime	Node.js with native ES Modules	Confirmed	server/package.json:7 ("type": "module")
Backend Framework	Express ^4.17.1	Confirmed	server/package.json:18, server/index.js:4, 25
Frontend Framework	React ^17.0.2 & react-dom ^17.0.2	Confirmed	client/package.json:25, 27, client/src/index.js:3–4, 16–20
Build Pipeline	Create React App via react-scripts 4.0.3	Confirmed	client/package.json:34, 44–47
Node 22 Build Incompatibility	CRA 4.0.3 / Webpack 4 fails on Node 17+ (digital envelope legacy crypto)	Likely	react-scripts@4.0.3 relies on Webpack 4.44.2 which uses MD4 crypto algorithms unsupported by OpenSSL 3.0 in Node 17+
State Management	Redux ^4.1.0 with redux-thunk ^2.3.0	Confirmed	client/package.json:38–39, client/src/index.js:8–14
UI Component Library	Material-UI v4 (@material-ui/core 4.11.4)	Confirmed	client/package.json:10–13
HTTP Client	Axios ^0.21.1 with request interceptor	Confirmed	client/package.json:19, client/src/api/index.js:1, 6–12
Package Manager (Actual)	npm	Confirmed	Presence of server/package-lock.json and client/package-lock.json; commands in README.md:96–98, 118–120
Package Manager (Broken)	Yarn referenced in Dockerfiles	Confirmed	server/Dockerfile:14 runs yarn start-prod (nonexistent script); client/Dockerfile:7 copies yarn.lock (missing file)
2.2 Verified Server Routes Map

All routes were verified against their router files and controller exports.

Group 1: User & Authentication (server/routes/userRoutes.js mounted at /users)
POST /users/signin [Confirmed]
Location: server/routes/userRoutes.js:6 -> server/controllers/user.js:18–42
Parameters: Body { email, password }
Auth: None (Public)
Operations: User.findOne, ProfileModel.findOne, bcrypt.compare, jwt.sign
POST /users/signup [Confirmed]
Location: server/routes/userRoutes.js:7 -> server/controllers/user.js:46–68
Parameters: Body { email, password, confirmPassword, firstName, lastName, bio }
Auth: None (Public)
Operations: User.findOne, bcrypt.hash, User.create, jwt.sign
POST /users/forgot [Confirmed]
Location: server/routes/userRoutes.js:8 -> server/controllers/user.js:85–133
Parameters: Body { email }
Auth: None (Public)
Operations: User.findOne, crypto.randomBytes, user.save(), nodemailer.sendMail()
POST /users/reset [Confirmed]
Location: server/routes/userRoutes.js:9 -> server/controllers/user.js:137–156
Parameters: Body { password, token }
Auth: Token validation (User.findOne({ resetToken, expireToken: { $gt: Date.now() } }))
Operations: bcrypt.hash, user.save()
Group 2: Invoices (server/routes/invoices.js mounted at /invoices)
GET /invoices/count [Confirmed]
Location: server/routes/invoices.js:6 -> server/controllers/invoices.js:22–33
Parameters: Query ?searchQuery=<userId>
Auth: None
Operations: InvoiceModel.countDocuments({ creator: searchQuery })
GET /invoices/:id [Confirmed]
Location: server/routes/invoices.js:7 -> server/controllers/invoices.js:68–78
Parameters: Param :id
Auth: None
Operations: InvoiceModel.findById(id)
GET /invoices [Confirmed]
Location: server/routes/invoices.js:8 -> server/controllers/invoices.js:9–19
Parameters: Query ?searchQuery=<userId>
Auth: None
Operations: InvoiceModel.find({ creator: searchQuery })
POST /invoices [Confirmed]
Location: server/routes/invoices.js:9 -> server/controllers/invoices.js:53–66
Parameters: Body (Invoice JSON object)
Auth: None
Operations: new InvoiceModel(invoice).save()
PATCH /invoices/:id [Confirmed]
Location: server/routes/invoices.js:10 -> server/controllers/invoices.js:81–91
Parameters: Param :id, Body (fields to update)
Auth: None
Operations: mongoose.Types.ObjectId.isValid(_id), InvoiceModel.findByIdAndUpdate(_id, ...)
DELETE /invoices/:id [Confirmed]
Location: server/routes/invoices.js:11 -> server/controllers/invoices.js:93–101
Parameters: Param :id
Auth: None
Operations: InvoiceModel.findByIdAndRemove(id)
(Unmounted) getInvoices (server/controllers/invoices.js:36–48): Defined in controller but not registered in server/routes/invoices.js.
Group 3: Clients (server/routes/clients.js mounted at /clients)
GET /clients [Confirmed]
Location: server/routes/clients.js:6 -> server/controllers/clients.js:37–51
Parameters: Query ?page=<number>
Auth: None
Operations: ClientModel.countDocuments({}), ClientModel.find().limit(8).skip(...) (global unauthenticated dump)
GET /clients/user [Confirmed]
Location: server/routes/clients.js:7 -> server/controllers/clients.js:90–100
Parameters: Query ?searchQuery=<userId>
Auth: None
Operations: ClientModel.find({ userId: searchQuery })
POST /clients [Confirmed]
Location: server/routes/clients.js:8 -> server/controllers/clients.js:53–65
Parameters: Body (Client JSON object)
Auth: None
Operations: new ClientModel(...).save()
PATCH /clients/:id [Confirmed]
Location: server/routes/clients.js:9 -> server/controllers/clients.js:67–77
Parameters: Param :id, Body
Auth: None
Operations: ClientModel.findByIdAndUpdate(_id, ...)
DELETE /clients/:id [Confirmed]
Location: server/routes/clients.js:10 -> server/controllers/clients.js:79–88
Parameters: Param :id
Auth: None
Operations: ClientModel.findByIdAndRemove(id)
(Unmounted) getClient (server/controllers/clients.js:24–34): Not registered in server/routes/clients.js.
Group 4: Profiles (server/routes/profile.js mounted at /profiles)
GET /profiles/:id [Confirmed]
Location: server/routes/profile.js:6 -> server/controllers/profile.js:18–28
Parameters: Param :id
Auth: None
Operations: ProfileModel.findById(id)
GET /profiles [Confirmed]
Location: server/routes/profile.js:8 -> server/controllers/profile.js:69–81 (getProfilesByUser)
Parameters: Query ?searchQuery=<userId>
Auth: None
Operations: ProfileModel.findOne({ userId: searchQuery })
POST /profiles [Confirmed]
Location: server/routes/profile.js:9 -> server/controllers/profile.js:30–65
Parameters: Body { name, email, phoneNumber, businessName, contactAddress, logo, website, userId }
Auth: None
Operations: ProfileModel.findOne({ email }), newProfile.save()
PATCH /profiles/:id [Confirmed]
Location: server/routes/profile.js:10 -> server/controllers/profile.js:101–111
Parameters: Param :id, Body
Auth: None
Operations: ProfileModel.findByIdAndUpdate(_id, ...)
DELETE /profiles/:id [Confirmed]
Location: server/routes/profile.js:11 -> server/controllers/profile.js:113–122
Parameters: Param :id
Auth: None
Operations: ProfileModel.findByIdAndRemove(id)
(Unmounted / Commented Out):
getProfiles (server/controllers/profile.js:8–16): Commented out in server/routes/profile.js:7.
getProfilesBySearch (server/controllers/profile.js:85–98): Never mounted on server/routes/profile.js.
Group 5: Root & Utility Endpoints (server/index.js)
POST /send-pdf [Confirmed]: Lines 53–78. Generates PDF to disk, sends email with attachment. Auth: None.
POST /create-pdf [Confirmed]: Lines 87–94. Generates PDF to disk via pdfTemplate(req.body). Auth: None.
GET /fetch-pdf [Confirmed]: Lines 97–99. Serves ${__dirname}/invoice.pdf. Auth: None.
GET / [Confirmed]: Lines 102–104. Returns string 'SERVER IS RUNNING'. Auth: None.
2.3 Verified Data Model & Relationships
Entity	Primary Key	Key Fields & Types	Relationships	Schema Defaults / Anomalies	Evidence
User	_id (ObjectId)	name (String, req), email (String, req, unique), password (String, req), resetToken (String), expireToken (Date)	Parent to Profile, Invoices, and Clients via soft string matching	Unique index on email	server/models/userModel.js:3–11 [Confirmed]
Profile	_id (ObjectId)	name, email (req, unique), phoneNumber, businessName, contactAddress, paymentDetails, logo, website, userId ([String])	1-to-1 with User via userId[0] == User._id	Array used for singular scalar userId; unique index on email	server/models/ProfileModel.js:3–15 [Confirmed]
ClientModel	_id (ObjectId)	name, email, phone, address, userId ([String]), createdAt (Date)	1-to-many with User via userId[0] == User._id	default: new Date() evaluates once at boot time; no required fields	server/models/ClientModel.js:4–16 [Confirmed]
InvoiceModel	_id (ObjectId)	dueDate, currency, items (Array), rates, vat, total, subTotal, notes, status, invoiceNumber, type, creator ([String]), totalAmountReceived, client (Object), paymentRecords (Array), createdAt (Date)	1-to-many with User via creator[0] == User._id; denormalized snapshot copy of Client object	Embedded subdocuments for items and paymentRecords; default: new Date() evaluates once at boot time	server/models/InvoiceModel.js:3–25 [Confirmed]
2.4 Verified Gaps Audit

All 16 identified gaps have been re-verified against the code:

Unprotected Core Routes (server/routes/invoices.js:1–14, clients.js:1–12, profile.js:1–14): Confirmed. auth middleware is completely absent from all three routers.
Length-Based Token Bypass (server/middleware/auth.js:10, 22–24): Confirmed. Tokens with length >= 500 invoke jwt.decode() without cryptographic signature verification.
PDF Race Condition & Overwrite (server/index.js:57, 88, 98): Confirmed. All PDF operations use the static path 'invoice.pdf', causing file collisions across concurrent users.
PhantomJS SSRF / LFI (server/documents/index.js:115, 125–136): Confirmed. User strings are interpolated unescaped into HTML templates fed to PhantomJS.
Global Customer Data Exposure (server/controllers/clients.js:44–47): Confirmed. GET /clients queries all documents without user qualification.
Broken Production Dockerfiles (client/Dockerfile:7, server/Dockerfile:14): Confirmed. yarn.lock is missing in client/; start-prod script is missing in server/package.json.
Google Auth jti vs sub Mismatch (client/src/components/Login/Login.js:56, server/middleware/auth.js:24): Confirmed. Frontend stores ephemeral token ID jti, whereas backend queries permanent subject ID sub.
Express Route Shadowing on Profiles (server/routes/profile.js:6–8): Confirmed. /:id matches literal "search", intercepting client search requests.
Static Timestamp Execution (server/models/InvoiceModel.js:21, ClientModel.js:12): Confirmed. default: new Date() evaluates at Node module parse time.
Auth Error Request Hang (server/middleware/auth.js:29–31): Confirmed. catch block does not call next(error) or send an HTTP error status.
Single-User Invoice Profile Lookup (client/src/actions/invoiceActions.js:33): Confirmed. Frontend fetches the profile of the viewer rather than the invoice creator.
Exposed Cloudinary Preset (client/src/components/Settings/Form/Uploader.js:33, 41–43): Confirmed. Cloud name almpo and preset invoice are hardcoded in the frontend.
ReDoS in Search Query (server/controllers/profile.js:89–92): Confirmed. Query strings are compiled into RegExp without metacharacter escaping.
Promise.reject() Passed to res.send() (server/index.js:74, 90): Confirmed. Generates unhandled rejection warnings.
Unwrapped Synchronous LocalStorage Writes (client/src/components/Settings/Form/Form.js:53): Confirmed. Executes directly in the component render body.
Legacy Branding Remnants (server/controllers/user.js:121, server/documents/email.js:152): Confirmed. "Arc Invoicing" references remain in email templates.
3. Final Verification Classification Summary
Confirmed Findings: 100% of technical claims regarding data structures, routes, line numbers, authentication flow, and operational defects have been re-verified directly in the local repository files.
Likely Findings: Node 22 build incompatibility of react-scripts@4.0.3 (deduced from Webpack 4 OpenSSL 3.0 requirements without running the build).
Guessed Findings: None. Every assertion is backed by explicit source code citations.