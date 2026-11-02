# LibraryMS — Library Management System

A complete **MERN** Library Management System built as a college mini project. It lets a librarian run the day-to-day operations of a library — catalogue, members, issuing, returns, due dates, automatic fines and reporting — while members can browse books and track their own loans.

The app is backed by **MongoDB Atlas** and every action in the UI flows through a real REST API (React → Axios → Express → Mongoose → MongoDB Atlas). There is no mock data.

---

## Features

### Admin / Librarian
- **Dashboard** — total books, available copies, issued books, members, overdue count, outstanding fines
- **Chart** — books issued over time (last 12 months, Recharts)
- **Recent transactions** and quick actions (Add Book, Add Member, Issue Book, View Transactions)
- **Books CRUD** — add, edit, delete, search (title/author/ISBN), category & availability filters, pagination, book detail page
- **Members CRUD** — add (auto membership ID `LIB-001`), edit, delete, search, member detail page with active loans, history and outstanding fine
- **Issue Book** — select member → book → issue date → due date (default +14 days), with all business rules enforced server-side
- **Returns** — one-click return with automatic late days and fine calculation (₹10/day, configurable)
- **Transactions** — full history with status / member / book / date filters, search, pagination, detail modal, "mark fine as paid"
- **Overdue detection** — dynamic, computed from the current date (never stale)
- **Profile** — account info + change password

### Member
- Personal dashboard (issued books, overdue, outstanding fine, available books)
- Browse and search the catalogue, book details
- Own transaction history and due dates

### Cross-cutting
- JWT authentication, bcrypt password hashing, role-based authorization
- Centralized error handling with friendly messages
- Loading, error and empty states everywhere
- Toast notifications, modals, confirmation dialogs
- Fully responsive (mobile drawer sidebar, scrollable tables, stacked forms)

---

## Tech Stack

| Layer | Technology |
| --- | --- |
| Frontend | React 18, Vite, React Router 6, Axios, Tailwind CSS 4, Lucide React, Recharts |
| Backend | Node.js, Express.js, Mongoose |
| Database | MongoDB Atlas |
| Auth | JWT + bcryptjs |

---

## Architecture

```
React (Vite)  ──Axios──►  Express REST API  ──Mongoose──►  MongoDB Atlas
 /frontend                 /backend/src
  pages, components         routes → controllers → models
  context (Auth, Toast)     middleware (auth, role, error)
  services/api.js           utils (fine calculator)
```

---

## Folder Structure

```
library management/
├── backend/
│   ├── src/
│   │   ├── config/db.js               # MongoDB Atlas connection
│   │   ├── models/                    # User, Book, Member, Transaction
│   │   ├── controllers/               # auth, book, member, transaction, dashboard
│   │   ├── routes/                    # /api/* routers
│   │   ├── middleware/                # auth, role, error handlers
│   │   ├── utils/fineCalculator.js    # fine / overdue rules
│   │   ├── seed.js                    # demo data seeder
│   │   └── server.js
│   ├── .env.example
│   └── package.json
├── frontend/
│   ├── src/
│   │   ├── components/                # layout/, ui/
│   │   ├── pages/                     # Login, Dashboard, Books, Members, Issue, Returns, ...
│   │   ├── context/                   # AuthContext, ToastContext
│   │   ├── services/api.js            # Axios instance + interceptors
│   │   ├── hooks/                     # useDebounce
│   │   ├── utils/                     # formatting, constants
│   │   ├── App.jsx
│   │   └── main.jsx
│   └── package.json
├── .gitignore
└── README.md
```

---

## MongoDB Atlas Setup

