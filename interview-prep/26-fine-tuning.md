# 26. FINE-TUNING LLMs

## 26.1 What is Fine-Tuning?
Taking a pre-trained model and training it further on your specific data to improve performance for a particular task.

## 26.2 When to Fine-Tune vs RAG

| Criteria | Fine-Tuning | RAG |
|----------|------------|-----|
| Need latest data | No (static after training) | Yes (dynamic retrieval) |
| Custom behavior/tone | Yes | Somewhat (via prompting) |
| Cost | High (training + hosting) | Low (just API calls) |
| Speed to deploy | Slow (hours/days) | Fast (minutes) |
| Private data | Baked into model | Retrieved at query time |
| Best for | Style, format, specialized tasks | Knowledge retrieval, Q&A |

**Rule of thumb:** Try RAG first. Fine-tune only when RAG isn't enough.

## 26.3 Fine-Tuning with OpenAI

```javascript
// Training data format (JSONL)
{"messages": [{"role": "system", "content": "You classify reviews."}, {"role": "user", "content": "Great food!"}, {"role": "assistant", "content": "{\"sentiment\": \"positive\", \"score\": 0.95}"}]}
{"messages": [{"role": "system", "content": "You classify reviews."}, {"role": "user", "content": "Terrible service"}, {"role": "assistant", "content": "{\"sentiment\": \"negative\", \"score\": 0.9}"}]}

// Upload training file
const file = await openai.files.create({
  file: fs.createReadStream('training.jsonl'),
  purpose: 'fine-tune',
});

// Create fine-tuning job
const job = await openai.fineTuning.jobs.create({
  training_file: file.id,
  model: 'gpt-4o-mini-2024-07-18',
  hyperparameters: { n_epochs: 3 },
});

// Use fine-tuned model
const response = await openai.chat.completions.create({
  model: 'ft:gpt-4o-mini-2024-07-18:org:custom-name:id',
  messages: [{ role: 'user', content: 'Review: Not bad at all!' }],
});
```

