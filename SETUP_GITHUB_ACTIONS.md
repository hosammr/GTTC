# GitHub Actions & Deployment Setup Guide

This guide covers setting up automatic deployment to GitHub Pages and configuring GitHub Secrets for CI/CD.

## Step 1: Enable GitHub Pages

1. Go to your repository on GitHub
2. Navigate to **Settings** → **Pages**
3. Under **Source**, select:
   - **Deploy from a branch**
   - **Branch**: `main`
   - **Folder**: `/ (root)`
4. Click **Save**

Your site will be deployed to: `https://hosammr.github.io/GTTC`

## Step 2: Add GitHub Secrets

GitHub Secrets are encrypted environment variables used during CI/CD builds.

### How to add secrets:

1. Go to **Settings** → **Secrets and variables** → **Actions**
2. Click **New repository secret**
3. Add each secret below:

| Secret Name | Value |
|---|---|
| `SUPABASE_URL` | Your Supabase project URL (from Step 5 in SETUP_SUPABASE.md) |
| `SUPABASE_ANON_KEY` | Your Supabase anon key (from Step 5 in SETUP_SUPABASE.md) |

### Example:
- **Name**: `SUPABASE_URL`
- **Value**: `https://[YOUR-PROJECT-ID].supabase.co`
- Click **Add secret**

Repeat for `SUPABASE_ANON_KEY`.

## Step 3: Create GitHub Actions Workflow

1. Create the workflow directory:
```bash
mkdir -p .github/workflows
```

2. Create `.github/workflows/deploy.yml`:
```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Type check
        run: npm run type-check

      - name: Lint
        run: npm run lint

      - name: Build
        env:
          VITE_PUBLIC_SUPABASE_URL: ${{ secrets.SUPABASE_URL }}
          VITE_PUBLIC_SUPABASE_ANON_KEY: ${{ secrets.SUPABASE_ANON_KEY }}
          BASE_PATH: /GTTC
        run: npm run build

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v2
        with:
          path: './out'

  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v2
```

3. Commit and push:
```bash
git add .github/workflows/deploy.yml
git commit -m "Add GitHub Actions deployment workflow"
git push origin main
```

## Step 4: Verify Workflow Setup

1. Go to your repository
2. Click the **Actions** tab
3. You should see a workflow run named "Deploy to GitHub Pages"
4. Click on it to see the build logs

### Expected steps in the workflow:
- ✅ Checkout code
- ✅ Setup Node.js
- ✅ Install dependencies
- ✅ Type check
- ✅ Lint
- ✅ Build
- ✅ Upload artifact
- ✅ Deploy to GitHub Pages

## Step 5: Monitor Deployments

### View deployment status:
1. Go to **Deployments** tab in your repository
2. Look for the latest deployment
3. Click to see logs and status

### View build logs:
1. Go to **Actions** tab
2. Click the latest workflow run
3. Click **Build** job to see full logs

## Step 6: First Deployment

Once everything is set up:

1. Make a commit to `main` branch:
```bash
git add .
git commit -m "Trigger initial deployment"
git push origin main
```

2. GitHub Actions will automatically:
   - Run linting and type checks
   - Build your site with environment variables from secrets
   - Deploy to GitHub Pages

3. After ~2 minutes, your site will be live at:
   ```
   https://hosammr.github.io/GTTC
   ```

## Troubleshooting Deployments

### ❌ Build fails with "Missing environment variables"
**Solution:**
1. Verify secrets are added in **Settings** → **Secrets and variables** → **Actions**
2. Verify secret names match exactly:
   - `SUPABASE_URL` (not `VITE_PUBLIC_SUPABASE_URL`)
   - `SUPABASE_ANON_KEY`
3. Click the failed workflow run and check logs for details

### ❌ GitHub Pages shows "No site here"
**Solution:**
1. Go to **Settings** → **Pages**
2. Verify **Source** is set to "GitHub Actions"
3. Wait 2-3 minutes for initial deployment
4. Check **Deployments** tab for latest deployment status

