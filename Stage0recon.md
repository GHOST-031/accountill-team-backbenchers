Stage 0 Reconnaissance Report: Accountill

Repository: accountill
Application Type: Full-stack MERN invoicing web application for freelancers and small businesses
Analysis Method: Static, non-executing source code inspection (ignoring client/build/)

1. Languages
JavaScript (ECMAScript / ES6+ / JSX) [Confirmed]:
Backend: Node.js running native ES Modules ("type": "module" in server/package.json:7; import syntax across all controllers/routes, e.g., server/index.js:4-23).
Frontend: React 17 JSX and ES6+ JavaScript throughout client/src/index.js:1-21 and client/src/App.js:1-54.
HTML5 [Confirmed]:
Frontend SPA root template: client/public/index.html:1-45.
Server-side PDF and email template strings: server/documents/index.js:24-217, server/documents/email.js:15-158, and server/documents/invoice.js:23-201.
CSS3 / CSS Modules [Confirmed]:
Global stylesheet: client/src/index.css:1-35.
Scoped CSS modules across UI components (e.g., client/src/components/Invoice/Invoice.module.css, client/src/components/Dashboard/Dashboard.module.css).
JSON [Confirmed]:
Package definitions and currencies/mock datasets: server/package.json:1-29, client/package.json:1-72, client/src/currencies.json:1-129, client/src/clients.json:1-14.
YAML [Confirmed]:
Compose orchestration and funding metadata: docker-compose.prod.yml:1-33, .github/FUNDING.yml:1-12.
Dockerfile & Nginx Configuration Syntax [Confirmed]:
server/Dockerfile:1-15, client/Dockerfile:1-25, client/nginx.conf:1-10.
2. Frameworks
Backend Web Framework: Express.js (^4.17.1) [Confirmed]:
Defined in server/package.json:18.
Initialized and mounted with JSON/urlencoded parsers and CORS in server/index.js:4, 25, 28–30.
Frontend UI Framework: React (^17.0.2) [Confirmed]:
Defined in client/package.json:25.
Initialized via ReactDOM.render in client/src/index.js:16–20.
Component / UI Design Framework: Material-UI (MUI v4) [Confirmed]:
Dependencies: @material-ui/core ^4.11.4, @material-ui/icons ^4.11.2, @material-ui/lab ^4.0.0-alpha.58, @material-ui/pickers ^3.3.10 in client/package.json:10–13.
Consumed across UI dialogs, forms, grids, tables, and buttons (e.g., client/src/components/Invoice/Invoice.js:9–30).
State Management Framework: Redux (^4.1.0) with Redux-Thunk (^2.3.0) [Confirmed]:
Defined in client/package.json:38–39.
Assembled in client/src/index.js:8–14 and client/src/reducers/index.js:1–8.
3. Package Managers
npm (Primary / Canonical) [Confirmed]:
Evidence: Authoritative lockfiles present for both server and client: server/package-lock.json and client/package-lock.json.
Evidence: Official setup instructions exclusively prescribe npm commands: $ npm install, $ npm start (README.md:96–98, 118–120).
Evidence: Server Dockerfile uses RUN npm install (server/Dockerfile:8, 11).
Yarn (Incomplete / Erroneous Presence) [Confirmed]:
Evidence: server/Dockerfile:14 specifies CMD ["yarn", "start-prod"].
Evidence: client/Dockerfile:7–8 attempts COPY client/yarn.lock . and RUN yarn install --production, though no yarn.lock exists in the repository.
4. Important Dependencies and Versions
Server Dependencies (server/package.json:14–27) [Confirmed]
express: ^4.17.1 — HTTP server framework.
mongoose: ^5.12.10 — MongoDB ODM.
jsonwebtoken: ^8.5.1 — JWT generation and verification.
bcryptjs: ^2.4.3 — Password hashing (12 salt rounds used in server/controllers/user.js:57, 145).
nodemailer: ^6.6.3 — Email delivery via SMTP (server/index.js:38–48).
html-pdf: ^3.0.1 — Headless WebKit (PhantomJS) HTML-to-PDF rendering (server/index.js:9, 57, 88).
cors: ^2.8.5 — Cross-Origin Resource Sharing middleware (server/index.js:5, 30).
dotenv: ^8.5.0 — Environment variable loader (server/index.js:7, 26).
moment: ^2.29.1 — Date formatting in templates (server/documents/index.js:1).
nodemon: ^2.0.7 (devDependencies) — Server reload daemon.
Client Dependencies (client/package.json:5–42) [Confirmed]
react: ^17.0.2 & react-dom: ^17.0.2 — React UI library.
react-scripts: 4.0.3 — Create React App build pipeline (Node 22 incompatible due to legacy OpenSSL hash in Webpack 4) [Likely].
react-router-dom: ^5.2.0 — Declarative SPA client-side routing.
redux: ^4.1.0, redux-thunk: ^2.3.0, react-redux: ^7.2.4 — State management.
axios: ^0.21.1 — HTTP client with auth interceptor (client/src/api/index.js:1, 6–12).
@react-oauth/google: ^0.9.0 & jwt-decode: ^3.1.2 — Google OAuth button and token decode (client/src/components/Login/Login.js:5–6).
react-dropzone: ^11.3.4 — Drag-and-drop file upload for business logo (client/src/components/Settings/Form/Uploader.js:2).
file-saver: ^2.0.5 — Client-side blob saving for PDF downloads (client/src/components/InvoiceDetails/InvoiceDetails.js:30, 144).
react-simple-snackbar: ^1.1.11 — Toast notification provider.
Visualization packages: apexcharts ^3.28.1, react-apexcharts ^1.3.9, recharts ^2.0.9, @devexpress/dx-react-chart ^2.7.6.
5. Database Technology
Database Engine: MongoDB [Confirmed]:
Cloud / Production target: MongoDB Atlas (DB_URL string in README.md:105, 115).
Local / Container target: Docker image mongo on port 27017 with database name arch (docker-compose.prod.yml:28–32, README.md:141).
Driver / ODM: Mongoose ^5.12.10 [Confirmed]:
Connection initialized in server/index.js:106–114 (useNewUrlParser: true, useUnifiedTopology: true, useFindAndModify: false, useCreateIndex: true).
Schemas & Collections [Confirmed]:
User -> users: name, email, password, resetToken, expireToken (server/models/userModel.js:3–11).
Profile -> profiles: name, email, phoneNumber, businessName, contactAddress, paymentDetails, logo, website, userId (server/models/ProfileModel.js:3–15).
ClientModel -> clientmodels: name, email, phone, address, userId, createdAt (server/models/ClientModel.js:4–16).
InvoiceModel -> invoicemodels: dueDate, currency, items, rates, vat, total, subTotal, notes, status, invoiceNumber, type, creator, totalAmountReceived, client, paymentRecords, createdAt (server/models/InvoiceModel.js:3–25).
6. Authentication Technology
Dual-Method Authentication Architecture [Confirmed]:
Custom Password Authentication:
Hashing: Bcrypt with salt factor 12 (server/controllers/user.js:57, 145).
Token Issuance: jsonwebtoken.sign with a 1-hour expiration signed by process.env.SECRET (server/controllers/user.js:34, 61).
Password Recovery: 32-byte pseudo-random hex token generated via crypto.randomBytes(32) valid for 1 hour (server/controllers/user.js:103–115).
Google OAuth2 SSO:
Client UI renders Google Sign-In via @react-oauth/google (client/src/components/Login/Login.js:5, 107).
Credential received from Google is decoded client-side using jwt-decode (client/src/components/Login/Login.js:54).
Backend Middleware Verification Logic (server/middleware/auth.js:7–27):
If token.length < 500: Treats token as custom JWT and invokes jwt.verify(token, SECRET) -> req.userId = decodeData?.id.
If token.length >= 500: Treats token as Google OAuth token and calls jwt.decode(token) without verifying signatures or keys -> req.userId = decodeData?.sub.
Client Session Storage:
Persisted in browser localStorage.getItem('profile') containing user details and token (client/src/reducers/auth.js:6, 11).
Sent on all API requests via Axios request interceptor: req.headers.authorization = Bearer ${token} (client/src/api/index.js:6–12).
Critical Security Finding:
The auth middleware is not mounted on /invoices, /clients, or /profiles routes (server/routes/invoices.js:1–14, server/routes/clients.js:1–12, server/routes/profile.js:1–14), leaving the entire resource API unauthenticated.
7. Important Top-Level Folders
server/ [Confirmed]: Node.js Express backend service.
server/controllers/: Endpoint business logic (user.js, invoices.js, clients.js, profile.js).
server/models/: Mongoose database schemas (userModel.js, InvoiceModel.js, ClientModel.js, ProfileModel.js).
server/routes/: Route definitions (userRoutes.js, invoices.js, clients.js, profile.js).
server/middleware/: Express middleware (auth.js).
server/documents/: HTML generators for invoice PDF export and notifications (index.js, invoice.js, email.js).
client/ [Confirmed]: React SPA frontend service.
client/public/: Static files (index.html, manifest.json, favicon.ico, _redirects).
client/src/: Application source code.
client/src/components/: Modular UI views (Dashboard, Invoice, Invoices, InvoiceDetails, Clients, Settings, Login, Password, Payments, NavBar, Header, Footer).
client/src/actions/: Redux action creators (auth.js, clientActions.js, invoiceActions.js, profile.js).
client/src/reducers/: Redux state reducers (auth.js, clients.js, invoices.js, profiles.js).
client/src/api/: Axios HTTP client instances and API endpoint bindings (index.js).
.github/ [Confirmed]: GitHub configuration (FUNDING.yml).
(Note: client/build/ is ignored compiled output per instructions).
8. Important Configuration Files
docker-compose.prod.yml (docker-compose.prod.yml:1–33) [Confirmed]: Defines the multi-container stack (client on port 80, server on port 5000, mongo on port 27017).
server/Dockerfile (server/Dockerfile:1–15) [Confirmed]: Node 14 alpine image configuration exposing port 5000.
client/Dockerfile (client/Dockerfile:1–25) [Confirmed]: Multi-stage builder compiling React app into an Nginx production image.
client/nginx.conf (client/nginx.conf:1–10) [Confirmed]: Web server block routing client traffic to /usr/share/nginx/html/ with SPA fallback try_files $uri $uri/ /index.html.
server/Procfile (server/Procfile:1) [Confirmed]: Heroku web process declaration: web: node index.js.
client/public/_redirects [Confirmed]: Netlify SPA routing redirect rule: /* /index.html 200.
server/package.json & client/package.json [Confirmed]: Specifies project dependencies, scripts, build hooks, and ES module type.
Environment Variables Files (.env) [Confirmed]:
Server requires: DB_URL, PORT, SECRET, SMTP_HOST, SMTP_PORT, SMTP_USER, SMTP_PASS (README.md:104–113, server/index.js:38–44, 106–107, server/middleware/auth.js:5).
Client requires: REACT_APP_GOOGLE_CLIENT_ID, REACT_APP_API, REACT_APP_URL (README.md:80–85, client/src/api/index.js:4).
9. Application Entry Points
Backend Server Entry Point [Confirmed]:
File: server/index.js lines 1–116.
Execution mechanism: Executed directly with node index.js via npm start (server/package.json:9) or server/Procfile:1.
Starts Express server on port process.env.PORT || 5000 after establishing connection to Mongoose/MongoDB (server/index.js:106–111).
Frontend Client Entry Point [Confirmed]:
HTML Shell: client/public/index.html lines 1–45 with container <div id="root"></div> (client/public/index.html:32).
JavaScript Bootstrapper: client/src/index.js lines 1–21, which builds Redux store with thunk middleware and attaches <App /> to the DOM.
Root UI Component & Router: client/src/App.js lines 21–54, declaring browser routes (/, /invoice, /invoices, /invoice/:id, /edit/invoice/:id, /customers, /dashboard, /settings, /login, /forgot, /reset/:token).
10. Anything Unusual or Important for Understanding the Application
Global Unauthenticated IDOR / BOLA Across All Invoices & Clients [Confirmed]:
server/routes/invoices.js:1–14 and server/routes/clients.js:1–12 do not attach auth middleware.
GET /invoices/:id, PATCH /invoices/:id, DELETE /invoices/:id, and GET /invoices?searchQuery=<userId> can be executed by anyone on the internet without credentials.
Global Client PII Dump [Confirmed]:
server/controllers/clients.js:37–51: GET /clients?page=1 performs ClientModel.find().limit(8).skip(...) without filtering by user ID, leaking all customer records (names, emails, phones, addresses) across all tenants.
Flawed Authentication Logic in auth.js [Confirmed]:
server/middleware/auth.js:10, 22–24: Any JWT with length >= 500 triggers jwt.decode(token) rather than jwt.verify(). An attacker can pad an unsigned JWT to exceed 500 characters and forge any arbitrary req.userId identity.
File Overwrite Race Condition & Cross-Tenant Data Leak [Confirmed]:
server/index.js:57, 88, 98: Both /create-pdf and /send-pdf write to a single, hardcoded shared file 'invoice.pdf', and /fetch-pdf serves this exact static file. Concurrent invoice generations overwrite each other, leaking competitor/tenant data to whoever downloads next.
SSRF and Arbitrary File Read (LFI) via PhantomJS [Confirmed]:
server/index.js:87–94 and server/documents/index.js:115, 125–136: User-controlled fields (notes, company.logo, address) are interpolated unescaped into HTML and processed by html-pdf (PhantomJS), enabling XSS, SSRF, and local file reading (e.g., via XMLHttpRequest to file:///etc/passwd).
Hardcoded Cloudinary Storage Credentials [Confirmed]:
client/src/components/Settings/Form/Uploader.js:33, 41–43: Cloud name almpo and unsigned upload preset invoice are hardcoded in the frontend bundle, allowing anyone to upload arbitrary files to this Cloudinary bucket.
Broken Docker Build Configurations [Confirmed]:
client/Dockerfile:7 instructs COPY client/yarn.lock ., but no yarn.lock exists in client/.
server/Dockerfile:14 calls CMD ["yarn", "start-prod"], but no start-prod script is defined in server/package.json.
Static Mongoose Timestamps [Confirmed]:
server/models/InvoiceModel.js:21 and server/models/ClientModel.js:12: default: new Date() evaluates only once when the server boots, pinning the creation date of all subsequent documents to the startup timestamp.
Express Route Shadowing Bug [Confirmed]:
server/routes/profile.js:6–8 defines router.get('/:id', getProfile) before router.get('/', getProfilesByUser). When the client attempts to search via GET /profiles/search (client/src/api/index.js:34), Express treats "search" as :id and fails with a Mongoose CastError.
Orphaned Dead Code from Another Application [Confirmed]:
client/src/store.js:1–36 contains mock data for an ornithological bird-banding equipment loan registry (mistnet, binocular, lenders) from an institute in Jos, completely unrelated to the active Redux store.