# Secure Ticketing System for Laundry Mart - Implementation Plan

## 1. Project Overview

### 1.1 Purpose
A secure, digital ticketing system to manage laundry service requests, track order status, and facilitate customer-staff communication for a laundry mart.

### 1.2 Key Objectives (Small Business Focus)
- Replace paper tickets with simple digital system
- Easy for customers to use (no app download needed)
- Quick setup and deployment (under 1 hour)
- Low cost (free tier or under $20/month)
- Simple for staff to manage
- Basic reporting (daily revenue, ticket counts)
- Mobile-friendly (works on any phone)

---

## 2. Security Requirements

### 2.1 Authentication & Authorization (Simple for Small Business)
- **Basic authentication**: Email + password (secure, but simple)
- **Role-based access control (RBAC)**:
  - Customer: Create tickets, view own tickets (no login needed for viewing)
  - Staff: View all tickets, update status, mark as paid
  - Admin: Everything + manage staff accounts
- **Session management**: Secure session tokens (automatic with NextAuth)
- **Password policies**: Strong passwords (enforced by framework)
- **MFA**: Optional for admin only (not required for small business)

### 2.2 Data Protection
- **Encryption at rest**: Encrypt sensitive data in database
- **Encryption in transit**: HTTPS/TLS for all communications
- **PCI DSS compliance**: If handling payment card data
- **PII protection**: Encrypt customer personal information
- **Data retention policies**: Define how long to keep ticket data

### 2.3 Application Security (Essential Only)
- **Input validation**: Sanitize all user inputs (automatic with Prisma)
- **SQL injection prevention**: Use Prisma ORM (prevents SQL injection automatically)
- **XSS protection**: Built into Next.js and React
- **CSRF protection**: Automatic with Next.js
- **Rate limiting**: Basic rate limiting (optional, add if needed)
- **Audit logging**: Basic logging (ticket status changes, payments)

### 2.4 Infrastructure Security (Simplified)
- **Secure hosting**: Use reputable platform (Vercel, Railway, Render) - they handle security
- **HTTPS/SSL**: Automatic with modern platforms
- **Regular backups**: Use managed database backups (included with platforms)
- **Environment variables**: Use platform's secrets management
- **Basic monitoring**: Use platform's built-in monitoring or simple error tracking (Sentry)

---

## 3. System Architecture (Lightweight & Easy Deploy)

### 3.1 Simplified Architecture
```
┌─────────────────────────────────────────┐
│         Single Web Application          │
│  ┌──────────────┐    ┌──────────────┐  │
│  │   Frontend   │◄───┤    Backend   │  │
│  │  (React/UI)  │    │  (API Routes)│  │
│  └──────────────┘    └──────┬───────┘  │
│                              │          │
│         ┌────────────────────┼──────────┐
│         │                    │          │
│  ┌──────▼──────┐    ┌───────▼───────┐ │
│  │  Database   │    │ File Storage  │ │
│  │(PostgreSQL) │    │(Supabase/     │ │
│  │             │    │ Cloudinary)   │ │
│  └─────────────┘    └───────────────┘ │
└─────────────────────────────────────────┘
```

**Key Simplifications:**
- **Single deployable webapp** - Frontend and backend in one codebase
- **No separate services** - Everything runs together
- **Simple database** - Single PostgreSQL database (or SQLite for dev)
- **No caching/queues** - Keep it simple, add only if needed
- **One-click deployment** - Deploy entire app with single command

### 3.2 Recommended Technology Stack (Lightweight)

#### Option 1: Next.js (Recommended - Easiest)
- **Framework**: Next.js 14+ (Full-stack React framework)
- **Why**: Built-in API routes, server-side rendering, easy deployment
- **UI**: Tailwind CSS + shadcn/ui or NextUI
- **Database ORM**: Prisma or Drizzle
- **Authentication**: NextAuth.js (Auth.js)
- **Deployment**: Vercel (one-click deploy) or Railway/Render

#### Option 2: Remix (Alternative)
- **Framework**: Remix (Full-stack React framework)
- **Why**: Great for forms, simple data loading
- **Deployment**: Fly.io, Railway, or Vercel

