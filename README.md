# 🏕️ CampSaaS — Multi-Tenant Booking Platform

A production-ready SaaS booking system for camps, hotels, and rental properties.
Built with NestJS + Next.js + PostgreSQL + Prisma + Stripe.

---

## 🚀 Quick Start (Local Dev)

### Prerequisites
- Node.js 20+
- Docker & Docker Compose
- Git

### 1. Clone & setup env
```bash
git clone <your-repo>
cd campsaas
cp .env.example .env
# Edit .env and fill in your secrets
```

### 2. Start with Docker Compose
```bash
docker-compose up -d postgres redis
```

### 3. Backend setup
```bash
cd backend
npm install
npx prisma migrate dev --name init
npx prisma db seed
npm run start:dev
# → API running at http://localhost:4000
# → Swagger docs at http://localhost:4000/api/docs
```

### 4. Frontend setup
```bash
cd frontend
npm install
npm run dev
# → Dashboard at http://localhost:3000
```

### 5. Login
```
Owner: owner@nuweibacamps.com / Demo@123456
Admin: admin@campsaas.com / Admin@123456
```

---

## 🐳 Production Deployment (VPS)

### Requirements
- Ubuntu 22.04 VPS (min 2GB RAM)
- Domain pointing to your VPS IP
- Docker + Docker Compose installed

### Deploy
```bash
# On your VPS:
git clone <your-repo> /opt/campsaas
cd /opt/campsaas
cp .env.example .env
nano .env  # fill ALL variables

# Build and start
docker-compose up -d --build

# Run migrations
docker-compose exec backend npx prisma migrate deploy
docker-compose exec backend npx prisma db seed

# Check logs
docker-compose logs -f backend
```

### SSL with Let's Encrypt
```bash
apt install certbot
certbot certonly --standalone -d yourdomain.com

# Certs land at /etc/letsencrypt/live/yourdomain.com/
# Update docker/nginx.conf to enable the SSL server block
# Copy certs to docker/certs/
```

---

## 📁 Project Structure

```
campsaas/
├── backend/                    # NestJS API
│   ├── prisma/
│   │   ├── schema.prisma       # Full DB schema (15 models)
│   │   └── seed.ts             # Demo data seed
│   ├── src/
│   │   ├── app.module.ts       # Root module
│   │   ├── main.ts             # Bootstrap + Swagger
│   │   ├── config/             # App configuration
│   │   ├── prisma/             # Prisma service
│   │   ├── common/
│   │   │   ├── guards/         # JWT, Roles, Subscription guards
│   │   │   ├── decorators/     # @CurrentUser, @Roles
│   │   │   ├── filters/        # Global error handler
│   │   │   └── interceptors/   # Response transformer
│   │   └── modules/
│   │       ├── auth/           # Register, Login, JWT, Staff invite
│   │       ├── tenants/        # Org profile, API keys, Webhooks
│   │       ├── camps/          # Camp CRUD, blocked dates
│   │       ├── rooms/          # Room types, seasonal pricing
│   │       ├── bookings/       # Bookings, availability, receipts
│   │       ├── payments/       # Stripe subscriptions + webhooks
│   │       ├── chat/           # WhatsApp inbox, real-time WS
│   │       ├── ai/             # OpenAI booking assistant
│   │       ├── ical/           # iCal import/export + cron sync
│   │       ├── analytics/      # Revenue, occupancy, charts
│   │       └── admin/          # Super admin panel
│   └── Dockerfile
│
├── frontend/                   # Next.js 14 dashboard
│   ├── src/
│   │   ├── app/
│   │   │   ├── layout.tsx
│   │   │   ├── login/          # Login page
│   │   │   ├── register/       # Registration + free trial
│   │   │   └── dashboard/
│   │   │       ├── layout.tsx  # Sidebar navigation
│   │   │       ├── page.tsx    # Dashboard home
│   │   │       ├── bookings/   # Booking management
│   │   │       ├── billing/    # Stripe plans + history
│   │   │       └── analytics/  # Charts + occupancy
│   │   └── lib/
│   │       ├── api.ts          # All API calls (typed)
│   │       └── auth-store.ts   # Zustand auth state
│   └── Dockerfile
│
├── docker/
│   └── nginx.conf              # Reverse proxy config
├── docker-compose.yml          # Full stack orchestration
└── .env.example                # All environment variables
```

---

## 🔌 API Overview

