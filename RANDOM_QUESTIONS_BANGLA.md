# SchemaX প্রজেক্ট - র‍্যান্ডম প্রশ্নের উত্তর (বাংলায়)

## ১. Front end এর জন্য কি কি use করেছো?

### উত্তর:

আমরা **React-based Modern Web Stack** ব্যবহার করেছি:

#### Core Technologies:
- **React 18.3.1** - Main UI library
- **TypeScript** - Type safety এর জন্য
- **Vite** - Fast development server এবং build tool

#### UI Framework & Styling:
- **Tailwind CSS** - Utility-first CSS framework
- **shadcn/ui** - Pre-built accessible components
- **Radix UI** - Headless UI components (40+ components)
  - Dialog, Dropdown, Select, Tabs, Toast, etc.
- **Lucide React** - Modern icon library

#### State Management & Data Fetching:
- **React Query (@tanstack/react-query)** - Server state management
- **React Hook Form** - Form handling
- **Zod** - Schema validation

#### Routing:
- **React Router DOM v6** - Client-side routing

#### Other Libraries:
- **date-fns** - Date manipulation
- **recharts** - Charts এবং graphs
- **sonner** - Toast notifications
- **next-themes** - Dark/Light mode

**কোথায় দেখবেন:** `package.json` ফাইলের `dependencies` section

---

## ২. Backend এর জন্য কি কি use করেছো?

### উত্তর:

আমরা **Supabase** ব্যবহার করেছি যা একটি **complete Backend-as-a-Service (BaaS)**:

#### Database:
- **PostgreSQL** - Relational database
- **Supabase Client (@supabase/supabase-js)** - Database connection library

#### Backend Features:
1. **Database (PostgreSQL)**
   - 10টি tables
   - Foreign key relationships
   - Row Level Security (RLS)

2. **Authentication**
   - Email/Password authentication
   - Session management
   - JWT tokens

3. **Storage**
   - File upload (avatars, documents)
   - Public এবং private buckets

4. **Edge Functions**
   - Serverless functions
   - `create-booking`, `update-booking-status`, etc.

5. **Realtime**
   - Live notifications
   - Real-time data updates

**কোথায় দেখবেন:** 
- `src/integrations/supabase/client.ts` - Connection
- `supabase/migrations/` - Database schema
- `supabase/functions/` - Edge functions

---

## ৩. Supabase কেন use করেছো? SQL দিয়ে না করে PostgreSQL কেন use করেছো?

### উত্তর:

এই প্রশ্নটি একটু confusing! আসলে:

#### PostgreSQL হলো SQL-based Database!
- **PostgreSQL** একটি SQL database
- SQL হলো query language, PostgreSQL হলো database system
- PostgreSQL SQL ব্যবহার করে data query করে

#### Supabase কেন বেছে নিয়েছি:

**১. সহজ Setup:**
```typescript
// শুধু এই কোড লিখলেই database ready!
import { createClient } from '@supabase/supabase-js';
const supabase = createClient(URL, KEY);
```

**২. Built-in Features:**
- ✅ Authentication system (নিজে বানাতে হয় না)
- ✅ File storage (AWS S3 এর মতো)
- ✅ Real-time subscriptions
- ✅ Row Level Security (automatic security)
- ✅ Auto-generated REST API

**৩. Free Hosting:**
- Database cloud এ hosted
- Server maintain করতে হয় না
- Automatic backups

**৪. Developer Experience:**
```typescript
// Traditional SQL:
const result = await db.query('SELECT * FROM profiles WHERE id = $1', [userId]);

// Supabase (সহজ):
const { data } = await supabase.from('profiles').select('*').eq('id', userId);
```

**৫. TypeScript Support:**
- Auto-generated types (`types.ts`)
- Type-safe queries

#### PostgreSQL কেন (MySQL বা MongoDB না):
- **Relations** - Foreign keys, joins সহজ
- **ACID compliance** - Data integrity
- **JSON support** - NoSQL এর মতো flexibility
- **Powerful queries** - Complex queries সহজে করা যায়

---

## ৪. Query গুলো কোথায় লিখেছো?

### উত্তর:

Query গুলো **২ জায়গায়** লেখা আছে:

