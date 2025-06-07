# Practical 5: Implementing Cloud Bucket Storage with Supabase

## Table of Contents
- [Overview](#overview)
- [Understanding Cloud Storage](#understanding-cloud-storage)
- [Introduction to Supabase Storage](#introduction-to-supabase-storage)
- [Setting Up Supabase Storage](#setting-up-supabase-storage)
- [Part 1: Backend Implementation](#part-1-backend-implementation)
- [Part 2: Frontend Implementation](#part-2-frontend-implementation)
- [Part 3: Testing and Deployment](#part-3-testing-and-deployment)
- [Key Concepts](#key-concepts)
- [Resources](#resources)

## Overview

This practical upgrades our TikTok web application by migrating from local file storage to cloud storage using Supabase Storage. This enhancement improves our application's scalability, reliability, and provides better access control for user-uploaded content.

## Understanding Cloud Storage

### The Limitations of Local Storage

Our application previously stored files (videos and thumbnails) in a local uploads directory. This approach has several limitations:

1. **Disk Space:** Limited server disk space that fills up quickly with video files
2. **Scaling Issues:** Files on one server aren't available to others in multi-server deployments
3. **Reliability Concerns:** Server crashes or redeployments can result in lost files
4. **No CDN Benefits:** Local files don't leverage Content Delivery Networks for faster global access
5. **Backup Challenges:** No automatic backup system for local files

### Benefits of Cloud Storage

Cloud storage addresses these issues by providing:

1. **Scalability:** Virtually unlimited storage capacity
2. **Reliability:** Built-in redundancy and backup systems
3. **Performance:** Global CDN distribution for faster worldwide access
4. **Security:** Advanced permissions and access control systems
5. **Cost-effectiveness:** Pay-per-use model without infrastructure management

### Cloud Storage Flow in Web Applications


1. **Frontend Upload:** User selects file through browser
2. **Direct Upload:** File uploads directly to cloud storage (bypassing server)
3. **Metadata Storage:** Server stores file metadata (URLs) in database
4. **Access Control:** Cloud provider manages file permissions
5. **Content Serving:** Files served directly from cloud provider's CDN

## Introduction to Supabase Storage

### What is Supabase?

Supabase is an open-source Firebase alternative providing:
- Database (PostgreSQL)
- Authentication
- **Storage**
- Real-time subscriptions
- Edge Functions

### Supabase Storage Features

- File storage with customizable access controls
- Automatic content delivery through global CDN
- Fine-grained security rules
- Easy integration with existing Supabase projects

### Storage Organization

Supabase Storage organizes files in "buckets" with configurable security rules:
- **Public:** Anyone can access files
- **Private:** Only authenticated users can access files
- **Custom:** Access controlled by custom security policies

## Setting Up Supabase Storage

### Step 1: Create Supabase Account and Project

1. Visit [supabase.com](https://supabase.com) and sign up/login
2. Click "New Project" and enter project name (e.g., "tiktok")
3. Choose a strong database password (store securely)
4. Select region closest to target audience
5. Click "Create new project" and wait for creation


### Step 2: Create Storage Buckets

1. Navigate to "Storage" in the Supabase dashboard sidebar
2. Click "Create Bucket"
3. Create "videos" bucket with "Public" access level
4. Create "thumbnails" bucket with "Public" access level

### Step 3: Set Up Storage Policies

#### For Videos Bucket:

1. **Upload Policy for Authenticated Users:**
   - Policy name: "Authenticated users can upload videos"
   - Allowed Operation: All
   - Target roles: authenticated
   
2. **Public Access Policy:**
   - Policy name: "Public can view videos"
   - Allowed Operation: SELECT
   - Target roles: anon, authenticated

#### For Thumbnails Bucket:
Repeat similar policies for thumbnails bucket.

## Part 1: Backend Implementation

**Reference Repositories:**
- [TikTok Frontend](https://github.com/syangche/TikTok_Frontend.git)
- [TikTok Server](https://github.com/syangche/TikTok_Server.git)

### Step 1: Install Supabase SDK

```bash
cd server
npm install @supabase/supabase-js
```

### Step 2: Create Supabase Client Configuration

Create `src/lib/supabase.js`:

```javascript
const { createClient } = require('@supabase/supabase-js');

const supabaseUrl = process.env.SUPABASE_URL;
const supabaseKey = process.env.SUPABASE_SERVICE_KEY;

if (!supabaseUrl || !supabaseKey) {
  console.error('Missing Supabase credentials. Check your environment variables.');
}

const supabase = createClient(supabaseUrl, supabaseKey);

module.exports = supabase;
```

### Step 3: Update Environment Variables

Add to `.env` file:

```env
SUPABASE_URL=https://your-project-id.supabase.co
SUPABASE_SERVICE_KEY=your-service-key
SUPABASE_PUBLIC_KEY=your-public-key
SUPABASE_STORAGE_URL=https://your-project-id.supabase.co/storage/v1
```

**Finding Supabase Credentials:**
- Go to Settings > API in Supabase dashboard
- Copy "Project URL" for SUPABASE_URL
- Use "service_role" API key for SUPABASE_SERVICE_KEY
- Use "anon" public API key for SUPABASE_PUBLIC_KEY

### Step 4: Create Storage Service

Create `src/services/storageService.js` with file upload, download, and deletion functionality.

### Step 5: Update Video Controller

Update `videoController.js` to use Supabase for:
- `createVideo` function
- `deleteVideo` function

### Step 6: Update Prisma Schema

Add storage path fields to Video model:

```prisma
model Video {
  id                    Int       @id @default(autoincrement())
  userId                Int       @map("user_id")
  caption               String?
  videoUrl              String    @map("video_url")
  thumbnailUrl          String?   @map("thumbnail_url")
  audioName             String?   @map("audio_name")
  videoStoragePath      String?   @map("video_storage_path")
  thumbnailStoragePath  String?   @map("thumbnail_storage_path")
  views                 Int       @default(0)
  createdAt             DateTime  @default(now()) @map("created_at")
  updatedAt             DateTime  @default(now()) @updatedAt @map("updated_at")

  comments Comment[]
  likes    VideoLike[]
  user     User       @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@map("videos")
}
```

### Step 7: Create Migration Script

Create `scripts/migrateVideosToSupabase.js` for migrating existing local videos to Supabase.

## Part 2: Frontend Implementation

### Step 1: Install Supabase Client

```bash
cd tiktok_frontend
npm install @supabase/supabase-js
```

### Step 2: Create Supabase Client Configuration

Create `src/lib/supabase.js`:

```javascript
import { createClient } from '@supabase/supabase-js';

const supabaseUrl = process.env.NEXT_PUBLIC_SUPABASE_URL;
const supabaseKey = process.env.NEXT_PUBLIC_SUPABASE_PUBLIC_KEY;

if (!supabaseUrl || !supabaseKey) {
  console.warn('Missing Supabase credentials. Check your environment variables.');
}

const supabase = createClient(supabaseUrl, supabaseKey);

export default supabase;
```

### Step 3: Update Environment Variables

Create/update `.env.local`:

```env
NEXT_PUBLIC_SUPABASE_URL=https://your-project-id.supabase.co
NEXT_PUBLIC_SUPABASE_PUBLIC_KEY=your-public-key
```

### Step 4: Update Upload Service

Update `src/services/uploadService.js` to handle direct uploads to Supabase.

### Step 5: Update Upload Page Component

Modify `src/app/upload/page.jsx` to use direct Supabase uploads.

### Step 6: Update VideoCard Component

Update `src/components/ui/VideoCard.jsx` to handle Supabase URLs correctly:
- Update `getFullVideoUrl` function
- Ensure proper URL handling for Supabase storage

## Part 3: Testing and Deployment

### Step 1: Run Migration Script

If you have existing videos to migrate:

```bash
cd server
node scripts/migrateVideosToSupabase.js
```

### Step 2: Test Upload Functionality

1. Start both frontend and backend servers
2. Test video upload through the web interface
3. Verify files appear in Supabase Storage dashboard
4. Confirm videos play correctly from Supabase URLs

### Step 3: Test Access Control

1. Test public access to videos
2. Verify authenticated user upload permissions
3. Check file deletion functionality

### Step 4: Clean Up Local Storage

Once migration is successful:
1. Verify all videos play from Supabase URLs
2. Back up local uploads directory
3. Remove local file serving code

## Key Concepts

### Cloud Storage Architecture
- **Bucket Organization:** Logical containers for organizing files
- **CDN Integration:** Global content delivery for improved performance
- **Access Control:** Fine-grained permissions for file security

### Direct Upload Pattern
- **Client-side Upload:** Files go directly from browser to cloud storage
- **Reduced Server Load:** Server only handles metadata, not file transfer
- **Better Performance:** Leverages CDN for faster uploads

### Storage Security
- **Public vs Private Buckets:** Different access levels for different content types
- **Policy-based Access:** Flexible rules for file access control
- **Authentication Integration:** Seamless integration with user authentication

### Migration Strategies
- **Gradual Migration:** Moving existing content without downtime
- **Dual Storage:** Maintaining both local and cloud storage during transition
- **Data Integrity:** Ensuring no data loss during migration

## Benefits Achieved

### Scalability Improvements
- Unlimited storage capacity
- No server disk space constraints
- Better handling of traffic spikes

### Performance Enhancements
- Global CDN distribution
- Faster file access worldwide
- Reduced server load

### Reliability Features
- Built-in backup and redundancy
- High availability guarantee
- Disaster recovery capabilities

### Security Enhancements
- Advanced access control
- Secure file serving
- Authentication integration

## Resources

- [Supabase Storage Documentation](https://supabase.com/docs/guides/storage)
- [Supabase JavaScript Client](https://supabase.com/docs/reference/javascript)
- [File Upload Best Practices](https://developer.mozilla.org/en-US/docs/Web/API/File/Using_files_from_web_applications)
- [Content Delivery Networks Explained](https://developer.mozilla.org/en-US/docs/Glossary/CDN)
- [Video Hosting Considerations](https://web.dev/fast/#optimize-your-videos)