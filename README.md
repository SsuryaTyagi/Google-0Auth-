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
frontend/
│
├── src/
│   │
│   ├── assets/                # images, icons, fonts
│   │
│   ├── features/              # feature-based architecture
│   │   │
│   │   ├── auth/
│   │   │   │
│   │   │   ├── components/    # reusable UI components (auth specific)
│   │   │   │   ├── Button.jsx
│   │   │   │   ├── GoogleLoginButton.jsx
│   │   │   │   ├── Input.jsx
│   │   │   │   ├── ProtectedRoute.jsx
│   │   │   │
│   │   │   ├── hooks/         # custom hooks
│   │   │   │   └── useAuth.js
│   │   │   │
│   │   │   ├── pages/         # pages/screens
│   │   │   │   ├── LoginPage.jsx
│   │   │   │   ├── RegisterPage.jsx
│   │   │   │
│   │   │   ├── services/      # API calls
│   │   │   │   └── auth.api.js
│   │   │   │
│   │   │   ├── context/       # context API
│   │   │   │   └── AuthContext.jsx
│   │   │   │
│   │   │   ├── styles/        # auth specific styles
│   │   │   │   └── auth.scss
│   │   │   │
│   │   │   └── index.js       # barrel export (optional)
│   │
│   ├── App.jsx
│   ├── main.jsx
│   ├── index.css
│
├── .gitignore
├── package.json
├── vite.config.js
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
const bcrypt   = require('bcryptjs');

const UserSchema = new mongoose.Schema(
  {
    name: {
      type:     String,
      required: true,
      trim:     true,
    },
    email: {
      type:      String,
      required:  true,
      unique:    true,
      lowercase: true,
      trim:      true,
    },
    password: {
      type:   String,
      select: false, 
    },
    googleId: {
      type: String,
    },
    avatar: {
      type: String,
    },
    role: {
      type:    String,
      enum:    ['user', 'admin'],
      default: 'user',
    },

    // ── Extensible Fields (Uncomment to use) ─────────────────
    // phone:   { type: String },
    // address: { type: String },
    // city:    { type: String },
    // pincode: { type: String },
    // ─────────────────────────────────────────────────────────
  },
  { timestamps: true }
);

//hash password befor save db
UserSchema.pre('save', async function (next) {
  // Sirf tab hash karo jab password naya/changed ho
  if (!this.isModified('password') || !this.password) return next();
  this.password = await bcrypt.hash(this.password, 12);
  next();
});

// Password check method
UserSchema.methods.matchPassword = async function (enteredPassword) {
  return bcrypt.compare(enteredPassword, this.password);
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
 
const passport       = require('passport');
const GoogleStrategy = require('passport-google-oauth20').Strategy;
const User           = require('../models/User');

passport.use(
  new GoogleStrategy(
    {
      clientID:     process.env.GOOGLE_CLIENT_ID,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET,
      callbackURL:  'Your_backend_url/api/auth/google/callback',
    },
    async (accessToken, refreshToken, profile, done) => {
      try {
      
        let user = await User.findOne({ googleId: profile.id });

        if (user) return done(null, user);

        
        user = await User.findOne({ email: profile.emails[0].value });

        if (user) {
          user.googleId = profile.id;
          if (!user.avatar) user.avatar = profile.photos[0]?.value;
          await user.save();
          return done(null, user);
        }

        
        user = await User.create({
          name:     profile.displayName,
          email:    profile.emails[0].value,
          googleId: profile.id,
          avatar:   profile.photos[0]?.value || '',
        });

        return done(null, user);
      } catch (error) {
        return done(error, null);
      }
    }
  )
);

passport.serializeUser((user, done) => done(null, user.id));

passport.deserializeUser(async (id, done) => {
  try {
    const user = await User.findById(id);
    done(null, user);
  } catch (err) {
    done(err, null);
  }
});
```
 
---

 ## 8. Auth.middleware · JS
 
```bash
mkdir -p backend/
touch backend/middleware/auth.middleware.js
```
 
```javascript
// backend/middleware/auth.middleware.js
 
const jwt  = require('jsonwebtoken');
const User = require('../models/User');

const protect = async (req, res, next) => {
  const token = req.cookies?.token;

  if (!token) {
    return res.status(401).json({ success: false, message: 'Please login first' });
  }

  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    req.user      = await User.findById(decoded.id);
    next();
  } catch {
    res.status(401).json({ success: false, message: 'Invalid or expired token' });
  }
};

// Role check middleware
const requireRole = (...roles) => (req, res, next) => {
  if (!roles.includes(req.user?.role)) {
    return res.status(403).json({ success: false, message: 'Access denied' });
  }
  next();
};