### A) Frontend Code এ (TypeScript):
প্রতিটি page/component এ যেখানে data দরকার:

**উদাহরণ ১:** `src/pages/Dashboard.tsx` (লাইন ৪৬৭)
```typescript
const { data, error } = await supabase
  .from('profiles')
  .select('*')
  .eq('id', userId)
  .single();
```

**উদাহরণ ২:** `src/pages/CreateService.tsx` (লাইন ১০০)
```typescript
await supabase.from('services').insert({
  title: formData.title,
  price_cents: Math.round(parseFloat(formData.price) * 100)
});
```

### B) Database Migration Files এ (SQL):
Database structure তৈরির জন্য:

**লোকেশন:** `supabase/migrations/20250831174403_2e165ee6-72b0-4aa7-b0ed-215ff9af0d2e.sql`

```sql
CREATE TABLE public.profiles (
  id uuid primary key,
  role public.user_role not null,
  display_name text not null,
  avatar_url text,
  bio text,
  location text,
  hourly_rate integer,
  is_public boolean not null default false,
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now()
);
```

### সংক্ষেপে:
- **SQL queries** → Migration files এ (database structure)
- **Data queries** → React components এ (data fetch/insert)

---

## ৫. Table গুলো সব একসাথে কীভাবে add করেছো?

### উত্তর:

Tables **SQL Migration file** দিয়ে একসাথে তৈরি করা হয়েছে।

### Process:

**১. Migration File তৈরি:**
```bash
# Supabase CLI দিয়ে
supabase migration new initial_schema
```

**২. SQL Code লেখা:**
ফাইল: `supabase/migrations/20250831174403_2e165ee6-72b0-4aa7-b0ed-215ff9af0d2e.sql`

```sql
-- Step 1: Enums তৈরি
CREATE TYPE public.user_role AS ENUM ('freelancer', 'client');
CREATE TYPE public.booking_status AS ENUM ('pending','confirmed','completed','canceled');

-- Step 2: Tables তৈরি (ক্রমানুসারে)
CREATE TABLE public.profiles (...);
CREATE TABLE public.skills (...);
CREATE TABLE public.services (...);
CREATE TABLE public.bookings (...);
-- ... আরো tables

-- Step 3: Indexes তৈরি
CREATE INDEX idx_profiles_public_role ON public.profiles(is_public, role);

-- Step 4: Row Level Security (RLS) enable
ALTER TABLE public.profiles ENABLE ROW LEVEL SECURITY;

-- Step 5: Security Policies
CREATE POLICY "Users can view own profile" ON public.profiles ...;
```

**৩. Migration Run করা:**
```bash
supabase db push
```

### কেন এভাবে করা হয়েছে:
- ✅ **Version Control** - Git এ track করা যায়
- ✅ **Reproducible** - যেকোনো environment এ same setup
- ✅ **Rollback** - ভুল হলে undo করা যায়
- ✅ **Team Collaboration** - সবাই same schema পায়

### Table তৈরির Order:
```
1. Enums (user_role, booking_status, etc.)
2. Helper Functions (update_updated_at_column)
3. Base Tables (profiles, skills)
4. Dependent Tables (services → profiles এর উপর depend করে)
5. Junction Tables (freelancer_skills)
6. Indexes
7. RLS Policies
8. Storage Buckets
```

---

## ৬. Storage ফাঁকা কেন তোমাদের?

### উত্তর:

Storage ফাঁকা **নয়**, তবে **locally empty** দেখাতে পারে কারণ:

### Storage কোথায় আছে:
Storage **Supabase Cloud** এ আছে, local project folder এ নয়।

### Storage Buckets:
আমাদের **২টি bucket** আছে:

**১. `avatars` Bucket (Public):**
- User profile pictures
- Public access
- Anyone দেখতে পারে

**২. `documents` Bucket (Private):**
- Verification documents
- Private access
- শুধু owner দেখতে পারে

### কোথায় দেখবেন:

**A) Code এ:**
`supabase/migrations/20250831174403_2e165ee6-72b0-4aa7-b0ed-215ff9af0d2e.sql` (লাইন ২৬০-২৬৩)

