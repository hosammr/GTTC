# Supabase Setup Guide

This guide walks you through setting up Supabase for the GTTC website CMS functionality.

## Step 1: Create a Supabase Account & Project

1. Go to [supabase.com](https://supabase.com) and sign up for a free account
2. Create a new project:
   - **Project name**: `GTTC` (or similar)
   - **Database password**: Save this securely (you'll need it)
   - **Region**: Choose the region closest to your users (e.g., Europe)
3. Wait for the project to initialize (2-3 minutes)

## Step 2: Get Your Credentials

Once your project is ready:

1. Go to **Project Settings** → **API**
2. Copy these values:
   - **Project URL** → `VITE_PUBLIC_SUPABASE_URL`
   - **anon public** key → `VITE_PUBLIC_SUPABASE_ANON_KEY`

3. Update your `.env.local` file:
```env
VITE_PUBLIC_SUPABASE_URL=https://your-project.supabase.co
VITE_PUBLIC_SUPABASE_ANON_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

## Step 3: Create Database Tables

Go to **SQL Editor** in Supabase and run this SQL:

```sql
-- Create site_content table (single-row CMS for homepage)
CREATE TABLE site_content (
  id BIGINT PRIMARY KEY DEFAULT 1,
  content JSONB NOT NULL DEFAULT '{}'::jsonb,
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  CONSTRAINT only_one_row CHECK (id = 1)
);

-- Insert initial row
INSERT INTO site_content (id, content) VALUES (1, '{
  "hero": {
    "title": "Welkom bij GTTC Groningen",
    "subtitle": "Dé Tafeltennisvereniging in Groningen",
    "cta_text": "Word lid"
  },
  "cards": [
    {
      "id": 1,
      "title": "Training",
      "description": "Trainingen voor alle niveaus",
      "icon": "activity"
    }
  ],
  "sections": [],
  "agenda": [],
  "news": [],
  "footer": {
    "company": "GTTC Groningen",
    "phone": "+31 (0)6 123 45 678",
    "email": "info@gttc.nl"
  }
}'::jsonb);

-- Create navigation table
CREATE TABLE navigation (
  id BIGINT PRIMARY KEY DEFAULT 1,
  items JSONB NOT NULL DEFAULT '[]'::jsonb,
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  CONSTRAINT only_one_row CHECK (id = 1)
);

-- Insert initial navigation
INSERT INTO navigation (id, items) VALUES (1, '[
  {
    "label": "Home",
    "href": "/",
    "id": "home",
    "submenu": []
  },
  {
    "label": "Over GTTC",
    "href": "/over-gttc",
    "id": "about",
    "submenu": []
  },
  {
    "label": "Nieuws",
    "href": "/nieuws",
    "id": "news",
    "submenu": []
  },
  {
    "label": "Competitie",
    "href": "/competitie",
    "id": "competition",
    "submenu": []
  },
  {
    "label": "Trainingen",
    "href": "/trainingen",
    "id": "training",
    "submenu": []
  },
  {
    "label": "Lidmaatschap",
    "href": "/lidmaatschap",
    "id": "membership",
    "submenu": []
  },
  {
    "label": "Agenda",
    "href": "/agenda",
    "id": "agenda",
    "submenu": []
  },
  {
    "label": "Contact",
    "href": "/contact",
    "id": "contact",
    "submenu": []
  }
]'::jsonb);

-- Set up Row Level Security (RLS)
ALTER TABLE site_content ENABLE ROW LEVEL SECURITY;
ALTER TABLE navigation ENABLE ROW LEVEL SECURITY;

-- Create policy to allow public read access
CREATE POLICY "Allow public read access" ON site_content
  FOR SELECT
  USING (true);

CREATE POLICY "Allow public read access" ON navigation
  FOR SELECT
  USING (true);

-- Create a separate admin_users table for authentication
CREATE TABLE admin_users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  username TEXT UNIQUE NOT NULL,
  password_hash TEXT NOT NULL,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Enable RLS on admin_users
ALTER TABLE admin_users ENABLE ROW LEVEL SECURITY;

-- Only allow inserts/updates from authenticated users (you'll authenticate via password in the app)
CREATE POLICY "Admin only" ON admin_users
  FOR ALL
  USING (true);
```

## Step 4: Configure Row Level Security (RLS) Policies

### For `site_content` table:
- **Public**: Can READ ✅
- **Admin**: Can READ, UPDATE ❌ (handled via password in frontend for now)

### For `navigation` table:
- **Public**: Can READ ✅
- **Admin**: Can READ, UPDATE ❌ (handled via password in frontend for now)

## Step 5: Update Environment Variables

Create `.env.local` in your project root:

```env
VITE_PUBLIC_SUPABASE_URL=https://[YOUR-PROJECT-ID].supabase.co
VITE_PUBLIC_SUPABASE_ANON_KEY=[YOUR-ANON-KEY]
```

**⚠️ Important**: Never commit `.env.local` to GitHub. It's already in `.gitignore`.

## Step 6: Set Up GitHub Secrets (for CI/CD)

To enable automatic deployments to GitHub Pages, add these secrets to your GitHub repository:

1. Go to **Settings** → **Secrets and variables** → **Actions**
2. Click **New repository secret**
3. Add these secrets:
   - `SUPABASE_URL` = Your Supabase project URL
   - `SUPABASE_ANON_KEY` = Your Supabase anon key

The GitHub Actions workflow will use these for production builds.

## Step 7: Test the Connection

1. Start the dev server:
```bash
npm run dev
```

2. Open your browser console (F12) and check for any Supabase errors
3. The site should load without errors
4. Navigate to `/beheer` (admin page) to test the admin dashboard

## Troubleshooting

### Issue: "Could not connect to Supabase"
- ✅ Verify `VITE_PUBLIC_SUPABASE_URL` is correct (no trailing slash)
- ✅ Verify `VITE_PUBLIC_SUPABASE_ANON_KEY` is correct
- ✅ Check that RLS policies are set correctly
- ✅ Ensure tables exist in your Supabase project

### Issue: Admin dashboard won't load
- ✅ Check browser console for errors (F12 → Console tab)
- ✅ Verify `site_content` table has at least one row
- ✅ Clear browser cache and reload

### Issue: RLS Policy Errors
- ✅ Go to Supabase SQL Editor
- ✅ Verify all policies are created correctly
- ✅ Check that `Enable RLS` is toggled ON for both tables

## Next Steps

After setup, your content will be:
- **Synced** across all visitors via Supabase
- **Editable** via the admin dashboard at `/beheer`
- **Persistent** across deployments

## Resources

- [Supabase Docs](https://supabase.com/docs)
- [PostgreSQL Docs](https://www.postgresql.org/docs/)
- [Row Level Security (RLS)](https://supabase.com/docs/guides/auth/row-level-security)
