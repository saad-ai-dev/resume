# 34. PYTHON BASICS

## 34.1 Key Concepts (For AI/ML Context)

```python
# Variables (no type declaration needed)
name = "Saad"
age = 28
skills = ["Node.js", "Python", "AWS"]

# Functions
def analyze_sentiment(text: str) -> dict:
    """Analyze sentiment of text."""
    result = model.predict(text)
    return {"sentiment": result.label, "score": result.score}

# List comprehensions
positive = [r for r in reviews if r.sentiment == "positive"]

# Dictionary
user = {"name": "Saad", "age": 28}

# Classes
class ReviewAnalyzer:
    def __init__(self, model_name: str):
        self.model = load_model(model_name)

    def analyze(self, text: str) -> str:
        return self.model.predict(text)

# Async (asyncio)
import asyncio

async def fetch_reviews(location_id: int):
    async with aiohttp.ClientSession() as session:
        async with session.get(f"/api/reviews/{location_id}") as resp:
            return await resp.json()

# Virtual environments
# python -m venv venv
# source venv/bin/activate
# pip install -r requirements.txt
```

