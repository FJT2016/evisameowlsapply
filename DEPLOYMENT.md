# Deploying the Meowls e-Visa Portal

This app cannot run on GitHub Pages — it has a Python backend and a MongoDB
database. The cheapest working setup uses three free services:

| Piece              | Service           | What you'll do                              |
| ------------------ | ----------------- | ------------------------------------------- |
| Database           | MongoDB Atlas     | Create a free M0 cluster, copy the URI      |
| Backend (FastAPI)  | Render            | One-click deploy via `render.yaml`          |
| Frontend (React)   | Vercel            | Import repo, set one env var                |

Total cost: **$0**. Setup time once you have API keys: ~30 minutes.

---

## Before you start: get your API keys

1. **OpenAI** — https://platform.openai.com/api-keys → create a key starting with `sk-...`
2. **Resend** — https://resend.com → sign up, create an API key starting with `re_...`
3. (Optional) **VAPID keys** for push notifications:
   ```bash
   npx web-push generate-vapid-keys
   ```
   Save the public + private keys.

---

## Step 1 — MongoDB Atlas (database)

1. Go to https://www.mongodb.com/cloud/atlas/register and sign up.
2. **Build a Cluster** → choose **M0 Free**.
3. Pick any provider/region close to you. Click **Create**.
4. **Database Access** → Add a user with a username + password (save them).
5. **Network Access** → Add IP `0.0.0.0/0` (allow from anywhere).
6. **Database** → **Connect** → **Drivers** → copy the URI. It looks like:
   ```
   mongodb+srv://<user>:<password>@cluster0.xxxx.mongodb.net/?retryWrites=true&w=majority
   ```
   Replace `<user>` and `<password>`. **Save this — it's your `MONGO_URL`.**

---

## Step 2 — Render (backend)

1. Go to https://render.com and sign up with your GitHub account.
2. Authorize Render to read your `evisameowlsapply` repository.
3. Click **New +** → **Blueprint**.
4. Select your `evisameowlsapply` repo. Render will detect `render.yaml` and
   show one service: `meowls-evisa-backend`.
5. Click **Apply**. Render will ask you to fill in the secret env vars marked
   `sync: false`:

   | Variable           | Value                                                           |
   | ------------------ | --------------------------------------------------------------- |
   | `MONGO_URL`        | The URI from MongoDB Atlas (Step 1)                             |
   | `OPENAI_API_KEY`   | Your `sk-...` key                                               |
   | `RESEND_API_KEY`   | Your `re_...` key                                               |
   | `CORS_ORIGINS`     | Leave blank for now — fill in after Step 3                      |
   | `VAPID_PUBLIC_KEY` | From `npx web-push generate-vapid-keys` (or leave blank)        |
   | `VAPID_PRIVATE_KEY`| From `npx web-push generate-vapid-keys` (or leave blank)        |

6. Click **Create**. The first build takes 5–10 minutes.
7. When it's live, copy the URL Render gives you. It looks like:
   ```
   https://meowls-evisa-backend.onrender.com
   ```
   **Save this — it's your `BACKEND_URL`.**

> **Note on Render free tier:** the service sleeps after 15 minutes of
> inactivity and takes ~30 seconds to wake up on the next request. That's
> fine for testing; upgrade to a paid plan ($7/mo) if you want it always-on.

---

## Step 3 — Vercel (frontend)

1. Go to https://vercel.com and sign up with GitHub.
2. Click **Add New** → **Project** → import `evisameowlsapply`.
3. **Configure Project**:
   - **Root Directory:** click **Edit** and set it to `frontend`
   - **Framework Preset:** Create React App (auto-detected)
   - **Build Command / Output / Install:** leave defaults (Vercel reads
     `frontend/vercel.json`)
4. Expand **Environment Variables** and add:

   | Name                      | Value                                                         |
   | ------------------------- | ------------------------------------------------------------- |
   | `REACT_APP_BACKEND_URL`   | The Render URL from Step 2 (e.g. `https://meowls-evisa-backend.onrender.com`) |

5. Click **Deploy**. First build takes ~3 minutes.
6. When it's live, copy the Vercel URL. It looks like:
   ```
   https://evisameowlsapply.vercel.app
   ```

---

## Step 4 — Wire it back up (CORS)

The backend needs to know it's allowed to accept requests from the Vercel URL.

1. Go back to Render → your `meowls-evisa-backend` service → **Environment**.
2. Set `CORS_ORIGINS` to your Vercel URL (no trailing slash):
   ```
   https://evisameowlsapply.vercel.app
   ```
   For multiple origins (e.g. preview deployments), comma-separate:
   ```
   https://evisameowlsapply.vercel.app,https://your-preview.vercel.app
   ```
3. Save. Render will redeploy automatically.

---

## Step 5 — Smoke test

Open your Vercel URL. You should see the Home page. Then check:

- [ ] **Register** a test user → should land on the dashboard.
- [ ] **Apply** for a visa, upload a photo → submission saves.
- [ ] Log in as one of the seeded admins from `ADMIN_CREDENTIALS.md`.
- [ ] Approve the application → check the applicant's email inbox for the
      AI-generated visa PDF (and admin emails for copies).

If the dashboard loads but API calls fail with CORS errors, double-check
`CORS_ORIGINS` on Render matches your Vercel URL exactly.

If API calls just hang for ~30 seconds the first time, that's the Render
free-tier cold start — it'll be fast on subsequent requests.

---

## Updating

Push to your `main` branch on GitHub. Both Render and Vercel auto-deploy.

---

## Alternative: Railway instead of Render

If you prefer Railway for the backend:

1. https://railway.app → New Project → Deploy from GitHub repo.
2. Set the **Root Directory** to `backend`.
3. Add a **Start Command:** `uvicorn server:app --host 0.0.0.0 --port $PORT`.
4. Add the same env vars listed in Step 2.
5. Use the Railway-generated URL as `REACT_APP_BACKEND_URL` on Vercel.

Railway has a $5/mo free credit; no cold starts.
