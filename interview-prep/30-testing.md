# 30. TESTING (MOCHA, CHAI)

## 30.1 Testing Pyramid

```
         ┌──────┐
         │  E2E  │  Few (slow, expensive)
        ┌┴──────┴┐
        │ Integr. │  Some
       ┌┴────────┴┐
       │   Unit    │  Many (fast, cheap)
       └──────────┘
```

## 30.2 Mocha + Chai

```javascript
const { expect } = require('chai');
const chaiHttp = require('chai-http');
const app = require('../app');

chai.use(chaiHttp);

describe('Users API', () => {
  // Setup
  before(async () => { await seedDatabase(); });
  after(async () => { await cleanDatabase(); });

  describe('GET /api/users', () => {
    it('should return all users', async () => {
      const res = await chai.request(app).get('/api/users');

      expect(res).to.have.status(200);
      expect(res.body).to.be.an('array');
      expect(res.body).to.have.length.greaterThan(0);
      expect(res.body[0]).to.have.property('name');
      expect(res.body[0]).to.have.property('email');
    });

    it('should support pagination', async () => {
      const res = await chai.request(app)
        .get('/api/users')
        .query({ page: 1, limit: 5 });

      expect(res.body).to.have.length.at.most(5);
    });
  });

  describe('POST /api/users', () => {
    it('should create a new user', async () => {
      const res = await chai.request(app)
        .post('/api/users')
        .set('Authorization', `Bearer ${token}`)
        .send({ name: 'Test User', email: 'test@email.com' });

      expect(res).to.have.status(201);
      expect(res.body.name).to.equal('Test User');
    });

    it('should return 400 for invalid data', async () => {
      const res = await chai.request(app)
        .post('/api/users')
        .set('Authorization', `Bearer ${token}`)
        .send({ name: '' }); // Invalid

      expect(res).to.have.status(400);
      expect(res.body).to.have.property('error');
    });

    it('should return 401 without auth token', async () => {
      const res = await chai.request(app)
        .post('/api/users')
        .send({ name: 'Test' });

      expect(res).to.have.status(401);
    });
  });
});
```

## 30.3 Testing Best Practices
- Test behavior, not implementation
- One assertion per test (ideally)
- Use descriptive test names
- Arrange → Act → Assert pattern
- Mock external services (don't call real APIs in tests)
- Test edge cases and error paths

