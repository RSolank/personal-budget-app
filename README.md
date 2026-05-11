# Personal Budget App

MVP web app with user auth, transactions, tags, and settings.

## Run

### Backend

```bash
cd backend
npm install
npm run migrate
npm run dev
```

Server runs on http://localhost:4000. Database file: `backend/data.db`.

### Frontend

```bash
cd frontend
npm install
npm run dev
```

Frontend runs on http://localhost:5173. API base URL: `http://localhost:4000` (set `VITE_API_URL` to override).

## Features

- **Auth:** Register, login, logout (JWT in HTTP-only cookie, bcrypt with per-user salt)
- **Profile:** GET/PATCH `/api/users/me`
- **Tags:** GET/POST/PATCH `/api/tags` (hierarchy, user-created tags)
- **Transactions:** GET/POST/PATCH `/api/transactions` (manual entry, edit rules by source)
- **Dashboard:** List transactions, add manual, upload statement (placeholder)
- **Settings:** Profile tab, Categories tab (list tags, create user tags)
