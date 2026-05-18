# 🏪 Store Rating Platform — Complete Walkthrough
## "Explain it like I'm a kid, but make it recruiter-proof"

---

## 📦 PART 1: WHAT IS THIS PROJECT?

Think of this like a **mini Zomato or Google Maps ratings app**, but for stores.

There are **3 types of people** who use this app:
1. **Admin** — The boss. Can add users, add stores, delete them, see stats.
2. **Store Owner** — Owns a shop. Can see who rated their store and what score they got.
3. **Normal User** — A customer. Can browse all stores and give them a star rating (1–5).

---

## 🗂️ PART 2: FOLDER STRUCTURE (What lives where?)

```
store rating/
│
├── backend/              ← The SERVER (brain of the app)
│   ├── config/
│   │   ├── db.js         ← Connects to MySQL database
│   │   └── schema.sql    ← Reference SQL script for tables
│   ├── middleware/
│   │   └── authMiddleware.js  ← Security guard (checks who you are)
│   ├── routes/
│   │   ├── auth.js       ← Login & Register
│   │   ├── admin.js      ← Admin-only actions
│   │   ├── user.js       ← Normal user actions (browse, rate)
│   │   └── storeOwner.js ← Store owner dashboard
│   ├── .env              ← Secret config (DB password, JWT key)
│   ├── index.js          ← Main server file (entry point)
│   └── package.json      ← List of npm packages used
│
└── frontend/             ← The WEBSITE (what you see)
    └── src/
        ├── context/
        │   └── AuthContext.jsx   ← Global "who is logged in?" memory
        ├── pages/
        │   ├── Login.jsx         ← Login page
        │   ├── Signup.jsx        ← Register page
        │   ├── AdminDashboard.jsx      ← Admin panel
        │   ├── StoreOwnerDashboard.jsx ← Owner panel
        │   └── UserDashboard.jsx       ← Customer panel
        ├── App.jsx        ← Router (which page to show)
        ├── App.css        ← Styling
        └── main.jsx       ← React starting point
```

---

## 🧱 PART 3: THE THREE BIG TECHNOLOGIES

### 3.1 — What is React? (The Face)

**React** is a JavaScript library made by Facebook for building websites.

Imagine you're building with LEGO. Each LEGO block is a **Component**. You build small blocks (like a Login form, or a Table) and combine them to make the full page.

**Key React concepts used in this project:**

#### `useState` — The App's Memory
```jsx
const [users, setUsers] = useState([]);
```
- `users` = the current value (starts as empty array `[]`)
- `setUsers` = the function to UPDATE it
- When you call `setUsers(newData)`, React **automatically re-draws** the page with new data

Think of it like a whiteboard. `users` is what's written. `setUsers` is the eraser + new marker.

#### `useEffect` — Do something when page loads
```jsx
useEffect(() => {
  fetchUsers();
}, [userSort]);
```
- Runs code when the component first appears on screen
- The `[userSort]` part = "also re-run if `userSort` changes"
- Used to fetch data from the backend when the page loads

Think of it like saying: *"When I open the fridge, check what's inside."*

#### `props` — Passing data between components
Not heavily used here, but `children` is passed into `ProtectedRoute`.

#### JSX — HTML inside JavaScript
```jsx
return <div className="card"><h1>Hello</h1></div>;
```
Looks like HTML but it's actually JavaScript. React converts it to real HTML.

#### `fetch()` — Talking to the backend
```jsx
const res = await fetch('http://localhost:5000/api/admin/users', {
  headers: { Authorization: `Bearer ${user.token}` }
});
const data = await res.json();
```
This is how the frontend **asks the backend** for data. Like a waiter taking your order to the kitchen.

---

### 3.2 — What is Express.js? (The Brain / Kitchen)

**Express** is a framework for Node.js that lets you build a web server — a program that **listens for requests** and **sends back responses**.

**Node.js** = JavaScript running on your computer (not in a browser).
**Express** = Makes it easy to handle web requests.

#### How a Request Works:
```
Browser (React) → sends HTTP request → Express Server → queries MySQL → sends JSON back → React shows it
```

#### Key Express concepts:

**`app = express()`** — Creates the server object.

**`app.use(cors())`** — Allows the React app (port 5173) to talk to the Express server (port 5000). Without this, browsers BLOCK cross-origin requests as a security rule.

