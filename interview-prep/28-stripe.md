# 28. STRIPE PAYMENT INTEGRATION

## 28.1 Core Concepts

```
Customer → Subscription → Invoice → Payment

Key Objects:
- Customer: The person paying
- Product: What you sell (e.g., "Pro Plan")
- Price: How much ($30/month, $300/year)
- Subscription: Recurring billing
- PaymentIntent: One-time payment
- Invoice: Bill generated for subscription
```

## 28.2 Implementation

```javascript
const stripe = require('stripe')(process.env.STRIPE_SECRET_KEY);

// Create customer
const customer = await stripe.customers.create({
  email: 'saad@email.com',
  name: 'Saad Sohail',
  metadata: { userId: '123' },
});

// Create subscription
const subscription = await stripe.subscriptions.create({
  customer: customer.id,
  items: [{ price: 'price_xxx' }], // Monthly price ID
  payment_behavior: 'default_incomplete',
  expand: ['latest_invoice.payment_intent'],
});

// Webhook handling (CRITICAL for production)
app.post('/webhooks/stripe', express.raw({ type: 'application/json' }), (req, res) => {
  const sig = req.headers['stripe-signature'];
  const event = stripe.webhooks.constructEvent(req.body, sig, WEBHOOK_SECRET);

  switch (event.type) {
    case 'invoice.paid':
      // Activate subscription
      await activateSubscription(event.data.object);
      break;
    case 'invoice.payment_failed':
      // Notify user, retry
      await handlePaymentFailure(event.data.object);
      break;
    case 'customer.subscription.deleted':
      // Deactivate access
      await deactivateSubscription(event.data.object);
      break;
  }

  res.json({ received: true });
});
```

