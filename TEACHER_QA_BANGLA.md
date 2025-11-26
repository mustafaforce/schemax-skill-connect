# SchemaX প্রজেক্ট - শিক্ষকের প্রশ্নের উত্তর (বাংলায়)

## প্রজেক্ট সম্পর্কে সংক্ষিপ্ত তথ্য

**প্রজেক্টের নাম:** SchemaX - Freelancer Marketplace  
**টেকনোলজি স্ট্যাক:** React + TypeScript + Supabase (PostgreSQL)  
**উদ্দেশ্য:** একটি ফ্রিল্যান্সার মার্কেটপ্লেস যেখানে ক্লায়েন্ট এবং ফ্রিল্যান্সাররা সংযুক্ত হতে পারে

---

## ১. ফ্রন্টএন্ড এবং ব্যাকএন্ড কোথায় কানেক্ট করা হয়েছে?

### উত্তর:

ফ্রন্টএন্ড এবং ব্যাকএন্ড **Supabase Client** এর মাধ্যমে কানেক্ট করা হয়েছে। এটি একটি centralized connection যা পুরো অ্যাপ্লিকেশনে ব্যবহার করা হয়।

### কানেকশন ফাইল:
**ফাইল লোকেশন:** `src/integrations/supabase/client.ts`

```typescript
import { createClient } from '@supabase/supabase-js';
import type { Database } from './types';

const SUPABASE_URL = "https://zdmymcefzmbckohzupgt.supabase.co";
const SUPABASE_PUBLISHABLE_KEY = "eyJhbGc..."; // API Key

export const supabase = createClient<Database>(SUPABASE_URL, SUPABASE_PUBLISHABLE_KEY, {
  auth: {
    storage: localStorage,
    persistSession: true,
    autoRefreshToken: true,
  }
});
```

### কীভাবে কাজ করে:
1. **Supabase URL**: এটি আমাদের ব্যাকএন্ড ডাটাবেসের ঠিকানা
2. **API Key**: এটি authentication এর জন্য ব্যবহৃত হয়
3. **createClient**: এই ফাংশন একটি client object তৈরি করে যা দিয়ে আমরা database এর সাথে কথা বলতে পারি
4. **localStorage**: ইউজারের লগইন সেশন ব্রাউজারে সেভ রাখে

### কোথায় ব্যবহার করা হয়:
প্রতিটি পেজে এই client import করা হয়:
```typescript
import { supabase } from "@/integrations/supabase/client";
```

---

## ২. কোন কোন টেবিলে ডাটা পুশ (Insert) বা ফেচ (Fetch) করা হয়?

আমাদের প্রজেক্টে **১০টি প্রধান টেবিল** আছে। নিচে প্রতিটি টেবিলের জন্য কোড উদাহরণ দেওয়া হলো:

---

### ২.১ **profiles** টেবিল (ইউজার প্রোফাইল)

#### কোথায় ডাটা Fetch করা হয়:
**ফাইল:** `src/pages/Dashboard.tsx` (লাইন ৪৬৭-৪৭১)

```typescript
const { data, error } = await supabase
  .from('profiles')
  .select('*')
  .eq('id', userId)
  .single();
```

**ব্যাখ্যা:**
- `.from('profiles')` - profiles টেবিল থেকে ডাটা নিচ্ছি
- `.select('*')` - সব কলাম সিলেক্ট করছি
- `.eq('id', userId)` - যে ইউজারের ID match করবে
- `.single()` - একটি মাত্র রেকর্ড রিটার্ন করবে

#### কোথায় ডাটা Insert/Update করা হয়:
**ফাইল:** `src/pages/ProfileSetup.tsx` (লাইন ৬৬)

```typescript
const { error } = await supabase
  .from('profiles')
  .insert({
    id: session.user.id,
    display_name: formData.displayName,
    role: formData.role,
    bio: formData.bio || null,
    location: formData.location || null,
    hourly_rate: formData.hourlyRate ? parseFloat(formData.hourlyRate) : null,
    is_public: formData.isPublic
  });
```

**ব্যাখ্যা:**
- `.insert({...})` - নতুন প্রোফাইল তৈরি করছি
- সব ফিল্ড একসাথে object আকারে পাঠানো হচ্ছে

---

### ২.২ **services** টেবিল (ফ্রিল্যান্সার সার্ভিস)

#### কোথায় ডাটা Insert করা হয়:
**ফাইল:** `src/pages/CreateService.tsx` (লাইন ১০০-১১১)

