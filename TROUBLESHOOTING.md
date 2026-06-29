# Troubleshooting Guide

## Common Issues & Solutions

### 🔴 Build Fails: "Cannot find module"

**Error message:**
```
Error: Cannot find module '@/components/...'
```

**Solution:**
1. Clear node_modules and reinstall:
```bash
rm -rf node_modules package-lock.json
npm install
```

2. Verify `tsconfig.app.json` has the path alias:
```json
"paths": {
  "@/*": ["./src/*"]
}
```

3. Make sure the file actually exists at the specified path

---

### 🔴 Build Fails: TypeScript Errors

**Error message:**
```
TS2304: Cannot find name 'React'
```

**Solution:**
- This is expected! The `eslint.config.ts` auto-imports React
- The ESLint config declares these as globals, so they work in the app
- For TypeScript checking, run: `npm run type-check`
- If you see errors, ensure `vite-env.d.ts` exists and contains the global declarations

---

### 🔴 Vite Dev Server Won't Start

**Error message:**
```
EADDRINUSE: address already in use :::3000
```

**Solution:**
Port 3000 is already in use. Either:

1. Kill the process using port 3000:
```bash
# On macOS/Linux
lsof -i :3000 | grep LISTEN | awk '{print $2}' | xargs kill -9

# On Windows
netstat -ano | findstr :3000
taskkill /PID [PID] /F
```

2. Or use a different port:
```bash
npm run dev -- --port 3001
```

---

### 🔴 Supabase Connection Fails

**Error message:**
```
Failed to fetch from Supabase
```

**Solution:**
1. Check `.env.local` exists and has correct values:
```bash
cat .env.local
```

2. Verify environment variables are loaded:
   - In browser console: `console.log(import.meta.env.VITE_PUBLIC_SUPABASE_URL)`
   - Should show your Supabase URL

3. Check Supabase project status:
   - Go to https://supabase.com/dashboard
   - Verify your project is "Healthy"

4. Verify RLS policies:
   - Go to Supabase → SQL Editor
   - Run: `SELECT * FROM site_content;`
   - Should return data without errors

---

### 🔴 Admin Dashboard Won't Load

**Error message:**
```
Cannot read property 'content' of undefined
```

**Solution:**
1. Verify `site_content` table exists and has data:
```sql
SELECT * FROM site_content;
```

2. If empty, insert initial data:
```sql
INSERT INTO site_content (id, content) VALUES (1, '{}'::jsonb)
ON CONFLICT (id) DO NOTHING;
```

3. Check browser console for more detailed errors

---

### 🔴 GitHub Actions Deployment Fails

**Error message in Actions tab:**
```
Build step failed
```

**Solution:**
1. Check the workflow logs:
   - Go to **Actions** tab in your GitHub repo
   - Click the failed workflow run
   - Expand "Build" section to see full logs

2. Common reasons:
   - **Missing secrets**: Verify `SUPABASE_URL` and `SUPABASE_ANON_KEY` are set in repo settings
   - **Dependency issues**: Try pushing again (sometimes npm cache needs refresh)
   - **TypeScript errors**: Run `npm run type-check` locally to catch issues

3. Add secrets if missing:
   - Go to **Settings** → **Secrets and variables** → **Actions**
   - Add `SUPABASE_URL` and `SUPABASE_ANON_KEY`

---

### 🔴 GitHub Pages Shows 404

**Symptom:**
```
404 Page not found
```

**Solution:**
1. Verify GitHub Pages is enabled:
   - Go to **Settings** → **Pages**
   - **Source** should be "GitHub Actions"

2. Check deployment status:
   - Go to **Deployments** tab
   - Look for recent deployment
   - Click to see logs

3. Verify base path in `vite.config.ts`:
   - Should be `base: process.env.BASE_PATH || "/"`
   - GitHub Actions passes `BASE_PATH=/GTTC`

4. The live site URL should be:
   ```
   https://hosammr.github.io/GTTC
   ```

---

### 🔴 Linting Fails in CI/CD

**Error message:**
```
ESLint errors found
```

**Solution:**
1. Fix issues locally:
```bash
npm run lint
```

2. If you want to ignore certain warnings, update `eslint.config.ts`:
```typescript
rules: {
  'rule-name': 'warn' // or 'off'
}
```

3. For no warnings in build:
```bash
npm run lint -- --max-warnings 0
```

---

### 🔴 CSS/Tailwind Not Loading

**Symptom:**
- Page looks unstyled
- No colors or spacing applied

**Solution:**
1. Verify `tailwind.config.ts` content paths:
```typescript
content: [
  "./index.html",
  "./src/**/*.{js,ts,jsx,tsx}",
]
```

2. Verify `src/main.tsx` imports styles:
```typescript
import './index.css' // or similar
```

3. Rebuild:
```bash
npm run dev
```

4. Check browser DevTools:
   - Inspect element
   - Verify Tailwind classes are applied
   - Check Network tab for CSS file

---

### 🔴 Hot Module Replacement (HMR) Not Working

**Symptom:**
- Changes don't auto-reload in dev server
- Have to manually refresh

**Solution:**
1. Check Vite config `server.host`:
```typescript
server: {
  host: "0.0.0.0",
  port: 3000,
}
```

2. If using SSH tunnel or docker:
   - Set `VITE_HMR_HOST` environment variable
   - Example: `VITE_HMR_HOST=localhost npm run dev`

3. Try hard refresh: `Ctrl+Shift+R` (Windows) or `Cmd+Shift+R` (Mac)

---

### 🟡 Performance Issues

**Symptom:**
- Dev server is slow
- Build takes a long time

**Solution:**
1. Clear cache:
```bash
rm -rf node_modules .vite
npm install
```

2. Check what's taking time:
```bash
npm run build -- --reporter verbose
```

3. Reduce build scope (if needed):
   - Comment out unused routes
   - Lazy-load components

---

## Debug Mode

Enable verbose logging:

```bash
# For dev server
DEBUG=* npm run dev

# For build
DEBUG=* npm run build

# For ESLint
npm run lint -- --debug
```

## Getting Help

If you're stuck:

1. **Check browser console** (F12 → Console tab)
2. **Check terminal output** for stack traces
3. **Check GitHub Actions logs** for CI/CD issues
4. **Check Supabase logs** in dashboard
5. **Search [GitHub Issues](https://github.com/hosammr/GTTC/issues)**
6. **Ask in discussions** if needed

## Performance Optimization Checklist

- [ ] `npm run build` completes without warnings
- [ ] `npm run lint` passes with 0 errors
- [ ] `npm run type-check` passes
- [ ] Bundle size is reasonable (`npm run build` output shows sizes)
- [ ] GitHub Actions workflow completes in < 5 minutes
- [ ] GitHub Pages deployment succeeds
- [ ] Site loads in < 2 seconds on 4G
- [ ] Lighthouse score > 80 (run in Chrome DevTools)

---

**Last Updated**: June 2026