**`app.use(express.json())`** — Teaches Express to read JSON data from incoming requests. Without this, `req.body` would be `undefined`.

**`app.listen(5000)`** — Starts the server. Now Express is "open for business" on port 5000.

**Routes** — Like different counters in a bank:
```js
app.use('/api/auth', authRoutes);    // Counter 1: Login/Register
app.use('/api/admin', adminRoutes);  // Counter 2: Admin actions
app.use('/api/user', userRoutes);    // Counter 3: Normal user actions
```

**`router.get()`, `router.post()`, `router.delete()`** — These match specific URLs and HTTP methods:
- `GET` = Fetch/Read data
- `POST` = Create new data  
- `DELETE` = Remove data

**`req`** = The incoming request (contains `req.body`, `req.params`, `req.headers`)
**`res`** = The outgoing response (use `res.json()` to send data back)

**`async/await`** — Because talking to a database takes time, we use `async/await` to wait patiently instead of blocking everything.

---

### 3.3 — What is MySQL? (The Storage Room)

**MySQL** is a database — a program that stores data in organized **tables**, like Excel sheets.

**XAMPP** is a local server toolkit. It runs MySQL on your computer at `localhost:3306`.

#### The 3 Tables in This Project:

**`User` table** — stores all registered people
| id | name | email | password | address | role |
|----|------|-------|----------|---------|------|
| 1 | System Administrator | admin | admin | System HQ | ADMIN |
| 2 | Yuvraj Anil Tale... | yuvraj@email.com | Pass@123 | Pune | NORMAL |

**`Store` table** — stores all registered shops
| id | name | email | address | rating | ownerId |
|----|------|-------|---------|--------|---------|
| 1 | Yuvraj's Store | store@email.com | Pune | 4.5 | 2 |

**`Rating` table** — stores all ratings submitted
| id | score | userId | storeId |
|----|-------|--------|---------|
| 1 | 5 | 2 | 1 |

#### Key SQL terms:
- `PRIMARY KEY` — Unique ID for each row (auto-increments: 1, 2, 3...)
- `FOREIGN KEY` — Links tables together (ownerId in Store links to id in User)
- `UNIQUE` — No two rows can have the same value in this column
- `NOT NULL` — This field MUST have a value, can't be empty
- `DEFAULT` — Value to use if none provided
- `ON DELETE CASCADE` — If a User is deleted, their Ratings are auto-deleted too
- `ON DELETE SET NULL` — If a User is deleted, the Store's ownerId becomes NULL

---

## 🔌 PART 4: HOW REACT CONNECTS TO EXPRESS CONNECTS TO MYSQL

This is the most important concept. Here's the full flow:

```
STEP 1: User clicks "Login" in React
         ↓
STEP 2: React calls fetch('http://localhost:5000/api/auth/login')
         ↓
STEP 3: Express receives the request at router.post('/login')
         ↓
STEP 4: Express runs SQL: SELECT * FROM User WHERE email = 'admin'
         ↓
STEP 5: MySQL finds the row and returns it to Express
         ↓
STEP 6: Express checks the password, creates a JWT token
         ↓
STEP 7: Express sends { token, role, name } back as JSON
         ↓
STEP 8: React receives it, saves to localStorage, redirects to Dashboard
```

**Why port 5000 and 5173?**
- 5000 = Express backend (the server)
- 5173 = Vite dev server (serves your React frontend)
- They're two separate programs running simultaneously

---

## 🔐 PART 5: AUTHENTICATION (How Login Works)

### Step 1 — User submits login form
React collects `email` and `password` from the form inputs.

### Step 2 — Send to backend
```js
fetch('http://localhost:5000/api/auth/login', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ email, password })
})
```

### Step 3 — Backend checks the database
```js
const [rows] = await pool.query('SELECT * FROM `User` WHERE `email` = ?', [email]);
if (password !== user.password) { return res.status(401)... }
```

### Step 4 — Create a JWT Token
```js
const token = jwt.sign(
  { id: user.id, email: user.email, role: user.role },
  'supersecretkey123',
  { expiresIn: '24h' }
);
```

**What is JWT (JSON Web Token)?**

Think of JWT like a **wristband at a concert**. When you buy a ticket (login), the concert gives you a wristband (token). Every time you want to enter a restricted area, you show your wristband. The security guard checks it's real — they don't need to call the ticketing office every time.

