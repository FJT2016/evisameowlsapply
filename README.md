# Meowls e-Visa Portal

A full-stack e-visa application portal for the fictional country of **Meowls**, modeled
after real-world online visa systems (e.g. the Indian e-Visa flow).

Applicants register, submit a visa application with a photo, and track its status.
Government admins review applications from a dedicated dashboard, and the backend
auto-generates an AI-written visa document (with the applicant's photo embedded) and
emails it to the applicant and the full admin group on approval. Rejections also
trigger a notification email.

> **Heads up:** the site you are looking at right now is just this repository's
> GitHub page — it renders this README. The actual web app is a React + FastAPI +
> MongoDB stack and has to be run by a server. See [Running locally](#running-locally)
> below. GitHub itself does not run servers, so you cannot just "open the site" from
> the repo URL.

---

## Tech stack

| Layer        | Tech                                                                |
| ------------ | ------------------------------------------------------------------- |
| Frontend     | React 19, React Router 7, Tailwind CSS, Radix UI, CRA + CRACO       |
| Backend      | FastAPI, Motor (async MongoDB), Passlib/bcrypt, JWT auth            |
| Database     | MongoDB                                                             |
| AI / Email   | OpenAI (visa document generation), Resend (transactional email)     |
| PWA          | Service worker, Web Push (VAPID), installable on iOS/Android        |

## Repository layout

```
.
├── frontend/          React app (CRA + CRACO + Tailwind)
│   ├── public/        index.html, manifest.json, service worker
│   └── src/
│       ├── pages/     Home, Login, Register, Dashboard, ApplyVisa,
│       │              ApplicationDetails, TrackApplication,
│       │              AdminDashboard, AdminReview
│       ├── components/
│       ├── hooks/
│       └── lib/
├── backend/           FastAPI service
│   ├── server.py      App entrypoint (wires routers, retry worker)
│   ├── config.py      Env vars (MONGO_URL, OPENAI_API_KEY, RESEND_API_KEY, ...)
│   ├── database.py
│   ├── models.py
│   ├── routes/        auth, applications, admin, notifications, push
│   └── services/      auth, email, pdf, push
├── tests/             End-to-end / integration tests
├── backend_test.py
└── design_guidelines.json
```

## Features

- **User flow:** register → log in → submit visa application with photo upload →
  view status on dashboard → track via reference.
- **Admin flow:** 8 seeded admin accounts (see `ADMIN_CREDENTIALS.md`) review
  applications, approve or reject from the admin dashboard.
- **AI-generated visa document:** on approval the backend calls OpenAI to draft
  the visa text and embeds the applicant's uploaded photo into the PDF.
- **Email notifications via Resend:** approval and rejection emails are sent to
  the applicant and **all** admin emails (placed in the `to` array, not CC, so
  every admin receives delivery confirmation).
- **PWA support:** installable on mobile, push notifications via VAPID.

## Running locally

You need **Node 18+**, **Python 3.11+**, **MongoDB**, **Yarn**, and API keys for
OpenAI and Resend.

### 1. Backend

```bash
cd backend
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt

cat > .env <<'EOF'
MONGO_URL=mongodb://localhost:27017
DB_NAME=meowls_evisa
CORS_ORIGINS=http://localhost:3000
RESEND_API_KEY=your_resend_key
SENDER_EMAIL=onboarding@resend.dev
OPENAI_API_KEY=your_openai_key
VAPID_PUBLIC_KEY=...
VAPID_PRIVATE_KEY=...
EOF

uvicorn server:app --reload --port 8000
```

The API will be available at `http://localhost:8000/api/...`.

### 2. Frontend

```bash
cd frontend
yarn install
yarn start
```

The dev server runs at `http://localhost:3000`.

### 3. Admin accounts

See [`ADMIN_CREDENTIALS.md`](./ADMIN_CREDENTIALS.md) for the seeded admin emails
and passwords used to access `/admin`.

## Deploying

This app needs a real server runtime — it will not work as a static GitHub Pages
site. Typical options:

- **Frontend:** Vercel, Netlify, or Cloudflare Pages (build command `yarn build`,
  output `frontend/build`).
- **Backend:** Render, Railway, Fly.io, or any container host that can run a
  Python ASGI app.
- **Database:** MongoDB Atlas (free tier works).

Set the environment variables listed above on whatever host you choose, and point
the frontend at the backend URL via `CORS_ORIGINS` on the backend.

## Documentation

- `ADMIN_CREDENTIALS.md` — seeded admin accounts.
- `EMAIL_NOTIFICATION_FEATURES.md` — email behavior and admin distribution list.
- `auth_testing.md` — auth flow test notes.
- `test_email_delivery.md` — Resend delivery verification.
- `design_guidelines.json` — UI design tokens.

## License

Private project. All rights reserved.
