# Chatbot, Monitoring & Customer Features Guide

## 🤖 Chatbot Integration

### Option 1: Crisp (Recommended - Easiest)

**Why Crisp?**
- ✅ Free tier: Unlimited chats, 2 seats
- ✅ Live chat + chatbot in one
- ✅ Mobile apps for staff
- ✅ Easy setup (one script tag)
- ✅ Pre-built chatbot rules

#### Setup Steps

1. **Sign up at crisp.chat** (free)

2. **Get your website ID**
   - Go to Settings → Website Settings
   - Copy your Website ID

3. **Add to Next.js**
   ```typescript
   // app/layout.tsx or _app.tsx
   import Script from 'next/script'

   export default function RootLayout({ children }) {
     return (
       <html>
         <body>
           {children}
           <Script id="crisp-widget">
             {`
               window.$crisp=[];
               window.CRISP_WEBSITE_ID="${process.env.NEXT_PUBLIC_CRISP_WEBSITE_ID}";
               (function(){
                 d=document;
                 s=d.createElement("script");
                 s.src="https://client.crisp.chat/l.js";
                 s.async=1;
                 d.getElementsByTagName("head")[0].appendChild(s);
               })();
             `}
           </Script>
         </body>
       </html>
     )
   }
   ```

4. **Add environment variable**
   ```env
   NEXT_PUBLIC_CRISP_WEBSITE_ID="your-website-id"
   ```

5. **Configure Chatbot Rules** (in Crisp dashboard)
   - Go to Settings → Chatbot
   - Add rules like:
     - "What are your hours?" → "We're open Mon-Sat 8am-8pm"
     - "How much does washing cost?" → "Wash: $5, Dry: $3, Fold: $2"
     - "Check my ticket status" → Connect to API

6. **Connect to Ticket API** (Advanced)
   ```typescript
   // app/api/crisp-webhook/route.ts
   import { NextRequest, NextResponse } from 'next/server'

   export async function POST(request: NextRequest) {
     const data = await request.json()
     
     // Handle chatbot messages
     if (data.type === 'message:received') {
       const message = data.data.content.text
       
       // Check if asking about ticket
       if (message.includes('ticket') || message.includes('status')) {
         // Extract ticket number
         const ticketNumber = extractTicketNumber(message)
         
         if (ticketNumber) {
           // Fetch ticket from database
           const ticket = await getTicket(ticketNumber)
           
           // Send response back via Crisp API
           await sendCrispMessage(data.session_id, {
             type: 'text',
             content: `Your ticket ${ticketNumber} is ${ticket.status}`
           })
         }
       }
     }
     
     return NextResponse.json({ success: true })
   }
   ```

### Option 2: Tawk.to (Completely Free)

**Why Tawk.to?**
- ✅ 100% free forever
- ✅ Unlimited agents
- ✅ Live chat + chatbot
- ✅ Mobile apps

#### Setup Steps

1. **Sign up at tawk.to** (free)

2. **Get widget code**
   - Go to Administration → Channels → Chat Widget
   - Copy the JavaScript code

3. **Add to Next.js**
   ```typescript
   // app/layout.tsx
   import Script from 'next/script'

   export default function RootLayout({ children }) {
     return (
       <html>
         <body>
           {children}
           <Script
             id="tawk-to"
             strategy="lazyOnload"
             dangerouslySetInnerHTML={{
               __html: `
                 var Tawk_API=Tawk_API||{}, Tawk_LoadStart=new Date();
                 (function(){
                   var s1=document.createElement("script"),s0=document.getElementsByTagName("script")[0];
                   s1.async=true;
                   s1.src='https://embed.tawk.to/YOUR_PROPERTY_ID/YOUR_WIDGET_ID';
                   s1.charset='UTF-8';
                   s1.setAttribute('crossorigin','*');
                   s0.parentNode.insertBefore(s1,s0);
                 })();
               `
             }}
           />
         </body>
       </html>
     )
   }
   ```