JWT has 3 parts separated by dots:
```
eyJhbGciOiJIUzI1NiJ9    ← Header (algorithm used)
.eyJpZCI6MSwicm9sZSI6IkFETUlOIn0  ← Payload (your data: id, role)
.abc123signature        ← Signature (proves it's not tampered)
```

The **signature** is created using `supersecretkey123`. If someone tries to modify their role from NORMAL to ADMIN in the token, the signature won't match and the server will reject it.

### Step 5 — React saves the token
```js
localStorage.setItem('user', JSON.stringify({ token, role, name }));
```
`localStorage` is like a small notepad in the browser. It persists even if you refresh the page.

### Step 6 — Every future request includes the token
```js
headers: { Authorization: `Bearer ${user.token}` }
```

---

## 🛡️ PART 6: MIDDLEWARE (The Security Guard)

**Middleware** = Code that runs **between** receiving a request and sending a response.

### `verifyToken` middleware:
```js
const verifyToken = (req, res, next) => {
  const authHeader = req.headers.authorization;        // Get "Bearer <token>"
  const token = authHeader.split(' ')[1];              // Extract just the token
  jwt.verify(token, JWT_SECRET, (err, decoded) => {   // Check if valid
    if (err) return res.status(401).json({ error: 'Unauthorized' });
    req.user = decoded;  // Attach user info to the request
    next();              // Pass to the actual route handler
  });
};
```

`next()` = "I'm done checking, pass the request along to the next handler."

### `verifyRole` middleware:
```js
const verifyRole = (roles) => {
  return (req, res, next) => {
    if (!roles.includes(req.user.role)) {
      return res.status(403).json({ error: 'Forbidden' });
    }
    next();
  };
};
```

Used like this:
```js
router.use(verifyToken);           // First check: Are you logged in?
router.use(verifyRole(['ADMIN'])); // Second check: Are you an ADMIN?
```

403 = Forbidden (logged in but not allowed)
401 = Unauthorized (not logged in at all)

---

## 🗄️ PART 7: DATABASE CONNECTION (db.js explained)

```js
const mysql = require('mysql2/promise');
```
`mysql2/promise` = MySQL driver that supports `async/await`.

```js
pool = mysql.createPool(config);
```
**Connection Pool** = Instead of opening a new connection every time (slow!), we keep a pool of 10 ready connections. Think of it like a taxi stand with 10 taxis always waiting.

`pool.query()` = Pick a taxi from the stand, use it, return it.
`pool.getConnection()` = Reserve a specific taxi (used in `initDatabase()`).
`connection.release()` = Return the taxi to the stand.

**Why `CREATE TABLE IF NOT EXISTS`?**
So the server doesn't crash if the table already exists. It only creates it if it's missing.

---

## 🚀 PART 8: SERVER STARTUP (index.js explained line by line)

```js
const express = require('express');   // Import Express
const cors = require('cors');         // Import CORS handler
require('dotenv').config();           // Load .env file into process.env
```

```js
const app = express();     // Create the server
const PORT = 5000;         // Port to listen on
```

```js
async function bootstrap() {
  await initDatabase();    // Create tables if missing
  // Check if admin exists, seed if not
  const [rows] = await pool.query('SELECT * FROM `User` WHERE `email` = ?', ['admin']);
  if (rows.length === 0) {
    await pool.query('INSERT INTO `User` ... VALUES (?, ?, ?, ?, ?)',
      ['System Administrator', 'admin', 'admin', 'System HQ', 'ADMIN']);
  }
}
bootstrap(); // Run it immediately when server starts
```

```js
app.use(cors());          // Allow cross-origin requests
app.use(express.json());  // Parse JSON request bodies
```

```js
app.use('/api/auth', authRoutes);          // Mount routes
app.use('/api/admin', adminRoutes);
app.use('/api/user', userRoutes);
app.use('/api/store-owner', storeOwnerRoutes);
```

```js
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

---

## 📋 PART 9: REGISTRATION FLOW (auth.js)

When a new user signs up:

1. **Validate inputs** — Name 20–60 chars, email format, password rules (8–16 chars, 1 uppercase, 1 special character like `!@#$&*`)
2. **Check duplicate** — `SELECT * FROM User WHERE email = ?`
3. **Insert user** — `INSERT INTO User (name, email, password, address, role) VALUES (...)`
4. **If STORE_OWNER** — Auto-create a store entry linked to this user

