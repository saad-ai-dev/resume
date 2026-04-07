# 32. MONITORING (SENTRY, CODECLIMATE)

## 32.1 Sentry (Error Monitoring)
```javascript
const Sentry = require('@sentry/node');

Sentry.init({
  dsn: process.env.SENTRY_DSN,
  environment: process.env.NODE_ENV,
  tracesSampleRate: 0.1, // 10% of transactions
});

// Auto-captures unhandled errors
// Manual capture:
try {
  riskyOperation();
} catch (error) {
  Sentry.captureException(error, {
    tags: { module: 'payments' },
    extra: { userId: '123' },
  });
}
```

## 32.2 CodeClimate (Code Quality)
```
Metrics tracked:
- Maintainability (A-F grade)
- Code duplication
- Complexity (cyclomatic)
- Test coverage percentage
- Code smells
```

