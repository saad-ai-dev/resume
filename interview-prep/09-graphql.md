# 9. GRAPHQL

## 9.1 What is GraphQL?
GraphQL is a **query language for APIs** developed by Facebook (2015). Clients specify exactly what data they need — no more, no less.

## 9.2 Core Concepts

```graphql
# Schema Definition
type User {
  id: ID!
  name: String!
  email: String
  posts: [Post!]!
}

type Post {
  id: ID!
  title: String!
  author: User!
}

type Query {
  user(id: ID!): User
  users(page: Int, limit: Int): [User!]!
}

type Mutation {
  createUser(input: CreateUserInput!): User!
  updateUser(id: ID!, input: UpdateUserInput!): User!
}

# Client Query
query {
  user(id: "123") {
    name
    email
    posts {
      title
    }
  }
}
```

## 9.3 Key Concepts
- **Query** — Read data (like GET)
- **Mutation** — Write data (like POST/PUT/DELETE)
- **Subscription** — Real-time data (WebSocket)
- **Resolver** — Function that returns data for a field
- **Schema** — Contract between client and server

