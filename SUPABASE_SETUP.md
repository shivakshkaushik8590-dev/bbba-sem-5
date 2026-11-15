# ☕ 2 AM TO PM — Supabase Setup & Deployment Guide

This guide walks you through establishing the complete **Supabase** backend for the **2 AM TO PM — Haute European Cafe & Salon** web application.

---

## 📋 Prerequisites
1. A Supabase account ([https://supabase.com](https://supabase.com)).
2. Web browser or terminal access.

---

## Step 1: Create a Supabase Project

1. Log in to the [Supabase Dashboard](https://app.supabase.com).
2. Click **New Project** and select your Organization.
3. Configure your project:
   - **Name:** `2am-to-pm-salon`
   - **Database Password:** Choose a strong password and save it securely.
   - **Region:** Choose a region closest to your patrons (e.g., `West Europe (London / Frankfurt)` for European salon latency).
4. Click **Create new project** and wait ~2 minutes for provisioning to complete.

---

## Step 2: Retrieve API Credentials

1. In your project dashboard, navigate to **Project Settings** (gear icon) > **API**.
2. Copy the following keys:
   - **Project URL:** `https://your-project-id.supabase.co`
   - **Project API Keys > `anon` `public`:** (Safe for browser client)
   - **Project API Keys > `service_role` `secret`:** (Keep strictly server-side!)

---

## Step 3: Configure Environment Variables

Create or update `frontend/.env.local` (or `.env` in `frontend/`):

```bash
# Public Vite Client Variables
VITE_SUPABASE_URL=https://your-project-id.supabase.co
VITE_SUPABASE_ANON_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...your-anon-key...

# App Details
VITE_APP_ENV=development
VITE_APP_NAME="2AM to PM — Haute European Cafe & Salon"
```

> **Security Reminder:** Never commit `.env.local` or `.env` files containing live keys to Git. The provided `.gitignore` already ignores all `.env*` files except `.env.example`.

---

## Step 4: Execute Database Migrations

You can apply the migrations using either the **Supabase Dashboard SQL Editor** or the **Supabase CLI**.

### Option A: Using the Supabase Dashboard (Recommended for quick setup)

1. In the Supabase Dashboard, go to the **SQL Editor** tab.
2. Click **New Query**.
3. Open and paste the contents of the migration files in this exact order:

#### 1. Schema & Tables:
Copy from [`supabase/migrations/20261104000001_initial_schema.sql`](file:///c:/Users/shiva/Downloads/stitch_2am_to_pm_luxury_cafe/supabase/migrations/20261104000001_initial_schema.sql)  
Click **Run**. This creates:
- Extensions (`uuid-ossp`, `pgcrypto`)
- Custom ENUM types (`user_role`, `reservation_status`, `salon_atmosphere_type`, `vigil_period_type`)
- 9 tables (`profiles`, `customers`, `reservations`, `menu_categories`, `menu_items`, `contact_messages`, `newsletter_subscribers`, `gallery_images`, `site_settings`)
- Foreign keys, indexes, and automated `updated_at` triggers

#### 2. Row Level Security & Storage:
Copy from [`supabase/migrations/20261104000002_rls_and_security.sql`](file:///c:/Users/shiva/Downloads/stitch_2am_to_pm_luxury_cafe/supabase/migrations/20261104000002_rls_and_security.sql)  
Click **Run**. This establishes:
- Helper functions (`is_admin()`, `is_staff()`)
- Granular RLS policies for each table
- Storage buckets (`menu-images`, `gallery`, `avatars`, `documents`) and storage access policies

#### 3. Initial Brand Seed Data:
Copy from [`supabase/migrations/20261104000003_seed_data.sql`](file:///c:/Users/shiva/Downloads/stitch_2am_to_pm_luxury_cafe/supabase/migrations/20261104000003_seed_data.sql)  
Click **Run**. This populates:
- 3 Menu Categories (`Artisan Coffee`, `Salon Patisserie`, `Late Night Libations`)
- 12 Signature offerings with prices and portion volumes
- 5 Archival gallery visual records with descriptions
- Site settings for salon hours (Vigil, Grand Day, Nocturne) and addresses

---

### Option B: Using Supabase CLI

If you have Supabase CLI installed locally:
```bash
# Link to your remote project
supabase link --project-ref your-project-id

# Push migrations to remote database
supabase db push
```

---

## Step 5: Configure Supabase Authentication

1. In your dashboard, go to **Authentication** > **Providers** > **Email**.
2. Ensure **Enable Email provider** is turned on.
3. (Optional / Recommended for Luxury Hospitality): Enable **Sign in with Magic Link** so VIP patrons can authenticate without traditional password friction.
4. Under **Authentication** > **URL Configuration**:
   - **Site URL:** Set to `http://localhost:3000` (for local development) or your production domain.
   - **Redirect URLs:** Add `http://localhost:3000/**`.

---

## Step 6: Verify Storage Buckets

1. In your dashboard, open the **Storage** section.
2. Confirm the four buckets have been created by the migration:
   - `menu-images` (Public)
   - `gallery` (Public)
   - `avatars` (Public)
   - `documents` (Private)
3. If creating buckets manually in the UI, make sure to mark `menu-images` and `gallery` as **Public Buckets**.

---

## Step 7: Test the Database Connection

### Method 1: Using the Automated Test Script
Run the built-in diagnostic test runner:
```bash
# From the frontend directory:
npm run test:db
```
or run directly with Node:
```bash
node frontend/src/lib/testConnection.js
```

The script performs the following automated verifications:
1. Validates presence and structure of `VITE_SUPABASE_URL` and `VITE_SUPABASE_ANON_KEY`.
2. Measures API round-trip latency to PostgreSQL.
3. Tests read access on public tables (`site_settings`, `menu_categories`).
4. Verifies GoTrue Auth service responsiveness.
5. Verifies Supabase Storage API endpoint status.

### Method 2: Testing from the Running Application
1. Start the Vite development server:
   ```bash
   cd frontend
   npm run dev
   ```
2. Open [http://localhost:3000](http://localhost:3000).
3. Scroll to **Reserve a Salon Table** (`#visit-us`):
   - Fill in your name, honorary email, party size, and atmosphere.
   - Click **Confirm Salon Table Privilege**.
   - Check your Supabase dashboard under **Table Editor** > **reservations**; the submitted booking will appear with `status = 'pending'`.
4. In the Footer, enter an email in **The Salon Bulletin** and click **Join**:
   - Check **Table Editor** > **newsletter_subscribers**; the email will appear.

---

## 🔧 Troubleshooting

| Issue | Cause | Resolution |
| :--- | :--- | :--- |
| `PGRST116` or `42P01: relation does not exist` | Database tables have not yet been created. | Re-run `20261104000001_initial_schema.sql` in the Supabase SQL Editor. |
| `new row violates row-level security policy` | A write operation was attempted that failed RLS check (e.g. invalid date or email). | Ensure date is in the future (`>= CURRENT_DATE`) and email is properly formatted. |
| Storage upload fails with `403 Forbidden` | Patron is not logged in or lacks staff role. | Uploads to `menu-images` and `gallery` require `is_staff() = true`. Patrons can only upload to `avatars/{userId}/`. |
| Changes in tables don't reflect immediately in API | PostgREST schema cache needs refresh. | In SQL Editor run: `NOTIFY pgrst, 'reload schema';` |
| `Failed to fetch` in browser console | CORS policy or invalid Supabase URL. | Verify your `VITE_SUPABASE_URL` in `frontend/.env.local` matches your project ref exactly. |

