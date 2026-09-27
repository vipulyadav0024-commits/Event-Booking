# Eventora — MERN Event Booking App

A full-stack event booking platform built with MongoDB, Express, React (Vite) and Node.js. Users can browse events, book tickets, and manage their account with email OTP verification.

---

## 🔗 Live Links

| Service | URL |
|---|---|
| **Frontend (Vercel)** | https://event-booking-vipul-89bf.vercel.app |
| **Backend API (Render)** | https://event-booking-9qxp.onrender.com |
| **API health check** | https://event-booking-9qxp.onrender.com/api/events |
| **GitHub Repo** | https://github.com/vipulyadav0024-commits/Event-Booking |

> ⚠️ The backend is on Render's free tier — it spins down after inactivity. The first request after idle can take 30–60 seconds to respond while it wakes up.

---

## 🗂 Project Structure

```
Event-Booking/
├── client/          → React + Vite frontend (deployed to Vercel)
│   └── src/utils/axios.js   → API base URL config
├── server/          → Express + MongoDB backend (deployed to Render)
│   ├── server.js            → App entry point, CORS config
│   ├── controllers/         → Route logic (auth, events, bookings)
│   ├── routes/               → API route definitions
│   ├── models/                → Mongoose schemas
│   └── utils/email.js        → Transactional email (Resend)
```

---

## ⚙️ Tech Stack

- **Frontend:** React, Vite, Axios
- **Backend:** Node.js, Express, Mongoose
- **Database:** MongoDB Atlas (free M0 cluster)
- **Auth:** JWT + bcrypt password hashing
- **Email/OTP delivery:** Resend (HTTPS-based email API)
- **Hosting:** Render (backend), Vercel (frontend)

---

## 🔑 Environment Variables

### Backend (Render → Environment tab)

| Key | Description |
|---|---|
| `MONGO_URI` | MongoDB Atlas connection string (includes DB user, password, and `eventora` database name) |
| `JWT_SECRET` | Any random secret string, used to sign login tokens |
| `EMAIL_USER` | Gmail address (kept for legacy/reference; no longer used for sending) |
| `EMAIL_PASS` | Gmail App Password (legacy; no longer used for sending) |
| `RESEND_API_KEY` | API key from resend.com — used to actually send OTP + booking emails |
| `PORT` | `5000` |

### Frontend (Vercel → Settings → Environments → Production)

| Key | Description |
|---|---|
| `VITE_API_URL` | `https://event-booking-9qxp.onrender.com/api` — points the frontend at the live backend |

---

## 🚀 Deployment Summary

### 1. Database — MongoDB Atlas
- Free M0 cluster (`Cluster0`)
- Database user created under **Database Access**
- **Network Access** set to `0.0.0.0/0` (allow all IPs) so Render can connect
- Connection string copied from **Connect → Drivers**, special characters in the password URL-encoded (e.g. `!` → `%21`)

### 2. Backend — Render
- New Web Service → connected to GitHub repo
- **Root Directory:** `server`
- **Build Command:** `npm install`
- **Start Command:** `node server.js`
- Environment variables added (see table above)
- Auto-deploys on every push to `main`

### 3. Frontend — Vercel
- New Project → same GitHub repo
- **Root Directory:** `client` (auto-detected as Vite)
- `VITE_API_URL` environment variable added under Production
- Auto-deploys on every push to `main`

### 4. CORS
`server/server.js` allows requests from `localhost:5173` (local dev) and any `*.vercel.app` domain, so preview deployments work automatically without needing to update CORS every time:

```js
app.use(cors({
  origin: function (origin, callback) {
    if (!origin) return callback(null, true);
    if (origin === 'http://localhost:5173' || origin.endsWith('.vercel.app')) {
      return callback(null, true);
    }
    return callback(new Error('Not allowed by CORS'));
  },
  credentials: true
}));
```

### 5. Email / OTP delivery
Originally used Gmail SMTP via `nodemailer`, but **Render's free tier blocks outbound SMTP ports (465/587)**, causing `ETIMEDOUT` errors. Switched to **Resend**, which sends email over HTTPS (not blocked) — see `server/utils/email.js`.

> Note: on Resend's free plan without a verified custom domain, emails can only be sent to the account owner's own inbox. To send OTPs to any user's email address, verify a domain at resend.com/domains and update the `from` address in `email.js`.

---

## 🧪 Local Development

**Backend:**
```bash
cd server
npm install
npm run dev
```

**Frontend:**
```bash
cd client
npm install
npm run dev
```

Create a `server/.env` file locally with the same keys listed above (using `localhost` Mongo URI or your Atlas string).

---

## 🐛 Known Issues / Notes

- Free Render backend cold-starts after ~15 min of inactivity (30–60s wake-up delay).
- Resend free plan currently restricted to sending to the account owner's email only, until a domain is verified.
- Account signup requires OTP email verification before login is allowed (`isVerified` flag in the `users` collection).

---

## 📋 Quick Troubleshooting

| Symptom | Likely Cause | Fix |
|---|---|---|
| CORS error in browser console | Frontend URL not allowed in `server.js` | Confirm CORS origin logic covers your Vercel domain |
| "Cannot GET /api/..." | Wrong Root Directory or Start Command on Render | Root Directory = `server`, Start Command = `node server.js` |
| Registration/login hangs then fails | Render free instance waking from sleep | Wait ~60s and retry |
| OTP email never arrives | Email service misconfigured or blocked | Check Render **Logs** tab for errors; confirm `RESEND_API_KEY` is set |
| "Invalid credentials" on login | Wrong password, or account doesn't exist | Check `users` collection in MongoDB Atlas → Browse Collections |