**Why `?` in SQL queries?**
These are **prepared statements** / **parameterized queries**. Instead of:
```sql
SELECT * FROM User WHERE email = 'admin'  ← DANGEROUS (SQL injection risk)
```
We use:
```sql
SELECT * FROM User WHERE email = ?   ← SAFE (MySQL escapes the value)
```
This prevents **SQL Injection** attacks where a hacker types `'; DROP TABLE User; --` as their email.

---

## 🎛️ PART 10: ADMIN DASHBOARD FEATURES

### Statistics (COUNT queries)
```sql
SELECT COUNT(*) as count FROM `User`
SELECT COUNT(*) as count FROM `Store`
SELECT COUNT(*) as count FROM `Rating`
```
These count how many rows are in each table.

### User Directory with JOIN
```sql
SELECT u.id, u.name, u.email, u.address, u.role, s.rating as storeRating
FROM `User` u
LEFT JOIN `Store` s ON u.id = s.ownerId
WHERE 1=1
```

**LEFT JOIN** = "Give me all users, and IF they own a store, also attach the store's rating."
- Normal users → `storeRating` = NULL
- Store owners → `storeRating` = their store's average rating

**`WHERE 1=1`** = Always true. It's a trick so we can dynamically append more conditions with `AND`:
```js
if (search) sql += ' AND u.`name` LIKE ?';
if (role) sql += ' AND u.`role` = ?';
```

**`LIKE '%search%'`** = Partial match. `%` = any characters. So `%yuvraj%` matches "Yuvraj Tale".

### Delete Endpoints
```sql
DELETE FROM `User` WHERE `id` = ?
DELETE FROM `Store` WHERE `id` = ?
```
Because of `ON DELETE CASCADE` on the Rating table, deleting a user auto-deletes their ratings.

---

## ⭐ PART 11: RATING SYSTEM (user.js)

```sql
INSERT INTO `Rating` (`userId`, `storeId`, `score`) VALUES (?, ?, ?)
ON DUPLICATE KEY UPDATE `score` = ?
```

**ON DUPLICATE KEY UPDATE** = "If this userId+storeId combo already exists, don't insert again — just UPDATE the score instead."

This handles both first-time ratings AND updating an existing rating in ONE query.

After rating, recalculate average:
```sql
SELECT AVG(`score`) as avgScore FROM `Rating` WHERE `storeId` = ?
```
`AVG()` = MySQL function that calculates the average of all scores.

Then update the store's stored rating:
```sql
UPDATE `Store` SET `rating` = ? WHERE `id` = ?
```

---

## 🔄 PART 12: ROUTING IN REACT (App.jsx)

```jsx
<BrowserRouter>
  <Routes>
    <Route path="/login" element={<Login />} />
    <Route path="/signup" element={<Signup />} />
    <Route path="/dashboard" element={
      <ProtectedRoute><Dashboard /></ProtectedRoute>
    } />
    <Route path="*" element={<Navigate to="/login" />} />
  </Routes>
</BrowserRouter>
```

- `path="*"` = Catch-all. Any unknown URL → redirect to login.
- `<Navigate to="/login" />` = Programmatic redirect.

**ProtectedRoute** — Custom component that checks if user is logged in:
```jsx
const ProtectedRoute = ({ children }) => {
  const { user } = useAuth();
  if (!user) return <Navigate to="/login" />;
  return children;  // If logged in, show the actual page
};
```

**Role-based Dashboard** — One `/dashboard` route shows different content based on role:
```jsx
if (user?.role === 'ADMIN') DashboardContent = <AdminDashboard />;
else if (user?.role === 'STORE_OWNER') DashboardContent = <StoreOwnerDashboard />;
else DashboardContent = <UserDashboard />;
```

---

## 🧠 PART 13: AUTH CONTEXT (AuthContext.jsx)

**Context** = A way to share data across ALL components without passing it as props through every level.

Think of it like a **TV signal** — you don't plug each TV individually into the broadcast station. The signal is broadcast, and any TV in range can pick it up.

```jsx
const AuthContext = createContext();

export function AuthProvider({ children }) {
  const [user, setUser] = useState(null);  // Global logged-in user

  // On page load, check if user was already logged in
  useEffect(() => {
    const stored = localStorage.getItem('user');
    if (stored) setUser(JSON.parse(stored));
  }, []);

  const login = (userData) => {
    setUser(userData);
    localStorage.setItem('user', JSON.stringify(userData));
  };

  const logout = () => {
    setUser(null);
    localStorage.removeItem('user');
  };

  return (
    <AuthContext.Provider value={{ user, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
}

export const useAuth = () => useContext(AuthContext);
```