#### Option 3: Express + React (Traditional)
- **Frontend**: React + Vite
- **Backend**: Express.js (Node.js)
- **Database**: PostgreSQL with Prisma or TypeORM
- **Deployment**: Railway, Render, or Fly.io

#### Database
- **Development**: SQLite (zero setup, file-based)
- **Production**: PostgreSQL (managed service: Supabase, Railway, or Neon)
- **ORM**: Prisma (recommended - easy migrations, type-safe)

#### File Storage (For Photos, Receipts, Documents)
**What needs storage:**
- Customer photo uploads (items being dropped off)
- Receipt PDFs (generated invoices)
- Document attachments (optional)

**Recommended Solutions (Lightweight & Easy):**

1. **Supabase Storage** (Best if using Supabase database)
   - Free tier: 1GB storage, 2GB bandwidth/month
   - Built-in CDN, image transformations
   - Easy integration with Supabase
   - Cost: $0 (free tier) or $25/month (Pro: 100GB)

2. **Cloudinary** (Best for images)
   - Free tier: 25GB storage, 25GB bandwidth/month
   - Automatic image optimization, resizing
   - Great for photo uploads
   - Cost: $0 (free tier) or $89/month (Advanced)

3. **Vercel Blob** (Best if using Vercel)
   - Free tier: 1GB storage
   - Seamless integration with Vercel
   - Simple API
   - Cost: $0 (free tier) or $0.15/GB/month

4. **Cloudflare R2** (Cheapest option)
   - Free tier: 10GB storage, unlimited egress
   - S3-compatible API
   - No egress fees (unlike S3)
   - Cost: $0.015/GB/month (no free tier, but very cheap)

5. **AWS S3** (Traditional option)
   - Free tier: 5GB storage (first year)
   - Reliable but more complex setup
   - Cost: ~$0.023/GB/month + transfer costs

**Recommendation:** Start with **Supabase Storage** (if using Supabase) or **Cloudinary** (if you need image optimization). Both have generous free tiers and are easy to integrate.

#### Deployment Platforms (Easy Deploy)
1. **Vercel** (Best for Next.js)
   - Free tier available
   - Automatic deployments from GitHub
   - Built-in SSL, CDN, edge functions
   - One-click deploy

2. **Railway** (Best for full-stack apps)
   - Free tier: $5 credit/month
   - Automatic deployments
   - Built-in PostgreSQL
   - Simple pricing

3. **Render** (Good alternative)
   - Free tier available (with limitations)
   - Automatic SSL
   - PostgreSQL add-on available

4. **Fly.io** (Good for global deployment)
   - Free tier available
   - Global edge deployment
   - Simple CLI deployment

#### Simplified Infrastructure
- **No Docker required** (unless you want it)
- **No Kubernetes** - unnecessary for small/medium scale
- **No separate cache/queue** - add only if performance requires it
- **Environment variables** - Use platform's built-in secrets management
- **Backups** - Use managed database backups (included with most platforms)

---

## 4. Core Features & Functionality

### 4.1 Customer Features
- **Ticket Creation**:
  - Service selection (wash, dry, fold, iron, dry clean, etc.)
  - Item details (quantity, special instructions)
  - Pickup/delivery preferences
  - Photo upload (optional, for special items)
  - QR code generation for quick ticket lookup
- **Ticket Tracking**:
  - Real-time status updates (auto-refresh)
  - Estimated completion time
  - SMS/Email/Push notifications
  - QR code scanning to view ticket status
  - Share ticket link with family/friends
- **Payment**:
  - Multiple payment methods (cash, card, digital wallet)
  - Payment status tracking
  - Receipt generation and download
  - Payment reminders
- **Customer Support**:
  - **Live Chatbot** - 24/7 automated support
    - Answer common questions (hours, pricing, services)
    - Check ticket status
    - Help with ticket creation
    - Escalate to human support when needed
  - **Live Chat** (optional) - Connect with staff in real-time
  - **FAQ Section** - Self-service help center