```sql
INSERT INTO storage.buckets (id, name, public) 
VALUES ('avatars','avatars', true);

INSERT INTO storage.buckets (id, name, public) 
VALUES ('documents','documents', false);
```

**B) Supabase Dashboard এ:**
1. https://supabase.com এ যান
2. Project select করুন
3. Storage → Buckets দেখুন

### Storage ব্যবহার:
`src/pages/Profile.tsx` (লাইন ১৮৯-১৯৯)

```typescript
// Avatar upload
const { error } = await supabase.storage
  .from('avatars')
  .upload(filePath, file);

// Public URL পাওয়া
const { data } = supabase.storage
  .from('avatars')
  .getPublicUrl(filePath);
```

### কেন ফাঁকা দেখায়:
- Local development এ files cloud এ upload হয়
- Local folder এ files থাকে না
- Supabase dashboard এ গেলে files দেখা যাবে

---

## ৭. Storage এর পর থেকে যেগুলো আছে বলতে পারে, ওগুলো কী?

### উত্তর:

Storage এর পর **Supabase Dashboard** এ যা যা আছে:

### ১. **SQL Editor**
- Custom SQL queries লেখার জায়গা
- Database explore করা
- Manual data insert/update

### ২. **Database**
- Tables দেখা
- Table structure
- Data browse করা
- Foreign keys, indexes

### ৩. **Authentication**
- Users list
- Email templates
- Auth providers (Google, GitHub, etc.)
- JWT settings

### ৪. **Edge Functions**
- Serverless functions
- API endpoints
- Custom backend logic

আমাদের functions:
```
- create-booking
- update-booking-status
- send-notification
- search-freelancers
```

### ৫. **Realtime**
- Live data subscriptions
- WebSocket connections
- Broadcast channels

### ৬. **Logs**
- API requests
- Error logs
- Performance metrics

### ৭. **API**
- Auto-generated REST API
- API documentation
- API keys

### ৮. **Project Settings**
- Database connection string
- API keys
- Custom domains
- Billing

### ৯. **Table Editor**
- Visual table editing
- Add/remove columns
- Change data types

### ১০. **Reports**
- Database size
- API usage
- Storage usage
- Active users

---

## ৮. Project Overview এ কি কি আছে?

### উত্তর:

Supabase Project Overview এ যা দেখা যায়:

### A) **Project Information:**
- Project Name: `schemax-skill-connect`
- Project ID: `zdmymcefzmbckohzupgt`
- Region: (যেখানে hosted)
- Created date

### B) **Quick Stats:**
- Database size
- API requests (last 24h)
- Active users
- Storage used

### C) **Connection Strings:**
```
Database URL: postgresql://...
API URL: https://zdmymcefzmbckohzupgt.supabase.co
```

### D) **API Keys:**
```typescript
// anon/public key (frontend এ use করা হয়)
const SUPABASE_PUBLISHABLE_KEY = "eyJhbGc...";

// service_role key (backend এ use করা হয়)
const SUPABASE_SERVICE_KEY = "secret...";
```

### E) **Recent Activity:**
- Latest migrations
- Recent API calls
- Error logs

### F) **Quick Actions:**
- SQL Editor
- Table Editor
- API Documentation

### G) **Resources:**
- Documentation links
- Community support
- GitHub repository

---

## ৯. নিচে যে link গুলো আছে ওগুলো বলতে পারে, এগুলো কিসের link?

### উত্তর:

Supabase Dashboard এর নিচের links:

### A) **API Documentation Links:**
```
https://zdmymcefzmbckohzupgt.supabase.co/rest/v1/
```
- Auto-generated REST API
- প্রতিটি table এর জন্য endpoint
- Example: `/rest/v1/profiles`, `/rest/v1/services`

### B) **GraphQL Endpoint:**
```
https://zdmymcefzmbckohzupgt.supabase.co/graphql/v1
```
- GraphQL API (optional)

### C) **Realtime Endpoint:**
```
wss://zdmymcefzmbckohzupgt.supabase.co/realtime/v1
```
- WebSocket connection
- Live data updates

### D) **Storage URL:**
```
https://zdmymcefzmbckohzupgt.supabase.co/storage/v1
```
- File upload/download
- Public file URLs