module.exports = { protect, requireRole };
```
 
---

## .9 Authentication Routes
 
```bash
mkdir -p backend/routes
touch backend/routes/auth.js
```
 
```javascript
// backend/routes/auth.js
 
const express  = require('express');
const jwt      = require('jsonwebtoken');
const passport = require('passport');
const User     = require('../models/User');
const { protect } = require('../middleware/auth.middleware');

const router = express.Router();

// ── Helper: JWT Cookie Set ────────────────────────────────────
const sendToken = (res, user, statusCode = 200) => {
  const token = jwt.sign({ id: user._id }, process.env.JWT_SECRET, {
    expiresIn: process.env.JWT_EXPIRE || '7d',
  });

  res.cookie('token', token, {
    httpOnly: true,
    secure:   process.env.NODE_ENV === 'production',
    sameSite: process.env.NODE_ENV === 'production' ? 'strict' : 'lax',
    maxAge:   7 * 24 * 60 * 60 * 1000,
  });

  res.status(statusCode).json({
    success: true,
    user: {
      id:     user._id,
      name:   user.name,
      email:  user.email,
      avatar: user.avatar,
      role:   user.role,
    },
  });
};

// ── POST /api/auth/register ───────────────────────────────────
router.post('/register', async (req, res) => {
  try {
    const { name, email, password } = req.body;

    if (!name || !email || !password) {
      return res.status(400).json({ success: false, message: 'All fields are required' });
    }

    if (password.length < 6) {
      return res.status(400).json({ success: false, message: 'Password must be at least 6 characters' });
    }

    const exists = await User.findOne({ email });
    if (exists) {
      return res.status(400).json({ success: false, message: 'Email already registered' });
    }

    const user = await User.create({ name, email, password });
    sendToken(res, user, 201);

  } catch (error) {
    res.status(500).json({ success: false, message: 'Server error' });
  }
});

// ── POST /api/auth/login ──────────────────────────────────────
router.post('/login', async (req, res) => {
  try {
    const { email, password } = req.body;

    if (!email || !password) {
      return res.status(400).json({ success: false, message: 'Email and password required' });
    }

    // password field select karo (model mein select: false hai)
    const user = await User.findOne({ email }).select('+password');

    if (!user || !user.password) {
      return res.status(401).json({ success: false, message: 'Invalid email or password' });
    }

    const isMatch = await user.matchPassword(password);
    if (!isMatch) {
      return res.status(401).json({ success: false, message: 'Invalid email or password' });
    }

    sendToken(res, user);

  } catch (error) {
    res.status(500).json({ success: false, message: 'Server error' });
  }
});

// ── GET /api/auth/google ──────────────────────────────────────
router.get('/google',
  passport.authenticate('google', { scope: ['profile', 'email'], prompt: 'select_account' })
);

// ── GET /api/auth/google/callback ────────────────────────────
router.get('/google/callback',
  passport.authenticate('google', { session: false, failureRedirect: `${process.env.CLIENT_URL}/login?error=google_failed` }),
  (req, res) => {
    sendToken(res, req.user);
    res.redirect(`${process.env.CLIENT_URL}/`);
  }
);

// ── GET /api/auth/me ──────────────────────────────────────────
router.get('/me', protect, (req, res) => {
  res.json({ success: true, user: req.user });
});

// ── POST /api/auth/logout ─────────────────────────────────────
router.post('/logout', (req, res) => {
  res.clearCookie('token');
  res.json({ success: true, message: 'Logged out' });
});

module.exports = router;
```
 
---
 
## 10. Express Server Setup (server.js)
 
```javascript
// backend/server.js
 
require('dotenv').config();

const express      = require('express');
const mongoose     = require('mongoose');
const cors         = require('cors');
const cookieParser = require('cookie-parser');
const session      = require('express-session');
const passport     = require('passport');

require('./config/passport');  //Is important 

const authRoutes = require('./routes/auth');

const app = express();

// ── Middleware ────────────────────────────────────────────────
app.use(cors({ origin: process.env.CLIENT_URL, credentials: true }));
app.use(express.json());
app.use(cookieParser());
app.use(session({
  secret:            process.env.SESSION_SECRET,
  resave:            false,
  saveUninitialized: false,
}));
app.use(passport.initialize());
app.use(passport.session());

// ── Routes ────────────────────────────────────────────────────
app.use('/api/auth', authRoutes);

app.get('/', (req, res) => res.json({ message: 'Auth API is running ✅' }));

