# 19. BULLMQ — JOB QUEUES & BACKGROUND PROCESSING

## 19.1 What is BullMQ?
BullMQ is a **Redis-backed job queue** for Node.js. Used for background processing, scheduled tasks, and async workflows.

## 19.2 Why Use Job Queues?
- Heavy tasks that would block the API response
- Scheduled/recurring tasks (cron-like)
- Retry failed operations
- Rate-limiting external API calls
- Decoupling services

## 19.3 Implementation

```javascript
const { Queue, Worker } = require('bullmq');
const Redis = require('ioredis');

const connection = new Redis({ host: 'localhost', port: 6379 });

// Create queue
const reviewQueue = new Queue('review-processing', { connection });

// Add jobs
await reviewQueue.add('fetch-reviews', {
  locationId: 123,
  companyId: 456,
}, {
  attempts: 3,                    // Retry 3 times on failure
  backoff: { type: 'exponential', delay: 5000 }, // 5s, 10s, 20s
  removeOnComplete: 100,         // Keep last 100 completed jobs
  removeOnFail: 50,              // Keep last 50 failed jobs
});

// Scheduled job (every hour)
await reviewQueue.add('sync-reviews', { all: true }, {
  repeat: { pattern: '0 * * * *' }, // cron pattern
});

// Worker (processes jobs)
const worker = new Worker('review-processing', async (job) => {
  console.log(`Processing ${job.name} with data:`, job.data);

  switch (job.name) {
    case 'fetch-reviews':
      await fetchReviewsFromGoogle(job.data.locationId);
      break;
    case 'analyze-sentiment':
      await analyzeSentiment(job.data.reviewId);
      break;
    case 'send-notification':
      await sendEmail(job.data);
      break;
  }
}, {
  connection,
  concurrency: 5, // Process 5 jobs in parallel
});

// Events
worker.on('completed', (job) => console.log(`Job ${job.id} completed`));
worker.on('failed', (job, err) => console.error(`Job ${job.id} failed:`, err));
```

## 19.4 Your BullMQ Usage at Obenan/Global Software Consulting
- Review fetching from Google Business
- Sentiment analysis pipeline
- Report generation
- Email notification queues
- Scheduled data sync jobs

