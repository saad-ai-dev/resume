# 24. AI & LLMs

## 24.1 What is an LLM?
Large Language Model — a neural network trained on massive text data that can generate, understand, and reason about text. Examples: GPT-4, Claude, Gemini.

## 24.2 OpenAI API

```javascript
const OpenAI = require('openai');
const openai = new OpenAI({ apiKey: process.env.OPENAI_API_KEY });

// Chat Completion
const response = await openai.chat.completions.create({
  model: 'gpt-4o',
  messages: [
    { role: 'system', content: 'You are a helpful assistant.' },
    { role: 'user', content: 'Analyze this review for sentiment.' },
  ],
  temperature: 0,       // 0 = deterministic, 1 = creative
  max_tokens: 500,
  response_format: { type: 'json_object' }, // Force JSON output
});

const answer = response.choices[0].message.content;

// Streaming
const stream = await openai.chat.completions.create({
  model: 'gpt-4o',
  messages: [{ role: 'user', content: 'Tell me a story' }],
  stream: true,
});

for await (const chunk of stream) {
  const text = chunk.choices[0]?.delta?.content || '';
  process.stdout.write(text); // Real-time output
}

// Function Calling (Tool Use)
const response = await openai.chat.completions.create({
  model: 'gpt-4o',
  messages: [{ role: 'user', content: 'What is the weather in Lahore?' }],
  tools: [{
    type: 'function',
    function: {
      name: 'getWeather',
      description: 'Get current weather for a city',
      parameters: {
        type: 'object',
        properties: {
          city: { type: 'string', description: 'City name' },
        },
        required: ['city'],
      },
    },
  }],
});
```

## 24.3 Claude API (Anthropic)

```javascript
const Anthropic = require('@anthropic-ai/sdk');
const anthropic = new Anthropic({ apiKey: process.env.CLAUDE_API_KEY });

const response = await anthropic.messages.create({
  model: 'claude-sonnet-4-6',
  max_tokens: 1024,
  system: 'You are a sentiment analysis expert.',
  messages: [
    { role: 'user', content: 'Analyze this review: "Great food but slow service"' },
  ],
});

console.log(response.content[0].text);

// Tool Use
const response = await anthropic.messages.create({
  model: 'claude-sonnet-4-6',
  max_tokens: 1024,
  tools: [{
    name: 'search_reviews',
    description: 'Search reviews for a business location',
    input_schema: {
      type: 'object',
      properties: {
        locationId: { type: 'string' },
        sentiment: { type: 'string', enum: ['positive', 'negative', 'neutral'] },
      },
      required: ['locationId'],
    },
  }],
  messages: [{ role: 'user', content: 'Find negative reviews for location 123' }],
});
```

## 24.4 Model Comparison

| Feature | GPT-4o | Claude Opus | Claude Sonnet | Claude Haiku | GPT-4o Mini |
|---------|--------|-------------|---------------|-------------|-------------|
| **Quality** | Excellent | Excellent | Very Good | Good | Good |
| **Speed** | Fast | Slower | Fast | Very Fast | Very Fast |
| **Cost** | $$$ | $$$$ | $$ | $ | $ |
| **Context** | 128K | 200K | 200K | 200K | 128K |
| **Best for** | General tasks | Complex reasoning | Balanced | Quick tasks | Cheap, fast |
| **Coding** | Excellent | Excellent | Very Good | Good | Good |

**Interview Q: How do you choose which model to use?**
- **Simple classification** → Haiku or GPT-4o Mini (cheapest)
- **Complex reasoning** → Opus or GPT-4o
- **Production chatbot** → Sonnet or GPT-4o (balance of quality/cost)
- **Cost-sensitive high-volume** → Haiku + caching
- **Safety-critical** → Claude (better guardrails)

