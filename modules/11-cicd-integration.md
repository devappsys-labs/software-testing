# 11 — CI/CD Integration

> 🔴 **Advanced**

[← Back to Index](../README.md)

---

This is where testing becomes **a quality gate**, not just a developer tool. Every push is validated automatically before it can merge or deploy.

## The Full Pipeline

```mermaid
flowchart TD
    PR["🔀 Pull Request\nOpened / Updated"]

    subgraph "Fast Checks (< 2 min)"
    LINT["🔍 Lint\nESLint, Prettier"]
    TYPE["📝 Type Check\nTypeScript"]
    UNIT["⚡ Unit Tests\n(Vitest/Jest)"]
    end

    subgraph "Build & Integration (5–10 min)"
    BUILD["🔨 Build\nNext.js / Vite"]
    INT["🔗 Integration Tests\n(Test DB)"]
    end

    subgraph "Extended Checks (optional)"
    E2E["🌐 E2E Tests\n(Playwright)"]
    PERF["📊 Performance\n(k6 smoke)"]
    SEC["🔒 Security Scan\n(Snyk / Semgrep)"]
    end

    MERGE["✅ Merge to Main"]
    DEPLOY["🚀 Deploy to Staging"]

    PR --> LINT & TYPE & UNIT
    LINT & TYPE & UNIT -->|all pass| BUILD
    BUILD --> INT
    INT -->|pass| E2E & PERF & SEC
    E2E & PERF & SEC -->|all pass| MERGE
    MERGE --> DEPLOY

    style PR fill:#3498db,color:#fff
    style MERGE fill:#27ae60,color:#fff
    style DEPLOY fill:#9b59b6,color:#fff
```

---

## 11.1 GitHub Actions — Full Test Pipeline

```yaml
# .github/workflows/test.yml
name: Test Suite

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  # ─── FAST CHECKS ──────────────────────────────────────────────────────────
  lint-and-typecheck:
    name: Lint & Type Check
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci
      - run: npm run lint
      - run: npm run typecheck

  unit-tests:
    name: Unit Tests
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci
      - run: npm run test:unit -- --coverage
      - uses: actions/upload-artifact@v4
        with:
          name: coverage-report
          path: coverage/

  # ─── INTEGRATION TESTS ───────────────────────────────────────────────────
  integration-tests:
    name: Integration Tests
    runs-on: ubuntu-latest
    needs: [lint-and-typecheck, unit-tests]

    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_USER: testuser
          POSTGRES_PASSWORD: testpass
          POSTGRES_DB: testdb
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432

      redis:
        image: redis:7
        options: >-
          --health-cmd "redis-cli ping"
          --health-interval 10s
        ports:
          - 6379:6379

    env:
      DATABASE_URL: postgresql://testuser:testpass@localhost:5432/testdb
      REDIS_URL: redis://localhost:6379
      NODE_ENV: test

    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci
      - run: npm run db:migrate
      - run: npm run test:integration

  # ─── E2E TESTS ────────────────────────────────────────────────────────────
  e2e-tests:
    name: E2E Tests
    runs-on: ubuntu-latest
    needs: [integration-tests]

    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci
      - run: npx playwright install --with-deps chromium
      - run: npm run build
      - run: npm run start &
      - run: npx wait-on http://localhost:3000 --timeout 30000
      - run: npm run test:e2e
      - uses: actions/upload-artifact@v4
        if: failure()
        with:
          name: playwright-report
          path: playwright-report/

  # ─── SECURITY ─────────────────────────────────────────────────────────────
  security:
    name: Security Scan
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm audit --audit-level=high
      - uses: actions/setup-python@v5
        with:
          python-version: '3.12'
      - run: pip install semgrep && semgrep --config=auto src/

  # ─── COVERAGE GATE ────────────────────────────────────────────────────────
  coverage-check:
    name: Coverage Gate
    runs-on: ubuntu-latest
    needs: [unit-tests]
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: coverage-report
          path: coverage/
      - name: Check coverage threshold
        run: |
          COVERAGE=$(cat coverage/coverage-summary.json | jq '.total.lines.pct')
          echo "Line coverage: ${COVERAGE}%"
          if (( $(echo "$COVERAGE < 80" | bc -l) )); then
            echo "❌ Coverage ${COVERAGE}% is below the 80% threshold"
            exit 1
          fi
          echo "✅ Coverage check passed"
```

---

## 11.2 GitLab CI Pipeline

```yaml
# .gitlab-ci.yml
stages:
  - validate
  - test
  - e2e
  - security
  - deploy

variables:
  NODE_VERSION: "20"
  POSTGRES_DB: testdb
  POSTGRES_USER: testuser
  POSTGRES_PASSWORD: testpass

default:
  image: node:20-alpine
  cache:
    key: ${CI_COMMIT_REF_SLUG}
    paths:
      - node_modules/

lint:
  stage: validate
  script:
    - npm ci
    - npm run lint
    - npm run typecheck

unit-tests:
  stage: test
  script:
    - npm ci
    - npm run test:unit -- --coverage
  coverage: '/Lines\s*:\s*(\d+\.?\d*)%/'
  artifacts:
    reports:
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml

integration-tests:
  stage: test
  services:
    - name: postgres:16
      alias: db
  variables:
    DATABASE_URL: postgresql://testuser:testpass@db/testdb
  script:
    - npm ci
    - npm run db:migrate
    - npm run test:integration

e2e:
  stage: e2e
  image: mcr.microsoft.com/playwright:v1.44.0
  script:
    - npm ci
    - npm run build
    - npm run start &
    - npx wait-on http://localhost:3000
    - npm run test:e2e
  artifacts:
    when: on_failure
    paths:
      - playwright-report/
```

---

## 11.3 Makefile for Local Development

Run the same checks locally before pushing:

```makefile
# Makefile

.PHONY: test test-unit test-integration test-e2e test-all check

test-unit:
	@echo "🔬 Running unit tests..."
	npx vitest run --coverage

test-integration:
	@echo "🔗 Running integration tests..."
	DATABASE_URL=postgresql://testuser:testpass@localhost:5432/testdb \
	  npx vitest run tests/integration

test-e2e:
	@echo "🌐 Running E2E tests..."
	npx playwright test

check: test-unit
	@echo "✅ Fast checks passed — safe to push"

test-all: test-unit test-integration test-e2e
	@echo "✅ Full test suite passed"
```

---

**← Previous:** [Contract Testing](./10-contract-testing.md) · **Next →** [Test Strategy by Role](./12-test-strategy-by-role.md)
