# Food Surplus Router — Backend API & Database Service ⚙️

Robust, production-ready backend engine and database service for the **Food Surplus Router** platform, powered by **Prisma ORM**, **PostgreSQL**, and Next.js Route Handlers.

---

## 🏗️ Core Architecture & Modules

### 1. 🗄️ Database & Schema (`prisma/schema.prisma`)
* **ORM:** Prisma 5 with Cloud PostgreSQL (Neon / Supabase).
* **Models:**
  * `User`: Multi-role schema (`ADMIN`, `RESTAURANT`, `SHELTER`), KYC compliance status, compliance score, and donor tiering.
  * `SurplusListing`: Geo-coordinates, pickup windows, expiry time, portions, and ESG emission attributes.
  * `Claim`: Atomic multi-shelter claims with interactive transactions to prevent double-claiming race conditions.
  * `Payment`: Direct UPI transactions tracking ₹20 platform fees and verified 12-digit UTR references.
  * `RazorpayTransaction`: Consolidated financial ledger for admin revenue transparency.
  * `DisputeTicket`: Moderation tickets for compromised or uncollected food items.

### 2. 🔐 Authentication & Session (`lib/auth.ts`, `lib/session.ts`)
* NextAuth.js with bcrypt password hashing (12 salt rounds).
* Stateless JWT strategy with role-based route middleware guards (`ADMIN`, `RESTAURANT`, `SHELTER`).

### 3. 💳 Direct UPI Payment Engine (`lib/upi-payments.ts`)
* Generates NPCI-compliant deep-link intent URIs (`upi://pay?pa=...&am=20.00&cu=INR&tr=...`).
* Locks the platform convenience fee strictly to ₹20.00.
* Generates dynamic high-res QR codes on the server for instant mobile scanning (Google Pay, PhonePe, Paytm, BHIM).
* `/api/payments/verify` strictly enforces 12-digit UTR input and blocks duplicate transaction re-use.

### 4. ⚡ Real-Time Events (`app/api/events/route.ts`)
* Server-Sent Events (SSE) push stream alerting connected shelters instantly when local restaurants post fresh surplus food.

---

## 📡 Key API Endpoints

| Method | Endpoint | Description |
| :--- | :--- | :--- |
| `POST` | `/api/auth/register` | Register new Restaurant or Shelter organization |
| `POST` | `/api/auth/[...nextauth]` | Login and JWT token issuance |
| `GET` / `POST` | `/api/listings` | Browse available listings / Post new surplus |
| `POST` | `/api/claims` | Atomically claim food surplus |
| `POST` | `/api/payments/create-order` | Generate ₹20 UPI payment intent & QR code |
| `POST` | `/api/payments/verify` | Verify 12-digit UTR and approve food claim |
| `PATCH` | `/api/admin/kyc` | Approve / Reject shelter documentation |
| `PATCH` | `/api/admin/donors` | Update restaurant scale tier & capacity |
| `GET` | `/api/admin/esg` | Aggregate carbon offset & landfill diversion metrics |
| `GET` | `/api/events` | Real-time SSE stream for live surplus food alerts |

---

## 🚀 Database Setup & Migration

### 1. Install Dependencies
```bash
npm install
```

### 2. Configure Environment (`.env`)
```env
DATABASE_URL="postgresql://username:password@your-host.neon.tech/neondb?sslmode=require"
NEXTAUTH_SECRET="your-32-char-random-secret"
PLATFORM_FEE_INR="20"
PLATFORM_PAYEE_NAME="Food Surplus Router"
PLATFORM_UPI_NUMBER="8328099342"
PLATFORM_UPI_ID="8328099342@upi"
```

### 3. Sync Database Schema
```bash
npx prisma db push
```

### 4. Seed Admin & Operational Data
```bash
npm run db:seed
```
* Seeds the primary admin: `vokkerlasnehith@gmail.com` with password `Snehith@18`.
