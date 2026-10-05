This report identifies concrete defects, security vulnerabilities, correctness bugs, and operational mismatches discovered by static analysis of the accountill codebase.

Gap 1: Absence of Authentication & Authorization on Core Resource Routes
Underlying Claim Classification: [Confirmed]
1. Problem: The authentication middleware is never imported or mounted on invoice, client, or profile routes.
2. Exact Evidence:
server/routes/invoices.js:1–14: Neither imports nor references auth middleware. Routes like GET /invoices/:id, PATCH /invoices/:id, DELETE /invoices/:id, and POST /invoices are directly attached to controllers.
server/routes/clients.js:1–12: No auth middleware attached to customer routes.
server/routes/profile.js:1–14: No auth middleware attached to profile routes.
3. Why it Matters: Any unauthenticated client can read, overwrite, or delete invoices, customer records, and freelancer profiles across the entire system.
4. Affected User/System: Entire database, all registered freelancers, and their customer data.
5. Suggested Fix: Import auth from ../middleware/auth.js into server/routes/invoices.js, server/routes/clients.js, and server/routes/profile.js, and verify that req.userId matches the record's creator or userId before returning or mutating data.
6. Severity: Critical
Gap 2: Flawed Token Validation in Auth Middleware Permitting Identity Forgery
Underlying Claim Classification: [Confirmed]
1. Problem: The middleware distinguishes custom tokens from Google OAuth tokens solely based on string length, and decodes long tokens without verifying cryptographic signatures.
2. Exact Evidence: server/middleware/auth.js:9–25:
javascript
const token = req.headers.authorization.split(" ")[1]
const isCustomAuth = token.length < 500 
if(token && isCustomAuth) {
    decodeData = jwt.verify(token, SECRET)
    req.userId = decodeData?.id
} else {
    decodeData = jwt.decode(token)
    req.userId = decodeData?.sub
}
3. Why it Matters: jwt.decode() does not verify signatures. Anyone can craft an unsigned JWT containing an arbitrary sub claim, pad it with whitespace or dummy claims so the length exceeds 500 characters, and completely bypass authentication to act as any user.
4. Affected User/System: The entire authentication boundary and any controller relying on req.userId.
5. Suggested Fix: Use the official Google Auth library (google-auth-library or OAuth2Client.verifyIdToken) to validate Google tokens against Google's public JWKS, and remove length-based token branching.
6. Severity: Critical
Gap 3: File Collision Race Condition in PDF Generation and Download
Underlying Claim Classification: [Confirmed]
1. Problem: Generated PDF files are written to a single, static file path on disk (invoice.pdf) shared across all users and concurrent requests.
2. Exact Evidence:
server/index.js:57: pdf.create(pdfTemplate(req.body), options).toFile('invoice.pdf', ...)
server/index.js:88: pdf.create(pdfTemplate(req.body), {}).toFile('invoice.pdf', ...)
server/index.js:97–99: app.get('/fetch-pdf', (req, res) => { res.sendFile(${__dirname}/invoice.pdf) })
3. Why it Matters: If User A creates an invoice while User B is downloading theirs, User B receives User A's generated invoice. Furthermore, GET /fetch-pdf requires neither authentication nor an invoice ID parameter, allowing anyone to poll the endpoint and continuously harvest generated invoices.
4. Affected User/System: All invoicing workflows, financial privacy, tenant separation.
5. Suggested Fix: Stream the PDF buffer directly in the HTTP response using pdf.create(...).toStream((err, stream) => stream.pipe(res)) rather than writing to a shared file on disk.
6. Severity: High
Gap 4: Server-Side Request Forgery (SSRF) and Local File Inclusion in html-pdf
Underlying Claim Classification: [Confirmed]
1. Problem: Unsanitized user input is concatenated directly into HTML template strings and processed by html-pdf (PhantomJS).
2. Exact Evidence:
server/documents/index.js:115: ${company?.logo ?Preview unavailable:
___
}
server/documents/index.js:125–136, 163–172, 213: Raw interpolation of notes, company.businessName, email, address, and item.itemName.
server/index.js:87–88: Public POST /create-pdf executes pdf.create(pdfTemplate(req.body), {}).
3. Why it Matters: Because PhantomJS is a headless WebKit browser that executes JavaScript and accesses local URLs, an attacker can embed <iframe src="file:///etc/passwd"> or execute XMLHttpRequest to read server files or query internal network services (SSRF) and render the output into the resulting PDF.
4. Affected User/System: Host server filesystem, internal cloud metadata endpoints (e.g. AWS/GCP metadata).
5. Suggested Fix: HTML-escape all interpolated user strings (e.g., via he or escape-html) and disable external and local resource loading in PhantomJS options (localToRemoteUrlAccessEnabled: false).
6. Severity: High
Gap 5: Global Customer Data Exposure via Unfiltered GET /clients
Underlying Claim Classification: [Confirmed]
1. Problem: The getClients controller queries and paginates all clients in the database without filtering by user.
2. Exact Evidence: server/controllers/clients.js:44–47:
javascript
const total = await ClientModel.countDocuments({});
const clients = await ClientModel.find().sort({ _id: -1 }).limit(LIMIT).skip(startIndex);
res.json({ data: clients, currentPage: Number(page), numberOfPages: Math.ceil(total / LIMIT)});
3. Why it Matters: Any anonymous internet user can call GET /clients?page=1, 2, ... and scrape the names, emails, phone numbers, and addresses of all customers registered by all users on the platform.
4. Affected User/System: All customer records and freelancers' client lists.
5. Suggested Fix: Scope the query strictly to the authenticated user ID: ClientModel.find({ userId: req.userId }).
6. Severity: High
Gap 6: Production Docker Build & Startup Failures
Underlying Claim Classification: [Confirmed]
1. Problem: Neither the client nor the server Docker container can build or run using the repository's production Docker setup.
2. Exact Evidence:
client/Dockerfile:7: COPY client/yarn.lock . fails during build because no yarn.lock exists in client/ (only package-lock.json is present).
server/Dockerfile:14: CMD ["yarn", "start-prod"] fails on container start because no start-prod script is defined in server/package.json:8–10.
3. Why it Matters: The documented container deployment instructions (docker-compose -f docker-compose.prod.yml build in README.md:159) fail immediately.
4. Affected User/System: Operators, DevOps, and users attempting to containerize or deploy the application.
5. Suggested Fix:
In client/Dockerfile: Remove client/yarn.lock and use COPY client/package*.json . with npm install or npm ci.
In server/Dockerfile: Replace CMD ["yarn", "start-prod"] with CMD ["npm", "start"].
6. Severity: High
Gap 7: Google Login ID Mismatch Breaking Business Profile Association
Underlying Claim Classification: [Confirmed]
1. Problem: Upon Google login, the client saves the profile using the ephemeral token ID (jti), while the server authenticates using the user subject ID (sub).
2. Exact Evidence:
client/src/components/Login/Login.js:54–56:
javascript
const result = jwtDecode(res.credential);
dispatch(createProfile({ name: result?.name, email: result?.email, userId: result?.jti, ... }))
server/middleware/auth.js:24: req.userId = decodeData?.sub
3. Why it Matters: jti changes on every login (unique JWT ID), whereas sub is the user's permanent Google ID. Storing jti in ProfileModel.userId breaks user-profile lookup across sessions, resulting in empty business settings and missing company branding on invoices.
4. Affected User/System: All users authenticating via Google Sign-In.
5. Suggested Fix: In client/src/components/Login/Login.js:56, replace userId: result?.jti with userId: result?.sub.
6. Severity: Medium
Gap 8: Express Route Shadowing Breaking /profiles/search
Underlying Claim Classification: [Confirmed]
1. Problem: GET /profiles/:id is declared before any search route, and /profiles/search is not registered on the router.
2. Exact Evidence:
server/routes/profile.js:6–8:
javascript
router.get('/:id', getProfile)
router.get('/', getProfilesByUser)
client/src/api/index.js:34: API.get('/profiles/search?searchQuery=...')
3. Why it Matters: When the client calls /profiles/search, Express routes "search" into getProfile as :id. Mongoose executes ProfileModel.findById("search"), which fails with a CastError (Cast to ObjectId failed for value "search").
4. Affected User/System: Profile search functionality in frontend and API consumers.
5. Suggested Fix: Add router.get('/search', getProfilesBySearch) before router.get('/:id', getProfile) in server/routes/profile.js.
6. Severity: Medium
Gap 9: Static Timestamp Evaluation at Module Load in Mongoose Models
Underlying Claim Classification: [Confirmed]
1. Problem: Mongoose schemas invoke new Date() statically at module evaluation time rather than passing a reference to Date.now.
2. Exact Evidence:
server/models/InvoiceModel.js:21: createdAt: { type: Date, default: new Date() }
server/models/ClientModel.js:12: createdAt: { type: Date, default: new Date() }
3. Why it Matters: default: new Date() evaluates once when Node boots. Any invoice or customer created without an explicit client-supplied createdAt will receive the startup timestamp of the server process rather than the actual document creation time.
4. Affected User/System: Audit logs, invoice creation dates, and time-based sorting.
5. Suggested Fix: Change default: new Date() to default: Date.now.
6. Severity: Medium
Gap 10: Hanging Requests on Auth Middleware Failures
Underlying Claim Classification: [Confirmed]
1. Problem: The catch block in auth.js logs errors to the console without sending an HTTP response or calling next(error).
2. Exact Evidence: server/middleware/auth.js:29–31:
javascript
} catch (error) {
    console.log(error)
}
3. Why it Matters: If a client sends a request without an Authorization header, .split(" ") throws a TypeError: Cannot read property 'split' of undefined. Because next() is never reached and no response is returned, the connection hangs until the client times out.
4. Affected User/System: All incoming requests with invalid or missing auth headers.
5. Suggested Fix: Add an error response inside the catch block: return res.status(401).json({ message: "Invalid or missing token" });.
6. Severity: Medium
Gap 11: Invoice View Loads Viewer's Business Profile Rather Than Creator's Profile
Underlying Claim Classification: [Confirmed]
1. Problem: When fetching invoice details, the frontend queries the profile of whoever is currently logged in, rather than the invoice issuer.
2. Exact Evidence: client/src/actions/invoiceActions.js:27–34:
javascript
export const getInvoice = (id) => async (dispatch)=> {
    const user = JSON.parse(localStorage.getItem('profile'))
    try {
        const { data } = await api.fetchInvoice(id)
        const businessDetails = await api.fetchProfilesByUser({search: user?.result?._id || user?.result?.googleId})
        const invoiceData = {...data, businessDetails}
3. Why it Matters: If a customer opens the invoice link sent to their email (or if another user views the invoice), the invoice header displays the viewer's business profile or blank information rather than the business details of the freelancer who issued the invoice.
4. Affected User/System: End clients receiving and paying invoices.
5. Suggested Fix: Fetch the issuer's profile using the invoice's creator identity: await api.fetchProfilesByUser({ search: data.creator[0] }).
6. Severity: Medium
Gap 12: Hardcoded Cloudinary Upload Credentials in Frontend Bundle
Underlying Claim Classification: [Confirmed]
1. Problem: The Cloudinary cloud name and unsigned upload preset are hardcoded in the frontend.
2. Exact Evidence: client/src/components/Settings/Form/Uploader.js:33, 41–43:
javascript
const url = "https://api.cloudinary.com/v1_1/almpo/image/upload";
formData.append("file", acceptedFile);
formData.append("upload_preset", "invoice");
3. Why it Matters: Anyone inspecting client source code can post arbitrary files and media to this Cloudinary account, exhausting bandwidth/storage limits or hosting unapproved content.
4. Affected User/System: Storage quotas and third-party media infrastructure.
5. Suggested Fix: Use backend-signed uploads with ephemeral signatures generated via Cloudinary SDK on an authenticated endpoint.
6. Severity: Medium
Gap 13: Regular Expression Denial of Service (ReDoS) in Search
Underlying Claim Classification: [Confirmed]
1. Problem: searchQuery is passed unescaped directly into new RegExp().
2. Exact Evidence: server/controllers/profile.js:89–92:
javascript
const name = new RegExp(searchQuery, "i");
const email = new RegExp(searchQuery, "i");
const profiles = await ProfileModel.find({ $or: [ { name }, { email } ] });
3. Why it Matters: Supplying catastrophic backtracking regex patterns in ?searchQuery= freezes the single-threaded Node.js event loop, preventing all concurrent users from processing requests.
4. Affected User/System: Backend server availability and event loop.
5. Suggested Fix: Escape regex characters or use MongoDB text indexes ($text: { $search: searchQuery }).
6. Severity: Medium
Gap 14: Unhandled Promise Rejections Passed to res.send()
Underlying Claim Classification: [Confirmed]
1. Problem: Passing Promise.reject() into Express res.send() causes unhandled promise rejections.
2. Exact Evidence:
server/index.js:74: if(err) { res.send(Promise.reject()); }
server/index.js:90: if(err) { res.send(Promise.reject()); }
3. Why it Matters: Express does not handle rejected promises passed directly into res.send(). In modern Node.js runtimes, unhandled rejections trigger process warnings or terminate the process.
4. Affected User/System: Node.js runtime stability during PDF generation failures.
5. Suggested Fix: Replace with res.status(500).json({ error: err.message }).
6. Severity: Low
Gap 15: Redundant Synchronous LocalStorage Writes on Form Renders
Underlying Claim Classification: [Confirmed]
1. Problem: localStorage.setItem is called directly in the component render body rather than inside a hook.
2. Exact Evidence: client/src/components/Settings/Form/Form.js:53:
javascript
localStorage.setItem('profileDetail', JSON.stringify({...profiles}))
3. Why it Matters: Executes on every state update and keystroke, introducing unnecessary synchronous disk I/O on the browser's main thread.
4. Affected User/System: Browser UI responsiveness on the Settings page.
5. Suggested Fix: Wrap the statement inside a useEffect hook: useEffect(() => { localStorage.setItem(...); }, [profiles]);.
6. Severity: Low
Gap 16: Upstream Rebranding String Mismatches
Underlying Claim Classification: [Confirmed]
1. Problem: While the repository was rebranded from "Arc Invoice" to "Accountill" (README.md:8–10), old branding remains in user-facing transactional emails and navigation assets.
2. Exact Evidence:
server/controllers/user.js:121: "You requested for password reset from Arc Invoicing application"
server/documents/email.js:152: alt="arc-invoice"
client/src/components/NavBar/NavBar.js:21: alt="arc-invoice"
3. Why it Matters: Produces inconsistent brand presentation and user distrust during password resets.
4. Affected User/System: End users performing password recovery.
5. Suggested Fix: Update all residual template strings to refer to "Accountill".
6. Severity: Low
Ranked Summary of Gaps
Rank	Gap Description	Severity	Area	Evidence
1	Completely unauthenticated invoice, client, and profile CRUD routes	Critical	Security / Auth	server/routes/invoices.js:1–14
2	Unverified JWT decoding allowing arbitrary identity forgery	Critical	Security / Auth	server/middleware/auth.js:10, 22–24
3	Shared invoice.pdf file collision causing cross-user data leakage	High	Data Integrity	server/index.js:57, 88, 98
4	Unsanitized template interpolation enabling SSRF and file read via PhantomJS	High	Security / RCE	server/documents/index.js:115, 125
5	Unfiltered GET /clients endpoint dumping all customer PII	High	Privacy / Data Leak	server/controllers/clients.js:44–47
6	Broken Dockerfiles (missing yarn.lock, missing start-prod script)	High	DevOps / Build	client/Dockerfile:7, server/Dockerfile:14
7	Google login ephemeral jti used instead of permanent sub	Medium	Correctness	client/src/components/Login/Login.js:56
8	Express route shadowing making /profiles/search fail with CastError	Medium	Correctness	server/routes/profile.js:6–8
9	Static default: new Date() evaluated at module load time	Medium	Data Integrity	server/models/InvoiceModel.js:21
10	Auth middleware hanging on errors instead of returning 401	Medium	Reliability	server/middleware/auth.js:29–31
11	Single-user profile lookup bug when displaying invoices to clients	Medium	UX / Correctness	client/src/actions/invoiceActions.js:33
12	Hardcoded Cloudinary unsigned upload preset	Medium	Security	client/src/components/Settings/Form/Uploader.js:33
13	ReDoS vulnerability in profile search query	Medium	Security	server/controllers/profile.js:89–92
14	Promise.reject() in res.send() causing unhandled rejection warnings	Low	Stability	server/index.js:74, 90
15	Naked localStorage.setItem in component render body	Low	Performance	client/src/components/Settings/Form/Form.js:53
16	Legacy "Arc" branding remnants in emails and navigation tags	Low	Documentation	server/controllers/user.js:121