// ── Start ─────────────────────────────────────────────────────
mongoose
  .connect(process.env.MONGO_URI)
  .then(() => {
    console.log('✅ MongoDB connected');
    app.listen(process.env.PORT || 5000, () =>
      console.log(`✅ Server running on http://localhost:${process.env.PORT || 5000}`)
    );
  })
  .catch((err) => {
    console.error('❌ MongoDB error:', err.message);
    process.exit(1);
  });
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
 
## 11. React Frontend — Login Button
 
```jsx
// frontend/src/components/GoogleLoginButton.jsx
 
import React from 'react';
import { googleLoginURL } from '../services/auth.api';

function GoogleLoginButton({ text = 'Continue with Google' }) {
  return (
    <button
      type="button"
      className="btn-google"
      onClick={() => (window.location.href = googleLoginURL)}
    >
      <svg className="google-icon" viewBox="0 0 48 48">
        <path fill="#EA4335" d="M24 9.5c3.54 0 6.71 1.22 9.21 3.6l6.85-6.85C35.9 2.38 30.47 0 24 0 14.62 0 6.51 5.38 2.56 13.22l7.98 6.19C12.43 13.72 17.74 9.5 24 9.5z"/>
        <path fill="#4285F4" d="M46.98 24.55c0-1.57-.15-3.09-.38-4.55H24v9.02h12.94c-.58 2.96-2.26 5.48-4.78 7.18l7.73 6c4.51-4.18 7.09-10.36 7.09-17.65z"/>
        <path fill="#FBBC05" d="M10.53 28.59c-.48-1.45-.76-2.99-.76-4.59s.27-3.14.76-4.59l-7.98-6.19C.92 16.46 0 20.12 0 24c0 3.88.92 7.54 2.56 10.78l7.97-6.19z"/>
        <path fill="#34A853" d="M24 48c6.48 0 11.93-2.13 15.89-5.81l-7.73-6c-2.15 1.45-4.92 2.3-8.16 2.3-6.26 0-11.57-4.22-13.47-9.91l-7.98 6.19C6.51 42.62 14.62 48 24 48z"/>
      </svg>
      {text}
    </button>
  );
}

export default GoogleLoginButton;
```
```jsx
// frontend/src/components/Input · JSX
 
import React, { useState } from 'react';

// Usage:
// <Input label="Email"    type="email"    value={email}    onChange={setEmail} />
// <Input label="Password" type="password" value={password} onChange={setPassword} />
function Input({ label, type = 'text', value, onChange, placeholder, required }) {
  const [showPass, setShowPass] = useState(false);
  const isPassword = type === 'password';

  return (
    <div className="input-group">
      {label && <label className="input-label">{label}</label>}
      <div className="input-wrapper">
        <input
          className="input-field"
          type={isPassword ? (showPass ? 'text' : 'password') : type}
          value={value}
          onChange={(e) => onChange(e.target.value)}
          placeholder={placeholder || label}
          required={required}
        />
        {isPassword && (
          <button
            type="button"
            className="input-eye"
            onClick={() => setShowPass((prev) => !prev)}
            tabIndex={-1}
          >
            {showPass ? '🙈' : '👁️'}
          </button>
        )}
      </div>
    </div>
  );
}

export default Input;
```
```jsx
// frontend/src/auth/hooks/Auth.api.JS
 
const API = process.env.REACT_APP_API_URL || 'http://localhost:5000/api';

// ── Helper ────────────────────────────────────────────────────
const request = (url, options = {}) =>
  fetch(`${API}${url}`, {
    headers:     { 'Content-Type': 'application/json' },
    credentials: 'include',
    ...options,
  }).then((r) => r.json());

// ── APIs ──────────────────────────────────────────────────────
export const registerAPI = (data) =>
  request('/auth/register', { method: 'POST', body: JSON.stringify(data) });

export const loginAPI = (data) =>
  request('/auth/login', { method: 'POST', body: JSON.stringify(data) });

export const getMeAPI    = ()  => request('/auth/me');
export const logoutAPI   = ()  => request('/auth/logout', { method: 'POST' });

// Google OAuth — browser redirect
export const googleLoginURL = `${API}/auth/google`;
```
 
Create `frontend/src/pages/Login.jsx`:
 
