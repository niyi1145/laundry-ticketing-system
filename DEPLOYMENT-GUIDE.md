# Quick Deployment Guide - Laundry Ticketing System

## 🚀 Fastest Path to Production

### Recommended Stack: Next.js + Vercel + Supabase

**Why?**
- ✅ Next.js: Full-stack in one codebase
- ✅ Vercel: One-click deploy, free tier, automatic SSL
- ✅ Supabase: Free PostgreSQL, built-in auth option
- ✅ Total setup time: ~30 minutes
- ✅ Monthly cost: $0-5 (free tier)

---

## Step-by-Step Deployment

### 1. Create Project (5 minutes)

```bash
# Create Next.js project
npx create-next-app@latest laundry-ticketing \
  --typescript \
  --tailwind \
  --app \
  --no-src-dir

cd laundry-ticketing

# Install essential packages
npm install prisma @prisma/client next-auth
npm install @auth/prisma-adapter
npm install bcryptjs
npm install -D @types/bcryptjs
```

### 2. Set Up Database (5 minutes)

#### Option A: Supabase (Recommended - Free)
1. Go to https://supabase.com
2. Create new project
3. Go to Settings → Database
4. Copy connection string
5. Format: `postgresql://postgres:[PASSWORD]@[HOST]:5432/postgres`
6. **Bonus**: Supabase includes free file storage (1GB) - no extra setup needed!

#### Option B: Neon (Alternative - Free)
1. Go to https://neon.tech
2. Create project
3. Copy connection string
4. **Note**: You'll need separate storage (see Step 2.5)

#### Initialize Prisma
```bash
# Initialize Prisma
npx prisma init

# Edit prisma/schema.prisma - set provider to "postgresql"
# Add your DATABASE_URL to .env

# Create initial schema
npx prisma migrate dev --name init

# Generate Prisma Client
npx prisma generate
```

### 2.5 Set Up File Storage (5 minutes)

#### Option A: Supabase Storage (If using Supabase - Easiest)
1. In Supabase dashboard → Storage
2. Create bucket: `ticket-photos` (public)
3. Create bucket: `receipts` (private)
4. Copy API keys from Settings → API
5. Add to `.env`:
   ```
   NEXT_PUBLIC_SUPABASE_URL="https://xxx.supabase.co"
   NEXT_PUBLIC_SUPABASE_ANON_KEY="your-anon-key"
   SUPABASE_SERVICE_ROLE_KEY="your-service-role-key"
   ```

#### Option B: Cloudinary (Best for images)
1. Go to https://cloudinary.com
2. Sign up (free tier: 25GB)
3. Copy credentials from Dashboard
4. Install: `npm install cloudinary`
5. Add to `.env`:
   ```
   CLOUDINARY_CLOUD_NAME="your-cloud-name"
   CLOUDINARY_API_KEY="your-api-key"
   CLOUDINARY_API_SECRET="your-api-secret"
   ```

#### Option C: Vercel Blob (If using Vercel)
1. Go to Vercel dashboard → Storage → Blob
2. Create storage
3. Copy token
4. Add to `.env`:
   ```
   BLOB_READ_WRITE_TOKEN="vercel_blob_token"
   ```

### 3. Push to GitHub (2 minutes)

```bash
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/yourusername/laundry-ticketing.git
git push -u origin main
```

### 4. Deploy to Vercel (5 minutes)

1. Go to https://vercel.com
2. Sign up/Login with GitHub
3. Click "Add New Project"
4. Import your `laundry-ticketing` repository
5. Vercel auto-detects Next.js
6. Add environment variables:
   ```
   DATABASE_URL=your-supabase-connection-string
   NEXTAUTH_SECRET=generate-random-string-here
   NEXTAUTH_URL=https://your-app.vercel.app
   ```
7. Click "Deploy"
8. Wait 2-3 minutes
9. **Done!** Your app is live at `https://your-app.vercel.app`

### 5. Configure Custom Domain (Optional - 5 minutes)

