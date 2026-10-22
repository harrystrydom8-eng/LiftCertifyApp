# LiftCertify - web app (phase 1)

Inspection certificates for lifting equipment and NDT, as a web app for many client companies.

**Stack:** Next.js (React + TypeScript) · Supabase (PostgreSQL database, logins, storage) · Vercel (hosting)

## What phase 1 does

- Sign in with email and password
- Roles per user, enforced **in the database**: Admin, LMI, LTI, NDT (Level 1-3), Office, Manager
- Five divisions; each user only sees the divisions their roles allow
- Unit register (each serial number stored once per customer)
- Certificates with the checklist of their equipment type: OK / Defect / N/A / Repaired
- Approval: LMI approves lifting certificates, NDT Level 2/3 approve NDT reports, Admin never.
  Expired qualifications pause the role. Changing an approved certificate removes the approval.
- Audit trail of every creation, change, checklist result and approval
- Every client company's data is kept separate by row-level security

---

## 1. Install the tools (once)

1. **Node.js LTS** - https://nodejs.org (choose the LTS version)
2. **Git** - https://git-scm.com
3. **VS Code** - you already have it

## 2. Create the database (once)

1. Make a free account at https://supabase.com and create a **new project**.
   Choose the region closest to your users and a strong database password.
2. In the project: **SQL Editor > New query**. Paste the whole of
   `supabase/migrations/0001_init.sql` and click **Run**.
3. New query again: paste `supabase/seed.sql` and **Run** (demo company KGOLAGANO with
   equipment types, checklists, a customer and units).
4. **Authentication > Users > Add user > Create new user**: your email and a password
   (tick "Auto confirm user").
5. Link that login to the company and give it roles. SQL Editor, change the email and name, **Run**:

```sql
insert into public.profiles (id, company_id, full_name, roles, lmi_reg_no, lmi_expiry)
select u.id, c.id, 'KOBUS LE GRANGE', array['admin','lmi'], 'LMI 201612060', '2027-12-31'
from auth.users u, public.companies c
where u.email = 'you@example.com' and c.name = 'KGOLAGANO';
```

For an NDT technician use `array['ndt']` and add `ndt_level, ndt_scheme, ndt_cert_no, ndt_expiry`.

## 3. Run it on your computer

1. Open this folder in VS Code (**File > Open Folder**).
2. Copy `.env.example` to a new file `.env.local` and fill in the two values from
   Supabase **Project Settings > API** (the project URL and the anon / publishable key).
3. In the VS Code terminal (**Terminal > New Terminal**):

```
npm install
npm run dev
```

4. Open http://localhost:3000 and sign in.

## 4. Put it on GitHub (PRIVATE)

This is your product's code - always a **private** repository.
VS Code: **Source Control > Publish to GitHub > Publish to GitHub private repository**.
`.env.local` is never uploaded (it is in `.gitignore`).

## 5. Put it online (Vercel)

1. https://vercel.com > sign in with GitHub > **Add New > Project** > choose the repository.
2. Add the two environment variables from `.env.local`.
3. **Deploy**. Every push to GitHub updates the live app.
4. Later: add your domain, e.g. `app.liftcertify.com`.

---

## Project map

| Path | What it is |
| --- | --- |
| `supabase/migrations/0001_init.sql` | Tables, role rules, approval and audit triggers, row-level security |
| `supabase/seed.sql` | Demo data |
| `lib/roles.ts` | The role rules for the screens (the database enforces them) |
| `lib/session.ts` | Signed-in user and profile |
| `lib/supabase/*` | Supabase connection |
| `middleware.ts` | Sends anyone not signed in to /login |
| `app/login` | Sign-in page |
| `app/(app)/page.tsx` | Start: figures and divisions |
| `app/(app)/divisions/[slug]` | Certificates of a division |
| `app/(app)/certificates/new` | New certificate (and new unit) |
| `app/(app)/certificates/[id]` | Certificate: checklist, approval, audit trail |
| `app/(app)/approvals` | Waiting for your approval |

## Roadmap

| Phase | Adds |
| --- | --- |
| 2 | Customers, units and equipment types screens; users and roles screen; company logo |
| 3 | PDF certificates in your layout, email to the client, inspection tags |
| 4 | Photos, defect report, re-inspections, load test criteria, NDT / MPI details |
| 5 | Bulk load from Excel, jobs and timesheets, dashboards and forecast |
| 6 | Customer portal |
| 7 | Mobile / offline use on site (installable app, syncs when back in signal) |
| 8 | Subscriptions and billing, sign-up for new client companies |