### E) **Auth URL:**
```
https://zdmymcefzmbckohzupgt.supabase.co/auth/v1
```
- Authentication endpoints
- Login, signup, password reset

### F) **Connection String:**
```
postgresql://postgres:[password]@db.zdmymcefzmbckohzupgt.supabase.co:5432/postgres
```
- Direct database connection
- psql, pgAdmin দিয়ে connect করা যায়

### কোথায় ব্যবহার হয়:
`src/integrations/supabase/client.ts`
```typescript
const SUPABASE_URL = "https://zdmymcefzmbckohzupgt.supabase.co";
```

---

## ১০. আমি যদি query change করে দেই তাহলে এইটা দিলে এইটা কি হবে?

### উত্তর:

Query change করলে **কোথায় change করছেন** তার উপর depend করে:

### Case 1: Frontend Code এ Query Change

**আগে:**
```typescript
const { data } = await supabase
  .from('profiles')
  .select('*')
  .eq('id', userId);
```

**পরে:**
```typescript
const { data } = await supabase
  .from('profiles')
  .select('display_name, bio')  // শুধু 2টি column
  .eq('role', 'freelancer');    // different filter
```

**কি হবে:**
- ✅ Different data আসবে
- ✅ Database structure change হবে না
- ✅ শুধু যে data fetch হচ্ছে তা change হবে
- ⚠️ যদি column না থাকে তাহলে error

### Case 2: Migration File এ Query Change

**আগে:**
```sql
CREATE TABLE profiles (
  id uuid primary key,
  display_name text not null
);
```

**পরে:**
```sql
CREATE TABLE profiles (
  id uuid primary key,
  display_name text not null,
  phone_number text  -- নতুন column
);
```

**কি হবে:**
- ⚠️ **নতুন migration** run করতে হবে
- ✅ Database structure change হবে
- ✅ নতুন column add হবে
- ⚠️ Existing data এ `phone_number` NULL হবে

### Case 3: Wrong Query

**ভুল Query:**
```typescript
const { data } = await supabase
  .from('profiles')
  .select('wrong_column')  // এই column নেই
  .eq('id', userId);
```

**কি হবে:**
```javascript
{
  data: null,
  error: {
    message: "column 'wrong_column' does not exist",
    code: "42703"
  }
}
```

### Best Practice:
```typescript
// Always check error
const { data, error } = await supabase.from('profiles').select('*');

if (error) {
  console.error('Query failed:', error.message);
  return;
}

// Use data
console.log(data);
```

---

## ১১. ER Diagram এর কার সাথে কার relation আছে?

### উত্তর:

আমাদের database এর **Entity Relationship Diagram**:

```
┌─────────────┐
│  profiles   │ (Main table)
│  - id (PK)  │
│  - role     │
│  - name     │
└──────┬──────┘
       │
       ├──────────────────────────────────┐
       │                                  │
       ▼                                  ▼
┌─────────────┐                    ┌──────────────┐
│  services   │                    │  bookings    │
│  - id (PK)  │                    │  - id (PK)   │
│  - freelancer_id (FK) ────────┐  │  - client_id (FK) ──┐
│  - title    │                 │  │  - freelancer_id (FK)│
│  - price    │                 │  │  - service_id (FK) ──┤
└──────┬──────┘                 │  └──────┬───────┘       │
       │                        │         │               │
       │                        │         │               │
       ▼                        │         ▼               │
┌──────────────┐                │  ┌─────────────┐       │
│ categories   │                │  │   reviews   │       │
│  - id (PK)   │                │  │  - id (PK)  │       │
│  - name      │                │  │  - booking_id (FK) ─┘
└──────┬───────┘                │  │  - client_id (FK)
       │                        │  │  - freelancer_id (FK)
       ▼                        │  │  - rating
┌──────────────┐                │  └─────────────┘
│subcategories │                │
│  - id (PK)   │                │
│  - category_id (FK)           │
└──────────────┘                │
                                │
       ┌────────────────────────┘
       │
       ▼
┌──────────────────┐
│availability_slots│
│  - id (PK)       │
│  - freelancer_id (FK)
│  - start_time    │
│  - end_time      │
└──────────────────┘

┌─────────────────┐
│freelancer_skills│ (Junction Table)
│  - freelancer_id (FK) ──→ profiles
│  - skill_id (FK) ──────→ skills
└─────────────────┘

┌──────────────┐
│    skills    │
│  - id (PK)   │
│  - name      │
└──────────────┘

┌──────────────┐
│notifications │
│  - id (PK)   │
│  - user_id   │ (no FK, flexible)
│  - type      │
└──────────────┘

┌──────────────┐
│verifications │
│  - id (PK)   │
│  - freelancer_id (FK) ──→ profiles
│  - status    │
└──────────────┘
```