1. Create a free cluster at [cloud.mongodb.com](https://cloud.mongodb.com).
2. Under **Database → Connect → Drivers**, copy the connection string, e.g.
   `mongodb+srv://<user>:<pass>@cluster0.xxxxx.mongodb.net/libraryms?retryWrites=true&w=majority`
3. Allow your IP address (Network Access) and create a database user.
4. Put the string in `backend/.env` as `MONGODB_URI` (see below).

---

## Environment Variables

Create `backend/.env` (copy from `backend/.env.example`):

```env
MONGODB_URI=your_mongodb_atlas_connection_string
JWT_SECRET=your_jwt_secret_change_this
JWT_EXPIRES_IN=7d
PORT=5000
CLIENT_URL=http://localhost:5173
FINE_PER_DAY=10
```

`backend/.env` is git-ignored — never commit secrets.

---

## Installation & Running

### 1. Backend

```bash
cd backend
npm install
cp .env.example .env      # then edit .env with your Atlas URI + JWT secret
npm run seed              # (optional) load demo data into Atlas
npm run dev               # starts http://localhost:5000
```

### 2. Frontend

```bash
cd frontend
npm install
npm run dev               # starts http://localhost:5173
```

The Vite dev server proxies `/api` to the backend on port 5000, so no extra CORS setup is needed in development.

> Run the seeder only when you want fresh demo data — it **clears** existing collections first.

---

## Demo Credentials

| Role | Email | Password |
| --- | --- | --- |
| Admin / Librarian | `admin@library.com` | `Admin@123` |
| Member | `aarav@example.com` | `Member@123` |

(6 members, 16 books across Programming / Database / Fiction / Science / History / Self Help, and ~18 transactions including overdue and fined loans are seeded.)

---

## API Overview

### Auth
| Method | Endpoint | Access |
| --- | --- | --- |
| POST | `/api/auth/login` | Public |
| POST | `/api/auth/register` | Admin |
| GET | `/api/auth/me` | Authenticated |
| PUT | `/api/auth/change-password` | Authenticated |

### Books
| Method | Endpoint | Access |
| --- | --- | --- |
| GET | `/api/books` (search, category, availability, pagination) | Authenticated |
| GET | `/api/books/:id` | Authenticated |
| POST | `/api/books` | Admin |
| PUT | `/api/books/:id` | Admin |
| DELETE | `/api/books/:id` | Admin |

### Members
| Method | Endpoint | Access |
| --- | --- | --- |
| GET | `/api/members`, `/api/members/:id` | Admin |
| POST | `/api/members` | Admin |
| PUT | `/api/members/:id` | Admin |
| DELETE | `/api/members/:id` | Admin |

### Transactions
| Method | Endpoint | Access |
| --- | --- | --- |
| GET | `/api/transactions` (status/member/book/date/search filters) | Authenticated |
| GET | `/api/transactions/overdue` | Authenticated |
| GET | `/api/transactions/fines/summary` | Admin |
| POST | `/api/transactions/issue` | Admin |
| PUT | `/api/transactions/:id/return` | Authenticated |
| PUT | `/api/transactions/:id/pay-fine` | Authenticated |

### Dashboard
| Method | Endpoint | Access |
| --- | --- | --- |
| GET | `/api/dashboard/stats` | Authenticated (role-aware) |
| GET | `/api/dashboard/recent-transactions` | Authenticated |
| GET | `/api/dashboard/issue-statistics` | Admin |

All responses use a consistent shape: `{ success, data, message?, pagination? }`.

---

## Business Rules

1. A book cannot be issued when `availableCopies <= 0`
2. Issuing decreases `availableCopies`; returning increases it (never above `totalCopies`)
3. A member cannot hold two active issues of the same book
4. Due date cannot be before issue date
5. Fine = `late days × FINE_PER_DAY` (default ₹10), always calculated automatically
6. A transaction is **overdue** when today > due date and it is not returned — computed dynamically
7. Books with active issues and members with active loans cannot be deleted
8. Admin routes are protected by role-based authorization

---

## Screenshots

> _Add screenshots here after running the app (Login, Dashboard, Books, Issue, Returns, Transactions)._

---

## Future Improvements

- Book reservations / holds queue
- Email reminders for upcoming and overdue due dates
- PDF receipt generation for issue/return
- Book cover upload instead of URL
- Reports (most borrowed books, most delayed members)
- Refresh tokens and session management

---

## Author

Built as a college mini project on the MERN stack.