### Option 3: Custom AI Chatbot (OpenAI)

**Why Custom?**
- ✅ Full control
- ✅ Can access your database
- ✅ More intelligent responses
- ✅ Cost: ~$0.002 per message

#### Setup Steps

1. **Install OpenAI SDK**
   ```bash
   npm install openai
   ```

2. **Create Chat API**
   ```typescript
   // app/api/chat/route.ts
   import OpenAI from 'openai'
   import { NextRequest, NextResponse } from 'next/server'

   const openai = new OpenAI({
     apiKey: process.env.OPENAI_API_KEY,
   })

   export async function POST(request: NextRequest) {
     const { message, ticketNumber } = await request.json()

     // Get ticket info if provided
     let ticketInfo = ''
     if (ticketNumber) {
       const ticket = await getTicket(ticketNumber)
       ticketInfo = `Customer's ticket: ${ticket.status}, Amount: $${ticket.total_amount}`
     }

     const completion = await openai.chat.completions.create({
       model: "gpt-3.5-turbo",
       messages: [
         {
           role: "system",
           content: `You are a helpful assistant for a laundry mart. 
           You can help customers with:
           - Business hours (Mon-Sat 8am-8pm)
           - Pricing (Wash: $5, Dry: $3, Fold: $2)
           - Ticket status
           - General questions
           ${ticketInfo}`
         },
         {
           role: "user",
           content: message
         }
       ],
       max_tokens: 150,
     })

     return NextResponse.json({
       response: completion.choices[0].message.content
     })
   }
   ```

3. **Frontend Component**
   ```typescript
   // components/Chatbot.tsx
   'use client'
   import { useState } from 'react'

   export function Chatbot() {
     const [messages, setMessages] = useState([])
     const [input, setInput] = useState('')

     const sendMessage = async () => {
       const res = await fetch('/api/chat', {
         method: 'POST',
         headers: { 'Content-Type': 'application/json' },
         body: JSON.stringify({ message: input })
       })
       
       const { response } = await res.json()
       setMessages([...messages, { user: input, bot: response }])
       setInput('')
     }

     return (
       <div className="fixed bottom-4 right-4 w-80 bg-white shadow-lg rounded-lg">
         {/* Chat UI */}
       </div>
     )
   }
   ```

---

## 📊 Monitoring Setup

### 1. Error Tracking: Sentry

#### Setup Steps

1. **Sign up at sentry.io** (free tier: 5,000 events/month)

2. **Install Sentry**
   ```bash
   npm install @sentry/nextjs
   ```

3. **Initialize Sentry**
   ```bash
   npx @sentry/wizard@latest -i nextjs
   ```
   This creates `sentry.client.config.ts` and `sentry.server.config.ts`

4. **Add environment variables**
   ```env
   SENTRY_DSN="your-sentry-dsn"
   SENTRY_AUTH_TOKEN="your-auth-token"
   ```

5. **Test error tracking**
   ```typescript
   // Test in your code
   import * as Sentry from "@sentry/nextjs"

   try {
     // Your code
   } catch (error) {
     Sentry.captureException(error)
   }
   ```

### 2. Analytics: Vercel Analytics (Built-in)

#### Setup Steps

1. **Install Vercel Analytics**
   ```bash
   npm install @vercel/analytics
   ```

2. **Add to app**
   ```typescript
   // app/layout.tsx
   import { Analytics } from '@vercel/analytics/react'

   export default function RootLayout({ children }) {
     return (
       <html>
         <body>
           {children}
           <Analytics />
         </body>
       </html>
     )
   }
   ```

3. **View in Vercel Dashboard**
   - Go to your project → Analytics
   - See page views, performance, etc.

### 3. Uptime Monitoring: UptimeRobot

#### Setup Steps

1. **Sign up at uptimerobot.com** (free: 50 monitors)

2. **Add Monitor**
   - Monitor Type: HTTP(s)
   - URL: Your website URL
   - Interval: 5 minutes
   - Alert Contacts: Your email

3. **Get alerts** when site is down

### 4. Logging: Axiom (Optional)

#### Setup Steps

1. **Sign up at axiom.co** (free tier available)

2. **Install Axiom**
   ```bash
   npm install @axiomhq/next
   ```

3. **Configure**
   ```typescript
   // next.config.js
   const { withAxiom } = require('@axiomhq/next')

   module.exports = withAxiom({
     // Your config
   })
   ```

4. **Log events**
   ```typescript
   import { log } from '@axiomhq/next'

   log.info('Ticket created', { ticketId: '123' })
   ```

---

## 🎯 Customer-Friendly Features

### 1. QR Code for Tickets

#### Setup Steps

1. **Install QR code library**
   ```bash
   npm install qrcode
   ```

2. **Generate QR Code**
   ```typescript
   // app/api/tickets/[id]/qrcode/route.ts
   import QRCode from 'qrcode'
   import { NextResponse } from 'next/server'

   export async function GET(
     request: Request,
     { params }: { params: { id: string } }
   ) {
     const ticketUrl = `${process.env.NEXT_PUBLIC_APP_URL}/tickets/${params.id}`
     
     const qrCode = await QRCode.toDataURL(ticketUrl)
     
     return NextResponse.json({ qrCode })
   }
   ```

3. **Display QR Code**
   ```typescript
   // components/TicketQRCode.tsx
   'use client'
   import { useEffect, useState } from 'react'

   export function TicketQRCode({ ticketId }: { ticketId: string }) {
     const [qrCode, setQrCode] = useState('')

     useEffect(() => {
       fetch(`/api/tickets/${ticketId}/qrcode`)
         .then(res => res.json())
         .then(data => setQrCode(data.qrCode))
     }, [ticketId])

     return <img src={qrCode} alt="Ticket QR Code" />
   }
   ```

### 2. Progressive Web App (PWA)

#### Setup Steps

1. **Install PWA plugin**
   ```bash
   npm install next-pwa
   ```

2. **Configure next.config.js**
   ```javascript
   const withPWA = require('next-pwa')({
     dest: 'public',
     register: true,
     skipWaiting: true,
   })

   module.exports = withPWA({
     // Your config
   })
   ```

3. **Create manifest.json**
   ```json
   // public/manifest.json
   {
     "name": "Laundry Mart",
     "short_name": "Laundry",
     "description": "Laundry ticketing system",
     "start_url": "/",
     "display": "standalone",
     "background_color": "#ffffff",
     "theme_color": "#000000",
     "icons": [
       {
         "src": "/icon-192.png",
         "sizes": "192x192",
         "type": "image/png"
       },
       {
         "src": "/icon-512.png",
         "sizes": "512x512",
         "type": "image/png"
       }
     ]
   }
   ```

4. **Add to layout**
   ```typescript
   // app/layout.tsx
   export default function RootLayout({ children }) {
     return (
       <html>
         <head>
           <link rel="manifest" href="/manifest.json" />
         </head>
         <body>{children}</body>
       </html>
     )
   }
   ```

### 3. Push Notifications

#### Setup Steps

1. **Install web-push**
   ```bash
   npm install web-push
   ```

2. **Generate VAPID keys**
   ```bash
   npx web-push generate-vapid-keys
   ```

3. **Subscribe to notifications**
   ```typescript
   // components/NotificationButton.tsx
   'use client'
   import { useState } from 'react'

   export function NotificationButton() {
     const [subscribed, setSubscribed] = useState(false)

     const subscribe = async () => {
       const registration = await navigator.serviceWorker.ready
       const subscription = await registration.pushManager.subscribe({
         userVisibleOnly: true,
         applicationServerKey: process.env.NEXT_PUBLIC_VAPID_PUBLIC_KEY
       })

       await fetch('/api/notifications/subscribe', {
         method: 'POST',
         body: JSON.stringify(subscription)
       })

       setSubscribed(true)
     }

     return (
       <button onClick={subscribe} disabled={subscribed}>
         {subscribed ? 'Subscribed' : 'Enable Notifications'}
       </button>
     )
   }
   ```

### 4. Real-time Status Updates

#### Setup Steps

1. **Use Server-Sent Events (SSE)**
   ```typescript
   // app/api/tickets/[id]/stream/route.ts
   import { NextRequest } from 'next/server'

   export async function GET(
     request: NextRequest,
     { params }: { params: { id: string } }
   ) {
     const stream = new ReadableStream({
       async start(controller) {
         const encoder = new TextEncoder()
         
         // Poll database every 5 seconds
         const interval = setInterval(async () => {
           const ticket = await getTicket(params.id)
           const data = encoder.encode(`data: ${JSON.stringify(ticket)}\n\n`)
           controller.enqueue(data)
         }, 5000)

         request.signal.addEventListener('abort', () => {
           clearInterval(interval)
           controller.close()
         })
       }
     })

     return new Response(stream, {
       headers: {
         'Content-Type': 'text/event-stream',
         'Cache-Control': 'no-cache',
         'Connection': 'keep-alive'
       }
     })
   }
   ```

2. **Client-side**
   ```typescript
   // components/TicketStatus.tsx
   'use client'
   import { useEffect, useState } from 'react'

   export function TicketStatus({ ticketId }: { ticketId: string }) {
     const [status, setStatus] = useState('')

     useEffect(() => {
       const eventSource = new EventSource(`/api/tickets/${ticketId}/stream`)
       
       eventSource.onmessage = (event) => {
         const ticket = JSON.parse(event.data)
         setStatus(ticket.status)
       }

       return () => eventSource.close()
     }, [ticketId])

     return <div>Status: {status}</div>
   }
   ```

### 5. Share Ticket Link

#### Implementation

```typescript
// components/ShareTicket.tsx
'use client'

