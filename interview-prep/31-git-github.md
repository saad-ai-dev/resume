# 31. GIT, GITHUB & BRANCHING STRATEGIES

## 31.1 Git Flow
```
main ─────────────────────────────────────────────
  │                              ↑
  └── develop ─────────────────────────────────────
       │           ↑          ↑
       └── feature/login ─────┘
       └── feature/reviews ───┘
       └── hotfix/security ───────────── merge to main
```

## 31.2 Common Commands
```bash
# Branching
git checkout -b feature/new-api
git push -u origin feature/new-api

# Rebase (clean history)
git checkout feature/new-api
git rebase main

# Squash commits
git rebase -i HEAD~3  # Squash last 3 commits

# Stash (save work temporarily)
git stash
git stash pop

# Cherry-pick (take specific commit)
git cherry-pick abc123

# Undo last commit (keep changes)
git reset --soft HEAD~1

# View changes
git log --oneline --graph
git diff --staged
```

## 31.3 PR Best Practices
- Small, focused PRs (< 400 lines)
- Descriptive title and description
- Link to Jira ticket
- Include screenshots for UI changes
- Request specific reviewers
- Address all comments before merge

