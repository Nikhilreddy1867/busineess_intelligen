# Business Intelligence — Query to Canvas

Upload a CSV, ask questions in plain English, and get SQL-generated charts and tables. Runs **fully locally** using a small LLM (Ollama) and Supabase — no cloud API keys required.

---

## Prerequisites

- **Node.js 18+** and **npm** — [install with nvm](https://github.com/nvm-sh/nvm#installing-and-updating)
- **Docker Desktop** — [install for Mac](https://docs.docker.com/desktop/install/mac-install/) (required for local Supabase)
- **Supabase CLI** — `brew install supabase/tap/supabase`
- **Ollama** — [ollama.com](https://ollama.com) (for the local LLM)

---

## One-time setup

### 1. Clone and install dependencies

```bash
git clone https://github.com/Pramodsai29/busineess_intelligen.git
cd busineess_intelligen
npm install
```

### 2. Install and run Ollama (local model)

```bash
# Install from https://ollama.com if needed, then:
ollama pull llama3.2:3b
ollama serve   # If you see "address already in use", Ollama is already running — that's fine
```

### 3. Point the app at local Supabase

Create a file **`.env.local`** in the project root:

```env
VITE_SUPABASE_URL=http://127.0.0.1:54321
VITE_SUPABASE_PUBLISHABLE_KEY=sb_publishable_ACJWlzQHlZjBrEguHvfOxg_3BJgxAaH
```

> After you run `supabase start` (below), the CLI prints your **Publishable** key. If it differs from the one above, copy it from the terminal into `VITE_SUPABASE_PUBLISHABLE_KEY` and restart the dev server.

### 4. Start Docker

Open **Docker Desktop** and wait until it is running.

### 5. Start Supabase and apply migrations

```bash
cd busineess_intelligen
supabase stop --yes    # Clean any previous run
supabase start         # Wait until you see "Started supabase local development setup"
supabase db reset      # Applies migrations (creates execute_admin_sql, etc.)
```

Keep the terminal open; Supabase must stay running. Note the **Project URL** (e.g. `http://127.0.0.1:54321`) and **Publishable** key from the output — use them in `.env.local` if they differ.

---

## Running the project

Use **three terminals** (or run the last two in the background).

### Terminal 1 — Supabase (if not already running)

```bash
cd busineess_intelligen
supabase start
```

Leave this running.

### Terminal 2 — Edge Functions (local model + CSV upload)

```bash
cd busineess_intelligen
MODEL_PROVIDER=local \
LOCAL_MODEL_URL=http://host.docker.internal:11434/api/chat \
LOCAL_MODEL_NAME=llama3.2:3b \
supabase functions serve --no-verify-jwt
```

This serves both `generate-dashboard` (SQL/charts via Ollama) and `upload-csv`. Leave it running.

### Terminal 3 — Frontend

```bash
cd busineess_intelligen
npm run dev
```

Open the URL Vite prints (e.g. **http://localhost:8080/** or **http://localhost:8081/**). Upload a CSV and ask questions in plain English; the app uses your **local Ollama model** (no API key).

---

## Quick reference

| Service           | URL / command |
|-------------------|----------------|
| App (Vite)        | `npm run dev` → e.g. http://localhost:8080 |
| Supabase API      | http://127.0.0.1:54321 |
| Supabase Studio   | http://127.0.0.1:54323 |
| Edge Functions   | http://127.0.0.1:54321/functions/v1/ |

---

## Troubleshooting

- **"Failed to send a request to the Edge Function"** — Supabase or Edge Functions aren’t running. Run `supabase start` and then `supabase functions serve ...` (Terminals 1 and 2).
- **"Edge Function returned a non-2xx status"** — Often due to missing DB setup. Run `supabase db reset` and try again.
- **"Could not find the function public.execute_admin_sql"** — Migrations not applied. Run `supabase db reset`.
- **"Could not find the table 'public.xxx' in the schema cache"** — PostgREST schema cache. Restart Supabase: `supabase stop --yes` then `supabase start`, then upload again.
- **Ollama "address already in use"** — Ollama is already running; no need to run `ollama serve` again.

---

## Tech stack

- **Frontend:** Vite, React, TypeScript, shadcn-ui, Tailwind CSS
- **Backend:** Supabase (Postgres, Edge Functions)
- **LLM:** Local Ollama (e.g. `llama3.2:3b`) — no cloud API