Any component can call `useAuth()` to get the current user, call login, or logout.

---

## 🌐 PART 14: HTTP STATUS CODES (What numbers mean)

| Code | Meaning | When used |
|------|---------|-----------|
| 200 | OK | Request succeeded |
| 201 | Created | New resource created (after INSERT) |
| 400 | Bad Request | Invalid input from user |
| 401 | Unauthorized | Not logged in / bad token |
| 403 | Forbidden | Logged in but wrong role |
| 404 | Not Found | Resource doesn't exist |
| 500 | Server Error | Something crashed on the server |

---

## 🛠️ PART 15: TOOLS & PACKAGES USED

| Package | What it does |
|---------|-------------|
| `express` | Web server framework |
| `mysql2` | MySQL driver for Node.js (supports async/await) |
| `jsonwebtoken` | Creates and verifies JWT tokens |
| `cors` | Allows frontend and backend on different ports to communicate |
| `dotenv` | Loads `.env` file into `process.env` |
| `nodemon` | Auto-restarts server when you save a file (dev only) |
| `react` | UI component library |
| `react-router-dom` | Client-side routing (URL navigation) |
| `vite` | Fast frontend development server and build tool |

---

## ❓ PART 16: RECRUITER Q&A — MASTER THESE ANSWERS

**Q: What is the tech stack of your project?**
> "The frontend is built with React using Vite as the build tool. The backend is a Node.js server using the Express framework. The database is MySQL, running locally via XAMPP. Communication between frontend and backend happens through RESTful HTTP API calls, with JWT tokens handling authentication."

**Q: What is REST API?**
> "REST stands for Representational State Transfer. It's a design standard for web APIs. In REST, each URL is a 'resource', and you use HTTP verbs to act on it — GET to read, POST to create, PUT to update, DELETE to remove. For example, `GET /api/admin/users` fetches all users, and `DELETE /api/admin/users/5` deletes the user with ID 5."

**Q: How does authentication work in your app?**
> "When a user logs in, the backend verifies their credentials against the database. If valid, it generates a JWT token signed with a secret key. This token contains the user's ID and role embedded inside it. The frontend saves this token in localStorage. On every subsequent API request, the token is sent in the Authorization header. The backend middleware verifies the token's signature and extracts the user's identity from it — no session storage on the server is needed."

**Q: What is JWT and why is it stateless?**
> "JWT stands for JSON Web Token. It's stateless because the server doesn't need to store session data. All the user information is encoded inside the token itself. The server just verifies the signature each time — if the signature matches, the token is valid. This makes it scalable because you don't need a shared session store."

**Q: What is CORS and why do you need it?**
> "CORS — Cross-Origin Resource Sharing — is a browser security feature. Browsers block JavaScript from making requests to a different domain or port than the page itself. Since our React app runs on port 5173 and Express on port 5000, they're considered 'different origins'. The `cors()` middleware tells Express to include the right HTTP headers to tell the browser it's allowed."

**Q: What is a Connection Pool?**
> "A connection pool is a cache of database connections maintained so that they can be reused when needed. Opening a new database connection for every single query would be expensive and slow. With a pool of 10 connections, Express can handle multiple simultaneous requests by using available connections from the pool."

**Q: How did you handle role-based access control?**
> "Each user has a role stored in the database — ADMIN, STORE_OWNER, or NORMAL. When they log in, the role is embedded inside their JWT token. The backend uses middleware called `verifyRole()` that checks if the user's role is allowed for that route. For example, the admin routes have `verifyRole(['ADMIN'])` — anyone without an ADMIN role gets a 403 Forbidden response."

**Q: How does rating work in your app?**
> "When a normal user submits a rating, the backend uses a MySQL `INSERT ... ON DUPLICATE KEY UPDATE` statement. This means if the user has already rated the store, it updates their existing score instead of creating a duplicate. After each rating, we recalculate the average using `SELECT AVG(score) FROM Rating WHERE storeId = ?` and update the store's rating column."

