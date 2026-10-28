# AD Aquatics — Bring the Ocean Home

A modern, responsive, and secure web application for **AD Aquatics**, featuring a dynamic category showcase, real product inventory management, WhatsApp ordering integration, smooth underwater intro animation, and an administrative dashboard protected by database-level Row Level Security (RLS).

---

## Table of Contents

1. [Features Overview](#features-overview)
2. [Project Structure](#project-structure)
3. [Prerequisites & Installation](#prerequisites--installation)
4. [Running Locally](#running-locally)
5. [Supabase Backend Configuration](#supabase-backend-configuration)
   - [Database Schema & Migrations](#database-schema--migrations)
   - [Row Level Security (RLS) Policies](#row-level-security-rls-policies)
   - [Storage Bucket & Media Policies](#storage-bucket--media-policies)
   - [Secure Initial Administrator Provisioning](#secure-initial-administrator-provisioning)
6. [Offline / Local Development Sandbox](#offline--local-development-sandbox)
7. [WhatsApp Business Ordering Setup](#whatsapp-business-ordering-setup)
8. [Admin Dashboard Guide](#admin-dashboard-guide)
   - [Category Management & Safe Deletion](#category-management--safe-deletion)
   - [Product Catalog Management](#product-catalog-management)
   - [Image Library](#image-library)
   - [Website Settings](#website-settings)
9. [Production Deployment & Security Checklist](#production-deployment--security-checklist)

---

## 1. Features Overview

- **Warm Minimalist Palette:** Clean, premium aesthetic built with Warm Ivory (`#F7F3EA`), Soft Cream (`#EFE7D8`), Warm White (`#FFFEFA`), Deep Charcoal (`#26312F`), Muted Ocean Teal (`#397F78`), and Deep Teal (`#28665F`).
- **Underwater Intro Experience:** A lightweight animation showing a fish swimming across an underwater gradient with rising bubbles, concluding with the glowing AD Aquatics emblem reveal and a skip button.
- **Dynamic Category Department Boxes:** Initially configured with five core departments (**Fish**, **Food**, **Tank**, **Decor**, **Working Parts**). Categories can be added, reordered, edited, deactivated, or deleted safely from the dashboard.
- **Public Offers Page (`/offers`):** Displays published, enabled promotional offers currently within their active start and end dates. Includes support for percentage discounts, fixed savings, custom promotional messages, linked products or departments, and direct WhatsApp deal inquiries. Displays a clean empty state when no active offers exist.
- **Admin Offer Management:** Dedicated "Manage Offers" section inside the admin dashboard with full CRUD, start/end scheduling, banner image upload, live card preview, and safe deletion.
- **Empty Inventory State Integrity:** Zero fake or placeholder products or offers are prefilled. Real products and offers are entered and maintained by authorized administrators.
- **WhatsApp Direct Ordering & Offer Inquiries:** Pre-fills detailed messages with product or offer specifications and validity details, connecting directly to the authorized business phone number.
- **Safe Category Deletion:** Protects inventory from orphaned records. If a category contains products, deletion is blocked until products are either reassigned to an active category or the category is archived.
- **Enterprise-Grade Security:** Row Level Security (RLS) ensures public visitors can only read active categories, published products, and currently valid offers, while mutation operations strictly require verified administrator permissions.

---

## 2. Project Structure

```
ad-aquatics/
├── public/
│   ├── logo.png                       # Official 3D AD Aquatics emblem
│   └── favicon.svg
├── src/
│   ├── components/
│   │   ├── admin/
│   │   │   ├── AdminLayout.tsx        # Dashboard shell with responsive navigation
│   │   │   ├── AdminLogin.tsx         # Secure admin login form
│   │   │   ├── AdminOverview.tsx      # Metrics, health checks, and quick actions
│   │   │   ├── CategoryManagement.tsx # Category CRUD and ordering
│   │   │   ├── ProductManagement.tsx  # Product catalog CRUD, filters, and drafts
│   │   │   ├── OfferManagement.tsx    # Offer management (CRUD, scheduling, linking)
│   │   │   ├── ImageManagement.tsx    # Media library and URL clipboard helper
│   │   │   ├── SafeDeleteModal.tsx    # Category deletion and reassignment safety modal
│   │   │   └── WebsiteSettingsView.tsx# Brand, WhatsApp, About, and Banner configuration
│   │   ├── common/
│   │   │   ├── Footer.tsx             # Public footer with brand info and admin trigger
│   │   │   ├── IntroAnimation.tsx     # Swimming fish, bubbles, and logo intro
│   │   │   ├── Navbar.tsx             # Responsive header with search and links
│   │   │   ├── Toast.tsx              # Toast notification system
│   │   │   └── WhatsAppModal.tsx      # WhatsApp order confirmation & guidance modal
│   │   ├── home/
│   │   │   ├── AboutSection.tsx       # Brand story, health standards, and guarantees
│   │   │   ├── CategorySection.tsx    # Dynamic department cards
│   │   │   ├── ContactSection.tsx     # Direct inquiry composer and verified contacts
│   │   │   ├── Hero.tsx               # Ocean ambiance hero with tagline and CTAs
│   │   │   └── ProductSection.tsx     # Searchable catalog with empty states
│   │   └── offers/
│   │       └── OffersPage.tsx         # Public Offers page with validity filter & WhatsApp
│   ├── lib/
│   │   ├── auth.ts                    # Supabase Auth and sandbox session handling
│   │   ├── constants.ts               # Default settings and 5 initial categories
│   │   ├── db.ts                      # Centralized data access layer with persistent fallback
│   │   ├── supabase.ts                # Supabase client instantiation
│   │   └── whatsapp.ts                # WhatsApp message & offer inquiry URL generator
│   ├── types/
│   │   └── index.ts                   # TypeScript interfaces (Offer, Product, Category...)
│   ├── App.tsx                        # Router, public website, offers route, dashboard
│   ├── index.css                      # Tailwind CSS v4, warm ivory CSS variables, keyframes
│   └── main.tsx
├── supabase/
│   └── migrations/
│       ├── 20261015000000_ad_aquatics_schema.sql # PostgreSQL schema, RLS, storage
│       └── 20261028000000_ad_aquatics_offers.sql # Offers table, RLS, indexes
├── .env.example                       # Environment template with placeholder variables
├── index.html
├── package.json
└── vite.config.ts
```

---

## 3. Prerequisites & Installation

- **Node.js**: v18.0.0 or higher (v20+ recommended)
- **npm** or **pnpm** or **yarn**

Install project dependencies:

```bash
npm install
```

---

## 4. Running Locally

Start the Vite development server:

```bash
npm run dev
```

Open your browser at the local URL (typically `http://localhost:5173`).

---

## 5. Supabase Backend Configuration

### Step A: Create a Supabase Project
1. Log in to [Supabase](https://supabase.com) and create a new project.
2. Note your **Project URL** and **anon public API key** in **Project Settings -> API**.

### Step B: Configure Environment Variables
Copy `.env.example` to `.env`:

```bash
cp .env.example .env
```

Edit `.env` and set your credentials:

```env
VITE_SUPABASE_URL=https://your-project-id.supabase.co
VITE_SUPABASE_ANON_KEY=your-actual-anon-key-here
```

> **Warning:** NEVER paste your `service_role` key into client environment files or frontend source code.

### Step C: Apply Database Migrations & Schema
1. In the Supabase Dashboard, open the **SQL Editor**.
2. Open the file `supabase/migrations/20261015000000_ad_aquatics_schema.sql` from this repository.
3. Paste the contents into the SQL Editor and click **Run**.

This script sets up:
- The `categories` table with slug uniqueness and display ordering.
- The `products` table with strict foreign-key protection (`ON DELETE RESTRICT`).
- The `admin_users` table and `public.is_admin()` security helper function.
- The `website_settings` table for persistent site configuration.
- The `aquatics-media` storage bucket.
- Row Level Security (RLS) on all tables and storage objects.
- Initial seed data for the 5 core categories (**Fish**, **Food**, **Tank**, **Decor**, **Working Parts**).

### Step D: Secure Initial Administrator Provisioning
Public self-registration for administrators is strictly disabled. To provision the first administrator:

1. In Supabase Dashboard, navigate to **Authentication -> Users**.
2. Click **Add User** -> **Create User**.
3. Enter the administrator's email and password.
4. Copy the generated `User ID` (UUID).
5. Open the **SQL Editor** and run:

```sql
INSERT INTO public.admin_users (user_id, email, role)
VALUES ('<PASTE-COPIED-USER-UUID-HERE>', 'admin@adaquatics.com', 'superadmin');
```

This user is now verified as an administrator and can log in at `/admin/login`.

---

## 6. Offline / Local Development Sandbox

When running without Supabase credentials in `.env`, the application automatically activates a local development sandbox:
- Data persists in browser `localStorage`.
- The initial 5 categories are loaded automatically.
- Products added via the admin dashboard are saved locally.
- Development administrator credentials:
  - **Email:** `admin@adaquatics.com`
  - **Password:** `AquaticsAdmin2026!`
- A banner inside the dashboard indicates whether the backend is running in **Supabase Connected** mode or **Local Offline Store** mode.

---

## 7. WhatsApp Business Ordering Setup

1. Log in to the Admin Dashboard at `/admin/login`.
2. Navigate to **Website Settings**.
3. Under **WhatsApp Ordering & Inquiries**, enter your full WhatsApp Business phone number including country code (e.g., `+14155552671` or `447911123456`).
4. Click **Save All Settings**.

Now, when visitors browse products on the website and click **Order on WhatsApp**, the app formats an order message and opens WhatsApp directly with the seller.

---

## 8. Admin Dashboard Guide

Access the admin dashboard at `/admin` (or `/admin/login`).

### Category Management & Safe Deletion
- **Add New Category:** Click "Add New Category", provide name, optional image URL/file, and description.
- **Reorder:** Use the Up / Down controls to change category sequence on the homepage.
- **Toggle Visibility:** Click "Active / Hidden" to show or hide departments from visitors.
- **Safe Deletion:**
  - If a category contains 0 products, it is deleted after confirmation.
  - If a category contains products, deletion is blocked. The modal allows you to safely reassign all products to another active category before completing deletion, or deactivate the category instead.

### Product Catalog Management
- **Add Product:** Specify title, category, price, stock status (In Stock, Low Stock, Made to Order, Out of Stock), description, and image.
- **Drafts vs Published:** Products can be saved as drafts while preparing inventory.
- **Catalog Filter:** Switch between table view and grouped-by-department view.

### Image Library
- Upload product photography or department hero banners (JPG, PNG, WebP, SVG up to 5MB).
- Use **Copy Link** to paste image URLs directly into product forms.

---

## 9. Production Deployment & Security Checklist

1. **Verify Environment Variables:** Confirm `VITE_SUPABASE_URL` and `VITE_SUPABASE_ANON_KEY` are configured in your deployment platform (e.g. Vercel, Netlify, Cloudflare Pages).
2. **Confirm RLS is Active:** Run `SELECT * FROM pg_tables WHERE schemaname = 'public';` in Supabase to ensure RLS is enabled on all tables.
3. **Verify Admin Provisioning:** Verify that only trusted administrators exist in `public.admin_users`.
4. **Test WhatsApp Link:** Ensure the business phone number format has been verified in live browser tests.
5. **Run Production Build:**
   ```bash
   npm run build
   ```
   Preview the production build locally:
   ```bash
   npm run preview
   ```
