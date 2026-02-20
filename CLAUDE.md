# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Personal portfolio website for Niccolo Piccioli (Software Developer & Music Producer). All user-facing text is in Italian. The backend handles a contact form that saves messages to SQLite, with rate limiting and input validation.

## Stack

- **Frontend**: React 18 + TypeScript (Vite), react-icons, plain CSS with CSS variables
- **Backend**: Django 4.2 + Django REST Framework, django-cors-headers, python-dotenv, SQLite
- **Build tool**: Vite 6.x (migrated from Create React App)

## Environment Variables

### Backend (`backend/.env`)
All sensitive configuration is loaded from environment variables via python-dotenv. See `backend/.env.example` for the full list:
- `DJANGO_SECRET_KEY` — required, Django secret key
- `DJANGO_DEBUG` — `True`/`False`, defaults to `False`
- `DJANGO_ALLOWED_HOSTS` — comma-separated hostnames
- `CORS_ALLOWED_ORIGINS` — comma-separated allowed origins (e.g. `http://localhost:5173`)

### Frontend (`frontend/.env.development`)
- `VITE_API_URL` — backend API base URL (e.g. `http://localhost:8000`)

Access in code via `import.meta.env.VITE_API_URL`.

## Commands

### Backend
```bash
cd backend
python3 -m venv venv
source venv/bin/activate            # activate virtualenv
pip install -r requirements.txt
cp .env.example .env                # then edit .env with real values
python manage.py migrate
python manage.py runserver           # runs on localhost:8000
```

### Frontend
```bash
cd frontend
npm install
npm run dev                          # Vite dev server (localhost:3000)
npm run build                        # production build to frontend/build/
npm run preview                      # preview production build
```

## Architecture

### Backend
- `myproject/` — Django project config (settings.py, root urls.py, wsgi.py)
- `contact/` — single Django app: one model (`ContactMessage`), one endpoint, rate limiting

**API**: `POST /api/contact/` — validates name/email/message (length + format), rate limits per IP (5 req/hour), saves to SQLite. Returns JSON with Italian messages.

**Security features**:
- SECRET_KEY, DEBUG, ALLOWED_HOSTS all from env vars
- CORS configured via django-cors-headers (allowed origins from env)
- HTTP security headers: HSTS, X-Frame-Options DENY, XSS filter, content-type nosniff, referrer policy
- Rate limiting on contact endpoint (5 requests per IP per hour)
- Input validation: max lengths (name 100, email 254, message 5000), email regex

### Frontend
Single-page app in `src/App.tsx` with these sections (each a component):
- **Navbar** — sticky, with Intersection Observer for active section highlighting
- **Hero** — intro with CTAs
- **About** ("Chi Sono") — bio
- **MusicProducer** ("Il Lato Creativo") — music production card
- **Skills** ("Competenze Tecniche") — 4-card grid with react-icons
- **Experience** ("Esperienza e Formazione") — timeline
- **Projects** ("Progetti") — 3-card grid (portfolio, Django app, music)
- **Contact** ("Contattami") — form posting to `VITE_API_URL/api/contact/`
- **Footer** — email, phone, GitHub, LinkedIn links

**CSS animations**: fade-in on scroll via Intersection Observer + CSS transitions (no external libs).

Styles are in `src/index.css` using CSS variables for the dark theme (`--bg-dark`, `--accent-blue`, `--accent-purple`, etc.).

### Build & Config Files
- `frontend/vite.config.ts` — Vite config (React plugin, port 3000, output to `build/`)
- `frontend/tsconfig.json` — TypeScript config (ESNext target/module, bundler resolution, strict)
- `frontend/index.html` — Vite entry point (root level, with SEO meta tags and Open Graph)
