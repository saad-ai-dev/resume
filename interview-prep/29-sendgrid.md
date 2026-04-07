# 29. SENDGRID EMAIL INTEGRATION

```javascript
const sgMail = require('@sendgrid/mail');
sgMail.setApiKey(process.env.SENDGRID_API_KEY);

// Send email
await sgMail.send({
  to: 'user@email.com',
  from: 'noreply@obenan.com',
  subject: 'New Review Alert',
  templateId: 'd-xxxxx',           // SendGrid dynamic template
  dynamicTemplateData: {
    name: 'Saad',
    reviewCount: 5,
    locationName: 'Main Street Branch',
  },
});

// Webhook verification (verify events are from SendGrid)
app.post('/webhooks/sendgrid', (req, res) => {
  const signature = req.headers['x-twilio-email-event-webhook-signature'];
  const timestamp = req.headers['x-twilio-email-event-webhook-timestamp'];

  // Verify signature...

  for (const event of req.body) {
    switch (event.event) {
      case 'delivered': /* track delivery */ break;
      case 'bounce':    /* handle bounce */  break;
      case 'open':      /* track opens */    break;
      case 'click':     /* track clicks */   break;
    }
  }

  res.sendStatus(200);
});
```

