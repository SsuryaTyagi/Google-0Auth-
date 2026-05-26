# 🔐 Google OAuth 2.0 — Complete Integration Guide
 
> **React (Frontend) + Node.js / Express (Backend) + MongoDB**  
> Step-by-step, zero steps missed. Production-ready.
 
---
 
![Google OAuth Banner](https://developers.google.com/static/identity/images/g-logo.png)
 
![Node.js](https://img.shields.io/badge/Node.js-18%2B-339933?style=for-the-badge&logo=node.js&logoColor=white)
![React](https://img.shields.io/badge/React-18%2B-61DAFB?style=for-the-badge&logo=react&logoColor=black)
![MongoDB](https://img.shields.io/badge/MongoDB-6%2B-47A248?style=for-the-badge&logo=mongodb&logoColor=white)
![Express](https://img.shields.io/badge/Express-4%2B-000000?style=for-the-badge&logo=express&logoColor=white)
![JWT](https://img.shields.io/badge/JWT-Auth-000000?style=for-the-badge&logo=jsonwebtokens&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-blue?style=for-the-badge)
 
---
 
## 📋 Table of Contents
 
1. [How Google OAuth Works (Flow Diagram)](#1-how-google-oauth-works-flow-diagram)
2. [Project Folder Structure](#2-project-folder-structure)
3. [Google Cloud Console Setup](#3-google-cloud-console-setup)
4. [Install All Required Packages](#4-install-all-required-packages)
5. [Environment Variables (.env)](#5-environment-variables-env)
6. [MongoDB User Model](#6-mongodb-user-model)
7. [Passport Strategy Configuration](#7-passport-strategy-configuration)
8. [Authentication Routes](#8-authentication-routes)
9. [Express Server Setup (server.js)](#9-express-server-setup-serverjs)
10. [React Frontend — Login Button](#10-react-frontend--login-button)
11. [React Auth Context](#11-react-auth-context)
12. [Protected Route (Frontend Guard)](#12-protected-route-frontend-guard)
13. [Dashboard Page (After Login)](#13-dashboard-page-after-login)
14. [App.jsx — Final Wiring](#14-appjsx--final-wiring)
15. [JWT + Refresh Token System](#15-jwt--refresh-token-system)
16. [Role-Based Access Control (RBAC)](#16-role-based-access-control-rbac)
17. [Error Handling](#17-error-handling)
18. [Testing the Full Flow](#18-testing-the-full-flow)
19. [Production Deployment Checklist](#19-production-deployment-checklist)
20. [Common Errors & Fixes](#20-common-errors--fixes)
---
 
## 1. How Google OAuth Works (Flow Diagram)
 
```
┌──────────┐     Click "Login"      ┌─────────────┐
│   User   │ ──────────────────────▶│   React App  │
└──────────┘                        └──────┬──────┘
                                           │ Redirect to
                                           │ /auth/google
                                    ┌──────▼──────┐
                                    │  Express +   │
                                    │  Passport    │
                                    └──────┬──────┘
                                           │ Redirect to
                                           │ Google
                                    ┌──────▼──────┐
                                    │   Google    │
                                    │ Consent     │
                                    │  Screen     │
                                    └──────┬──────┘
                                           │ User approves
                                           │ Authorization Code
                                    ┌──────▼──────┐
                                    │  Callback   │
                                    │  Route      │
                                    │ /auth/google│
                                    │ /callback   │
                                    └──────┬──────┘
                                           │
                              ┌────────────┼────────────┐
                              │            │            │
                       ┌──────▼──┐  ┌─────▼────┐ ┌────▼────┐
                       │  Save   │  │  Issue   │ │Redirect │
                       │User in  │  │   JWT    │ │Frontend │
                       │MongoDB  │  │  Token   │ │/dashboard│
                       └─────────┘  └──────────┘ └─────────┘
```
 
**Simple explanation:**
1. User clicks "Sign in with Google"
2. Browser goes to your backend `/auth/google`
3. Passport redirects to Google login page
4. User approves → Google sends a code to your callback URL
5. Passport exchanges code for user profile
6. You save/find user in MongoDB
7. You create a JWT token
8. User is redirected to your frontend dashboard — **logged in!**
---
 
## 2. Project Folder Structure
 
```
my-app/
├── backend/
│   ├── config/
│   │   └── passport.js          ← Passport Google Strategy
│   ├── models/
│   │   └── User.js              ← MongoDB User Schema
│   ├── routes/
│   │   └── auth.js              ← /auth/google, /callback, /me, /logout
│   ├── middleware/
│   │   └── authMiddleware.js    ← JWT verify + role check
│   ├── .env                     ← Secret keys (NEVER commit this)
│   ├── .env.example             ← Template (safe to commit)
│   └── server.js                ← Express entry point
│
├── frontend/
│   ├── src/
│   │   ├── context/
│   │   │   └── AuthContext.jsx  ← Global auth state
│   │   ├── components/
│   │   │   ├── GoogleLoginButton.jsx
│   │   │   └── ProtectedRoute.jsx
│   │   ├── pages/
│   │   │   ├── Login.jsx
│   │   │   └── Dashboard.jsx
│   │   └── App.jsx
│   └── package.json
│
└── README.md
```
 
---
 
## 3. Google Cloud Console Setup
 
> ⚠️ **This is the most important step. Do NOT skip any sub-step.**
 
### Step 3.1 — Create a New Project
 
1. Go to 👉 [https://console.cloud.google.com](https://console.cloud.google.com)
2. Sign in with your Google account
3. Click the **project dropdown** at the top-left (next to "Google Cloud" logo)
![Google Cloud Top Bar](https://i.imgur.com/placeholder-topbar.png)
 
> **Screenshot location:** Top navigation bar → project name dropdown
 
4. Click **"New Project"** button (top right of the popup)
5. Fill in:
   - **Project name:** `MyApp-OAuth` (or any name)
   - **Location:** No organization (for personal projects)
6. Click **Create**
7. Wait ~10 seconds, then select your new project from the dropdown
---
 
### Step 3.2 — Enable the Google+ API / People API
 
1. In the left sidebar → click **"APIs & Services"** → **"Library"**
![API Library Menu](https://i.imgur.com/placeholder-apilibrary.png)
 
> **Screenshot location:** Left sidebar → APIs & Services → Library
 
2. In the search bar, type: `Google People API`
3. Click on **"Google People API"** from results
4. Click the blue **"Enable"** button
5. Also search for **"Google+ API"** → Enable it too
---
 
### Step 3.3 — Configure OAuth Consent Screen
 
> This is what users see before they approve login.
 
1. Left sidebar → **"APIs & Services"** → **"OAuth consent screen"**
![OAuth Consent Screen Menu](https://i.imgur.com/placeholder-consent.png)
 
2. Choose **"External"** → Click **Create**
![External User Type](https://i.imgur.com/placeholder-external.png)
 
3. Fill in the form:
| Field | What to enter |
|-------|---------------|
| **App name** | Your app name (e.g., "MyApp") |
| **User support email** | Your email |
| **Developer contact email** | Your email |

4. Click **Save and Continue**

5. On **Audience** page:
   - Select: ✅ **External**
   - Click **Next**

6. On **Contact Information** page:
   - Verify your email
   - Click **Next**

7. On **Scopes** page → Click **"Add or Remove Scopes"**

8. Search and select these 3 scopes:
   - ✅ `../auth/userinfo.email`
   - ✅ `../auth/userinfo.profile`
   - ✅ `openid`

9. Click **Update** → **Save and Continue**

10. On **Test Users** page:
   - Add your email as a test user

11. Click **Save and Continue** → **Back to Dashboard**
---
 
### Step 3.4 — Create OAuth 2.0 Credentials
 
1. Left sidebar → **"APIs & Services"** → **"Credentials"**
2. Click **"+ Create Credentials"** → **"OAuth 2.0 Client ID"**
![Create Credentials Button](https://i.imgur.com/placeholder-credentials.png)
 
3. Fill in:
| Field | Value |
|-------|-------|
| **Application type** | Web application |
| **Name** | `MyApp Web Client` |
| **Authorized JavaScript origins** | `http://localhost:3000` |
| **Authorized redirect URIs** | `http://localhost:5000/auth/google/callback` |
 
> ⚠️ **CRITICAL:** The redirect URI must match **EXACTLY** what you use in code — same protocol, port, and path.
 
4. Click **Create**
5. A popup appears with your keys:
```
Client ID:      xxxxxxxxxx.apps.googleusercontent.com
Client Secret:  GOCSPX-xxxxxxxxxxxxxxxxxx
```
 
6. **Copy both values** — you will paste them in `.env`
7. Click **OK**
![Credentials Created Popup](https://i.imgur.com/placeholder-popup.png)
 
---
 
## 4. Install All Required Packages
 
### Backend Packages
 
```bash
cd backend
npm init -y
npm install express passport passport-google-oauth20 express-session mongoose dotenv cors jsonwebtoken cookie-parser express-rate-limi
npm install --save-dev nodemon
```
 
### What each package does:
 
| Package | Purpose |
|---------|---------|
| `express` | Web framework for Node.js |
| `passport` | Authentication middleware |
| `passport-google-oauth20` | Google OAuth 2.0 strategy for Passport |
| `express-session` | Session management |
| `mongoose` | MongoDB ODM |
| `dotenv` | Load .env variables |
| `cors` | Allow cross-origin requests from React |
| `jsonwebtoken` | Create and verify JWT tokens |
| `cookie-parser` | Parse cookies from requests |
| `express-rate-limit` | Prevent brute-force attacks |
| `nodemon` | Auto-restart server on file changes (dev only) |
 
### Frontend Packages
 
```bash
cd frontend
npx create-react-app .   # if not already created
npm install react-router-dom axios
```
 
---
 
## 5. Environment Variables (.env)
 
Create a `.env` file in your `backend/` folder:
 
```bash
touch backend/.env
```
 
```env
# ─── Server ─────────────────────────────────────
PORT=5000
NODE_ENV=development
 
# ─── Google OAuth ────────────────────────────────
GOOGLE_CLIENT_ID=your_client_id_here.apps.googleusercontent.com
GOOGLE_CLIENT_SECRET=GOCSPX-your_secret_here
 
# ─── Session ─────────────────────────────────────
SESSION_SECRET=use_a_very_long_random_string_here_minimum_32_chars
 
# ─── JWT ─────────────────────────────────────────
JWT_SECRET=any_long_random_string_min_32_chars
JWT_EXPIRE=7d
 
# ─── Database ────────────────────────────────────
MONGO_URI=mongodb://localhost:27017/myapp_oauth
 
# ─── Frontend URL ────────────────────────────────
CLIENT_URL=http://localhost:3000
```
 
Create `.env.example` (this one is safe to commit to Git):
 
```env
PORT=5000
NODE_ENV=development
GOOGLE_CLIENT_ID=
GOOGLE_CLIENT_SECRET=
SESSION_SECRET=
JWT_SECRET=
JWT_REFRESH_SECRET=
JWT_EXPIRE=15m
JWT_REFRESH_EXPIRE=30d
MONGO_URI=
CLIENT_URL=http://localhost:3000
```
 
Add `.env` to `.gitignore`:
 
```bash
echo ".env" >> .gitignore
echo "node_modules/" >> .gitignore
```
 
> 🔐 **NEVER push your `.env` file to GitHub. Your Google secret will be exposed.**
 
---
 
## 6. MongoDB User Model
 
```bash
mkdir -p backend/models
touch backend/models/User.js
```
 
```javascript
// backend/models/User.js
 
const mongoose = require('mongoose');
 
const UserSchema = new mongoose.Schema(
  {
    googleId: {
      type: String,
      required: true,
      unique: true,
      index: true,
    },
    displayName: {
      type: String,
      required: true,
      trim: true,
    },
    firstName: {
      type: String,
      trim: true,
    },
    lastName: {
      type: String,
      trim: true,
    },
    email: {
      type: String,
      required: true,
      unique: true,
      lowercase: true,
      trim: true,
    },
    avatar: {
      type: String,   // Google profile picture URL
    },
    role: {
      type: String,
      enum: ['user', 'admin', 'moderator'],
      default: 'user',
    },
    isActive: {
      type: Boolean,
      default: true,
    },
    lastLogin: {
      type: Date,
      default: Date.now,
    },
    refreshToken: {
      type: String,   // Store hashed refresh token
      select: false,  // Never return in queries by default
    },
  },
  {
    timestamps: true, // Adds createdAt and updatedAt automatically
  }
);
 
// Virtual: full name
UserSchema.virtual('fullName').get(function () {
  return `${this.firstName} ${this.lastName}`.trim();
});
 
// Method: return safe user object (no sensitive fields)
UserSchema.methods.toSafeObject = function () {
  return {
    id: this._id,
    displayName: this.displayName,
    email: this.email,
    avatar: this.avatar,
    role: this.role,
    createdAt: this.createdAt,
  };
};
 
module.exports = mongoose.model('User', UserSchema);
```
 
---
 
## 7. Passport Strategy Configuration
 
```bash
mkdir -p backend/config
touch backend/config/passport.js
```
 
```javascript
// backend/config/passport.js
 
const passport = require('passport');
const { Strategy: GoogleStrategy } = require('passport-google-oauth20');
const User = require('../models/User');
 
passport.use(
  new GoogleStrategy(
    {
      clientID:     process.env.GOOGLE_CLIENT_ID,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET,
      callbackURL:  '/auth/google/callback',
      // Pass back the entire request to the callback
      passReqToCallback: true,
    },
    async (req, accessToken, refreshToken, profile, done) => {
      try {
        // Step 1: Check if user already exists in DB
        let user = await User.findOne({ googleId: profile.id });
 
        if (user) {
          // Step 2a: User exists → update last login time
          user.lastLogin = new Date();
          await user.save();
          return done(null, user);
        }
 
        // Step 2b: New user → create in database
        const newUser = await User.create({
          googleId:    profile.id,
          displayName: profile.displayName,
          firstName:   profile.name?.givenName  || '',
          lastName:    profile.name?.familyName || '',
          email:       profile.emails[0].value,
          avatar:      profile.photos[0]?.value || '',
        });
 
        return done(null, newUser);
 
      } catch (error) {
        console.error('Passport Google Strategy Error:', error);
        return done(error, null);
      }
    }
  )
);
 
// Serialize: store only user ID in session
passport.serializeUser((user, done) => {
  done(null, user.id);
});
 
// Deserialize: fetch full user from DB using session ID
passport.deserializeUser(async (id, done) => {
  try {
    const user = await User.findById(id);
    done(null, user);
  } catch (error) {
    done(error, null);
  }
});
 
module.exports = passport;
```
 
---
 
## 8. Authentication Routes
 
```bash
mkdir -p backend/routes
touch backend/routes/auth.js
```
 
```javascript
// backend/routes/auth.js
 
const express    = require('express');
const passport   = require('passport');
const jwt        = require('jsonwebtoken');
const router     = express.Router();
 
// ─── Helper: Generate Tokens ──────────────────────────────────
const generateAccessToken = (user) => {
  return jwt.sign(
    { id: user._id, email: user.email, role: user.role },
    process.env.JWT_SECRET,
    { expiresIn: process.env.JWT_EXPIRE || '15m' }
  );
};
 
const generateRefreshToken = (user) => {
  return jwt.sign(
    { id: user._id },
    process.env.JWT_REFRESH_SECRET,
    { expiresIn: process.env.JWT_REFRESH_EXPIRE || '30d' }
  );
};
 
const sendTokenCookies = (res, accessToken, refreshToken) => {
  const isProduction = process.env.NODE_ENV === 'production';
 
  res.cookie('accessToken', accessToken, {
    httpOnly: true,
    secure:   isProduction,
    sameSite: isProduction ? 'strict' : 'lax',
    maxAge:   15 * 60 * 1000, // 15 minutes
  });
 
  res.cookie('refreshToken', refreshToken, {
    httpOnly: true,
    secure:   isProduction,
    sameSite: isProduction ? 'strict' : 'lax',
    maxAge:   30 * 24 * 60 * 60 * 1000, // 30 days
    path:     '/auth/refresh', // Only sent to refresh endpoint
  });
};
 
// ─── Route 1: Start Google OAuth ─────────────────────────────
// GET /auth/google
// User lands here when they click "Sign in with Google"
router.get(
  '/google',
  passport.authenticate('google', {
    scope: ['profile', 'email'],
    prompt: 'select_account', // Always show account picker
  })
);
 
// ─── Route 2: Google Callback ────────────────────────────────
// GET /auth/google/callback
// Google redirects here after user approves
router.get(
  '/google/callback',
  passport.authenticate('google', {
    failureRedirect: `${process.env.CLIENT_URL}/login?error=auth_failed`,
    session: false,
  }),
  (req, res) => {
    try {
      const user = req.user;
 
      // Generate tokens
      const accessToken  = generateAccessToken(user);
      const refreshToken = generateRefreshToken(user);
 
      // Set tokens as httpOnly cookies
      sendTokenCookies(res, accessToken, refreshToken);
 
      // Redirect to frontend dashboard
      res.redirect(`${process.env.CLIENT_URL}/dashboard`);
 
    } catch (error) {
      console.error('Callback Error:', error);
      res.redirect(`${process.env.CLIENT_URL}/login?error=server_error`);
    }
  }
);
 
// ─── Route 3: Get Current User ───────────────────────────────
// GET /auth/me
// Frontend calls this to check if user is logged in
router.get('/me', requireAuth, async (req, res) => {
  try {
    const User = require('../models/User');
    const user = await User.findById(req.user.id).select('-refreshToken');
 
    if (!user) {
      return res.status(404).json({ success: false, error: 'User not found' });
    }
 
    res.json({ success: true, user: user.toSafeObject() });
  } catch (error) {
    res.status(500).json({ success: false, error: 'Server error' });
  }
});
 
// ─── Route 4: Refresh Access Token ──────────────────────────
// POST /auth/refresh
router.post('/refresh', (req, res) => {
  const token = req.cookies.refreshToken;
 
  if (!token) {
    return res.status(401).json({ success: false, error: 'No refresh token' });
  }
 
  try {
    const payload      = jwt.verify(token, process.env.JWT_REFRESH_SECRET);
    const User         = require('../models/User');
    
    User.findById(payload.id).then((user) => {
      if (!user || !user.isActive) {
        return res.status(401).json({ success: false, error: 'User not found' });
      }
 
      const newAccessToken = generateAccessToken(user);
      const isProduction   = process.env.NODE_ENV === 'production';
 
      res.cookie('accessToken', newAccessToken, {
        httpOnly: true,
        secure:   isProduction,
        sameSite: isProduction ? 'strict' : 'lax',
        maxAge:   15 * 60 * 1000,
      });
 
      res.json({ success: true, message: 'Token refreshed' });
    });
 
  } catch (error) {
    res.clearCookie('accessToken');
    res.clearCookie('refreshToken');
    res.status(401).json({ success: false, error: 'Invalid refresh token' });
  }
});
 
// ─── Route 5: Logout ────────────────────────────────────────
// POST /auth/logout
router.post('/logout', (req, res) => {
  res.clearCookie('accessToken');
  res.clearCookie('refreshToken', { path: '/auth/refresh' });
  res.json({ success: true, message: 'Logged out successfully' });
});
 
// ─── Middleware: Require Authentication ──────────────────────
function requireAuth(req, res, next) {
  const token = req.cookies?.accessToken
    || req.headers.authorization?.split(' ')[1];
 
  if (!token) {
    return res.status(401).json({ success: false, error: 'Not authenticated' });
  }
 
  try {
    req.user = jwt.verify(token, process.env.JWT_SECRET);
    next();
  } catch (error) {
    if (error.name === 'TokenExpiredError') {
      return res.status(401).json({ success: false, error: 'Token expired' });
    }
    res.status(401).json({ success: false, error: 'Invalid token' });
  }
}
 
module.exports = { router, requireAuth };
```
 
---
 
## 9. Express Server Setup (server.js)
 
```javascript
// backend/server.js
 
require('dotenv').config();
 
const express      = require('express');
const session      = require('express-session');
const passport     = require('passport');
const mongoose     = require('mongoose');
const cors         = require('cors');
const cookieParser = require('cookie-parser');
const rateLimit    = require('express-rate-limit');
 
// Load passport configuration
require('./config/passport');
 
// Import routes
const { router: authRouter } = require('./routes/auth');
 
const app = express();
 
// ─── Security: Rate Limiting ─────────────────────────────────
const authLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max:      20,              // max 20 auth requests per window
  message:  { error: 'Too many requests, please try again later.' },
  standardHeaders: true,
  legacyHeaders:   false,
});
 
// ─── Middleware ───────────────────────────────────────────────
app.use(cors({
  origin:      process.env.CLIENT_URL,   // Only allow your React app
  credentials: true,                     // Allow cookies
  methods:     ['GET', 'POST', 'PUT', 'DELETE', 'PATCH'],
  allowedHeaders: ['Content-Type', 'Authorization'],
}));
 
app.use(express.json({ limit: '10kb' }));
app.use(express.urlencoded({ extended: true }));
app.use(cookieParser());
 
// Session (needed by Passport)
app.use(session({
  secret:            process.env.SESSION_SECRET,
  resave:            false,
  saveUninitialized: false,
  cookie: {
    secure:   process.env.NODE_ENV === 'production',
    httpOnly: true,
    maxAge:   24 * 60 * 60 * 1000, // 1 day
    sameSite: process.env.NODE_ENV === 'production' ? 'strict' : 'lax',
  },
}));
 
// Initialize Passport
app.use(passport.initialize());
app.use(passport.session());
 
// ─── Routes ───────────────────────────────────────────────────
app.use('/auth', authLimiter, authRouter);
 
// Health check
app.get('/health', (req, res) => {
  res.json({ status: 'OK', timestamp: new Date().toISOString() });
});
 
// 404 handler
app.use((req, res) => {
  res.status(404).json({ error: 'Route not found' });
});
 
// Global error handler
app.use((err, req, res, next) => {
  console.error('Global Error:', err.stack);
  res.status(err.status || 500).json({
    error: process.env.NODE_ENV === 'production'
      ? 'Internal server error'
      : err.message,
  });
});
 
// ─── Connect DB & Start Server ────────────────────────────────
const startServer = async () => {
  try {
    await mongoose.connect(process.env.MONGO_URI, {
      useNewUrlParser:    true,
      useUnifiedTopology: true,
    });
    console.log('✅ MongoDB connected successfully');
 
    const PORT = process.env.PORT || 5000;
    app.listen(PORT, () => {
      console.log(`✅ Server running on http://localhost:${PORT}`);
      console.log(`✅ Auth URL: http://localhost:${PORT}/auth/google`);
    });
 
  } catch (error) {
    console.error('❌ Failed to connect to MongoDB:', error.message);
    process.exit(1);
  }
};
 
startServer();
```
 
Add to `backend/package.json`:
 
```json
{
  "scripts": {
    "start":   "node server.js",
    "dev":     "nodemon server.js",
    "test":    "echo \"Error: no test specified\" && exit 1"
  }
}
```
 
---
 
## 10. React Frontend — Login Button
 
```jsx
// frontend/src/components/GoogleLoginButton.jsx
 
import React from 'react';
 
export default function GoogleLoginButton({ text = 'Sign in with Google' }) {
  const handleGoogleLogin = () => {
    // Redirect the browser to the backend OAuth route
    window.location.href = `${process.env.REACT_APP_API_URL}/auth/google`;
  };
 
  return (
    <button
      onClick={handleGoogleLogin}
      style={{
        display:        'flex',
        alignItems:     'center',
        justifyContent: 'center',
        gap:            '12px',
        padding:        '12px 24px',
        background:     '#ffffff',
        color:          '#1f1f1f',
        border:         '1px solid #dadce0',
        borderRadius:   '8px',
        fontSize:       '16px',
        fontWeight:     '500',
        cursor:         'pointer',
        transition:     'box-shadow 0.2s ease',
        minWidth:       '240px',
      }}
      onMouseEnter={(e) => e.target.style.boxShadow = '0 2px 8px rgba(0,0,0,0.2)'}
      onMouseLeave={(e) => e.target.style.boxShadow = 'none'}
    >
      {/* Official Google icon SVG */}
      <svg width="20" height="20" viewBox="0 0 48 48">
        <path fill="#EA4335" d="M24 9.5c3.54 0 6.71 1.22 9.21 3.6l6.85-6.85C35.9 2.38 30.47 0 24 0 14.62 0 6.51 5.38 2.56 13.22l7.98 6.19C12.43 13.72 17.74 9.5 24 9.5z"/>
        <path fill="#4285F4" d="M46.98 24.55c0-1.57-.15-3.09-.38-4.55H24v9.02h12.94c-.58 2.96-2.26 5.48-4.78 7.18l7.73 6c4.51-4.18 7.09-10.36 7.09-17.65z"/>
        <path fill="#FBBC05" d="M10.53 28.59c-.48-1.45-.76-2.99-.76-4.59s.27-3.14.76-4.59l-7.98-6.19C.92 16.46 0 20.12 0 24c0 3.88.92 7.54 2.56 10.78l7.97-6.19z"/>
        <path fill="#34A853" d="M24 48c6.48 0 11.93-2.13 15.89-5.81l-7.73-6c-2.15 1.45-4.92 2.3-8.16 2.3-6.26 0-11.57-4.22-13.47-9.91l-7.98 6.19C6.51 42.62 14.62 48 24 48z"/>
      </svg>
      {text}
    </button>
  );
}
```
 
Create `frontend/src/pages/Login.jsx`:
 
```jsx
// frontend/src/pages/Login.jsx
 
import React, { useEffect } from 'react';
import { useNavigate, useSearchParams } from 'react-router-dom';
import { useAuth } from '../context/AuthContext';
import GoogleLoginButton from '../components/GoogleLoginButton';
 
export default function Login() {
  const { user }           = useAuth();
  const navigate           = useNavigate();
  const [searchParams]     = useSearchParams();
  const error              = searchParams.get('error');
 
  // If already logged in, redirect to dashboard
  useEffect(() => {
    if (user) navigate('/dashboard');
  }, [user, navigate]);
 
  const errorMessages = {
    auth_failed:  'Google login failed. Please try again.',
    server_error: 'Something went wrong on our end. Please try again.',
  };
 
  return (
    <div style={{ display: 'flex', flexDirection: 'column', alignItems: 'center', justifyContent: 'center', minHeight: '100vh', background: '#f8f9fa' }}>
      <div style={{ background: 'white', padding: '48px 40px', borderRadius: '16px', boxShadow: '0 4px 24px rgba(0,0,0,0.08)', textAlign: 'center', maxWidth: '400px', width: '100%' }}>
        <h1 style={{ fontSize: '28px', fontWeight: '700', margin: '0 0 8px' }}>Welcome Back</h1>
        <p style={{ color: '#666', margin: '0 0 32px' }}>Sign in to your account to continue</p>
 
        {error && (
          <div style={{ background: '#FEF2F2', color: '#DC2626', padding: '12px 16px', borderRadius: '8px', marginBottom: '24px', fontSize: '14px' }}>
            ⚠️ {errorMessages[error] || 'An error occurred. Please try again.'}
          </div>
        )}
 
        <GoogleLoginButton />
 
        <p style={{ marginTop: '24px', fontSize: '12px', color: '#999' }}>
          By signing in, you agree to our Terms of Service and Privacy Policy.
        </p>
      </div>
    </div>
  );
}
```
 
---
 
## 11. React Auth Context
 
```bash
mkdir -p frontend/src/context
touch frontend/src/context/AuthContext.jsx
```
 
```jsx
// frontend/src/context/AuthContext.jsx
 
import React, { createContext, useContext, useState, useEffect, useCallback } from 'react';
 
const AuthContext = createContext(null);
 
const API_URL = process.env.REACT_APP_API_URL || 'http://localhost:5000';
 
export function AuthProvider({ children }) {
  const [user,    setUser]    = useState(null);
  const [loading, setLoading] = useState(true);
  const [error,   setError]   = useState(null);
 
  // ── Check if user is logged in (on app load) ──────────────
  const checkAuth = useCallback(async () => {
    try {
      const response = await fetch(`${API_URL}/auth/me`, {
        credentials: 'include',  // ← IMPORTANT: sends cookies
      });
 
      if (response.ok) {
        const data = await response.json();
        setUser(data.user);
      } else if (response.status === 401) {
        // Try refreshing the token
        await refreshToken();
      } else {
        setUser(null);
      }
    } catch (err) {
      console.error('Auth check failed:', err);
      setUser(null);
    } finally {
      setLoading(false);
    }
  }, []);
 
  // ── Refresh access token ──────────────────────────────────
  const refreshToken = async () => {
    try {
      const response = await fetch(`${API_URL}/auth/refresh`, {
        method:      'POST',
        credentials: 'include',
      });
 
      if (response.ok) {
        // Token refreshed, try /me again
        const meResponse = await fetch(`${API_URL}/auth/me`, {
          credentials: 'include',
        });
        if (meResponse.ok) {
          const data = await meResponse.json();
          setUser(data.user);
        }
      } else {
        setUser(null);
      }
    } catch (err) {
      setUser(null);
    }
  };
 
  // ── Logout ────────────────────────────────────────────────
  const logout = async () => {
    try {
      await fetch(`${API_URL}/auth/logout`, {
        method:      'POST',
        credentials: 'include',
      });
    } catch (err) {
      console.error('Logout error:', err);
    } finally {
      setUser(null);
    }
  };
 
  useEffect(() => {
    checkAuth();
  }, [checkAuth]);
 
  const value = {
    user,
    loading,
    error,
    logout,
    refreshToken,
    isAuthenticated: !!user,
    isAdmin:         user?.role === 'admin',
  };
 
  return (
    <AuthContext.Provider value={value}>
      {children}
    </AuthContext.Provider>
  );
}
 
// Custom hook — use this in any component
export const useAuth = () => {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used inside <AuthProvider>');
  }
  return context;
};
```
 
---
 
## 12. Protected Route (Frontend Guard)
 
```jsx
// frontend/src/components/ProtectedRoute.jsx
 
import React from 'react';
import { Navigate, useLocation } from 'react-router-dom';
import { useAuth } from '../context/AuthContext';
 
export default function ProtectedRoute({
  children,
  requiredRole = null,   // Optional: 'admin', 'moderator', etc.
}) {
  const { user, loading, isAuthenticated } = useAuth();
  const location = useLocation();
 
  // Show loading spinner while checking auth
  if (loading) {
    return (
      <div style={{ display: 'flex', justifyContent: 'center', alignItems: 'center', height: '100vh' }}>
        <div style={{ textAlign: 'center' }}>
          <div style={{ fontSize: '32px', marginBottom: '16px' }}>⏳</div>
          <p>Checking authentication...</p>
        </div>
      </div>
    );
  }
 
  // Not logged in → redirect to login, remember where they came from
  if (!isAuthenticated) {
    return <Navigate to="/login" state={{ from: location }} replace />;
  }
 
  // Logged in but wrong role
  if (requiredRole && user.role !== requiredRole) {
    return <Navigate to="/unauthorized" replace />;
  }
 
  return children;
}
```
 
---
 
## 13. Dashboard Page (After Login)
 
```jsx
// frontend/src/pages/Dashboard.jsx
 
import React from 'react';
import { useAuth } from '../context/AuthContext';
 
export default function Dashboard() {
  const { user, logout, isAdmin } = useAuth();
 
  return (
    <div style={{ maxWidth: '800px', margin: '0 auto', padding: '32px 24px' }}>
 
      {/* Profile Card */}
      <div style={{ display: 'flex', alignItems: 'center', gap: '16px', background: 'white', padding: '24px', borderRadius: '16px', boxShadow: '0 2px 12px rgba(0,0,0,0.08)', marginBottom: '24px' }}>
        <img
          src={user.avatar}
          alt={user.displayName}
          style={{ width: '72px', height: '72px', borderRadius: '50%', border: '3px solid #e8f0fe' }}
        />
        <div>
          <h1 style={{ margin: '0 0 4px', fontSize: '22px' }}>
            Welcome, {user.displayName}! 👋
          </h1>
          <p style={{ margin: '0 0 8px', color: '#666' }}>{user.email}</p>
          <span style={{
            background: isAdmin ? '#FEF3C7' : '#E8F5E9',
            color:      isAdmin ? '#92400E' : '#2E7D32',
            padding:    '2px 10px',
            borderRadius: '99px',
            fontSize:   '12px',
            fontWeight: '600',
          }}>
            {user.role.toUpperCase()}
          </span>
        </div>
      </div>
 
      {/* Stats Grid */}
      <div style={{ display: 'grid', gridTemplateColumns: 'repeat(3, 1fr)', gap: '16px', marginBottom: '24px' }}>
        {[
          { label: 'Account Type', value: 'Google OAuth' },
          { label: 'Role',         value: user.role },
          { label: 'Member Since', value: new Date(user.createdAt).getFullYear() },
        ].map((stat) => (
          <div key={stat.label} style={{ background: 'white', padding: '20px', borderRadius: '12px', boxShadow: '0 2px 8px rgba(0,0,0,0.06)', textAlign: 'center' }}>
            <p style={{ margin: '0 0 4px', fontSize: '13px', color: '#888' }}>{stat.label}</p>
            <p style={{ margin: 0, fontSize: '18px', fontWeight: '600' }}>{stat.value}</p>
          </div>
        ))}
      </div>
 
      {/* Logout Button */}
      <button
        onClick={logout}
        style={{ padding: '12px 24px', background: '#DC2626', color: 'white', border: 'none', borderRadius: '8px', fontSize: '15px', fontWeight: '500', cursor: 'pointer' }}
      >
        Sign Out
      </button>
    </div>
  );
}
```
 
---
 
## 14. App.jsx — Final Wiring
 
Create `frontend/.env`:
 
```env
REACT_APP_API_URL=http://localhost:5000
```
 
```jsx
// frontend/src/App.jsx
 
import React from 'react';
import { BrowserRouter, Routes, Route, Navigate } from 'react-router-dom';
import { AuthProvider } from './context/AuthContext';
import ProtectedRoute from './components/ProtectedRoute';
import Login     from './pages/Login';
import Dashboard from './pages/Dashboard';
 
// Unauthorized page
const Unauthorized = () => (
  <div style={{ textAlign: 'center', padding: '60px' }}>
    <h2>🚫 Access Denied</h2>
    <p>You don't have permission to view this page.</p>
    <a href="/dashboard">Go back</a>
  </div>
);
 
export default function App() {
  return (
    <AuthProvider>
      <BrowserRouter>
        <Routes>
          {/* Public routes */}
          <Route path="/"             element={<Navigate to="/dashboard" replace />} />
          <Route path="/login"        element={<Login />} />
          <Route path="/unauthorized" element={<Unauthorized />} />
 
          {/* Protected routes — any logged-in user */}
          <Route
            path="/dashboard"
            element={
              <ProtectedRoute>
                <Dashboard />
              </ProtectedRoute>
            }
          />
 
          {/* Admin-only route */}
          <Route
            path="/admin"
            element={
              <ProtectedRoute requiredRole="admin">
                <div><h2>Admin Panel</h2></div>
              </ProtectedRoute>
            }
          />
 
          {/* 404 */}
          <Route path="*" element={<div style={{ textAlign: 'center', padding: '60px' }}><h2>404 — Page Not Found</h2></div>} />
        </Routes>
      </BrowserRouter>
    </AuthProvider>
  );
}
```
 
---
 
## 15. JWT + Refresh Token System
 
> The access token is short-lived (15 min). The refresh token is long-lived (30 days). This keeps your app secure while keeping users logged in.
 
```
Access Token:   Short-lived (15min)  ─── Sent on every API request
Refresh Token:  Long-lived (30 days) ─── Stored in httpOnly cookie, only sent to /auth/refresh
```
 
**How auto-refresh works in the frontend:**
 
```jsx
// frontend/src/api/axiosInstance.js
 
import axios from 'axios';
 
const api = axios.create({
  baseURL:     process.env.REACT_APP_API_URL,
  withCredentials: true, // Always send cookies
});
 
// Response interceptor: auto-refresh on 401
api.interceptors.response.use(
  (response) => response, // Success: pass through
  async (error) => {
    const originalRequest = error.config;
 
    // If 401 and not already retrying
    if (error.response?.status === 401 && !originalRequest._retry) {
      originalRequest._retry = true;
 
      try {
        // Attempt token refresh
        await axios.post(
          `${process.env.REACT_APP_API_URL}/auth/refresh`,
          {},
          { withCredentials: true }
        );
 
        // Retry the original request
        return api(originalRequest);
 
      } catch (refreshError) {
        // Refresh failed → redirect to login
        window.location.href = '/login';
        return Promise.reject(refreshError);
      }
    }
 
    return Promise.reject(error);
  }
);
 
export default api;
```
 
---
 
## 16. Role-Based Access Control (RBAC)
 
```javascript
// backend/middleware/authMiddleware.js
 
const jwt = require('jsonwebtoken');
 
// ─── Middleware 1: Verify JWT ────────────────────────────────
exports.requireAuth = (req, res, next) => {
  const token = req.cookies?.accessToken
    || req.headers.authorization?.split(' ')[1];
 
  if (!token) {
    return res.status(401).json({ success: false, error: 'Authentication required' });
  }
 
  try {
    req.user = jwt.verify(token, process.env.JWT_SECRET);
    next();
  } catch (error) {
    if (error.name === 'TokenExpiredError') {
      return res.status(401).json({ success: false, error: 'Token expired', code: 'TOKEN_EXPIRED' });
    }
    res.status(401).json({ success: false, error: 'Invalid token' });
  }
};
 
// ─── Middleware 2: Check Role ────────────────────────────────
exports.requireRole = (...roles) => {
  return (req, res, next) => {
    if (!req.user) {
      return res.status(401).json({ success: false, error: 'Not authenticated' });
    }
 
    if (!roles.includes(req.user.role)) {
      return res.status(403).json({
        success: false,
        error:   `Access denied. Required role: ${roles.join(' or ')}`,
      });
    }
 
    next();
  };
};
 
// ─── Usage Example in Routes ────────────────────────────────
/*
const { requireAuth, requireRole } = require('../middleware/authMiddleware');
 
// Any logged-in user
router.get('/profile',          requireAuth,                    getProfile);
 
// Only admins
router.get('/admin/users',      requireAuth, requireRole('admin'),              getAllUsers);
 
// Admins and moderators
router.delete('/posts/:id',     requireAuth, requireRole('admin', 'moderator'), deletePost);
 
// Promote user to admin (super admin only)
router.patch('/users/:id/role', requireAuth, requireRole('admin'),              updateUserRole);
*/
```
 
---
 
## 17. Error Handling
 
### Backend global error handler
 
```javascript
// In server.js (already included above)
 
app.use((err, req, res, next) => {
  // Log error (use a proper logger like winston in production)
  console.error({
    message:   err.message,
    stack:     err.stack,
    url:       req.url,
    method:    req.method,
    timestamp: new Date().toISOString(),
  });
 
  res.status(err.status || 500).json({
    success: false,
    error:   process.env.NODE_ENV === 'production'
      ? 'Something went wrong. Please try again.'
      : err.message,
  });
});
```
 
### Frontend error boundary
 
```jsx
// frontend/src/components/ErrorBoundary.jsx
 
import React from 'react';
 
export default class ErrorBoundary extends React.Component {
  state = { hasError: false, error: null };
 
  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }
 
  render() {
    if (this.state.hasError) {
      return (
        <div style={{ textAlign: 'center', padding: '60px' }}>
          <h2>Something went wrong</h2>
          <p>{this.state.error?.message}</p>
          <button onClick={() => window.location.href = '/'}>Go Home</button>
        </div>
      );
    }
    return this.props.children;
  }
}
```
 
Wrap your app:
 
```jsx
// In index.js
import ErrorBoundary from './components/ErrorBoundary';
 
root.render(
  <ErrorBoundary>
    <App />
  </ErrorBoundary>
);
```
 
---
 
## 18. Testing the Full Flow
 
### Start both servers
 
**Terminal 1 — Backend:**
```bash
cd backend
npm run dev
# ✅ MongoDB connected successfully
# ✅ Server running on http://localhost:5000
# ✅ Auth URL: http://localhost:5000/auth/google
```
 
**Terminal 2 — Frontend:**
```bash
cd frontend
npm start
# React app running on http://localhost:3000
```
 
### Manual test steps
 
| # | Action | Expected Result |
|---|--------|-----------------|
| 1 | Go to `http://localhost:3000/login` | Login page shows with Google button |
| 2 | Click "Sign in with Google" | Redirected to Google account picker |
| 3 | Select your Google account | Redirected to `/auth/google/callback` |
| 4 | Callback processes | JWT cookies are set, redirected to `/dashboard` |
| 5 | Check MongoDB | New user document created with googleId |
| 6 | Refresh the page | User stays logged in (cookie persists) |
| 7 | Call `GET /auth/me` | Returns logged-in user data |
| 8 | Click "Sign Out" | Cookies cleared, redirected to `/login` |
| 9 | Try to access `/dashboard` directly | Redirected to `/login` |
 
### Test with curl (backend only)
 
```bash
# Test health check
curl http://localhost:5000/health
 
# Test /me without token (should return 401)
curl http://localhost:5000/auth/me
 
# Test /me with invalid token (should return 401)
curl -H "Authorization: Bearer invalidtoken" http://localhost:5000/auth/me
```
 
---
 
## 19. Production Deployment Checklist
 
```
BEFORE GOING LIVE — Check every single item:
 
GOOGLE CONSOLE
  ☐ Add your production domain to "Authorized JavaScript origins"
      Example: https://myapp.com
  ☐ Add production callback to "Authorized redirect URIs"
      Example: https://myapp.com/auth/google/callback (note: https)
  ☐ Remove http://localhost entries from production credentials
  ☐ Publish OAuth consent screen (remove "Testing" status)
 
ENVIRONMENT VARIABLES
  ☐ Set NODE_ENV=production
  ☐ Update CLIENT_URL to your real domain
  ☐ Use strong random secrets (min 64 characters) for JWT_SECRET
  ☐ Use strong random secret for SESSION_SECRET
  ☐ Use MongoDB Atlas or a managed DB (not localhost)
  ☐ Never commit .env to version control
 
SECURITY
  ☐ All cookies have secure: true (requires HTTPS)
  ☐ All cookies have sameSite: 'strict'
  ☐ CORS origin is set to exact production domain (no wildcard *)
  ☐ HTTPS is enforced on your server (use nginx or load balancer)
  ☐ Rate limiting is active on auth endpoints
  ☐ Helmet.js is installed: npm install helmet
      Add: app.use(require('helmet')()) in server.js
 
REACT FRONTEND
  ☐ REACT_APP_API_URL points to production backend URL
  ☐ Run: npm run build (creates /build folder)
  ☐ Serve /build with nginx or deploy to Vercel/Netlify
 
DATABASE
  ☐ MongoDB Atlas free tier or paid cluster
  ☐ IP whitelist configured
  ☐ DB credentials in environment variable (not in code)
  ☐ Backup strategy in place
 
MONITORING
  ☐ Auth success/failure events are logged
  ☐ Error tracking tool set up (e.g., Sentry)
  ☐ Uptime monitoring configured
```
 
---
 
## 20. Common Errors & Fixes
 
| Error | Cause | Fix |
|-------|-------|-----|
| `redirect_uri_mismatch` | Callback URL in code doesn't match Google Console | Must be **exactly** the same: same `http/https`, same port, same path, no trailing slash difference |
| `invalid_client` | Wrong `CLIENT_ID` or `CLIENT_SECRET` | Re-copy from Google Console → Credentials |
| `401 Not authenticated` on `/auth/me` | Cookie not being sent | Add `credentials: 'include'` to every `fetch()` call |
| CORS error on API calls | Missing `credentials: true` in CORS config | Set `credentials: true` in backend `cors()` config, use exact origin |
| `User not found` after OAuth | `deserializeUser` failing | Check MongoDB connection is active, `User` model imported in passport.js |
| Login loop (redirect back to /login) | Cookie not being set | Check `secure` flag — needs `https` in production; in dev use `secure: false` |
| `TokenExpiredError` | JWT has expired | Implement refresh token endpoint (Step 15) and auto-refresh logic |
| Google shows "This app isn't verified" | Consent screen not published | Fine for testing; publish screen in Google Console for production |
| `Cannot read property 'id' of undefined` | Passport strategy returning `null` user | Check `done(null, user)` is called correctly in strategy |
| Session not persisting | Missing `saveUninitialized` / bad cookie config | Check `express-session` config; ensure `resave: false`, `saveUninitialized: false` |
 
---
 
## 🎉 Complete — Your Google OAuth is Ready!
 
```
Frontend (React)    http://localhost:3000
Backend (Express)   http://localhost:5000
Auth Start URL      http://localhost:5000/auth/google
Callback URL        http://localhost:5000/auth/google/callback
Get Current User    GET  http://localhost:5000/auth/me
Refresh Token       POST http://localhost:5000/auth/refresh
Logout              POST http://localhost:5000/auth/logout
Health Check        GET  http://localhost:5000/health
```
 
---
 
*Built with ❤️ — React + Node.js + Passport + MongoDB + JWT*