- **History**:
  - View past tickets
  - Reorder from history
  - Download receipts
  - View spending history and statistics
- **Mobile Experience**:
  - Progressive Web App (PWA) - Install on phone
  - Mobile-optimized interface
  - Push notifications
  - Offline ticket viewing (cached)

### 4.2 Staff Features (Simple & Practical)
- **Simple Dashboard**:
  - Today's tickets (all statuses)
  - Pending tickets (need attention)
  - Completed today (count)
- **Ticket Management** (One-Click Actions):
  - Update status (dropdown: Pending → In Progress → Ready → Completed)
  - Add quick note (optional)
  - Mark as paid (when customer pays)
- **Quick Actions**:
  - Print ticket (QR code + details)
  - Send "Ready" notification (email/SMS)

### 4.3 Manager/Admin Features (Essential Only)
- **Simple Dashboard**:
  - Today's revenue
  - This week's revenue
  - Tickets today (count)
  - Pending tickets (count)
- **Basic Reports**:
  - Daily revenue (last 30 days)
  - Service breakdown (wash vs dry vs fold)
  - Export to CSV (for Excel)
- **Settings** (Simple):
  - Service prices (edit in one place)
  - Business hours (display on site)
  - Staff accounts (add/remove staff)

---

## 5. Database Schema Design

### 5.1 Core Tables

#### Users Table
```sql
- id (UUID, Primary Key)
- email (String, Unique, Indexed)
- password_hash (String)
- role (Enum: customer, staff, manager, admin)
- first_name (String)
- last_name (String)
- phone (String)
- is_active (Boolean)
- mfa_enabled (Boolean)
- mfa_secret (String, Encrypted)
- created_at (Timestamp)
- updated_at (Timestamp)
- last_login (Timestamp)
```

#### Tickets Table
```sql
- id (UUID, Primary Key)
- ticket_number (String, Unique, Indexed) -- e.g., "LAU-2024-001234"
- customer_id (UUID, Foreign Key → Users)
- assigned_staff_id (UUID, Foreign Key → Users, Nullable)
- status (Enum: pending, in_progress, ready_for_pickup, completed, cancelled)
- priority (Enum: low, normal, high, urgent)
- service_type (String) -- JSON array of services
- items_description (Text)
- special_instructions (Text)
- photo_urls (JSON, Nullable) -- Array of photo URLs from storage
- receipt_url (String, Nullable) -- URL to generated receipt PDF
- pickup_date (Date)
- delivery_date (Date, Nullable)
- estimated_completion (Timestamp, Nullable)
- actual_completion (Timestamp, Nullable)
- total_amount (Decimal)
- payment_status (Enum: pending, partial, paid, refunded)
- payment_method (String, Nullable)
- created_at (Timestamp)
- updated_at (Timestamp)
```

#### Ticket_Status_History Table
```sql
- id (UUID, Primary Key)
- ticket_id (UUID, Foreign Key → Tickets)
- status (Enum)
- changed_by (UUID, Foreign Key → Users)
- notes (Text, Nullable)
- created_at (Timestamp)
```

#### Payments Table
```sql
- id (UUID, Primary Key)
- ticket_id (UUID, Foreign Key → Tickets)
- amount (Decimal)
- payment_method (String)
- transaction_id (String, Unique, Indexed)
- payment_status (Enum: pending, completed, failed, refunded)
- payment_gateway_response (JSON, Nullable) -- Encrypted
- created_at (Timestamp)
```

#### Audit_Logs Table
```sql
- id (UUID, Primary Key)
- user_id (UUID, Foreign Key → Users, Nullable)
- action (String) -- e.g., "CREATE_TICKET", "UPDATE_STATUS"
- resource_type (String) -- e.g., "TICKET", "USER"
- resource_id (UUID)
- ip_address (String)
- user_agent (String)
- details (JSON)
- created_at (Timestamp)
```

#### Notifications Table
```sql
- id (UUID, Primary Key)
- user_id (UUID, Foreign Key → Users)
- ticket_id (UUID, Foreign Key → Tickets, Nullable)
- type (Enum: email, sms, push, in_app)
- title (String)
- message (Text)
- status (Enum: pending, sent, failed)
- sent_at (Timestamp, Nullable)
- created_at (Timestamp)
```

