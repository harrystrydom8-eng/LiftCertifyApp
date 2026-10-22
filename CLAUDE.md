# LiftCertify - guide for AI assistants working on this code

LiftCertify is a multi-company (SaaS) web app for lifting equipment and NDT inspection
companies. It replaces an MS Access prototype called CertificateManager.

## Stack
- Next.js App Router, React Server Components, server actions, TypeScript (strict)
- Supabase: PostgreSQL + Auth + Storage, accessed with `@supabase/ssr`
- Plain CSS in `app/globals.css` (no Tailwind). Colours: navy #0B2545, steel #13579B,
  safety amber #F2A900 (key actions only, navy text on amber), bg #F4F6F9, lines #D9E1EA.
  Green = pass, red = fail / overdue - never use them decoratively.

## Non-negotiable rules
1. **Every tenant table has `company_id`** and row-level security. Never bypass RLS from the
   app (no service-role key in the browser or in normal server code).
2. **Role rules live in the database** (`has_role`, `can_see_division`, `can_create_in`,
   `can_approve_in` in `supabase/migrations`). `lib/roles.ts` mirrors them for the UI only.
   Change both together.
3. Roles: admin, lmi, lti, ndt (with ndt_level 1-3), office, manager. Users can have several;
   rights add up. An expired qualification pauses only that role.
4. Approval: LMI approves divisions 1-4, NDT Level 2/3 approve division 5. **Admin never approves.**
   Approval only through `approve_inspection()`. Any change to an approved certificate or its
   results removes the approval (trigger).
5. Divisions: 1 Lifting Tackle, 2 Lifting Machines, 3 Mobile Machinery,
   4 Specialised Equipment, 5 NDT. LTI sees only 1, NDT technicians only 5.
6. Units: a serial number is stored once per company + type + customer (`serial_key`).
7. Certificates are numbered per company + equipment type (prefix + number, e.g. CB12).
8. Any defect means the certificate is NOT ISSUED.
9. Schema changes go in a NEW file `supabase/migrations/000N_name.sql` - never edit 0001.

## Writing style for the UI
Sentence case, plain English, no exclamation marks, verbs on buttons ("Create certificate").
South African context: dates dd/mm/yyyy in documents, kg for loads.