### Relations বিস্তারিত:

#### 1. **profiles ↔ services** (One-to-Many)
```sql
services.freelancer_id → profiles.id
```
- একজন freelancer এর অনেক services থাকতে পারে
- একটি service একজন freelancer এর

#### 2. **profiles ↔ bookings** (One-to-Many, দুইবার)
```sql
bookings.client_id → profiles.id
bookings.freelancer_id → profiles.id
```
- একজন client অনেক booking করতে পারে
- একজন freelancer অনেক booking পেতে পারে

#### 3. **services ↔ bookings** (One-to-Many)
```sql
bookings.service_id → services.id
```
- একটি service এর অনেক booking হতে পারে

#### 4. **bookings ↔ reviews** (One-to-One)
```sql
reviews.booking_id → bookings.id
```
- একটি booking এর একটি review

#### 5. **profiles ↔ skills** (Many-to-Many)
```sql
freelancer_skills.freelancer_id → profiles.id
freelancer_skills.skill_id → skills.id
```
- একজন freelancer এর অনেক skills
- একটি skill অনেক freelancer এর

#### 6. **categories ↔ subcategories** (One-to-Many)
```sql
subcategories.category_id → categories.id
```
- একটি category এর অনেক subcategories

#### 7. **profiles ↔ availability_slots** (One-to-Many)
```sql
availability_slots.freelancer_id → profiles.id
```
- একজন freelancer এর অনেক time slots

#### 8. **profiles ↔ verifications** (One-to-Many)
```sql
verifications.freelancer_id → profiles.id
```
- একজন freelancer এর verification records

### Cascade Rules:

**ON DELETE CASCADE:**
- Profile delete হলে → সব services, bookings, reviews delete
- Service delete হলে → booking এ `service_id` NULL হয়ে যায়

**কোথায় দেখবেন:**
`supabase/migrations/20250831174403_2e165ee6-72b0-4aa7-b0ed-215ff9af0d2e.sql`

---

## ১২. Table কীভাবে add করবো?

### উত্তর:

নতুন table add করার **3টি উপায়**:

### উপায় ১: Migration File দিয়ে (Recommended)

**Step 1: Migration তৈরি**
```bash
cd /Users/mustafafahim/Downloads/schemax-skill-connect
supabase migration new add_messages_table
```

**Step 2: SQL লেখা**
ফাইল: `supabase/migrations/[timestamp]_add_messages_table.sql`

```sql
-- নতুন table তৈরি
CREATE TABLE IF NOT EXISTS public.messages (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  sender_id uuid NOT NULL REFERENCES public.profiles(id) ON DELETE CASCADE,
  receiver_id uuid NOT NULL REFERENCES public.profiles(id) ON DELETE CASCADE,
  content text NOT NULL,
  read boolean NOT NULL DEFAULT false,
  created_at timestamptz NOT NULL DEFAULT now()
);

-- Index তৈরি
CREATE INDEX idx_messages_receiver ON public.messages(receiver_id, read);

-- RLS enable
ALTER TABLE public.messages ENABLE ROW LEVEL SECURITY;

-- Security policy
CREATE POLICY "Users can view own messages" ON public.messages
FOR SELECT USING (
  auth.uid() = sender_id OR auth.uid() = receiver_id
);

CREATE POLICY "Users can send messages" ON public.messages
FOR INSERT WITH CHECK (auth.uid() = sender_id);
```

**Step 3: Migration run**
```bash
supabase db push
```

### উপায় ২: Supabase Dashboard দিয়ে

**Step 1:** Supabase Dashboard → Table Editor → New Table