#### Chat_Messages Table (Optional - for custom chatbot)
```sql
- id (UUID, Primary Key)
- ticket_id (UUID, Foreign Key → Tickets, Nullable)
- user_id (UUID, Foreign Key → Users, Nullable)
- message (Text)
- response (Text, Nullable)
- is_from_bot (Boolean)
- session_id (String, Indexed)
- created_at (Timestamp)
```

#### Service_Types Table (Reference Data)
```sql
- id (UUID, Primary Key)
- name (String)
- description (Text)
- base_price (Decimal)
- unit (String) -- e.g., "per_item", "per_kg"
- is_active (Boolean)
- created_at (Timestamp)
```

---

## 6. API Endpoints Design

### 6.1 Authentication Endpoints
```
POST   /api/auth/register          - Customer registration
POST   /api/auth/login             - Login (returns JWT)
POST   /api/auth/logout            - Logout
POST   /api/auth/refresh           - Refresh JWT token
POST   /api/auth/forgot-password   - Request password reset
POST   /api/auth/reset-password    - Reset password
POST   /api/auth/mfa/enable        - Enable MFA
POST   /api/auth/mfa/verify        - Verify MFA code
```

### 6.2 Ticket Endpoints
```
GET    /api/tickets                - List tickets (filtered by role)
POST   /api/tickets                - Create new ticket
GET    /api/tickets/:id             - Get ticket details
PUT    /api/tickets/:id             - Update ticket (authorized roles)
PATCH  /api/tickets/:id/status      - Update ticket status
GET    /api/tickets/:id/history    - Get ticket status history
POST   /api/tickets/:id/assign     - Assign ticket to staff
```

### 6.3 Payment Endpoints
```
POST   /api/payments               - Create payment
GET    /api/payments/:id           - Get payment details
POST   /api/payments/:id/verify   - Verify payment (webhook)
POST   /api/payments/:id/refund    - Process refund
```

### 6.4 Admin Endpoints
```
GET    /api/admin/users            - List users
POST   /api/admin/users            - Create user
PUT    /api/admin/users/:id        - Update user
DELETE /api/admin/users/:id        - Deactivate user
GET    /api/admin/analytics        - Get analytics data
GET    /api/admin/reports          - Generate reports
GET    /api/admin/audit-logs       - View audit logs
```

---

## 7. Implementation Phases (Simplified - 6-8 Weeks)

### Phase 1: Project Setup & Authentication (Week 1)
- [ ] Initialize Next.js/Remix project
- [ ] Set up database (SQLite for dev, PostgreSQL for prod)
- [ ] Configure Prisma ORM
- [ ] Set up authentication (NextAuth.js or similar)
- [ ] Create user registration/login pages
- [ ] Implement role-based access (customer, staff, admin)
- [ ] Deploy to staging (Vercel/Railway) - test deployment

### Phase 2: Core Ticket System (Week 2)
- [ ] Create database schema (tickets, users, payments)
- [ ] Set up file storage (Supabase Storage or Cloudinary)
- [ ] Configure file upload API endpoint
- [ ] Build ticket creation form (customer-facing)
- [ ] Add photo upload functionality
- [ ] Create ticket listing page with filters
- [ ] Implement ticket status updates
- [ ] Build ticket detail page (with photos)
- [ ] Add ticket assignment (for staff)

### Phase 3: Staff Dashboard (Week 3)
- [ ] Create staff dashboard layout
- [ ] Build ticket queue view
- [ ] Add ticket status update interface
- [ ] Implement ticket search/filter
- [ ] Add basic analytics (tickets today, pending, etc.)

### Phase 4: Payment Integration (Week 4)
- [ ] Integrate Stripe (or PayPal) payment gateway
- [ ] Create payment form
- [ ] Implement payment processing
- [ ] Add payment status tracking
- [ ] Generate receipt PDFs
- [ ] Upload receipts to storage
- [ ] Add receipt download functionality