export function ShareTicket({ ticketId }: { ticketId: string }) {
  const share = async () => {
    const url = `${window.location.origin}/tickets/${ticketId}`
    
    if (navigator.share) {
      await navigator.share({
        title: 'My Laundry Ticket',
        text: 'Check my laundry ticket status',
        url: url
      })
    } else {
      // Fallback: copy to clipboard
      await navigator.clipboard.writeText(url)
      alert('Link copied to clipboard!')
    }
  }

  return (
    <button onClick={share}>
      Share Ticket
    </button>
  )
}
```

---

## 📱 Quick Setup Checklist

### Chatbot
- [ ] Sign up for Crisp or Tawk.to
- [ ] Add script to layout
- [ ] Configure chatbot rules
- [ ] Test chatbot responses

### Monitoring
- [ ] Set up Sentry for error tracking
- [ ] Add Vercel Analytics
- [ ] Set up UptimeRobot
- [ ] Configure alert emails

### Customer Features
- [ ] Add QR code generation
- [ ] Set up PWA
- [ ] Add push notifications (optional)
- [ ] Implement real-time updates
- [ ] Add share functionality

---

## 💰 Cost Summary

### Free Tier (Everything Free)
- **Chatbot**: Tawk.to (free) or Crisp free tier
- **Error Tracking**: Sentry (5,000 events/month free)
- **Analytics**: Vercel Analytics (free) or Google Analytics (free)
- **Uptime**: UptimeRobot (50 monitors free)
- **Total**: $0/month

### Paid Tier (If Needed)
- **Crisp Pro**: $25/month
- **Sentry Pro**: $26/month
- **Total**: $51/month (optional)

---

## 🚀 Quick Start Commands

```bash
# Install all dependencies
npm install @sentry/nextjs @vercel/analytics qrcode next-pwa

# Set up Sentry
npx @sentry/wizard@latest -i nextjs

# Generate PWA icons (use online tool or design)
# Create public/icon-192.png and public/icon-512.png
```

---

**Pro Tip**: Start with free tiers for everything. You can always upgrade later when you need more features or higher limits!
