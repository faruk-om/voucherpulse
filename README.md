# VoucherPulse

Lightweight real-estate underwriting helper. Add properties, store basic financials, and see NOI & DSCR at a glance. Built with **Next.js 14**, **TypeScript**, and **Supabase**.

> Status: MVP (Create + List). Next up: Edit/Delete, auth, and styling.

---

## Features (MVP)

- 🔎 Property list with **NOI** and **DSCR** per deal  
- ➕ Add Property form (title, city, state, price, rent, mortgage/mo)  
- 🛢️ Supabase Postgres backend  
- ⚡ Next.js App Router + client components  
- 🧮 Simple, transparent formulas in code

---

## Tech Stack

- **Frontend:** Next.js 14 (App Router), React 18, TypeScript  
- **Backend:** Supabase (Postgres, Row Level Security policies)  
- **Auth (planned):** Supabase Auth  
- **Styling (planned):** TailwindCSS

---

## Quick Start

### 1) Prereqs
- **Node.js 20** (recommended via `nvm-windows`)
- A **Supabase** project

### 2) Clone & install
```bash
git clone https://github.com/faruk-om/voucherpulse.git
cd voucherpulse
npm install
```

### 3) Environment variables
Create `.env.local` in the project root:

```
NEXT_PUBLIC_SUPABASE_URL=https://<YOUR-PROJECT-REF>.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=<YOUR-ANON-KEY>
```

> Don’t commit `.env.local`. It’s already gitignored.

### 4) Run dev server
```bash
npm run dev
# http://localhost:3000
```

---

## Database

Create a `properties` table and the basic finance columns.

```sql
-- Base table
create table if not exists public.properties (
  id uuid primary key default gen_random_uuid(),
  title text not null,
  city  text,
  state text,
  price numeric,
  rent numeric,
  taxes numeric default 0,
  insurance numeric default 0,
  maintenance numeric default 0,
  hoa numeric default 0,
  mgmt_pct numeric default 10,   -- % of rent
  vacancy_pct numeric default 5, -- % of rent
  mortgage_mo numeric default 0, -- monthly debt service
  created_at timestamp default now()
);

-- During development you can either disable RLS or add simple policies:
alter table public.properties enable row level security;

-- (Option A) Open read/write for anon (dev only)
create policy "anon can read properties" on public.properties
for select to anon using (true);

create policy "anon can insert properties" on public.properties
for insert to anon with check (true);

-- (Option B) Quick dev shortcut (not for prod)
-- alter table public.properties disable row level security;
```

### Seed a couple of rows (optional)
```sql
insert into public.properties (title, city, state, price, rent, taxes, insurance, maintenance, hoa, mgmt_pct, vacancy_pct, mortgage_mo)
values
 ('2-Bed Apartment','Detroit','MI',1200,1500,250,100,100,0,10,5,900),
 ('Test numeric row','Detroit','MI',1350,1500,250,100,100,0,10,5,900);
```

---

## How DSCR is calculated

```text
NOI = rent
      - (rent * vacancy_pct/100)
      - (rent * mgmt_pct/100)
      - taxes - insurance - maintenance - hoa

DSCR = NOI / mortgage_mo
```

The calculation is implemented in `app/properties/page.tsx`.

---

## App Structure

```
app/
  page.tsx                 # Home – links to properties
  properties/
    page.tsx               # List view (NOI & DSCR per row)
    new/
      page.tsx             # Add Property form
lib/
  supabaseClient.ts        # Supabase browser client
```

---

## Scripts

```bash
npm run dev      # start local dev server
npm run build    # production build
npm run start    # run built app
```

---

## Roadmap

- [ ] Edit / Delete properties  
- [ ] Auth (Supabase) + per-user ownership  
- [ ] Tailwind UI (cards/table, color-coded DSCR)  
- [ ] Export deal sheet (PDF)  
- [ ] Simple comps & rent reasonability helper  
- [ ] Deployment (Vercel)

---

## Deploy (Vercel)

1. Push to GitHub (already done).  
2. Create a new Vercel project from this repo.  
3. Add the same env vars in **Vercel → Settings → Environment Variables**:  
   - `NEXT_PUBLIC_SUPABASE_URL`  
   - `NEXT_PUBLIC_SUPABASE_ANON_KEY`  
4. Deploy.

---

## Notes / Troubleshooting

- If you see **“relation… does not exist”**, run the SQL above to create the table.  
- If reads/inserts fail with 401/403, your **RLS** policies are too strict—use the dev policies above.  
- Windows users: if port 3000 is busy, Next.js will use **3001** automatically.

---

## Credits

Built by **Faruk** with a lot of grit and some nerdy guidance. 🛠️