### Phase 5: Customer Experience & Support (Week 5)
- [ ] Integrate chatbot (Crisp or Tawk.to)
- [ ] Set up chatbot responses (FAQ, ticket status)
- [ ] Add live chat widget (optional)
- [ ] Create FAQ page
- [ ] Add QR code generation for tickets
- [ ] Implement PWA (Progressive Web App)
- [ ] Add push notifications
- [ ] Set up email notifications (Resend or SendGrid)
- [ ] Add SMS notifications (optional - Twilio)
- [ ] Implement responsive design (mobile-friendly)
- [ ] Add loading states and error handling

### Phase 6: Admin Features & Monitoring (Week 6)
- [ ] Build admin dashboard
- [ ] Add user management (create staff accounts)
- [ ] Create basic reports (daily revenue, ticket counts)
- [ ] Set up error tracking (Sentry)
- [ ] Configure uptime monitoring (UptimeRobot)
- [ ] Add analytics (Vercel Analytics or Google Analytics)
- [ ] Set up logging and alerts
- [ ] Create system health dashboard

### Phase 6: Testing & Production Deploy (Week 6)
- [ ] Write basic tests (critical paths)
- [ ] Security review (input validation, SQL injection prevention)
- [ ] Performance optimization
- [ ] Set up production database
- [ ] Deploy to production
- [ ] Configure domain and SSL
- [ ] Test all features in production
- [ ] Create user documentation

### Optional Phase 7: Enhancements (As Needed)
- [ ] SMS notifications (Twilio)
- [ ] Advanced analytics dashboard
- [ ] Export reports (CSV/PDF)
- [ ] Photo upload for tickets
- [ ] Mobile app (PWA or native)

---

## 8. Security Checklist

