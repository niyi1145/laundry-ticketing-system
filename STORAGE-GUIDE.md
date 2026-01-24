# File Storage Implementation Guide

## Overview

Your laundry ticketing system needs file storage for:
- **Customer photos** (items being dropped off) - typically 2-5 photos per ticket
- **Receipt PDFs** (generated invoices) - ~100-200KB each
- **Optional**: Document attachments

## Storage Options Comparison

| Solution | Free Tier | Paid Tier | Best For | Setup Difficulty |
|----------|-----------|-----------|----------|------------------|
| **Supabase Storage** | 1GB | $25/mo (100GB) | If using Supabase DB | ⭐ Easy |
| **Cloudinary** | 25GB | $89/mo (Advanced) | Image optimization | ⭐⭐ Medium |
| **Vercel Blob** | 1GB | $0.15/GB | If using Vercel | ⭐ Easy |
| **Cloudflare R2** | None | $0.015/GB | Cost-effective | ⭐⭐ Medium |
| **AWS S3** | 5GB (1st year) | $0.023/GB | Enterprise | ⭐⭐⭐ Complex |

## Recommended: Supabase Storage

**Why?** If you're using Supabase for your database, storage is included and integrated.

### Setup Steps

1. **Create Storage Buckets**
   ```bash
   # In Supabase Dashboard → Storage
   # Create two buckets:
   # 1. "ticket-photos" (public) - for customer photos
   # 2. "receipts" (private) - for PDF receipts
   ```

2. **Install Supabase Client**
   ```bash
   npm install @supabase/supabase-js
   ```

3. **Create Supabase Client**
   ```typescript
   // lib/supabase.ts
   import { createClient } from '@supabase/supabase-js'

   const supabaseUrl = process.env.NEXT_PUBLIC_SUPABASE_URL!
   const supabaseAnonKey = process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!

   export const supabase = createClient(supabaseUrl, supabaseAnonKey)
   ```

4. **Upload Photo API Route**
   ```typescript
   // app/api/upload/route.ts
   import { supabase } from '@/lib/supabase'
   import { NextRequest, NextResponse } from 'next/server'

   export async function POST(request: NextRequest) {
     const formData = await request.formData()
     const file = formData.get('file') as File
     const ticketId = formData.get('ticketId') as string

     if (!file) {
       return NextResponse.json({ error: 'No file' }, { status: 400 })
     }

     const fileExt = file.name.split('.').pop()
     const fileName = `${ticketId}/${Date.now()}.${fileExt}`
     const fileBuffer = await file.arrayBuffer()

     const { data, error } = await supabase.storage
       .from('ticket-photos')
       .upload(fileName, fileBuffer, {
         contentType: file.type,
         upsert: false
       })

     if (error) {
       return NextResponse.json({ error: error.message }, { status: 500 })
     }

     const { data: { publicUrl } } = supabase.storage
       .from('ticket-photos')
       .getPublicUrl(data.path)

     return NextResponse.json({ url: publicUrl })
   }
   ```

5. **Upload from Frontend**
   ```typescript
   // components/PhotoUpload.tsx
   'use client'
   import { useState } from 'react'

   export function PhotoUpload({ ticketId }: { ticketId: string }) {
     const [uploading, setUploading] = useState(false)

     const handleUpload = async (e: React.ChangeEvent<HTMLInputElement>) => {
       const file = e.target.files?.[0]
       if (!file) return

       setUploading(true)
       const formData = new FormData()
       formData.append('file', file)
       formData.append('ticketId', ticketId)

       const res = await fetch('/api/upload', {
         method: 'POST',
         body: formData
       })

       const { url } = await res.json()
       // Save URL to database
       setUploading(false)
     }

     return (
       <input
         type="file"
         accept="image/*"
         onChange={handleUpload}
         disabled={uploading}
       />
     )
   }
   ```

## Alternative: Cloudinary

**Why?** Best for image optimization, transformations, and larger free tier.

### Setup Steps

1. **Sign up at cloudinary.com** (free tier: 25GB)

2. **Install Cloudinary**
   ```bash
   npm install cloudinary
   ```

3. **Upload API Route**
   ```typescript
   // app/api/upload/route.ts
   import { v2 as cloudinary } from 'cloudinary'
   import { NextRequest, NextResponse } from 'next/server'

   cloudinary.config({
     cloud_name: process.env.CLOUDINARY_CLOUD_NAME,
     api_key: process.env.CLOUDINARY_API_KEY,
     api_secret: process.env.CLOUDINARY_API_SECRET,
   })

   export async function POST(request: NextRequest) {
     const formData = await request.formData()
     const file = formData.get('file') as File

     const bytes = await file.arrayBuffer()
     const buffer = Buffer.from(bytes)

     return new Promise((resolve) => {
       cloudinary.uploader.upload_stream(
         { folder: 'ticket-photos' },
         (error, result) => {
           if (error) {
             resolve(NextResponse.json({ error: error.message }, { status: 500 }))
           } else {
             resolve(NextResponse.json({ url: result!.secure_url }))
           }
         }
       ).end(buffer)
     })
   }
   ```

## Alternative: Vercel Blob

**Why?** Seamless if using Vercel, simple API.

### Setup Steps