```typescript
const { error } = await supabase
  .from('services')
  .insert({
    freelancer_id: session.user.id,
    title: formData.title,
    description: formData.description || null,
    price_cents: Math.round(parseFloat(formData.price) * 100),
    duration_minutes: parseInt(formData.duration),
    category_id: formData.category_id || null,
    subcategory_id: formData.subcategory_id || null,
    is_active: formData.is_active
  });
```

**ব্যাখ্যা:**
- `price_cents: Math.round(parseFloat(formData.price) * 100)` - টাকাকে পয়সায় কনভার্ট করছি (৫০ টাকা = ৫০০০ পয়সা)
- `duration_minutes` - সার্ভিসের সময়কাল মিনিটে

#### কোথায় ডাটা Fetch করা হয়:
**ফাইল:** `src/pages/Dashboard.tsx` (লাইন ২৪৪-২৪৮)

```typescript
const { data, error } = await supabase
  .from('services')
  .select('*')
  .eq('freelancer_id', profile.id)
  .order('created_at', { ascending: false });
```

**ব্যাখ্যা:**
- `.eq('freelancer_id', profile.id)` - শুধু এই ফ্রিল্যান্সারের সার্ভিস দেখাবে
- `.order('created_at', { ascending: false })` - নতুন সার্ভিস আগে দেখাবে

#### কোথায় ডাটা Update করা হয়:
**ফাইল:** `src/pages/Dashboard.tsx` (লাইন ২৬৫-২৬৮)

```typescript
const { error } = await supabase
  .from('services')
  .update({ is_active: !currentStatus })
  .eq('id', serviceId);
```

**ব্যাখ্যা:**
- `.update({...})` - সার্ভিস activate/deactivate করছি
- `.eq('id', serviceId)` - নির্দিষ্ট সার্ভিস আপডেট করছি

#### কোথায় ডাটা Delete করা হয়:
**ফাইল:** `src/pages/Dashboard.tsx` (লাইন ২৯৭-৩০০)

```typescript
const { error } = await supabase
  .from('services')
  .delete()
  .eq('id', serviceId);
```

---

### ২.৩ **bookings** টেবিল (বুকিং তথ্য)

#### কোথায় ডাটা Fetch করা হয়:
**ফাইল:** `src/pages/Dashboard.tsx` (লাইন ৪৭-৬৮)

```typescript
let query = supabase
  .from('bookings')
  .select(`
    *,
    service:services(
      id,
      title,
      price_cents,
      duration_minutes,
      freelancer:profiles!services_freelancer_id_fkey(
        id,
        display_name,
        avatar_url
      )
    ),
    client:profiles!bookings_client_id_fkey(
      id,
      display_name,
      avatar_url
    )
  `)
  .order('start_time', { ascending: false });
```

**ব্যাখ্যা:**
- এখানে **JOIN query** ব্যবহার করা হয়েছে
- `service:services(...)` - services টেবিল থেকে related ডাটা নিচ্ছি
- `client:profiles!bookings_client_id_fkey(...)` - foreign key দিয়ে profiles টেবিল join করছি
- এটি একটি **nested query** - একবারে multiple টেবিল থেকে ডাটা আনছি

#### কোথায় ডাটা Insert করা হয়:
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

**ব্যাখ্যা:**
- এখানে **Supabase Edge Function** ব্যবহার করা হয়েছে
- `.functions.invoke('create-booking', {...})` - একটি server-side function call করছি
- এটি সরাসরি database এ insert না করে, একটি function এর মাধ্যমে করছি (security এর জন্য)

---

### ২.৪ **categories** টেবিল (ক্যাটাগরি)

#### কোথায় ডাটা Fetch করা হয়:
**ফাইল:** `src/pages/CreateService.tsx` (লাইন ৫৮-৬২)

```typescript
const { data } = await supabase
  .from('categories')
  .select('*')
  .order('name');
```

**ফাইল:** `src/components/home/Categories.tsx` (লাইন ৩৫)

```typescript
const { data } = await supabase
  .from("categories")
  .select("*");
```

---

### ২.৫ **subcategories** টেবিল (সাব-ক্যাটাগরি)

#### কোথায় ডাটা Fetch করা হয়:
**ফাইল:** `src/pages/CreateService.tsx` (লাইন ৭২-৭৬)

```typescript
const { data } = await supabase
  .from('subcategories')
  .select('*')
  .eq('category_id', categoryId)
  .order('name');
```