### ❌ "Cannot find module" errors in build
**Solution:**
1. Run locally first: `npm install && npm run build`
2. Fix any errors locally
3. Push to main: `git push origin main`
4. GitHub Actions will retry automatically

### ❌ Workflow not running
**Solution:**
1. Verify workflow file is at `.github/workflows/deploy.yml`
2. Verify you're pushing to `main` branch (not a different branch)
3. Check that the workflow YAML is valid:
   - Use [yamllint](https://www.yamllint.com/) to validate
4. Click **Actions** tab and look for any error messages

### ❌ Build passes but site isn't updated
**Solution:**
1. Hard refresh your browser: `Ctrl+Shift+R` (Windows) or `Cmd+Shift+R` (Mac)
2. Clear browser cache completely
3. Wait 5-10 minutes for GitHub Pages to cache update
4. Check **Deployments** tab to confirm latest deployment succeeded

## Environment Variables Reference

### Used during CI/CD builds:
| Variable | Source | Purpose |
|---|---|---|
| `VITE_PUBLIC_SUPABASE_URL` | `secrets.SUPABASE_URL` | Supabase project URL |
| `VITE_PUBLIC_SUPABASE_ANON_KEY` | `secrets.SUPABASE_ANON_KEY` | Supabase anonymous key |
| `BASE_PATH` | Hardcoded: `/GTTC` | GitHub Pages base path |

### Passed to npm scripts:
```yaml
env:
  VITE_PUBLIC_SUPABASE_URL: ${{ secrets.SUPABASE_URL }}
  VITE_PUBLIC_SUPABASE_ANON_KEY: ${{ secrets.SUPABASE_ANON_KEY }}
  BASE_PATH: /GTTC
run: npm run build
```

## Workflow Overview

```
┌─────────────────────────┐
│  Push to main branch    │
└────────────┬────────────┘
             │
             ▼
┌─────────────────────────┐
│  GitHub Actions starts  │
└────────────┬────────────┘
             │
             ▼
┌─────────────────────────┐     ┌──────────────────┐
│  Checkout code          │────▶│  Setup Node 18   │
└────────────┬────────────┘     └──────────────────┘
             │
             ▼
┌─────────────────────────┐     ┌──────────────────┐
│  npm ci (install)       │────▶│  Type check      │
└────────────┬────────────┘     └──────────────────┘
             │
             ▼
┌─────────────────────────┐     ┌──────────────────┐
│  npm run lint           │────▶│  npm run build   │
└────────────┬────────────┘     └──────────────────┘
             │
             ▼
┌─────────────────────────┐
│  Upload to GitHub Pages │
└────────────┬────────────┘
             │
             ▼
┌─────────────────────────┐
│  Live at hosammr.       │
│  github.io/GTTC         │
└─────────────────────────┘
```

## Best Practices

✅ **Do:**
- Keep `.env.local` out of version control (it's in `.gitignore`)
- Use GitHub Secrets for sensitive values
- Run `npm run lint` and `npm run type-check` locally before pushing
- Check workflow logs if deployment fails
- Test locally with `npm run build && npm run preview`

❌ **Don't:**
- Commit `.env` or `.env.local` files
- Use hardcoded secrets in the workflow file
- Push to main without testing locally
- Ignore GitHub Actions warnings or errors

## Next Steps

1. ✅ Add secrets to GitHub (SUPABASE_URL, SUPABASE_ANON_KEY)
2. ✅ Create `.github/workflows/deploy.yml` workflow file
3. ✅ Push to main branch
4. ✅ Check Actions tab for successful build
5. ✅ Visit https://hosammr.github.io/GTTC to see live site

## Additional Resources

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [GitHub Pages Deployment Guide](https://docs.github.com/en/pages/getting-started-with-github-pages)
- [GitHub Secrets Documentation](https://docs.github.com/en/actions/security-guides/encrypted-secrets)
- [Vite Deployment Guide](https://vitejs.dev/guide/static-deploy.html#github-pages)