**Step 2:** Form fill করুন:
- Table name: `messages`
- Columns add করুন:
  - `id` (uuid, primary key)
  - `sender_id` (uuid, foreign key → profiles)
  - `receiver_id` (uuid, foreign key → profiles)
  - `content` (text)
  - `read` (boolean, default: false)
  - `created_at` (timestamp)

**Step 3:** Save

### উপায় ৩: SQL Editor দিয়ে

**Step 1:** Dashboard → SQL Editor

**Step 2:** SQL paste করুন (উপরের মতো)

**Step 3:** Run

### TypeScript Types Update:

Table add করার পর types update করতে হবে:

```bash
# Types generate করুন
supabase gen types typescript --local > src/integrations/supabase/types.ts
```

তারপর code এ use করুন:
```typescript
const { data } = await supabase
  .from('messages')  // Auto-complete পাবেন
  .select('*');
```

---

## ১৩. Query গুলার মধ্যে exactly কোথায় change করে database এ change হবে?

### উত্তর:

Database এ change হয় **শুধুমাত্র** যখন **write operations** করা হয়:

### ১. INSERT Query (নতুন data add)

```typescript
const { error } = await supabase
  .from('profiles')
  .insert({                    // ← এখানে change হয়
    id: userId,
    display_name: 'John Doe',
    role: 'freelancer'
  });
```

**কি হয়:**
- `profiles` table এ নতুন row add হয়
- Database এ permanently save হয়

**কোথায় আছে:**
- `src/pages/ProfileSetup.tsx` (লাইন ৬৬)
- `src/pages/CreateService.tsx` (লাইন ১০০)

---

### ২. UPDATE Query (existing data change)

```typescript
const { error } = await supabase
  .from('services')
  .update({                    // ← এখানে change হয়
    is_active: false,
    title: 'New Title'
  })
  .eq('id', serviceId);        // ← কোন row change হবে
```

**কি হয়:**
- Existing row এর data update হয়
- শুধু specified columns change হয়

**কোথায় আছে:**
- `src/pages/Dashboard.tsx` (লাইন ২৬৫)
- `src/pages/Profile.tsx` (লাইন ১৫৪)

---

### ৩. DELETE Query (data মুছে ফেলা)

```typescript
const { error } = await supabase
  .from('services')
  .delete()                    // ← এখানে change হয়
  .eq('id', serviceId);
```

**কি হয়:**
- Row permanently delete হয়
- Cascade rules অনুযায়ী related data ও delete হতে পারে

**কোথায় আছে:**
- `src/pages/Dashboard.tsx` (লাইন ২৯৭)
- `src/pages/Availability.tsx` (লাইন ১৪৪)

---

### ৪. UPSERT Query (insert or update)

```typescript
const { error } = await supabase
  .from('profiles')
  .upsert({                    // ← এখানে change হয়
    id: userId,
    display_name: 'Updated Name'
  });
```

**কি হয়:**
- যদি row থাকে → update
- যদি না থাকে → insert

---

### যেগুলো Database Change করে না:

#### SELECT Query (শুধু read)
```typescript
const { data } = await supabase
  .from('profiles')
  .select('*');                // ← কোনো change হয় না
```
- শুধু data fetch করে
- Database unchanged থাকে

---

### Summary Table:

| Query Type | Database Change | Example |
|-----------|----------------|---------|
| `.select()` | ❌ No | Data পড়া |
| `.insert()` | ✅ Yes | নতুন row add |
| `.update()` | ✅ Yes | Existing row change |
| `.delete()` | ✅ Yes | Row মুছে ফেলা |
| `.upsert()` | ✅ Yes | Insert or update |

---

## ১৪. যখন কোনো কিছু order করবো তখন database এ change কোথায় আসবে?

### উত্তর:

যখন একটি **booking/order** করা হয়, তখন database এ **multiple changes** হয়:

### Booking Process Flow:

#### Step 1: User Service Book করে
**ফাইল:** `src/pages/BookService.tsx` (লাইন ১২৩-১৩১)

```typescript
const { data, error } = await supabase.functions.invoke('create-booking', {
  body: {
    freelancer_id: service.freelancer_id,
    service_id: service.id,
    start_time: startDateTime.toISOString(),
    end_time: endDateTime.toISOString(),
    notes: notes || null,
  },
});
```