Full Swagger docs at `/api/docs` when running locally.

### Base URL
```
http://localhost:4000/api/v1
```

### Auth
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/auth/register` | Register + create org (starts free trial) |
| POST | `/auth/login` | Login, returns JWT pair |
| POST | `/auth/refresh` | Refresh access token |
| GET  | `/auth/me` | Current user + org profile |
| POST | `/auth/invite` | Invite staff member |

### Bookings (key endpoints)
| Method | Endpoint | Auth |
|--------|----------|------|
| POST | `/bookings?orgId=xxx` | Public |
| GET  | `/bookings/availability` | Public |
| GET  | `/bookings` | Staff |
| GET  | `/bookings/calendar` | Staff |
| PUT  | `/bookings/:id` | Admin |
| DELETE | `/bookings/:id` | Admin |

### Payments
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/payments/status` | Subscription status |
| POST | `/payments/checkout` | Create Stripe checkout |
| POST | `/payments/portal` | Open billing portal |
| POST | `/payments/webhook/stripe` | Stripe webhook |

### iCal
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/ical/feed/:orgId/:campId` | Export .ics feed (public link) |
| GET | `/ical/check?url=&checkin=&checkout=` | Check external calendar |

---

## 💳 Subscription Plans

| Feature | Free Trial | Starter | Professional | Enterprise |
|---------|-----------|---------|--------------|------------|
| Camps | 1 | 2 | 10 | Unlimited |
| Rooms | 5 | 20 | 100 | Unlimited |
| Bookings/mo | 50 | 500 | 5,000 | Unlimited |
| Staff | 2 | 5 | 20 | Unlimited |
| WhatsApp | ❌ | ✅ | ✅ | ✅ |
| Analytics | ❌ | ✅ | ✅ | ✅ |
| Public API | ❌ | ❌ | ✅ | ✅ |
| White-label | ❌ | ❌ | ✅ | ✅ |

---

## 🔧 Key Environment Variables

```env
# Required for core functionality
DATABASE_URL=...
JWT_ACCESS_SECRET=...
JWT_REFRESH_SECRET=...

# Required for subscriptions
STRIPE_SECRET_KEY=sk_...
STRIPE_WEBHOOK_SECRET=whsec_...

# Required for AI features
OPENAI_API_KEY=sk-...

# Required for WhatsApp
WHATSAPP_ACCESS_TOKEN=...
WHATSAPP_PHONE_NUMBER_ID=...
WHATSAPP_VERIFY_TOKEN=...
```

---

## 🛡️ Security Features

- JWT with short-lived access tokens (15m) + refresh rotation
- bcrypt password hashing (12 rounds)
- Role-based access control (OWNER / ADMIN / STAFF)
- Subscription guard (blocks features beyond plan)
- Rate limiting via ThrottlerModule
- Input validation with class-validator
- Helmet security headers
- Multi-tenant data isolation (organizationId on every query)
- API key hashing (SHA-256, never stored plaintext)
- Stripe webhook signature verification

---

## 📅 iCal Integration

CampSaaS supports iCal for external calendar sync:

**Export**: Each camp/room gets a public `.ics` URL:
```
GET /api/v1/ical/feed/{orgId}/{campId}?roomTypeId={roomId}
```

Add this URL to Google Calendar, Airbnb host calendar, or any OTA platform.

**Import**: Add iCal URL to a room type's `icalUrl` field.
The system checks external calendars before confirming bookings, and a cron job syncs every 2 hours.

---

## 🤖 AI Booking Assistant

Powered by OpenAI (configurable model, default: `gpt-4o-mini`).

**Public widget** — embed on your website:
```javascript
fetch('/api/v1/ai/chat?orgId=YOUR_ORG_ID', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    campId: 'optional-camp-id',
    messages: [{ role: 'user', content: 'What rooms are available?' }]
  })
})
```

**Staff tools**: AI-suggested replies in the inbox, WhatsApp auto-reply generation.

---

## 🔮 Future Roadmap (Architecture Ready)

- **Custom domains** — `slug` field already on Organization
- **OTA sync** — BookingSource enum has `OTA` value, iCal export ready
- **Multi-currency** — currency field on every booking
- **Mobile app** — REST API is fully documented
- **Microservices** — NestJS modular architecture, each module can become a service

---

## 📞 Support

For deployment help or feature requests, contact via the platform's support email.