```jsx
// frontend/src/pages/Login.jsx
 
import React, { useState, useEffect } from 'react';
import { Link, useNavigate, useSearchParams } from 'react-router-dom';
import Input             from '../component/Input';
import Button            from '../component/Button';
import GoogleLoginButton from '../component/GoogleLoginButton';
import { useAuth }       from '../auth.context';
import { loginAPI }      from '../services/auth.api';
import '../Style/Style.css';

function LoginPage() {
  const { user, setUser }  = useAuth();
  const navigate           = useNavigate();
  const [searchParams]     = useSearchParams();
  const errorParam         = searchParams.get('error');

  const [email,    setEmail]    = useState('');
  const [password, setPassword] = useState('');
  const [error,    setError]    = useState(
    errorParam === 'google_failed' ? 'Google login failed. Please try again.' : ''
  );
  const [loading, setLoading] = useState(false);

  // Already logged in?  direct home page
  useEffect(() => {
    if (user) navigate('/');
  }, [user, navigate]);

  const handleSubmit = async (e) => {
    e.preventDefault();
    setError('');
    setLoading(true);

    const res = await loginAPI({ email, password });
    setLoading(false);

    if (res.success) {
      setUser(res.user);
      navigate('/');
    } else {
      setError(res.message || 'Login failed. Please check your credentials.');
    }
  };

  return (
    <div className="auth-page">
      <div className="auth-card">

        <div className="auth-header">
          <div className="auth-logo">🔐</div>
          <h1 className="auth-title">Welcome back</h1>
          <p className="auth-subtitle">Sign in to your account</p>
        </div>

        {error && <div className="auth-error">⚠️ {error}</div>}

        <form onSubmit={handleSubmit} className="auth-form">
          <Input
            label="Email"
            type="email"
            value={email}
            onChange={setEmail}
            placeholder="you@example.com"
            required
          />
          <Input
            label="Password"
            type="password"
            value={password}
            onChange={setPassword}
            placeholder="••••••••"
            required
          />
          <Button loading={loading}>Sign In</Button>
        </form>

        <div className="auth-divider">
          <span>or</span>
        </div>

        <GoogleLoginButton text="Sign in with Google" />

        <p className="auth-switch">
          Don't have an account? <Link to="/register">Create one</Link>
        </p>

      </div>
    </div>
  );
}

export default LoginPage;
```
 
---
Create `frontend/src/pages/Register.jsx`:
 
```jsx
// frontend/src/pages/Register.jsx
 
import React, { useState, useEffect } from 'react';
import { Link, useNavigate }  from 'react-router-dom';
import Input             from '../component/Input';
import Button            from '../component/Button';
import GoogleLoginButton from '../component/GoogleLoginButton';
import useAuth           from '../hooks/useAuth';
import { registerAPI }   from '../services/auth.api';
import '../Style/Style.scss';

function RegisterPage() {
  const { user, setUser } = useAuth();
  const navigate          = useNavigate();

  const [name,     setName]     = useState('');
  const [email,    setEmail]    = useState('');
  const [password, setPassword] = useState('');
  const [error,    setError]    = useState('');
  const [loading,  setLoading]  = useState(false);

  useEffect(() => {
    if (user) navigate('/');
  }, [user, navigate]);

  const handleSubmit = async (e) => {
    e.preventDefault();
    setError('');

    if (password.length < 6) {
      setError('Password must be at least 6 characters');
      return;
    }

    setLoading(true);
    const res = await registerAPI({ name, email, password });
    setLoading(false);

    if (res.success) {
      setUser(res.user);
      navigate('/');
    } else {
      setError(res.message || 'Registration failed');
    }
  };

  return (
    <div className="auth-page">
      <div className="auth-card">

        {/* Header */}
        <div className="auth-header">
          <div className="auth-logo">✨</div>
          <h1 className="auth-title">Create account</h1>
          <p className="auth-subtitle">Start your journey today</p>
        </div>

        {/* Error */}
        {error && <div className="auth-error">{error}</div>}

        {/* Register Form */}
        <form onSubmit={handleSubmit} className="auth-form">
          <Input
            label="Full Name"
            type="text"
            value={name}
            onChange={setName}
            placeholder="John Doe"
            required
          />
          <Input
            label="Email"
            type="email"
            value={email}
            onChange={setEmail}
            placeholder="you@example.com"
            required
          />
          <Input
            label="Password"
            type="password"
            value={password}
            onChange={setPassword}
            placeholder="Min. 6 characters"
            required
          />

          {/*
            <Input label="Phone"   type="tel"  value={phone}   onChange={setPhone}   />
            <Input label="Address" type="text" value={address} onChange={setAddress} />
            ────────────────────────────────────────────────────
          */}

          <Button loading={loading}>Create Account</Button>
        </form>

        {/* Divider */}
        <div className="auth-divider">
          <span>or</span>
        </div>

        {/* Google Register */}
        <GoogleLoginButton text="Sign up with Google" />

        {/* Switch to Login */}
        <p className="auth-switch">
          Already have an account?{' '}
          <Link to="/login">Sign in</Link>
        </p>

      </div>
    </div>
  );
}

export default RegisterPage;
```
 
