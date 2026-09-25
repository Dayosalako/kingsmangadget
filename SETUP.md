# Kingsman Modern Gadgets — Setup Guide

You got three files:
- `index.html` — the homepage
- `store.html` — the full product catalog customers browse and order from
- `admin.html` — where you (the owner) add/edit/delete products — **keep this link private**

All three currently show placeholder contact info and no live database. Do these two things and you're live.

## 1. Placeholder info to replace

In all three files, find and replace (Ctrl+F):
- `+2348000000000` → your real WhatsApp/phone number, no `+`, no spaces (e.g. `2348012345678`)
- `sales@kingsmangadgets.com` → your real email
- `Abuja, Nigeria` / `Abuja, FCT, Nigeria` → your real location

## 2. Connect a database (Supabase — free tier is enough)

The site needs somewhere to store products and product photos. We use [Supabase](https://supabase.com), a free hosted database — no server needed.

### a) Create a project
1. Go to supabase.com → sign up → **New Project**.
2. Pick any name/password/region, wait ~2 minutes for it to spin up.

### b) Create the products table
In your Supabase project, open **SQL Editor** → **New query**, paste this, click **Run**:

```sql
create table products (
  id uuid primary key default gen_random_uuid(),
  created_at timestamptz default now(),
  category text,
  brand text,
  name text not null,
  condition text default 'new',
  price numeric not null,
  old_price numeric,
  specs text[],
  in_stock boolean default true,
  hot boolean default false,
  images text[]
);

alter table products enable row level security;

-- Anyone can view products (the storefront needs this)
create policy "Public can view products"
  on products for select
  using (true);

-- Only signed-in users (you, the admin) can add/edit/delete
create policy "Authenticated users can manage products"
  on products for all
  using (auth.role() = 'authenticated')
  with check (auth.role() = 'authenticated');
```

### c) Create the image storage bucket
Go to **Storage** in the sidebar → **New bucket** → name it exactly `product-images` → toggle **Public bucket** ON → Create.

Then, in **SQL Editor**, run:

```sql
create policy "Public can view product images"
  on storage.objects for select
  using (bucket_id = 'product-images');

create policy "Authenticated users can upload product images"
  on storage.objects for insert
  with check (bucket_id = 'product-images' and auth.role() = 'authenticated');

create policy "Authenticated users can update product images"
  on storage.objects for update
  using (bucket_id = 'product-images' and auth.role() = 'authenticated');
```

### d) Create your admin login
Go to **Authentication** → **Users** → **Add user** → enter the email/password you'll use to sign in to `admin.html`. Under **Auto Confirm User**, tick it so you don't need to verify by email.

### e) Get your API keys
Go to **Project Settings** → **API**. Copy:
- **Project URL**
- **anon public** key

### f) Paste them into the files
In `index.html`, `store.html`, and `admin.html`, find this block near the bottom (inside a `<script>` tag):

```js
var SB_URL = 'YOUR_SUPABASE_URL';
var SB_KEY = 'YOUR_SUPABASE_ANON_KEY';
```

Replace both placeholder strings with the values you copied. Save all three files.

## 3. Host the files

Upload `index.html`, `store.html`, and `admin.html` to any static host (your existing hosting/cPanel, Netlify, Vercel, GitHub Pages, etc.) — no build step, no server, they work as plain files. Keep the `admin.html` URL out of public navigation/search engines; it's already tagged `noindex` but isn't password-protected beyond your Supabase login, so don't link to it from the storefront.

## 4. Add your first products

Go to `yoursite.com/admin.html`, sign in with the account you created in step 2d, and start adding products. They'll appear on `store.html` immediately, and the four newest/hot ones will surface on the homepage.

---

**Note on the WhatsApp ordering flow:** when a customer clicks "Order via WhatsApp," it opens a pre-filled message with their order details and sends it to the number you set in step 1. Nothing is saved anywhere else — treat WhatsApp as your order inbox, or add order-logging later if you want a paper trail.