### Development Phase
- [ ] Use parameterized queries (prevent SQL injection)
- [ ] Validate and sanitize all inputs
- [ ] Implement CSRF protection
- [ ] Use secure session management
- [ ] Encrypt sensitive data at rest
- [ ] Use HTTPS for all communications
- [ ] Implement proper error handling (don't expose sensitive info)
- [ ] Use environment variables for secrets
- [ ] Regular dependency updates (check for vulnerabilities)

### Deployment Phase (Simplified)
- [ ] Use platform's built-in SSL (automatic with Vercel/Railway/Render)
- [ ] Configure environment variables securely
- [ ] Enable platform's built-in monitoring
- [ ] Set up database backups (usually automatic with managed DB)
- [ ] Configure error tracking (Sentry - free tier available)
- [ ] Set up basic rate limiting (if needed)
- [ ] Review platform security settings

### Operational Phase
- [ ] Regular security updates
- [ ] Monitor for suspicious activities
- [ ] Regular backup verification
- [ ] Access review and audit
- [ ] Incident response plan
- [ ] Security training for staff

---

## 9. Compliance Considerations

### 9.1 Data Privacy Regulations
- **GDPR** (if serving EU customers):
  - Right to access data
  - Right to deletion
  - Data portability
  - Consent management
- **CCPA** (if serving California customers):
  - Consumer privacy rights
  - Data disclosure requirements

### 9.2 Payment Card Industry (PCI DSS)
- If handling card data directly, ensure PCI DSS compliance
- Consider using payment gateway (Stripe, PayPal) to offload PCI requirements

### 9.3 Industry Standards
- Follow OWASP Top 10 security practices
- Implement secure coding standards
- Regular security assessments

---

## 10. Scalability Considerations (Simplified)

### 10.1 Performance Optimization
- Database indexing on frequently queried fields
- Pagination for large datasets
- Optimize database queries (use Prisma query optimization)
- Lazy loading for images
- Static generation where possible (Next.js)

### 10.2 Scaling (When Needed)
- Most platforms auto-scale (Vercel, Railway handle this)
- Stateless design (already done with modern frameworks)
- Database connection pooling (handled by ORM)
- Add caching later if needed (Redis, or use platform's edge caching)

### 10.3 Monitoring (Simple)
- Platform's built-in monitoring (Vercel Analytics, Railway metrics)
- Error tracking: Sentry (free tier)
- Uptime monitoring: UptimeRobot (free) or Better Uptime
- Database monitoring: Use platform's database dashboard

---

## 11. Cost Estimation (Lightweight - Low Cost)

### Infrastructure Costs (Monthly)

#### Free Tier Option (Small Business)
- **Hosting (Vercel)**: $0 (free tier: 100GB bandwidth, unlimited requests)
- **Database (Supabase/Neon)**: $0 (free tier: 500MB storage, 2GB bandwidth)
- **Domain**: $10-15/year (Namecheap, Google Domains)
- **SSL**: $0 (included with hosting)
- **Total**: ~$0-2/month (just domain)

#### Paid Tier (Growing Business)
- **Hosting (Vercel Pro)**: $20/month (or Railway $5-10/month)
- **Database (Supabase Pro)**: $25/month (or Railway included)
- **Error Tracking (Sentry)**: $0-26/month (free tier available)
- **Email (Resend)**: $0-20/month (free tier: 3,000 emails/month)
- **Total**: ~$25-75/month

### Third-Party Services
- **Payment Gateway (Stripe)**: 2.9% + $0.30 per transaction (no monthly fee)
- **File Storage**:
  - Supabase Storage: $0 (free: 1GB) or $25/month (100GB)
  - Cloudinary: $0 (free: 25GB) or $89/month (Advanced)
  - Vercel Blob: $0 (free: 1GB) or $0.15/GB/month
  - Cloudflare R2: $0.015/GB/month (very cheap, no egress fees)
- **Chatbot & Support**:
  - Crisp: $0 (free: unlimited chats, 2 seats) or $25/month (Pro)
  - Tawk.to: $0 (completely free, unlimited)
  - Intercom: $0 (free tier limited) or $79/month
  - Custom AI (OpenAI): ~$0.002 per message
- **Monitoring & Analytics**:
  - Sentry: $0 (free: 5,000 events/month) or $26/month
  - Vercel Analytics: $0 (built-in) or $20/month (Pro)
  - UptimeRobot: $0 (free: 50 monitors) or $7/month
  - Google Analytics: $0 (free)
- **Email Service**: 
  - Resend: Free tier (3,000 emails/month)
  - SendGrid: Free tier (100 emails/day)
- **SMS Service (Optional)**: 
  - Twilio: $0.0075-0.01 per SMS
  - Or use email only (free)
- **Domain & SSL**: $10-20/year (SSL free with hosting)

### Storage Usage Estimates
**Typical usage per ticket:**
- 2-5 photos: ~2-5MB (compressed)
- 1 receipt PDF: ~100-200KB
- **Per 100 tickets/month**: ~200-500MB
- **Per 1000 tickets/month**: ~2-5GB

**Storage Recommendations:**
- **Small business** (<500 tickets/month): Free tier sufficient (1-25GB)
- **Medium business** (500-2000 tickets/month): May need paid tier (25-100GB)
- **Large business** (>2000 tickets/month): Consider Cloudflare R2 or paid tier

### Total Monthly Cost Estimate (Small Business)

#### Free Tier (Recommended for Start)
- **Hosting**: $0 (Vercel free tier)
- **Database**: $0 (Supabase free: 500MB)
- **Storage**: $0 (Supabase free: 1GB)
- **Chatbot**: $0 (Tawk.to - completely free)
- **Monitoring**: $0 (Sentry free, UptimeRobot free)
- **Email**: $0 (Resend free: 3,000/month)
- **Domain**: $10-15/year ($1-1.25/month)
- **Total**: $1-2/month (just domain)

#### If Business Grows (Still Affordable)
- **Hosting**: $0-20/month (Vercel Pro if needed)
- **Database**: $0-25/month (Supabase Pro if needed)
- **Storage**: Included in Supabase Pro
- **Everything else**: Still free
- **Total**: $25-50/month (only if you outgrow free tier)

**Key Point**: Most small laundry businesses can run on **$0-2/month** forever!

---

## 12. Risk Assessment

### High Risk
- **Data Breach**: Implement strong encryption, access controls
- **Payment Fraud**: Use secure payment gateway, implement fraud detection
- **System Downtime**: Implement redundancy, monitoring, backup systems

### Medium Risk
- **Performance Issues**: Plan for scalability, optimize queries
- **User Errors**: Implement validation, confirmation dialogs
- **Third-Party Failures**: Have backup providers, fallback mechanisms

### Low Risk
- **Feature Requests**: Plan for extensibility
- **UI/UX Issues**: Regular user feedback, iterative improvements

---

## 13. Success Metrics

### Technical Metrics
- System uptime: >99.9%
- API response time: <200ms (p95)
- Error rate: <0.1%
- Security incidents: 0

### Business Metrics
- Ticket processing time reduction: 30-50%
- Customer satisfaction: >4.5/5
- Staff productivity increase: 20-30%
- Revenue tracking accuracy: 100%

---

## 14. Quick Start Guide (Deploy in Minutes)

### Prerequisites
- Node.js 18+ installed
- Git installed
- GitHub account (for deployment)
- Account on deployment platform (Vercel/Railway/Render)

### One-Command Deployment Options

#### Deploy to Vercel (Fastest - ~5 minutes)
```bash
# After pushing to GitHub:
# 1. Go to vercel.com/new
# 2. Import your GitHub repository
# 3. Vercel auto-detects Next.js
# 4. Add environment variables
# 5. Click Deploy
# Done! Your app is live
```

#### Deploy to Railway (With Database - ~10 minutes)
```bash
# 1. Install Railway CLI
npm i -g @railway/cli

# 2. Login and initialize
railway login
railway init

# 3. Add PostgreSQL
railway add postgresql

# 4. Deploy
railway up

# Railway provides DATABASE_URL automatically
```

### Environment Variables Needed
```env
# Database
DATABASE_URL="postgresql://user:pass@host:5432/dbname"

# Authentication
NEXTAUTH_SECRET="your-secret-key-here"
NEXTAUTH_URL="https://your-domain.com"

# File Storage (choose one)
# Option 1: Supabase Storage
NEXT_PUBLIC_SUPABASE_URL="https://your-project.supabase.co"
NEXT_PUBLIC_SUPABASE_ANON_KEY="your-anon-key"
SUPABASE_SERVICE_ROLE_KEY="your-service-role-key"

# Option 2: Cloudinary
CLOUDINARY_CLOUD_NAME="your-cloud-name"
CLOUDINARY_API_KEY="your-api-key"
CLOUDINARY_API_SECRET="your-api-secret"

# Option 3: Vercel Blob
BLOB_READ_WRITE_TOKEN="vercel_blob_token"

# Payment (Stripe)
STRIPE_SECRET_KEY="sk_test_..."
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY="pk_test_..."

# Email (Optional)
RESEND_API_KEY="re_..."

# Chatbot (choose one)
NEXT_PUBLIC_CRISP_WEBSITE_ID="your-crisp-id"
# OR
NEXT_PUBLIC_TAWK_PROPERTY_ID="your-tawk-id"
NEXT_PUBLIC_TAWK_WIDGET_ID="your-tawk-widget-id"

# Monitoring
SENTRY_DSN="your-sentry-dsn"
SENTRY_AUTH_TOKEN="your-sentry-token"

# Push Notifications (Optional)
NEXT_PUBLIC_VAPID_PUBLIC_KEY="your-vapid-public-key"
VAPID_PRIVATE_KEY="your-vapid-private-key"

# Custom AI Chatbot (Optional - OpenAI)
OPENAI_API_KEY="sk-..."
```

### Deployment Checklist
- [ ] Code pushed to GitHub
- [ ] Database created (Supabase/Neon/Railway)
- [ ] Environment variables configured
- [ ] Domain configured (optional)
- [ ] SSL certificate active (automatic)
- [ ] Test login/registration
- [ ] Test ticket creation
- [ ] Test payment flow (test mode)

### Option 1: Next.js + Vercel (Recommended - Easiest)
```bash
# 1. Create Next.js project
npx create-next-app@latest laundry-ticketing --typescript --tailwind --app

# 2. Add dependencies
npm install prisma @prisma/client next-auth
npm install -D @types/node

# 3. Initialize Prisma
npx prisma init

# 4. Push to GitHub
git init
git add .
git commit -m "Initial commit"
git remote add origin <your-repo>
git push -u origin main

# 5. Deploy to Vercel
# - Go to vercel.com
# - Import your GitHub repo
# - Add environment variables
# - Deploy! (automatic)
```

### Option 2: Railway (Full-stack with Database)
```bash
# 1. Create project (same as above)
# 2. Install Railway CLI
npm i -g @railway/cli

# 3. Deploy
railway login
railway init
railway up

# 4. Add PostgreSQL
railway add postgresql
# Railway automatically provides DATABASE_URL
```

### Option 3: Render (Alternative)
1. Push code to GitHub
2. Go to render.com
3. Create new Web Service
4. Connect GitHub repo
5. Add PostgreSQL database
6. Deploy!

### Quick Setup Checklist
- [ ] Choose platform (Vercel recommended for Next.js)
- [ ] Create GitHub repository
- [ ] Initialize project with chosen framework
- [ ] Set up database (SQLite for dev, PostgreSQL for prod)
- [ ] Configure environment variables
- [ ] Deploy to staging
- [ ] Test deployment
- [ ] Configure custom domain (optional)

## 15. Next Steps

1. **Review and approve this plan**
2. **Choose technology stack** (Next.js recommended for easiest deployment)
3. **Set up project** using Quick Start Guide above
4. **Follow implementation phases** (6 weeks to production)
5. **Deploy and iterate** based on user feedback

---

## 16. Resources & References

### Framework Documentation
- Next.js: https://nextjs.org/docs
- Prisma: https://www.prisma.io/docs
- NextAuth.js: https://next-auth.js.org/
- Tailwind CSS: https://tailwindcss.com/docs

### Deployment Platforms
- Vercel: https://vercel.com/docs
- Railway: https://docs.railway.app/
- Render: https://render.com/docs
- Fly.io: https://fly.io/docs/

### Security
- OWASP Security Guidelines: https://owasp.org/
- Next.js Security: https://nextjs.org/docs/app/building-your-application/configuring/security-headers
- Prisma Security: https://www.prisma.io/docs/guides/security

### Payment Integration
- Stripe: https://stripe.com/docs
- Stripe + Next.js: https://stripe.com/docs/payments/quickstart

### Database
- Supabase (Free PostgreSQL): https://supabase.com/
- Neon (Serverless PostgreSQL): https://neon.tech/
- Prisma Migrations: https://www.prisma.io/docs/guides/migrate

---

## 17. Project Structure Example (Next.js)

```
laundry-ticketing/
├── app/                    # Next.js App Router
│   ├── (auth)/            # Auth routes
│   │   ├── login/
│   │   └── register/
│   ├── (customer)/        # Customer routes
│   │   ├── tickets/
│   │   └── tickets/[id]/
│   ├── (staff)/           # Staff routes
│   │   └── dashboard/
│   ├── (admin)/           # Admin routes
│   │   └── admin/
│   ├── api/               # API routes
│   │   ├── auth/
│   │   ├── tickets/
│   │   └── payments/
│   └── layout.tsx
├── components/            # React components
│   ├── ui/               # Reusable UI components
│   ├── forms/            # Form components
│   └── layouts/         # Layout components
├── lib/                  # Utilities
│   ├── prisma.ts         # Prisma client
│   ├── auth.ts           # Auth configuration
│   └── utils.ts          # Helper functions
├── prisma/
│   ├── schema.prisma     # Database schema
│   └── migrations/       # Database migrations
├── public/               # Static assets
├── .env.local           # Environment variables
├── .env.example         # Example env file
├── next.config.js       # Next.js config
├── tailwind.config.js   # Tailwind config
└── package.json
```

---

**Document Version**: 2.0 (Lightweight Edition)  
**Last Updated**: [Current Date]  
**Focus**: Easy deployment, single webapp, minimal infrastructure
