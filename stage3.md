Stage 3: Server Routes Technical Analysis

Scope: Backend API routes mounted in the Node.js / Express application (server/).
Methodology: Static source code analysis of routing declarations, controller handlers, middleware chains, and Mongoose operations. All claims are classified as Confirmed, Likely, or Guess and backed by line-numbered file references.

1. Top-Level Route Mounting Map

In server/index.js:32–35, the Express application registers four main route routers alongside four inline endpoints:

javascript
32: app.use('/invoices', invoiceRoutes)
33: app.use('/clients', clientRoutes)
34: app.use('/users', userRoutes)
35: app.use('/profiles', profile)

Note on Authentication & Middleware:
The authentication middleware server/middleware/auth.js:7–32 is not mounted on /invoices, /clients, or /profiles. As verified below, every route under these subtrees executes without authentication checks [Confirmed].

2. Route Specifications
A. User & Authentication Routes (/users)
1. POST /users/signin
Route Declaration: server/routes/userRoutes.js:6 [Confirmed]
Handler File & Lines: server/controllers/user.js:18–42 (signin) [Confirmed]
Input / Parameters:
Body (req.body): { email, password } (server/controllers/user.js:19)
Response:
200 OK: { result: existingUser, userProfile, token } (server/controllers/user.js:37)
400 Bad Request: { message: "Invalid credentials" } (server/controllers/user.js:31)
404 Not Found: { message: "User doesn't exist" } (server/controllers/user.js:27)
500 Internal Server Error: { message: "Something went wrong" } (server/controllers/user.js:40)
Authentication Requirement: None (Public) [Confirmed]
Authorization Requirement: None [Confirmed]
Important Database Operations:
User.findOne({ email }) (server/controllers/user.js:22)
ProfileModel.findOne({ userId: existingUser?._id }) (server/controllers/user.js:25)
2. POST /users/signup
Route Declaration: server/routes/userRoutes.js:7 [Confirmed]
Handler File & Lines: server/controllers/user.js:46–68 (signup) [Confirmed]
Input / Parameters:
Body (req.body): { email, password, confirmPassword, firstName, lastName, bio } (server/controllers/user.js:47)
Response:
200 OK: { result, userProfile, token } (server/controllers/user.js:63)
400 Bad Request: { message: "User already exist" } (server/controllers/user.js:53) or { message: "Password don't match" } (server/controllers/user.js:55)
500 Internal Server Error: { message: "Something went wrong" } (server/controllers/user.js:66)
Authentication Requirement: None (Public) [Confirmed]
Authorization Requirement: None [Confirmed]
Important Database Operations:
User.findOne({ email }) (server/controllers/user.js:50)
ProfileModel.findOne({ userId: existingUser?._id }) (server/controllers/user.js:51)
User.create({ email, password: hashedPassword, name:
𝑓
𝑖
𝑟
𝑠
𝑡
𝑁
𝑎
𝑚
𝑒
firstName{lastName}, bio }) (server/controllers/user.js:59)
3. POST /users/forgot
Route Declaration: server/routes/userRoutes.js:8 [Confirmed]
Handler File & Lines: server/controllers/user.js:85–133 (forgotPassword) [Confirmed]
Input / Parameters:
Body (req.body): { email } (server/controllers/user.js:87)
Response:
200 OK: { message: "check your email" } (server/controllers/user.js:128)
422 Unprocessable Entity: { error: "User does not exist in our database" } (server/controllers/user.js:111)
Authentication Requirement: None (Public) [Confirmed]
Authorization Requirement: None [Confirmed]
Important Database Operations:
User.findOne({ email: email }) (server/controllers/user.js:108)
user.save() persisting resetToken and expireToken (server/controllers/user.js:113–115)
4. POST /users/reset
Route Declaration: server/routes/userRoutes.js:9 [Confirmed]
Handler File & Lines: server/controllers/user.js:137–156 (resetPassword) [Confirmed]
Input / Parameters:
Body (req.body): { password, token } (server/controllers/user.js:138–139)
Response:
200 OK: { message: "password updated success" } (server/controllers/user.js:150)
422 Unprocessable Entity: { error: "Try again session expired" } (server/controllers/user.js:143)
Authentication Requirement: None (Validation via reset token and expiration lookup) [Confirmed]
Authorization Requirement: None [Confirmed]
Important Database Operations:
User.findOne({ resetToken: sentToken, expireToken: { $gt: Date.now() } }) (server/controllers/user.js:140)
user.save() setting new hashed password and clearing resetToken / expireToken (server/controllers/user.js:146–149)
B. Invoice Routes (/invoices)
5. GET /invoices/count
Route Declaration: server/routes/invoices.js:6 [Confirmed]
Handler File & Lines: server/controllers/invoices.js:22–33 (getTotalCount) [Confirmed]
Input / Parameters:
Query string: ?searchQuery=<userId> (req.query.searchQuery) (server/controllers/invoices.js:23)
Response:
200 OK: Number count (e.g. 12) (server/controllers/invoices.js:29)
404 Not Found: { message: error.message } (server/controllers/invoices.js:31)
Authentication Requirement: None [Confirmed]
Authorization Requirement: None [Confirmed]
Important Database Operations:
InvoiceModel.countDocuments({ creator: searchQuery }) (server/controllers/invoices.js:27)
6. GET /invoices/:id
Route Declaration: server/routes/invoices.js:7 [Confirmed]
Handler File & Lines: server/controllers/invoices.js:68–78 (getInvoice) [Confirmed]
Input / Parameters:
Route parameter: :id (req.params.id) (server/controllers/invoices.js:69)
Response:
200 OK: Invoice document object (server/controllers/invoices.js:74)
409 Conflict: { message: error.message } (server/controllers/invoices.js:76)
Authentication Requirement: None [Confirmed]
Authorization Requirement: None [Confirmed]
Important Database Operations:
InvoiceModel.findById(id) (server/controllers/invoices.js:72)
7. GET /invoices
Route Declaration: server/routes/invoices.js:8 [Confirmed]
Handler File & Lines: server/controllers/invoices.js:9–19 (getInvoicesByUser) [Confirmed]
Input / Parameters:
Query string: ?searchQuery=<userId> (req.query.searchQuery) (server/controllers/invoices.js:10)
Response:
200 OK: { data: invoices } (server/controllers/invoices.js:15)
404 Not Found: { message: error.message } (server/controllers/invoices.js:17)
Authentication Requirement: None [Confirmed]
Authorization Requirement: None [Confirmed]
Important Database Operations:
InvoiceModel.find({ creator: searchQuery }) (server/controllers/invoices.js:13)
8. POST /invoices
Route Declaration: server/routes/invoices.js:9 [Confirmed]
Handler File & Lines: server/controllers/invoices.js:53–66 (createInvoice) [Confirmed]
Input / Parameters:
Body (req.body): Full invoice document (dueDate, currency, items, rates, vat, total, subTotal, notes, status, invoiceNumber, type, creator, client, etc.) (server/controllers/invoices.js:55)
Response:
201 Created: Newly created invoice document (server/controllers/invoices.js:61)
409 Conflict: error.message string (server/controllers/invoices.js:63)
Authentication Requirement: None [Confirmed]
Authorization Requirement: None [Confirmed]
Important Database Operations:
new InvoiceModel(invoice).save() (server/controllers/invoices.js:57, 60)
9. PATCH /invoices/:id
Route Declaration: server/routes/invoices.js:10 [Confirmed]
Handler File & Lines: server/controllers/invoices.js:81–91 (updateInvoice) [Confirmed]
Input / Parameters:
Route parameter: :id (req.params.id) (server/controllers/invoices.js:82)
Body (req.body): Fields to update (server/controllers/invoices.js:83)
Response:
200 OK: Updated invoice document object (server/controllers/invoices.js:89)
404 Not Found: 'No invoice with that id' string (server/controllers/invoices.js:85)
Authentication Requirement: None [Confirmed]
Authorization Requirement: None [Confirmed]
Important Database Operations:
mongoose.Types.ObjectId.isValid(_id) validation check (server/controllers/invoices.js:85)
InvoiceModel.findByIdAndUpdate(_id, { ...invoice, _id }, { new: true }) (server/controllers/invoices.js:87)
10. DELETE /invoices/:id
Route Declaration: server/routes/invoices.js:11 [Confirmed]
Handler File & Lines: server/controllers/invoices.js:93–101 (deleteInvoice) [Confirmed]
Input / Parameters:
Route parameter: :id (req.params.id) (server/controllers/invoices.js:94)
Response:
200 OK: { message: 'Invoice deleted successfully' } (server/controllers/invoices.js:100)
404 Not Found: 'No invoice with that id' string (server/controllers/invoices.js:96)
Authentication Requirement: None [Confirmed]
Authorization Requirement: None [Confirmed]
Important Database Operations:
mongoose.Types.ObjectId.isValid(id) validation check (server/controllers/invoices.js:96)
InvoiceModel.findByIdAndRemove(id) (server/controllers/invoices.js:98)
C. Client / Customer Routes (/clients)
11. GET /clients
Route Declaration: server/routes/clients.js:6 [Confirmed]
Handler File & Lines: server/controllers/clients.js:37–51 (getClients) [Confirmed]
Input / Parameters:
Query string: ?page=<number> (req.query.page) (server/controllers/clients.js:38)
Response:
200 OK: { data: clients, currentPage: Number(page), numberOfPages: Math.ceil(total / LIMIT) } (server/controllers/clients.js:47)
404 Not Found: { message: error.message } (server/controllers/clients.js:49)
Authentication Requirement: None [Confirmed]
Authorization Requirement: None [Confirmed]
Important Database Operations:
ClientModel.countDocuments({}) (Counts across all users) (server/controllers/clients.js:44)
ClientModel.find().sort({ _id: -1 }).limit(8).skip(startIndex) (Fetches paginated clients globally across all users) (server/controllers/clients.js:45)
12. GET /clients/user
Route Declaration: server/routes/clients.js:7 [Confirmed]
Handler File & Lines: server/controllers/clients.js:90–100 (getClientsByUser) [Confirmed]
Input / Parameters:
Query string: ?searchQuery=<userId> (req.query.searchQuery) ([`server/controllers/clients.This request was blocked by Gemini's filters. They can occasionally trigger by mistake on safe coding, security, or biology-related queries. Please try rephrasing your prompt. You can send feedback
 or read more about our policies here
.