#### Step 2: Edge Function Execute হয়
**ফাইল:** `supabase/functions/create-booking/index.ts`

```typescript
// 1. bookings table এ নতুন row insert
const { data: booking, error } = await supabase
  .from('bookings')
  .insert({
    client_id: userId,
    freelancer_id: body.freelancer_id,
    service_id: body.service_id,
    start_time: body.start_time,
    end_time: body.end_time,
    status: 'pending',
    payment_status: 'unpaid',
    notes: body.notes
  })
  .select()
  .single();
```

#### Step 3: Related Tables Update

**A) availability_slots table:**
```typescript
// Time slot booked mark করা
await supabase
  .from('availability_slots')
  .update({ is_booked: true })
  .eq('freelancer_id', freelancerId)
  .gte('start_time', startTime)
  .lte('end_time', endTime);
```

**B) notifications table:**
```typescript
// Freelancer কে notification পাঠানো
await supabase
  .from('notifications')
  .insert({
    user_id: freelancerId,
    type: 'new_booking',
    payload: {
      booking_id: booking.id,
      client_name: clientName,
      service_title: serviceTitle
    },
    read: false
  });
```

### Database Changes Summary:

```
Order করার সময় যা হয়:
┌────────────────────────────────────────┐
│ 1. bookings table                      │
│    → নতুন booking row insert          │
│    → status: 'pending'                 │
│    → payment_status: 'unpaid'          │
└────────────────────────────────────────┘
           ↓
┌────────────────────────────────────────┐
│ 2. availability_slots table            │
│    → is_booked: true (update)          │
└────────────────────────────────────────┘
           ↓
┌────────────────────────────────────────┐
│ 3. notifications table                 │
│    → Freelancer notification (insert)  │
└────────────────────────────────────────┘
```

### Payment করার পর:

```typescript
// bookings table update
await supabase
  .from('bookings')
  .update({
    payment_status: 'paid',
    status: 'confirmed',
    stripe_session_id: sessionId
  })
  .eq('id', bookingId);
```

### Booking Complete হলে:

```typescript
// 1. bookings table update
await supabase
  .from('bookings')
  .update({ status: 'completed' })
  .eq('id', bookingId);

// 2. reviews table এ review add করা যায়
await supabase
  .from('reviews')
  .insert({
    booking_id: bookingId,
    client_id: clientId,
    freelancer_id: freelancerId,
    rating: 5,
    comment: 'Great service!'
  });
```

### Real-time Updates:

Booking হওয়ার সাথে সাথে:
```typescript
// Realtime subscription
supabase
  .channel('bookings')
  .on('postgres_changes', {
    event: 'INSERT',
    schema: 'public',
    table: 'bookings'
  }, (payload) => {
    console.log('New booking:', payload.new);
    // UI automatically update হয়
  })
  .subscribe();
```

---

## সংক্ষিপ্ত উত্তর (Quick Reference)

1. **Frontend:** React + TypeScript + Tailwind + shadcn/ui
2. **Backend:** Supabase (PostgreSQL + Auth + Storage + Functions)
3. **Supabase কেন:** সহজ setup, built-in features, free hosting
4. **Query কোথায়:** Frontend code এ + Migration files এ
5. **Tables add:** Migration file দিয়ে একসাথে
6. **Storage:** Cloud এ আছে (avatars, documents buckets)
7. **Storage পরে:** SQL Editor, Auth, Functions, Realtime, etc.
8. **Project Overview:** Stats, API keys, connection strings
9. **Links:** REST API, GraphQL, Realtime, Storage URLs
10. **Query change:** Frontend এ change করলে শুধু data আলাদা, migration এ করলে structure change
11. **ER Diagram:** profiles → services, bookings, reviews (foreign keys)
12. **Table add:** Migration file বা Dashboard দিয়ে
13. **Database change:** `.insert()`, `.update()`, `.delete()` এ
14. **Order করলে:** bookings + availability_slots + notifications update

---

**নোট:** এই ডকুমেন্ট প্রিন্ট করে নিয়ে যেতে পারেন। সব প্রশ্নের উত্তর এখানে আছে!
