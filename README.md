# FlyRank Auth API (Python / FastAPI)

A secure API built for the FlyRank Backend Track — Week 2, Assignment A4.
It uses [Supabase Auth](https://supabase.com/docs/guides/auth) as the
Identity Provider to handle sign up, log in, and log out, issues JSON Web
Tokens (JWTs) on login, and verifies those tokens with a reusable FastAPI
dependency to guard protected routes.

No passwords are hashed or tokens signed in this codebase — Supabase does
that. This server only ever forwards credentials to Supabase and verifies
the tokens it hands back.

## What this project is

- `POST /auth/signup` / `POST /auth/login` / `POST /auth/logout` — account
  lifecycle, backed by Supabase Auth.
- `GET /protected/profile` and `GET /protected/dashboard` — routes locked
  behind a bearer token, guarded by one shared dependency
  (`app/dependencies.py::get_current_user`).
- `GET /public/info` — open, unauthenticated route.
- Interactive Swagger docs at `/docs`, with a bearer-token "Authorize" lock
  on every protected route.

## Setup

1. **Create a Supabase project** at [supabase.com](https://supabase.com) —
   free, no credit card. In **Project Settings → API**, copy your
   **Project URL** and **anon key** (never the `service_role` key).
2. In **Authentication → Sign In / Providers → Email**, turn **off**
   "Confirm email" for local testing, so a fresh signup can log in
   immediately.
3. Copy the example env file and fill in your real values:
   ```bash
   cp .env.example .env
   ```
   ```env
   SUPABASE_URL=your_project_url
   SUPABASE_KEY=your_anon_key
   PORT=8000
   ```
4. Create a virtual environment and install dependencies:
   ```bash
   python3 -m venv .venv
   source .venv/bin/activate   # Windows: .venv\Scripts\activate
   pip install -r requirements.txt
   ```

## Run it

```bash
uvicorn app.main:app --reload --port 8000
```

You should see `Server running on port 8000 and connected to Supabase`
with no errors. Swagger UI is then live at `http://localhost:8000/docs`.

## Try it with curl

```bash
# Sign up
curl -i -X POST http://localhost:8000/auth/signup \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","password":"password123"}'

# Log in (copy the access_token from the response)
curl -i -X POST http://localhost:8000/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","password":"password123"}'

# Call a protected route
curl -i http://localhost:8000/protected/profile \
  -H "Authorization: Bearer <PASTE_ACCESS_TOKEN_HERE>"
```

Or use Swagger: click **Authorize**, paste the access token, then
**Try it out** on `/protected/profile` — no curl needed.

## API reference

| Method | Route                  | Auth required                     | Success | Failure                              |
|--------|-------------------------|------------------------------------|---------|----------------------------------------|
| POST   | `/auth/signup`          | none                                | 201     | 400 missing fields                    |
| POST   | `/auth/login`           | none                                | 200     | 400 missing fields, 401 bad creds     |
| POST   | `/auth/logout`          | `Authorization: Bearer <token>`    | 204     | 401 missing/invalid token             |
| GET    | `/protected/profile`    | `Authorization: Bearer <token>`    | 200     | 401 missing/invalid token             |
| GET    | `/protected/dashboard`  | `Authorization: Bearer <token>`    | 200     | 401 missing/invalid token             |
| GET    | `/public/info`          | none                                | 200     | —                                      |

## Swagger UI screenshot

_Add a screenshot of `/docs` here showing the lock icons on the protected
routes and a successful "Try it out" call on `/protected/profile`._

## AI vs me

_(Stage 7, optional) — after running your own AI-generated version in
`ai-version/`, summarize here: how it handled the `Bearer ` prefix, any
security flaws it introduced, and what your prompt left ambiguous._

## Project structure

```
app/
  main.py            # FastAPI app, mounts the three routers
  config.py           # Loads .env, creates the Supabase client
  dependencies.py      # Reusable get_current_user() auth guard
  routes/
    auth.py            # /auth/signup, /auth/login, /auth/logout
    public.py           # /public/info
    protected.py         # /protected/profile, /protected/dashboard
requirements.txt
.env.example
.gitignore
```