1. In Vercel dashboard → Settings → Domains
2. Add your domain
3. Follow DNS instructions
4. SSL is automatic

---

## Alternative: Railway Deployment

### Why Railway?
- ✅ Includes PostgreSQL database
- ✅ Simple pricing ($5/month credit)
- ✅ One command deploy

### Steps:

```bash
# Install Railway CLI
npm i -g @railway/cli

# Login
railway login

# Initialize project
railway init

# Add PostgreSQL
railway add postgresql

# Deploy
railway up

# Get database URL
railway variables
```

Railway automatically provides `DATABASE_URL` environment variable.

---

## Environment Variables Checklist

Create `.env.local` for local development:

```env
# Database
DATABASE_URL="postgresql://..."

# NextAuth
NEXTAUTH_SECRET="use-openssl-rand-base64-32"
NEXTAUTH_URL="http://localhost:3000"

# File Storage (choose one)
# Option 1: Supabase Storage
NEXT_PUBLIC_SUPABASE_URL="https://xxx.supabase.co"
NEXT_PUBLIC_SUPABASE_ANON_KEY="your-anon-key"
SUPABASE_SERVICE_ROLE_KEY="your-service-role-key"

# Option 2: Cloudinary
CLOUDINARY_CLOUD_NAME="your-cloud-name"
CLOUDINARY_API_KEY="your-api-key"
CLOUDINARY_API_SECRET="your-api-secret"

# Option 3: Vercel Blob
BLOB_READ_WRITE_TOKEN="vercel_blob_token"

# Stripe (for payments - get from stripe.com)
STRIPE_SECRET_KEY="sk_test_..."
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY="pk_test_..."

# Email (optional - use Resend free tier)
RESEND_API_KEY="re_..."
```

**Generate NEXTAUTH_SECRET:**
```bash
openssl rand -base64 32
```

---

## Quick Test After Deployment

1. ✅ Visit your deployed URL
2. ✅ Test registration
3. ✅ Test login
4. ✅ Create a test ticket
5. ✅ Check database (Supabase dashboard or Railway)

---

## Common Issues & Solutions

### Issue: Database connection error
**Solution:** 
- Check DATABASE_URL format
- Ensure database allows connections from Vercel IPs
- For Supabase: Check connection pooling settings

### Issue: Build fails
**Solution:**
- Check Node.js version (should be 18+)
- Review build logs in Vercel dashboard
- Ensure all dependencies are in package.json

### Issue: Environment variables not working
**Solution:**
- Restart deployment after adding env vars
- Check variable names match code exactly
- Use Vercel's environment variable UI

---

## Cost Breakdown

### Free Tier (Starting Out)
- **Vercel**: Free (100GB bandwidth/month)
- **Supabase**: Free (500MB database, 2GB bandwidth)
- **Domain**: $10-15/year (optional)
- **Total**: $0-1.25/month

### Paid Tier (Growing)
- **Vercel Pro**: $20/month (if needed)
- **Supabase Pro**: $25/month (if needed)
- **Total**: $25-50/month

---

## Next Steps After Deployment

1. ✅ Set up authentication
2. ✅ Create database schema
3. ✅ Build ticket creation form
4. ✅ Build staff dashboard
5. ✅ Add payment integration
6. ✅ Test thoroughly
7. ✅ Go live!

---

## Useful Commands

```bash
# Local development
npm run dev

# Database migrations
npx prisma migrate dev

# View database
npx prisma studio

# Build for production
npm run build

# Deploy to Vercel (if using Vercel CLI)
vercel

# Check Prisma schema
npx prisma format
```

---

## Support Resources

- **Next.js Docs**: https://nextjs.org/docs
- **Vercel Docs**: https://vercel.com/docs
- **Prisma Docs**: https://www.prisma.io/docs
- **Supabase Docs**: https://supabase.com/docs

---

**Pro Tip:** Start with the free tier, test everything, then upgrade only when needed. Most small businesses can run on free tier for months!
