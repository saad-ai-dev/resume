# 23. CI/CD

## 23.1 What is CI/CD?
- **CI (Continuous Integration):** Automatically build and test code on every push
- **CD (Continuous Delivery):** Automatically deploy tested code to staging/production

## 23.2 GitHub Actions
```yaml
# .github/workflows/deploy.yml
name: CI/CD Pipeline
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '20' }
      - run: npm ci
      - run: npm run lint
      - run: npm test

  deploy:
    needs: test
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Deploy to EC2
        run: |
          ssh -i ${{ secrets.SSH_KEY }} ec2-user@${{ secrets.EC2_HOST }} "
            cd /app && git pull && npm ci && pm2 restart all
          "
```

## 23.3 Branch Protection Rules
```
- Require pull request reviews (1-2 approvers)
- Require status checks to pass (tests, lint)
- Require branches to be up to date
- No direct pushes to main
- Require linear history (no merge commits)
```

