# Troubleshooting Guide

Common issues and solutions when working with Next.js, Supabase, and Vercel.

## Authentication Issues

### Issue: "Invalid login credentials"

**Possible causes:**
- Email/password combination is incorrect
- User hasn't confirmed their email (if email confirmation is enabled)
- User account was deleted

**Solutions:**
- Double-check credentials
- Check Supabase Dashboard → Authentication → Users
- Verify email confirmation settings

### Issue: Redirect URLs not working

**Symptoms:**
- Auth callback redirects to wrong URL
- "Invalid redirect URL" error

**Solutions:**
1. Check Supabase Dashboard → Authentication → URL Configuration
2. Ensure all redirect URLs are added:
   - `http://localhost:3000/auth/callback`
   - `https://your-app.vercel.app/auth/callback`
3. Verify `NEXT_PUBLIC_SITE_URL` is set correctly

### Issue: Email confirmation links go to localhost on Vercel

**Cause:** Not using correct environment variable priority

**Solution:** Use this pattern in your code:

```typescript
const siteUrl =
  process.env.NEXT_PUBLIC_SITE_URL ??      // Explicit production URL
  process.env.NEXT_PUBLIC_VERCEL_URL ??    // Auto-set by Vercel
  'http://localhost:3000';                 // Fallback
```

## Database Issues

### Issue: "relation does not exist"

**Cause:** Migrations haven't been run

**Solution:**
```bash
supabase db push
```

### Issue: "permission denied for table"

**Cause:** Row Level Security (RLS) policies blocking access

**Solutions:**
1. Check RLS policies in Supabase Dashboard
2. Verify user is authenticated
3. Ensure policies allow the operation

### Issue: Type errors with database types

**Cause:** Database types not generated or out of date

**Solution:**
```bash
supabase gen types typescript --project-id <project-ref> > src/lib/database.types.ts
```

### Issue: "Property 'full_name' does not exist on type 'never'" during build

**Cause:** TypeScript can't infer types when using `.select('*')` in Supabase queries

**Solution:** Use explicit column selects and import database types:

```typescript
import { Database } from '@/lib/database.types'

type Profile = Database['public']['Tables']['profiles']['Row']

// ❌ Wrong - TypeScript can't infer type
const { data: profile } = await supabase
  .from('profiles')
  .select('*')
  .eq('id', user.id)
  .single()

// ✅ Correct - Explicit columns allow type inference
const { data: profile } = await supabase
  .from('profiles')
  .select('full_name, role')
  .eq('id', user.id)
  .single()
```

## Build & Deployment Issues

### Issue: Build fails on Vercel

**Common causes:**
- Missing environment variables
- TypeScript errors
- Missing dependencies

**Solutions:**
1. Check Vercel build logs
2. Verify all environment variables are set
3. Run `npm run build` locally to catch errors
4. Ensure `database.types.ts` exists

### Issue: "Dynamic server usage" warnings

**Not an error!** These are informational messages indicating that pages using `cookies()` cannot be statically generated. This is expected behavior for authenticated pages.

### Issue: Service role key not working

**Symptoms:**
- Admin operations fail silently
- User invitations don't work

**Solutions:**
1. Verify `SUPABASE_SERVICE_ROLE_KEY` is set in Vercel
2. Check it's not exposed to the client
3. Redeploy after adding environment variables

## Development Issues

### Issue: Hot reload not working

**Solutions:**
- Restart dev server: `npm run dev`
- Clear `.next` folder: `rm -rf .next`
- Check for syntax errors

### Issue: "Cannot find module" errors

**Solutions:**
- Run `npm install`
- Check import paths are correct
- Verify TypeScript path aliases in `tsconfig.json`

### Issue: Supabase client errors

**Symptoms:**
- "Invalid API key"
- Connection errors

**Solutions:**
1. Verify `.env.local` has correct values
2. Check Supabase project is active
3. Regenerate API keys if needed

## Common Code Issues

### Issue: 406 Error when fetching data

**Cause:** Using `.single()` when no rows exist

**Solution:** Use `.maybeSingle()` instead:

```typescript
// ❌ Wrong
const { data } = await supabase
  .from('table')
  .select('*')
  .eq('id', id)
  .single(); // Returns 406 if no rows

// ✅ Correct
const { data } = await supabase
  .from('table')
  .select('*')
  .eq('id', id)
  .maybeSingle(); // Returns null if no rows
```

### Issue: Users reappear after deletion

**Cause:** Not filtering soft-deleted users

**Solution:** Add `WHERE deleted_at IS NULL` to queries:

```sql
SELECT * FROM auth.users
WHERE deleted_at IS NULL;
```

### Issue: Server action can't return Buffer

**Cause:** Server actions can only return serializable data

**Solution:** Convert to base64:

```typescript
// Server action
const base64Audio = content.toString('base64');
return { audio: base64Audio };

// Client component
const binaryString = atob(result.audio);
const bytes = new Uint8Array(binaryString.length);
for (let i = 0; i < binaryString.length; i++) {
  bytes[i] = binaryString.charCodeAt(i);
}
const blob = new Blob([bytes], { type: 'audio/mpeg' });
```

## Getting Help

If you're still stuck:

1. Check the [Supabase Documentation](https://supabase.com/docs)
2. Check the [Next.js Documentation](https://nextjs.org/docs)
3. Check the [Vercel Documentation](https://vercel.com/docs)
4. Review the [User Management Guide](./user-management.md) for detailed patterns
5. Open an issue on GitHub

## Debugging Tips

### Enable Debug Logging

Add console logs (but never log secrets!):

```typescript
console.log('Environment check:', {
  NEXT_PUBLIC_SITE_URL: process.env.NEXT_PUBLIC_SITE_URL,
  NEXT_PUBLIC_VERCEL_URL: process.env.NEXT_PUBLIC_VERCEL_URL,
  HAS_SERVICE_KEY: !!process.env.SUPABASE_SERVICE_ROLE_KEY,
});
```

### Check Vercel Logs

1. Go to Vercel Dashboard → Your Project → Functions
2. Click on a function to see logs
3. Check for errors or warnings

### Check Supabase Logs

1. Go to Supabase Dashboard → Logs
2. Filter by API, Auth, or Database
3. Look for errors or failed requests

---

*Last updated: Based on production experience*