**ব্যাখ্যা:**
- `.eq('category_id', categoryId)` - নির্দিষ্ট ক্যাটাগরির সাব-ক্যাটাগরি দেখাচ্ছি
- এটি একটি **filtered query**

---

### ২.৬ **availability_slots** টেবিল (ফ্রিল্যান্সার সময়সূচী)

#### কোথায় ডাটা Fetch করা হয়:
**ফাইল:** `src/pages/Availability.tsx` (লাইন ৬৬)

```typescript
const { data, error } = await supabase
  .from('availability_slots')
  .select('*')
  .eq('freelancer_id', profile.id)
  .order('start_time');
```

#### কোথায় ডাটা Insert করা হয়:
**ফাইল:** `src/pages/Availability.tsx` (লাইন ১১৩)

```typescript
const { error } = await supabase
  .from('availability_slots')
  .insert({
    freelancer_id: profile.id,
    start_time: slot.start_time,
    end_time: slot.end_time,
    is_booked: false
  });
```

#### কোথায় ডাটা Delete করা হয়:
**ফাইল:** `src/pages/Availability.tsx` (লাইন ১৪৪)

```typescript
const { error } = await supabase
  .from('availability_slots')
  .delete()
  .eq('id', slotId);
```

---

### ২.৭ **reviews** টেবিল (রিভিউ)

#### কোথায় ডাটা Fetch করা হয়:
**ফাইল:** `src/pages/FreelancerProfile.tsx` (লাইন ৯৪)

```typescript
const { data } = await supabase
  .from('reviews')
  .select(`
    *,
    client:profiles!reviews_client_id_fkey(display_name, avatar_url)
  `)
  .eq('freelancer_id', freelancerId)
  .order('created_at', { ascending: false });
```

**ব্যাখ্যা:**
- এখানেও **JOIN** ব্যবহার করা হয়েছে
- client এর নাম এবং ছবি profiles টেবিল থেকে নিচ্ছি

---

### ২.৮ **notifications** টেবিল (নোটিফিকেশন)

#### কোথায় ডাটা Fetch করা হয়:
**ফাইল:** `src/components/NotificationBell.tsx` (লাইন ৪৪)

```typescript
const { data, error } = await supabase
  .from('notifications')
  .select('*')
  .eq('user_id', userId)
  .order('created_at', { ascending: false })
  .limit(10);
```

**ব্যাখ্যা:**
- `.limit(10)` - শুধু ১০টি নোটিফিকেশন দেখাবে

#### কোথায় ডাটা Update করা হয়:
**ফাইল:** `src/components/NotificationBell.tsx` (লাইন ৯৪)

```typescript
await supabase
  .from('notifications')
  .update({ read: true })
  .eq('id', notificationId);
```

**ব্যাখ্যা:**
- নোটিফিকেশন পড়া হয়েছে mark করছি

---

### ২.৯ **freelancer_skills** টেবিল (ফ্রিল্যান্সার দক্ষতা)

এটি একটি **junction table** (many-to-many relationship এর জন্য)

---

### ২.১০ **verifications** টেবিল (ভেরিফিকেশন)

ফ্রিল্যান্সার verification এর জন্য ব্যবহৃত হয়

---

## ৩. Database Schema (টেবিল স্ট্রাকচার)

### Database Types ফাইল:
**ফাইল:** `src/integrations/supabase/types.ts`

এই ফাইলে সব টেবিলের structure TypeScript type হিসেবে define করা আছে:

```typescript
export type Database = {
  public: {
    Tables: {
      profiles: {
        Row: {
          id: string;
          display_name: string;
          bio: string | null;
          role: "freelancer" | "client";
          hourly_rate: number | null;
          // ... আরো ফিল্ড
        }
      },
      services: {
        Row: {
          id: string;
          title: string;
          price_cents: number;
          duration_minutes: number;
          // ... আরো ফিল্ড
        }
      }
      // ... বাকি টেবিল
    }
  }
}
```

---

## ৪. সাধারণ প্রশ্ন ও উত্তর

### প্রশ্ন: ডাটাবেস কোথায় হোস্ট করা?
**উত্তর:** Supabase cloud এ হোস্ট করা। এটি PostgreSQL database।

### প্রশ্ন: Authentication কীভাবে কাজ করে?
**উত্তর:** Supabase Auth ব্যবহার করা হয়েছে। ইউজার লগইন করলে একটি session তৈরি হয় যা localStorage এ সেভ থাকে।

**ফাইল:** `src/pages/Auth.tsx`

