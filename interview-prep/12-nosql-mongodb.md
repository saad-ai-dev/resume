# 12. NoSQL DATABASES (MongoDB)

## 12.1 Core Concepts

### Document Structure
```javascript
// MongoDB stores data as BSON (Binary JSON) documents
{
  _id: ObjectId("507f191e810c19729de860ea"),
  name: "Saad",
  email: "saad@email.com",
  age: 28,
  address: {                          // Embedded document
    city: "Lahore",
    country: "Pakistan"
  },
  skills: ["Node.js", "MongoDB"],     // Array
  posts: [                            // Array of embedded documents
    { title: "First Post", likes: 10 },
    { title: "Second Post", likes: 25 }
  ],
  createdAt: ISODate("2024-01-15")
}
```

### CRUD Operations
```javascript
// Create
db.users.insertOne({ name: "Saad", age: 28 });
db.users.insertMany([{ name: "Ali" }, { name: "Sara" }]);

// Read
db.users.find({ age: { $gt: 25 } }).sort({ name: 1 }).limit(10);
db.users.findOne({ email: "saad@email.com" });

// Update
db.users.updateOne(
  { _id: ObjectId("...") },
  { $set: { name: "Saad Sohail" }, $inc: { age: 1 } }
);

// Delete
db.users.deleteOne({ _id: ObjectId("...") });
```

### Query Operators
```javascript
// Comparison
$eq, $ne, $gt, $gte, $lt, $lte, $in, $nin

// Logical
$and, $or, $not, $nor

// Element
$exists, $type

// Array
$all, $elemMatch, $size

// Example
db.users.find({
  $and: [
    { age: { $gte: 25, $lte: 35 } },
    { skills: { $in: ["Node.js", "React"] } },
    { "address.city": { $exists: true } }
  ]
});
```

### Aggregation Pipeline
```javascript
db.orders.aggregate([
  { $match: { status: "completed" } },           // Filter
  { $group: {                                      // Group
    _id: "$customerId",
    totalSpent: { $sum: "$amount" },
    orderCount: { $sum: 1 }
  }},
  { $sort: { totalSpent: -1 } },                  // Sort
  { $limit: 10 },                                  // Limit
  { $lookup: {                                     // Join
    from: "customers",
    localField: "_id",
    foreignField: "_id",
    as: "customerInfo"
  }},
  { $project: {                                    // Select fields
    totalSpent: 1,
    orderCount: 1,
    customerName: { $arrayElemAt: ["$customerInfo.name", 0] }
  }}
]);
```

### Indexes in MongoDB
```javascript
// Single field
db.users.createIndex({ email: 1 });       // 1 = ascending, -1 = descending

// Compound
db.users.createIndex({ name: 1, age: -1 });

// Unique
db.users.createIndex({ email: 1 }, { unique: true });

// Text (full-text search)
db.posts.createIndex({ title: "text", body: "text" });
db.posts.find({ $text: { $search: "nodejs tutorial" } });

// Geospatial (2dsphere) — used at Zamulk for property search
db.properties.createIndex({ location: "2dsphere" });
db.properties.find({
  location: {
    $near: {
      $geometry: { type: "Point", coordinates: [74.35, 31.52] },
      $maxDistance: 5000 // meters
    }
  }
});

// TTL (auto-delete after time)
db.sessions.createIndex({ createdAt: 1 }, { expireAfterSeconds: 3600 });
```

### Embedding vs Referencing

| Strategy | When to Use | Example |
|----------|-------------|---------|
| **Embed** | 1-to-few, data accessed together, rarely changes | User + address |
| **Reference** | 1-to-many, large/growing arrays, data shared across docs | User + posts |

```javascript
// Embedded (denormalized)
{
  name: "Saad",
  address: { city: "Lahore", zip: "54000" }  // Lives inside user
}

// Referenced (normalized)
// users collection
{ _id: 1, name: "Saad" }
// posts collection
{ _id: 101, title: "Hello", authorId: 1 }    // References user
```

