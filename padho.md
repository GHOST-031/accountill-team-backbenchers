Reverse-Engineering Technical Report: Accountill

Target Repository: accountill
Stated Product: Invoicing App for Freelancers & Small Businesses
Methodology: Non-executing, static source code reverse engineering and architectural analysis. All findings are backed by line-numbered file citations from the workspace.

1. System Identity & Provenance

Product Origin & Rebranding [Confirmed]: The repository was originally named "Arc" / "Arc Invoice" developed by Panshak Solomon, and subsequently rebranded to "Accountill".

Evidence: README.md lines 8–12 ("I am pleased to inform you that the name of this repository has been changed from Arc Invoice to Accountill").
Evidence: Residual references to Arc in server/controllers/user.js lines 121–122 ("You requested for password reset from Arc Invoicing application"), server/documents/email.js line 152 (alt="arc-invoice"), client/src/components/NavBar/NavBar.js line 21 (alt="arc-invoice"), and commented API call in client/src/components/Header/Header.js line 49 (https://api.github.com/repos/panshak/arc).
Database name in Docker configuration is still arch: docker-compose.prod.yml line 141.

Unrelated Precursor / Dead Code Artifact [Confirmed]: The repository contains an orphaned Redux mock file client/src/store.js lines 1–36 containing equipment loan tracking data (gps, mistnet, camera, trap, binocular, lenders) from the "A.P. Leventis Ornithological Research Institute, Jos", which matches the address in sample invoice template server/documents/invoice.js lines 134–135. The actual Redux store is defined separately in client/src/index.js lines 8–15.
  

  

  
2. Architecture & Technology Stack

  
                     +---------------------------------------+
                     |         Browser / React 17 SPA        |
                     +---------------------------------------+
                        |                 |               |
           Unauthenticated/Auth APIs      |               | Direct Unsigned Upload
                        |                 |               v
                        v                 |        +-----------------+
        +----------------------------+    |        | Cloudinary API  |
        |  Express 4.17.1 Backend    |    |        | (almpo/invoice) |
        |  (Node.js / ES Modules)    |    |        +-----------------+
        +----------------------------+    |
         /      |            \            | Direct Google ID Token
        /       |             \           v
       v        v              v     +---------------------+
  +---------+ +------------+ +-----+ | Google OAuth2 API   |
  | MongoDB | | html-pdf   | | SMTP| +---------------------+
  | Atlas   | | PhantomJS  | | Host|
  +---------+ +------------+ +-----+

  
2.1 Backend Stack

  

  
Runtime & Framework [Confirmed]: Node.js with native ES Modules ("type": "module" in server/package.json line 7), Express ^4.17.1 (server/package.json line 18).

  
Database & ODM [Confirmed]: MongoDB via Mongoose ^5.12.10 (server/package.json line 22).

  
Authentication & Crypto [Confirmed]: jsonwebtoken ^8.5.1, bcryptjs ^2.4.3, Node standard crypto module (server/package.json lines 15, 20).

  
Document Generation & Email [Confirmed]: html-pdf ^3.0.1 (wraps PhantomJS headless browser), nodemailer ^6.6.3 (server/package.json lines 19, 23).

  

  
2.2 Frontend Stack

  

  
Core & Build [Confirmed]: React 17.0.2, react-scripts 4.0.3 (client/package.json lines 25, 34).

  
State Management & Routing [Confirmed]: Redux ^4.1.0, redux-thunk ^2.3.0, react-redux ^7.2.4, react-router-dom ^5.2.0 (client/package.json lines 32, 33, 38, 39).

  
UI Components [Confirmed]: Material-UI v4 (@material-ui/core ^4.11.4, @material-ui/icons ^4.11.2, @material-ui/pickers ^3.3.10, @material-ui/lab ^4.0.0-alpha.58) (client/package.json lines 10–13).

  
Visualization [Confirmed]: ApexCharts ^3.28.1, react-apexcharts ^1.3.9, Recharts ^2.0.9, DevExpress DX React Chart ^2.7.6 (client/package.json lines 7–9, 18, 26, 37).

  

  

  
3. Data Schema & Models

  
3.1 User Model (server/models/userModel.js lines 3–11) [Confirmed]

  
javascript
{
  name: { type: String, required: true },
  email: { type: String, required: true, unique: true },
  password: { type: String, required: true },
  resetToken: String,
  expireToken: Date
}

  
3.2 Profile Model (server/models/ProfileModel.js lines 3–15) [Confirmed]

  
javascript
{
  name: String,
  email: { type: String, required: true, unique: true },
  phoneNumber: String,
  businessName: String,
  contactAddress: String,
  paymentDetails: String,
  logo: String,
  website: String,
  userId: [String]   // Array of strings (stores owning user ID)
}

  
3.3 Client Model (server/models/ClientModel.js lines 4–16) [Confirmed]

  
javascript
{
  name: String,
  email: String,
  phone: String,
  address: String,
  userId: [String],
  createdAt: { type: Date, default: new Date() } // Bug: Static timestamp evaluated at module load
}

  
3.4 Invoice Model (server/models/InvoiceModel.js lines 3–25) [Confirmed]

  
javascript
{
  dueDate: Date,
  currency: String,
  items: [{ itemName: String, unitPrice: String, quantity: String, discount: String }],
  rates: String,
  vat: Number,
  total: Number,
  subTotal: Number,
  notes: String,
  status: String,
  invoiceNumber: String,
  type: String,
  creator: [String],  // Array of strings holding user ID
  totalAmountReceived: Number,
  client: { name: String, email: String, phone: String, address: String },
  paymentRecords: [{ amountPaid: Number, datePaid: Date, paymentMethod: String, note: String, paidBy: String }],
  createdAt: { type: Date, default: new Date() } // Bug: Static timestamp evaluated at module load
}

  

  
4. Complete API & Routing Matrix

  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
HTTP Method	Route Endpoint	Controller Function	Auth Middleware	Access Control / Security State
POST	/users/signin	signin (server/controllers/user.js#L18)	None	Public
POST	/users/signup	signup (server/controllers/user.js#L46)	None	Public
POST	/users/forgot	forgotPassword (server/controllers/user.js#L85)	None	Public (sends reset email)
POST	/users/reset	resetPassword (server/controllers/user.js#L137)	None	Public (validates token & expiration)
GET	/invoices/count	getTotalCount (server/controllers/invoices.js#L22)	None	Unauthenticated IDOR (takes ?searchQuery=<userId>)
GET	/invoices/:id	getInvoice (server/controllers/invoices.js#L68)	None	Unauthenticated IDOR (fetches any invoice by ID)
GET	/invoices	getInvoicesByUser (server/controllers/invoices.js#L9)	None	Unauthenticated IDOR (fetches all invoices for any ?searchQuery=<userId>)
POST	/invoices	createInvoice (server/controllers/invoices.js#L53)	None	Unauthenticated (creates invoice with arbitrary creator)
PATCH	/invoices/:id	updateInvoice (server/controllers/invoices.js#L81)	None	Unauthenticated IDOR (modifies any invoice in database)
DELETE	/invoices/:id	deleteInvoice (server/controllers/invoices.js#L93)	None	Unauthenticated IDOR (deletes any invoice in database)
GET	/clients	getClients (server/controllers/clients.js#L37)	None	Global PII Leak (dumps all clients for ALL users)
GET	/clients/user	getClientsByUser (server/controllers/clients.js#L90)	None	Unauthenticated IDOR (takes ?searchQuery=<userId>)
POST	/clients	createClient (server/controllers/clients.js#L53)	None	Unauthenticated (creates client under any user ID)
PATCH	/clients/:id	updateClient (server/controllers/clients.js#L67)	None	Unauthenticated IDOR (modifies any client)
DELETE	/clients/:id	deleteClient (server/controllers/clients.js#L79)	None	Unauthenticated IDOR (deletes any client)
GET	/profiles/:id	getProfile (server/controllers/profile.js#L18)	None	Unauthenticated IDOR (fetches profile by ID)
GET	/profiles	getProfilesByUser (server/controllers/profile.js#L69)	None	Unauthenticated IDOR (fetches profile for any ?searchQuery=<userId>)
POST	/profiles	createProfile (server/controllers/profile.js#L30)	None	Unauthenticated (creates profile)
PATCH	/profiles/:id	updateProfile (server/controllers/profile.js#L101)	None	Unauthenticated IDOR (modifies any profile)
DELETE	/profiles/:id	deleteProfile (server/controllers/profile.js#L113)	None	Unauthenticated IDOR (deletes any profile)
POST	/create-pdf	Anonymous inline handler (server/index.js#L87)	None	Unauthenticated SSRF / LFI / XSS
POST	/send-pdf	Anonymous inline handler (server/index.js#L53)	None	Unauthenticated Email Spoofing & Spam relay
GET	/fetch-pdf	Anonymous inline handler (server/index.js#L97)	None	Unauthenticated File Retrieval (Race Condition Leak)

  

  
5. Reverse-Engineering Security Audit & Vulnerabilities

  
Finding 1: Broken Object-Level Authorization (BOLA / IDOR) Across Entire Data Layer

  

  
Classification: [Confirmed]

  
Location:
  
  

  
server/routes/invoices.js lines 1–14

  
server/routes/clients.js lines 1–12

  
server/routes/profile.js lines 1–14

  

  
Mechanics:
The auth middleware defined in server/middleware/auth.js is never imported or mounted in server/routes/invoices.js, server/routes/clients.js, or server/routes/profile.js.
Although the frontend adds an Authorization: Bearer <token> header (client/src/api/index.js lines 6–12) and conditionally renders buttons based on invoice.creator (client/src/components/InvoiceDetails/InvoiceDetails.js line 204), the server does not enforce any authorization.

  
Impact:
Any unauthenticated user or external script can:
  
  

  
Retrieve any invoice via GET /invoices/:id or query all invoices of any user via GET /invoices?searchQuery=<targetUserId>.

  
Modify or tamper with any invoice data or payment records via PATCH /invoices/:id.

  
Delete any invoice via DELETE /invoices/:id.

  
Fetch, update, or delete any customer profile via /clients/:id and /profiles/:id.

  

  

  

  
Finding 2: Complete Authentication Bypass via Unverified Token Decoding in auth.js

  

  
Classification: [Confirmed]

  
Location: server/middleware/auth.js lines 7–27

  
Mechanics:
  
javascript
const token = req.headers.authorization.split(" ")[1]
const isCustomAuth = token.length < 500 
let decodeData;
if(token && isCustomAuth) {
    decodeData = jwt.verify(token, SECRET)
    req.userId = decodeData?.id
} else {
    decodeData = jwt.decode(token)
    req.userId = decodeData?.sub
}

  
  

  
The server differentiates between a custom JWT and a Google OAuth token strictly by token.length < 500.

  
For tokens where length >= 500, the server calls jwt.decode(token) instead of verifying against Google's public certs or JWKS.

  
jwt.decode() parses JSON without checking HMAC or RSA signatures.

  

  
Impact:
An attacker can create a self-signed or unsigned JWT containing { "sub": "<victim_user_id>" }, pad the payload/header with whitespace or dummy claims so the token length exceeds 500 bytes, and bypass authentication completely to impersonate any user identity (req.userId).

  

  

  
Finding 3: Server-Side Request Forgery (SSRF) & Arbitrary Local File Disclosure via html-pdf

  

  
Classification: [Confirmed]

  
Location:
  
  

  
server/index.js lines 53–58, 87–94

  
server/documents/index.js lines 112–217

  

  
Mechanics:
  
  

  
POST /create-pdf and POST /send-pdf accept unauthenticated JSON payloads and pass req.body directly to pdfTemplate(req.body).

  
pdfTemplate performs unescaped string interpolation of properties including company.logo, notes, company.businessName, name, items[].itemName, etc.

  
pdf.create(...) in html-pdf invokes PhantomJS, a headless WebKit browser.

  
PhantomJS parses HTML and executes embedded JavaScript.

  

  
Impact:
  
  

  
Local File Disclosure (LFI): Injecting <script>var x=new XMLHttpRequest();x.open('GET','file:///etc/passwd',false);x.send();document.write(x.responseText);</script> or <iframe src="file:///etc/passwd"></iframe> into notes renders system files directly into the generated PDF.

  
SSRF: Injecting <img src="http://169.254.169.254/latest/meta-data/"> or internal microservices triggers HTTP requests originating from the backend server.

  

  

  

  
Finding 4: Global File Collision & Race Condition Data Leakage

  

  
Classification: [Confirmed]

  
Location: server/index.js lines 57, 69, 88, 98

  
Mechanics:
  
  

  
Both /send-pdf and /create-pdf write the generated PDF to a single, static hardcoded file path: ${__dirname}/invoice.pdf.

  
/fetch-pdf serves this exact static file: res.sendFile('${__dirname}/invoice.pdf').

  
The PDF generation is asynchronous and shared across all concurrent requests and users.

  

  
Impact:
If User A calls /create-pdf and User B calls /create-pdf simultaneously, one user's PDF overwrites the other. When either user requests GET /fetch-pdf, they receive the competitor's/other client's invoice, leaking financial details, names, addresses, and line-item totals. Furthermore, any unauthenticated visitor can poll GET /fetch-pdf to continuously siphon generated invoices.

  

  

  
Finding 5: Global Tenant Customer Data Harvesting

  

  
Classification: [Confirmed]

  
Location: server/controllers/clients.js lines 37–51

  
Mechanics:
  
javascript
export const getClients = async (req, res) => {
    const { page } = req.query;
    const LIMIT = 8;
    const startIndex = (Number(page) - 1) * LIMIT;
    const total = await ClientModel.countDocuments({});
    const clients = await ClientModel.find().sort({ _id: -1 }).limit(LIMIT).skip(startIndex);
    res.json({ data: clients, currentPage: Number(page), numberOfPages: Math.ceil(total / LIMIT)});
}

  

  
Impact:
GET /clients?page=1 is unauthenticated and does not filter by user. Any attacker can iterate through pagination pages to harvest all customer records (names, emails, phone numbers, and physical addresses) belonging to every freelancer registered on the platform.

  

  

  
Finding 6: Unauthenticated Open Mail Relay & Spoofing

  

  
Classification: [Confirmed]

  
Location: server/index.js lines 53–78

  
Mechanics:
POST /send-pdf accepts arbitrary email, company.email, company.name, and company.businessName. It executes transporter.sendMail(...) without checking user authentication or sender legitimacy.

  
Impact:
The server acts as an open email relay for phishing campaigns, sending emails purporting to be from Accountill <hello@accountill.com> to arbitrary recipients.

  

  

  
Finding 7: Hardcoded Third-Party Storage Credentials (Cloudinary)

  

  
Classification: [Confirmed]

  
Location: client/src/components/Settings/Form/Uploader.js lines 33–44

  
Mechanics:
  
javascript
const url = "https://api.cloudinary.com/v1_1/almpo/image/upload";
formData.append("file", acceptedFile);
formData.append("upload_preset", "invoice");

  

  
Impact:
The Cloudinary cloud name (almpo) and unsigned upload preset (invoice) are hardcoded in the frontend. Any user or external actor can upload arbitrary files directly to this Cloudinary bucket, exhausting storage quotas or hosting malware.

  

  

  
Finding 8: Regular Expression Denial of Service (ReDoS)

  

  
Classification: [Confirmed]

  
Location: server/controllers/profile.js lines 89–92

  
Mechanics:
  
javascript
const name = new RegExp(searchQuery, "i");
const email = new RegExp(searchQuery, "i");
const profiles = await ProfileModel.find({ $or: [ { name }, { email } ] });

  searchQuery is taken from req.query and passed directly into new RegExp(...) without regex character escaping or input validation.

  
Impact:
An attacker passing complex polynomial/exponential regex patterns in ?searchQuery= can block the Node.js single-threaded event loop, creating a Denial of Service for all users.

  

  

  
Finding 9: Express Deadlocks / Hanging Requests on Auth Failure

  

  
Classification: [Confirmed]

  
Location: server/middleware/auth.js lines 29–31

  
Mechanics:
  
javascript
try {
    ...
    next()
} catch (error) {
    console.log(error)
}

  If req.headers.authorization is undefined (throwing a TypeError during .split(" ")), or if jwt.verify fails, execution drops into catch (error). The catch block neither calls res.status(...).json(...) nor calls next(error).

  
Impact:
Any request failing authentication hangs indefinitely until client timeout, consuming server connection handles.

  

  

  
6. Functional & Operational Deficiencies

  

  

  

Static Default Timestamps in Mongoose Schemas [Confirmed]:


  

  
server/models/InvoiceModel.js lines 19–22 and server/models/ClientModel.js lines 10–13:
createdAt: { type: Date, default: new Date() } executes new Date() when the JavaScript file is initially parsed by Node.js. All subsequently created documents share the exact timestamp of server startup rather than the document creation time.

  

  

  

  

Uncaught Rejected Promises Passed to res.send() [Confirmed]:


  

  
server/index.js lines 74–76, 90–92:
if(err) { res.send(Promise.reject()); } passes a rejected Promise directly to Express's res.send(), triggering unhandled promise rejection warnings/crashes in modern Node environments.

  

  

  

  

Google Sign-In Identifier Discrepancy [Confirmed]:


  

  
client/src/components/Login/Login.js line 56:
The client assigns userId: result?.jti when creating a profile upon Google login.

  
server/middleware/auth.js line 24:
The server extracts req.userId = decodeData?.sub.

  
Result: The JWT ID (jti) is a unique, one-time token identifier that changes on every login, whereas sub is the user's persistent Google account subject ID. Consequently, Google users fail to match their saved profiles across sessions.

  

  

  

  

Shadowed Route in Express Router [Confirmed]:


  

  
In server/routes/profile.js lines 6–8:
router.get('/:id', getProfile) is defined before router.get('/', getProfilesByUser).

  
In client/src/api/index.js line 34:
The frontend calls API.get('/profiles/search?searchQuery=...').

  
Result: Express matches the literal string "search" to /:id (id = "search"), passing it to ProfileModel.findById("search"), which throws a CastError (Cast to ObjectId failed for value "search").

  

  

  

  

Docker Build Configuration Failures [Confirmed]:


  

  
client/Dockerfile line 7: COPY client/yarn.lock . fails because no yarn.lock exists in client/ (only package-lock.json is committed).

  
server/Dockerfile line 14: CMD ["yarn", "start-prod"] fails because no start-prod script is defined in server/package.json.

  

  

  

  

  
7. Summary Classification Matrix

  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
Feature / Assessment Area	Status	Evidence Reference
Product Purpose: Invoicing for freelancers	Confirmed	README.md lines 1–4, 26–28
Origin / Upstream: Fork/rename of "Arc Invoice"	Confirmed	README.md lines 8–10, server/controllers/user.js line 121
MERN Architecture: Mongo/Express/React/Node	Confirmed	server/package.json, client/package.json
Authentication Enforcement on Invoices: Absent	Confirmed	server/routes/invoices.js lines 1–14
Authentication Enforcement on Clients: Absent	Confirmed	server/routes/clients.js lines 1–12
Authentication Enforcement on Profiles: Absent	Confirmed	server/routes/profile.js lines 1–14
Global Client PII Disclosure: Unfiltered query	Confirmed	server/controllers/clients.js lines 44–47
JWT Verification Bypass: token.length >= 500	Confirmed	server/middleware/auth.js lines 10, 22–24
SSRF / LFI via html-pdf: Unsanitized HTML into PhantomJS	Confirmed	server/index.js lines 57, 88, server/documents/index.js lines 112–217
Shared PDF File Overwrite Race Condition: Static invoice.pdf	Confirmed	server/index.js lines 57, 69, 88, 98
Hardcoded Cloudinary Preset: Public upload access	Confirmed	client/src/components/Settings/Form/Uploader.js lines 33–43
Production Dockerfiles Functional: Both fail	Confirmed	client/Dockerfile line 7, server/Dockerfile line 14
Node 22 / React Scripts 4 Compatibility: Broken build	Likely	react-scripts@4.0.3 relies on webpack 4 / OpenSSL 1.1 legacy crypto, known to fail on Node 17+