```typescript
const { data, error } = await supabase.auth.signInWithPassword({
  email: formData.email,
  password: formData.password,
});
```

### প্রশ্ন: Real-time features আছে কি?
**উত্তর:** হ্যাঁ, Supabase Realtime ব্যবহার করে notifications এবং chat features implement করা যায়।

### প্রশ্ন: File upload কীভাবে করা হয়?
**উত্তর:** Supabase Storage ব্যবহার করা হয়েছে avatar upload এর জন্য।

**ফাইল:** `src/pages/Profile.tsx` (লাইন ১৮৯-১৯৯)

```typescript
// Avatar upload
const { error: uploadError } = await supabase.storage
  .from('avatars')
  .upload(filePath, file);

// Get public URL
const { data: urlData } = supabase.storage
  .from('avatars')
  .getPublicUrl(filePath);

// Update profile with avatar URL
await supabase
  .from('profiles')
  .update({ avatar_url: urlData.publicUrl })
  .eq('id', user.id);
```

---

## ৫. কোড প্যাটার্ন সারাংশ

### ডাটা Fetch করার প্যাটার্ন:
```typescript
const { data, error } = await supabase
  .from('table_name')
  .select('*')
  .eq('column', value);
```

### ডাটা Insert করার প্যাটার্ন:
```typescript
const { error } = await supabase
  .from('table_name')
  .insert({ field1: value1, field2: value2 });
```

### ডাটা Update করার প্যাটার্ন:
```typescript
const { error } = await supabase
  .from('table_name')
  .update({ field: newValue })
  .eq('id', recordId);
```

### ডাটা Delete করার প্যাটার্ন:
```typescript
const { error } = await supabase
  .from('table_name')
  .delete()
  .eq('id', recordId);
```

### Join Query প্যাটার্ন:
```typescript
const { data } = await supabase
  .from('main_table')
  .select(`
    *,
    related_table:foreign_table(column1, column2)
  `);
```

---

## ৬. গুরুত্বপূর্ণ পয়েন্ট মনে রাখবেন

1. **Supabase Client** একবার তৈরি করা হয় `client.ts` ফাইলে, তারপর সব জায়গায় import করে ব্যবহার করা হয়

2. **সব database operation async/await** দিয়ে করা হয়

3. **Error handling** সব জায়গায় করা হয়েছে try-catch দিয়ে

4. **TypeScript types** ব্যবহার করা হয়েছে type safety এর জন্য

5. **Foreign key relationships** database level এ define করা আছে

6. **Row Level Security (RLS)** Supabase তে enable করা আছে security এর জন্য

---

## ৭. প্রতিটি পেজের কাজ

| পেজ | কোন টেবিল ব্যবহার করে | কী করে |
|-----|------------------------|---------|
| Dashboard.tsx | profiles, bookings, services | ইউজার ড্যাশবোর্ড দেখায় |
| CreateService.tsx | services, categories, subcategories | নতুন সার্ভিস তৈরি |
| BookService.tsx | services, bookings | সার্ভিস বুক করা |
| Profile.tsx | profiles | প্রোফাইল edit করা |
| Availability.tsx | availability_slots | সময়সূচী সেট করা |
| FreelancerProfile.tsx | profiles, reviews, services | ফ্রিল্যান্সার প্রোফাইল দেখা |
| Search.tsx | profiles, services | ফ্রিল্যান্সার খোঁজা |

---

## সংক্ষিপ্ত উত্তর (যদি শিক্ষক দ্রুত জানতে চান)

**প্রশ্ন: ফ্রন্টএন্ড-ব্যাকএন্ড কোথায় কানেক্ট?**  
**উত্তর:** `src/integrations/supabase/client.ts` ফাইলে Supabase client তৈরি করা হয়েছে যা পুরো অ্যাপে ব্যবহার করা হয়।

**প্রশ্ন: কোন টেবিলে ডাটা পুশ/ফেচ করা হয়?**  
**উত্তর:** ১০টি টেবিল: profiles, services, bookings, categories, subcategories, availability_slots, reviews, notifications, freelancer_skills, verifications। প্রতিটি পেজে `.from('table_name')` দিয়ে ডাটা নেওয়া/পাঠানো হয়।

---

**নোট:** এই ডকুমেন্টটি প্রিন্ট করে নিয়ে যেতে পারেন। প্রয়োজনে কোড লাইন নাম্বার দেওয়া আছে যাতে সহজে খুঁজে পাওয়া যায়।