1. **Install Vercel Blob**
   ```bash
   npm install @vercel/blob
   ```

2. **Upload API Route**
   ```typescript
   // app/api/upload/route.ts
   import { put } from '@vercel/blob'
   import { NextRequest, NextResponse } from 'next/server'

   export async function POST(request: NextRequest) {
     const formData = await request.formData()
     const file = formData.get('file') as File
     const ticketId = formData.get('ticketId') as string

     if (!file) {
       return NextResponse.json({ error: 'No file' }, { status: 400 })
     }

     const blob = await put(`tickets/${ticketId}/${file.name}`, file, {
       access: 'public',
       token: process.env.BLOB_READ_WRITE_TOKEN!,
     })

     return NextResponse.json({ url: blob.url })
   }
   ```

## Database Schema Update

Add storage URLs to your Tickets table:

```prisma
// prisma/schema.prisma
model Ticket {
  id                String   @id @default(uuid())
  ticket_number     String   @unique
  // ... other fields
  photo_urls        Json?    // Array of photo URLs
  receipt_url       String?  // URL to receipt PDF
  // ... rest of fields
}
```

## Receipt PDF Generation

### Using PDFKit or jsPDF

```typescript
// lib/generateReceipt.ts
import PDFDocument from 'pdfkit'
import { supabase } from './supabase'

export async function generateReceipt(ticket: Ticket) {
  const doc = new PDFDocument()
  const chunks: Buffer[] = []

  doc.on('data', (chunk) => chunks.push(chunk))
  doc.on('end', async () => {
    const pdfBuffer = Buffer.concat(chunks)
    
    // Upload to storage
    const fileName = `receipts/${ticket.ticket_number}.pdf`
    const { data, error } = await supabase.storage
      .from('receipts')
      .upload(fileName, pdfBuffer, {
        contentType: 'application/pdf',
        upsert: true
      })

    if (!error) {
      // Update ticket with receipt URL
      // ... save to database
    }
  })

  // Add content to PDF
  doc.text(`Receipt #${ticket.ticket_number}`)
  doc.text(`Amount: $${ticket.total_amount}`)
  // ... more content

  doc.end()
}
```

## Storage Usage Estimates

### Per Ticket
- **Photos**: 2-5 photos × 500KB-1MB each = 1-5MB
- **Receipt**: 1 PDF × 100-200KB = 0.1-0.2MB
- **Total per ticket**: ~1-5MB

### Monthly Estimates
- **100 tickets/month**: ~100-500MB
- **500 tickets/month**: ~500MB-2.5GB
- **1000 tickets/month**: ~1-5GB
- **5000 tickets/month**: ~5-25GB

## Cost Optimization Tips

1. **Compress images** before upload (use browser compression or server-side)
2. **Delete old photos** after ticket completion (optional, based on retention policy)
3. **Use image optimization** (Cloudinary auto-optimizes, Supabase can use transformations)
4. **Set up lifecycle policies** to archive old files

## Security Best Practices

1. **Validate file types** - Only allow images (jpg, png, webp)
2. **Limit file size** - Max 5MB per photo
3. **Use private buckets** for receipts (require authentication)
4. **Sanitize filenames** - Remove special characters
5. **Rate limiting** - Prevent abuse on upload endpoint

## Example: Complete Upload Flow

```typescript
// app/api/tickets/[id]/photos/route.ts
import { NextRequest, NextResponse } from 'next/server'
import { supabase } from '@/lib/supabase'

export async function POST(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  try {
    const formData = await request.formData()
    const files = formData.getAll('photos') as File[]
    
    const uploadedUrls: string[] = []

    for (const file of files) {
      // Validate file
      if (!file.type.startsWith('image/')) {
        return NextResponse.json(
          { error: 'Only images allowed' },
          { status: 400 }
        )
      }

      if (file.size > 5 * 1024 * 1024) {
        return NextResponse.json(
          { error: 'File too large (max 5MB)' },
          { status: 400 }
        )
      }

      // Upload to storage
      const fileName = `${params.id}/${Date.now()}-${file.name}`
      const fileBuffer = await file.arrayBuffer()

      const { data, error } = await supabase.storage
        .from('ticket-photos')
        .upload(fileName, fileBuffer, {
          contentType: file.type,
          upsert: false
        })

      if (error) throw error

      const { data: { publicUrl } } = supabase.storage
        .from('ticket-photos')
        .getPublicUrl(data.path)

      uploadedUrls.push(publicUrl)
    }

    // Update ticket with photo URLs
    // ... update database

    return NextResponse.json({ urls: uploadedUrls })
  } catch (error) {
    return NextResponse.json(
      { error: 'Upload failed' },
      { status: 500 }
    )
  }
}
```

---

## Quick Reference

### Supabase Storage
- **Free**: 1GB storage, 2GB bandwidth/month
- **Pro**: $25/month - 100GB storage
- **Docs**: https://supabase.com/docs/guides/storage

### Cloudinary
- **Free**: 25GB storage, 25GB bandwidth/month
- **Pro**: $89/month - Advanced features
- **Docs**: https://cloudinary.com/documentation

### Vercel Blob
- **Free**: 1GB storage
- **Paid**: $0.15/GB/month
- **Docs**: https://vercel.com/docs/storage/vercel-blob
