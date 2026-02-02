# Genesis 1:28 - Full-Stack Setup Guide

## What's Inside

```
G128-fullstack/
├── src/
│   ├── App.jsx                  # Entry point (routes to MSPAModern)
│   ├── MSPAModern.jsx           # Main UI (your original 22k+ line app)
│   ├── main.jsx                 # React mount
│   ├── index.css                # Tailwind styles
│   ├── components/              # (Ready for future component splitting)
│   ├── hooks/
│   │   └── useData.js           # Custom hooks: useAuth, useTasks, useClock, useTeamTasks
│   └── lib/
│       ├── supabase.js          # Supabase client + ALL API helper functions
│       └── store.js             # Zustand global state (replaces 100+ useState)
├── supabase/
│   └── migrations/
│       └── 001_initial_schema.sql  # Complete database schema (run once)
├── .env.example                 # Environment variables template
├── vercel.json                  # Vercel deployment config
├── package.json                 # Dependencies
└── README.md                    # This file
```

---

## Setup Steps (15 minutes)

### Step 1: Create Supabase Account (Free)

1. Go to [supabase.com](https://supabase.com)
2. Sign up with GitHub
3. Click "New Project"
4. Choose a name (e.g., "genesis-128")
5. Set a database password (save this!)
6. Select region closest to you
7. Click "Create Project" - wait 2 minutes

### Step 2: Create Database Tables

1. In Supabase dashboard, go to **SQL Editor**
2. Click "New Query"
3. Open the file: `supabase/migrations/001_initial_schema.sql`
4. Copy ALL contents and paste into the SQL editor
5. Click **Run**
6. You should see "Success" - your tables are created

### Step 3: Create Storage Buckets

1. In Supabase dashboard, go to **Storage**
2. Click "New Bucket" and create these 3:
   - `screenshots` (Public: ON)
   - `documents` (Public: OFF)
   - `avatars` (Public: ON)

### Step 4: Get Your API Keys

1. Go to **Settings > API**
2. Copy the **Project URL** (looks like `https://abc123.supabase.co`)
3. Copy the **anon/public key** (long string starting with `eyJ...`)

### Step 5: Setup the Project

```bash
# Unzip the project
unzip G128-fullstack.zip
cd G128-fullstack

# Install dependencies
npm install

# Create environment file
cp .env.example .env

# Edit .env and paste your Supabase URL and key
# VITE_SUPABASE_URL=https://your-project.supabase.co
# VITE_SUPABASE_ANON_KEY=your-key-here

# Start development server
npm run dev
```

Open http://localhost:5173 - your app is running!

### Step 6: Create First Admin User

1. In Supabase dashboard, go to **Authentication > Users**
2. Click "Add User" > "Create User"
3. Enter: email: `daniel@tsl.com.sg`, password: your choice
4. After user is created, go to **Table Editor > profiles**
5. Find Daniel's row and change `role` to `super_admin`

### Step 7: Deploy to Vercel (Free)

```bash
# Install Vercel CLI
npm i -g vercel

# Deploy
vercel

# It will ask questions:
# - Set up and deploy? Y
# - Which scope? (select your account)
# - Link to existing project? N
# - Project name: genesis-128
# - Directory: ./
# - Override settings? N

# Add environment variables
vercel env add VITE_SUPABASE_URL
vercel env add VITE_SUPABASE_ANON_KEY

# Redeploy with env vars
vercel --prod
```

Your app is now live at: `https://genesis-128.vercel.app`

---

## What the Backend Provides

| Feature | Before (localStorage) | After (Supabase) |
|---------|----------------------|-------------------|
| Data persistence | Browser only | Cloud database |
| Multi-user | Fake/mock data | Real user accounts |
| Team tasks | Sample data | Live employee data |
| Authentication | Simple login form | Secure auth with sessions |
| File storage | Not possible | Screenshots & documents in cloud |
| Real-time sync | None | Live updates across devices |
| Audit log | In-memory only | Permanent record |
| Security | None (client-side) | Row-level security per user |

---

## Database Tables

| Table | Purpose |
|-------|---------|
| `profiles` | User accounts, roles, departments |
| `task_entries` | IETL daily task log |
| `clock_records` | Clock in/out attendance |
| `task_verbs` | Verb library (Review, Prepare, etc.) |
| `task_products` | Product library (Email, Invoice, etc.) |
| `audit_log` | Action history |
| `leave_requests` | Leave/time-off requests |
| `documents` | Uploaded file records |
| `notifications` | User notifications |

---

## How to Wire Supabase into MSPAModern.jsx

The original MSPAModern.jsx currently uses localStorage. To switch to Supabase:

### Replace localStorage calls:

```javascript
// BEFORE (localStorage):
const [ietlTaskEntries, setIetlTaskEntries] = useState(() => 
  getFromStorage('g128_tasks', [])
);

// AFTER (Supabase):
import { useTasks } from './hooks/useData';
const { tasks, loading } = useTasks(todayStr);
```

### Replace login:

```javascript
// BEFORE:
if (email === 'daniel@tsl.com.sg') { setIsLoggedIn(true); }

// AFTER:
import { signIn } from './lib/supabase';
const { data } = await signIn(email, password);
```

### Replace task CRUD:

```javascript
// BEFORE:
setIetlTaskEntries(prev => [...prev, newTask]);

// AFTER:
import { createTask } from './lib/supabase';
const saved = await createTask({ ...newTask, user_id: user.id });
```

The hooks in `useData.js` handle all of this, including real-time subscriptions.

---

## Next Steps

1. Connect Supabase (follow steps above)
2. Replace localStorage with Supabase hooks gradually
3. Split MSPAModern.jsx into smaller components (optional but recommended)
4. Add email notifications via Supabase Edge Functions
5. Add PWA support for mobile app experience

---

## Tech Stack

- **Frontend**: React 18 + Vite + Tailwind CSS
- **Backend**: Supabase (PostgreSQL + Auth + Storage + Realtime)
- **State**: Zustand (global state management)
- **Hosting**: Vercel (free tier)
- **AI**: Claude API (already integrated in frontend)

Total monthly cost: **$0** (all free tiers)