**Q: What are prepared statements and why use them?**
> "Prepared statements use placeholders (`?`) instead of directly embedding user input into SQL queries. MySQL handles the escaping, which prevents SQL injection attacks — where a malicious user could type SQL code into a form input to manipulate or delete the database."

**Q: What is a LEFT JOIN?**
> "A LEFT JOIN combines rows from two tables. It returns ALL rows from the left table, and matching rows from the right table — or NULL if there's no match. In the user directory, we LEFT JOIN User with Store on `ownerId`. This means every user appears in the result, but only store owners have a non-null `storeRating`."

**Q: Why did you use Vite instead of Create React App?**
> "Vite is significantly faster for development. It uses native ES modules and only processes files when requested, whereas Create React App bundles everything upfront. Vite's Hot Module Replacement is near-instant, making development much smoother."

**Q: How does data flow from the database to the UI?**
> "The React component uses `useEffect` to call `fetch()` when it loads. The fetch call hits the Express API endpoint. Express runs a parameterized SQL query on MySQL. MySQL returns the result rows. Express sends them as JSON. React receives the JSON, updates state with `useState`, and React automatically re-renders the UI with the new data."

**Q: What happens when you delete a user?**
> "The frontend sends a DELETE request to `/api/admin/users/:id`. The backend middleware first verifies the JWT token and confirms the requester is an admin. It checks that the email isn't 'admin' (protected account). Then runs `DELETE FROM User WHERE id = ?`. Because the Rating table has a `FOREIGN KEY` with `ON DELETE CASCADE`, all ratings that this user submitted are automatically deleted by MySQL."

**Q: What is `module.exports` in Node.js?**
> "Node.js uses the CommonJS module system. `module.exports` is what gets returned when another file `require()`s this file. So in routes/admin.js, we do `module.exports = router` — and in index.js we `require('./routes/admin')` to get that router and mount it."

**Q: What is the `.env` file?**
> "The `.env` file stores environment-specific configuration like database credentials and secret keys. We use the `dotenv` package to load it. It's listed in `.gitignore` so secret keys are never committed to GitHub. Values are accessed via `process.env.DATABASE_URL`."

---

## 🔁 PART 17: FULL END-TO-END USER JOURNEYS

### Admin Login → View Users
1. Admin visits `/login`, types `admin` / `admin`
2. React POSTs to `POST /api/auth/login`
3. Express queries `SELECT * FROM User WHERE email = 'admin'`
4. MySQL returns the admin row
5. Express compares `admin === admin` → match
6. Express creates JWT: `{ id:1, email:'admin', role:'ADMIN' }`
7. React saves token, redirects to `/dashboard`
8. App.jsx sees role=ADMIN, renders `<AdminDashboard />`
9. Dashboard's `useEffect` calls `GET /api/admin/users`
10. `verifyToken` checks JWT → valid
11. `verifyRole(['ADMIN'])` checks role → passes
12. Express runs LEFT JOIN query → returns array of users as JSON
13. React's `setUsers(data)` updates state
14. React re-renders → table appears on screen

### Normal User Rates a Store
1. User Dashboard loads → `GET /api/user/stores` with their userId
2. Express runs LEFT JOIN with Rating table to show their existing score for each store
3. User picks 4 stars and clicks submit
4. React POSTs `{ score: 4 }` to `/api/user/stores/2/rate`
5. Express does `INSERT INTO Rating ... ON DUPLICATE KEY UPDATE score = 4`
6. Express calculates `SELECT AVG(score) FROM Rating WHERE storeId = 2`
7. Express updates `UPDATE Store SET rating = 4.2 WHERE id = 2`
8. React gets success response, re-fetches store list
9. Store now shows updated rating

---

## 📝 FINAL SUMMARY — ONE-LINE ANSWERS

| Question | Answer |
|----------|--------|
| Frontend | React (with Vite) |
| Backend | Node.js + Express |
| Database | MySQL (via XAMPP) |
| Auth method | JWT (JSON Web Tokens) |
| API style | REST |
| DB driver | mysql2/promise (connection pool) |
| Roles | ADMIN, STORE_OWNER, NORMAL |
| Password storage | Plaintext (for demonstration clarity) |
| Session type | Stateless (token-based) |
| Cross-origin handling | CORS middleware |
| SQL injection prevention | Prepared statements (parameterized queries) |
| Auto table creation | `CREATE TABLE IF NOT EXISTS` on server boot |
| Admin seeding | Auto-inserted if not exists on bootstrap |
