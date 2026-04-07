# 17. AUTHENTICATION & SECURITY

## 17.1 JWT (JSON Web Tokens)

### Structure
```
Header.Payload.Signature

eyJhbGciOiJIUzI1NiJ9.eyJ1c2VySWQiOjEsInJvbGUiOiJhZG1pbiJ9.signature
```

```javascript
// Header
{ "alg": "HS256", "typ": "JWT" }

// Payload
{ "userId": 1, "role": "admin", "iat": 1700000000, "exp": 1700003600 }

// Signature
HMACSHA256(base64(header) + "." + base64(payload), secret)
```

### Implementation
```javascript
const jwt = require('jsonwebtoken');

// Generate
function generateTokens(user) {
  const accessToken = jwt.sign(
    { userId: user.id, role: user.role },
    process.env.JWT_SECRET,
    { expiresIn: '15m' }
  );

  const refreshToken = jwt.sign(
    { userId: user.id },
    process.env.REFRESH_SECRET,
    { expiresIn: '7d' }
  );

  return { accessToken, refreshToken };
}

// Verify
function authMiddleware(req, res, next) {
  const token = req.headers.authorization?.split(' ')[1];
  if (!token) return res.status(401).json({ error: 'No token provided' });

  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    req.user = decoded;
    next();
  } catch (err) {
    if (err.name === 'TokenExpiredError') {
      return res.status(401).json({ error: 'Token expired' });
    }
    return res.status(401).json({ error: 'Invalid token' });
  }
}
```

### Access Token vs Refresh Token

| Feature | Access Token | Refresh Token |
|---------|-------------|---------------|
| Lifespan | Short (15 min) | Long (7-30 days) |
| Stored | Memory / HTTP header | HttpOnly cookie / DB |
| Purpose | Authenticate API requests | Get new access tokens |
| Contains | userId, role, permissions | userId only |

## 17.2 OAuth 2.0

```
┌────────┐     1. Redirect to Google     ┌──────────┐
│  User  │ ──────────────────────────────>│  Google  │
│(Browser)│<──────────────────────────────│  OAuth   │
│        │     2. User logs in & consents │          │
│        │     3. Redirect with auth code │          │
└────┬───┘                                └──────────┘
     │ 4. Send auth code to your server       │
┌────▼───┐     5. Exchange code for tokens ┌──┴───────┐
│  Your  │ ──────────────────────────────>│  Google  │
│ Server │<──────────────────────────────│  Token   │
│        │     6. Access token + user info │ Endpoint │
└────────┘                                └──────────┘
```

**Your Experience:** Google My Business OAuth for location sync, reviews, and insights.

## 17.3 Passport.js
```javascript
const passport = require('passport');
const { Strategy: JwtStrategy, ExtractJwt } = require('passport-jwt');

passport.use(new JwtStrategy({
  jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
  secretOrKey: process.env.JWT_SECRET,
}, async (payload, done) => {
  try {
    const user = await User.findByPk(payload.userId);
    if (user) return done(null, user);
    return done(null, false);
  } catch (err) {
    return done(err, false);
  }
}));

// Protect route
app.get('/api/profile', passport.authenticate('jwt', { session: false }), handler);
```

## 17.4 RBAC (Role-Based Access Control)

```javascript
// Role hierarchy
const permissions = {
  superadmin: ['read', 'write', 'delete', 'manage_users', 'manage_billing'],
  admin:      ['read', 'write', 'delete', 'manage_users'],
  manager:    ['read', 'write', 'delete'],
  user:       ['read', 'write'],
  viewer:     ['read'],
};

// Middleware
function authorize(...allowedRoles) {
  return (req, res, next) => {
    if (!allowedRoles.includes(req.user.role)) {
      return res.status(403).json({ error: 'Forbidden' });
    }
    next();
  };
}

// Usage
router.delete('/users/:id', authorize('admin', 'superadmin'), deleteUser);
```

## 17.5 Security Best Practices

| Threat | Prevention |
|--------|-----------|
| **SQL Injection** | Parameterized queries, ORMs |
| **XSS** | Sanitize input, Content-Security-Policy header |
| **CSRF** | CSRF tokens, SameSite cookies |
| **Brute Force** | Rate limiting, account lockout |
| **Password Storage** | bcrypt with salt (never plain text) |
| **Sensitive Data** | HTTPS only, encrypt at rest |
| **JWT Theft** | Short expiry, HttpOnly cookies, refresh rotation |

```javascript
const bcrypt = require('bcrypt');

// Hash password
const hash = await bcrypt.hash(password, 12); // 12 salt rounds

// Verify
const isMatch = await bcrypt.compare(password, hash);
```