---
 
## 12. React Auth Context
 
```bash
mkdir -p frontend/src/context
touch frontend/src/context/AuthContext.jsx
```
 
```jsx
// frontend/src/context/AuthContext.jsx
 
import React, { createContext, useContext, useState, useEffect, useCallback } from 'react';

export const AuthContext = createContext(null);

const API_URL = process.env.REACT_APP_API_URL || 'http://localhost:5000/api';

export function AuthProvider({ children }) {
  const [user,    setUser]    = useState(null);
  const [loading, setLoading] = useState(true);

  // App load par check karo — login hai ya nahi
  const checkAuth = useCallback(async () => {
    try {
      const res = await fetch(`${API_URL}/auth/me`, {
        credentials: 'include',  // cookies hamesha bhejo
      });

      if (res.ok) {
        const data = await res.json();
        setUser(data.user);
      } else {
        setUser(null);
      }
    } catch {
      setUser(null);
    } finally {
      setLoading(false);
    }
  }, []);

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

  return (
    <AuthContext.Provider value={{
      user,
      setUser,
      loading,
      logout,
      isAuthenticated: !!user,
      isAdmin:         user?.role === 'admin',
    }}>
      {children}
    </AuthContext.Provider>
  );
}

// Custom hook
export const useAuth = () => {
  const ctx = useContext(AuthContext);
  if (!ctx) throw new Error('useAuth must be used inside <AuthProvider>');
  return ctx;
};
```
 
---
 
## 13. Protected Route (Frontend Guard)
 
```jsx
// frontend/src/components/ProtectedRoute.jsx
 
import React from 'react';
import { Navigate, useLocation } from 'react-router-dom';
import { useAuth } from '../auth.context';

// Wrap any route that needs login:
// <Route path="/home" element={<Protected><HomePage /></Protected>} />
function Protected({ children, requiredRole = null }) {
  const { user, loading, isAuthenticated } = useAuth();
  const location = useLocation();

  if (loading) {
    return (
      <div className="auth-loading">
        <div className="spinner" />
      </div>
    );
  }

  if (!isAuthenticated) {
    // Login ke baad wapas isi page par aaye — state mein save karo
    return <Navigate to="/login" state={{ from: location }} replace />;
  }

  if (requiredRole && user.role !== requiredRole) {
    return <Navigate to="/unauthorized" replace />;
  }

  return children;
}

export default Protected;
```
 
---
 
## 13. Home Page (After Login)
 
```jsx
// frontend/src/pages/Home.jsx
 
import React from 'react';
import useAuth from './features/auth/hooks/useAuth';

function Home() {
  const { user, logout } = useAuth();

  return (
    <div style={{ maxWidth: 480, margin: '60px auto', padding: '32px', textAlign: 'center', fontFamily: 'sans-serif' }}>
      <img
        src={user?.avatar || `https://ui-avatars.com/api/?name=${user?.name}&background=4f46e5&color=fff`}
        alt={user?.name}
        style={{ width: 72, height: 72, borderRadius: '50%', marginBottom: 16 }}
      />
      <h2 style={{ margin: '0 0 4px' }}>Hello, {user?.name}! 👋</h2>
      <p style={{ color: '#6b7280', margin: '0 0 24px' }}>{user?.email}</p>
      <button
        onClick={logout}
        style={{
          padding: '10px 24px', background: '#ef4444', color: '#fff',
          border: 'none', borderRadius: 8, cursor: 'pointer', fontWeight: 600,
        }}
      >
        Sign Out
      </button>
    </div>
  );
}

export default Home;
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
import { AuthProvider } from './features/auth/auth.context';
import Protected        from './features/auth/component/Protected';
import LoginPage        from './features/auth/pages/LoginPage';
import RegisterPage     from './features/auth/pages/RegisterPage';

// ── Home page placeholder ─────────────────────────────────────
// Replace this with your actual home/dashboard component
import Home from './Home';

function App() {
  return (
    <AuthProvider>
      <BrowserRouter>
        <Routes>
          {/* Public */}
          <Route path="/login"    element={<LoginPage />} />
          <Route path="/register" element={<RegisterPage />} />

          {/* Protected — login zaroori */}
          <Route
            path="/"
            element={
              <Protected>
                <Home />
              </Protected>
            }
          />

          {/* 404 */}
          <Route path="*" element={<Navigate to="/" replace />} />
        </Routes>
      </BrowserRouter>
    </AuthProvider>
  );
}

export default App;
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
