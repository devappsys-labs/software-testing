# 🧪 The Complete Software Testing Guide
### From Zero to Senior — Real World Examples for Frontend & API Developers

> **Who is this for?** Absolute beginners writing their first test, mid-level devs wanting to improve coverage, and senior engineers designing test strategies. Every section is labeled with the skill level it targets.

---

## 📖 Table of Contents

| # | Topic | Level |
|---|-------|-------|
| 1 | [Why Testing Matters](#1-why-testing-matters) | 🟢 Beginner |
| 2 | [The Testing Pyramid](#2-the-testing-pyramid) | 🟢 Beginner |
| 3 | [Unit Testing](#3-unit-testing) | 🟢 Beginner |
| 4 | [Integration Testing](#4-integration-testing) | 🟡 Intermediate |
| 5 | [End-to-End Testing](#5-end-to-end-testing) | 🟡 Intermediate |
| 6 | [API Testing](#6-api-testing) | 🟡 Intermediate |
| 7 | [Frontend Testing](#7-frontend-testing) | 🟡 Intermediate |
| 8 | [Performance Testing](#8-performance-testing) | 🔴 Advanced |
| 9 | [Security Testing](#9-security-testing) | 🔴 Advanced |
| 10 | [Contract Testing](#10-contract-testing) | 🔴 Advanced |
| 11 | [CI/CD Integration](#11-cicd-integration) | 🔴 Advanced |
| 12 | [Test Strategy by Role](#12-test-strategy-by-role) | 🔴 Advanced |
| 13 | [Quick Reference Cheatsheet](#13-quick-reference-cheatsheet) | All |

---

## 1. Why Testing Matters

> 🟢 **Beginner**

Testing is not about finding bugs — it's about **preventing them from reaching users** and **giving developers confidence to change code**.

### The Cost of a Bug Over Time

```mermaid
xychart-beta
    title "Cost to Fix a Bug at Different Stages"
    x-axis ["Development", "Code Review", "QA", "Staging", "Production"]
    y-axis "Relative Cost" 0 --> 100
    bar [1, 5, 10, 25, 100]
```

A bug that costs **$1** to fix during development costs **$100** in production. This is why automated testing exists.

### What Good Tests Give You

```mermaid
mindmap
  root((Testing Benefits))
    Confidence
      Refactor safely
      Deploy faster
      Sleep well 😴
    Documentation
      Tests show intent
      Living docs that never go stale
    Design Pressure
      Untestable code = bad design
      Forces clean interfaces
    Speed
      Catch bugs in seconds
      Not days after deployment
```

---

## 2. The Testing Pyramid

> 🟢 **Beginner**

The testing pyramid describes the **ideal ratio** of test types in a healthy codebase.

```mermaid
graph TD
    E2E["🔺 E2E Tests<br/>Few, Slow, Expensive<br/>~10%"]
    INT["🔷 Integration Tests<br/>Some, Medium Speed<br/>~30%"]
    UNIT["🟩 Unit Tests<br/>Many, Fast, Cheap<br/>~60%"]

    E2E --> INT --> UNIT

    style E2E fill:#ff6b6b,color:#fff,stroke:#c0392b
    style INT fill:#f39c12,color:#fff,stroke:#e67e22
    style UNIT fill:#27ae60,color:#fff,stroke:#1e8449
```

| Layer | Count | Speed | Purpose |
|-------|-------|-------|---------|
| **Unit** | Hundreds–thousands | Milliseconds | Logic correctness |
| **Integration** | Dozens–hundreds | Seconds | Components work together |
| **E2E** | A handful | Minutes | Critical user flows work |

> **Anti-pattern**: The Ice Cream Cone — lots of slow E2E tests, few fast unit tests. This makes CI take 40+ minutes and gives developers no fast feedback loop.

```mermaid
graph TD
    subgraph "❌ Ice Cream Cone (Bad)"
    A["🔺 Lots of E2E Tests"]
    B["🔷 Some Integration"]
    C["🟩 Few Unit Tests"]
    A --> B --> C
    end
    subgraph "✅ Testing Pyramid (Good)"
    D["🔺 Few E2E Tests"]
    E["🔷 More Integration"]
    F["🟩 Many Unit Tests"]
    F --> E --> D
    end

    style A fill:#ff6b6b,color:#fff
    style D fill:#27ae60,color:#fff
```

---

## 3. Unit Testing

> 🟢 **Beginner** — Start here

A unit test tests **one function or class in isolation**, with all external dependencies replaced by mocks or stubs.

### The AAA Pattern

Every unit test follows three steps:

```
Arrange → Act → Assert
```

```mermaid
flowchart LR
    A["🔧 Arrange<br/>Set up test data<br/>and mocks"] --> B["⚡ Act<br/>Call the function<br/>under test"] --> C["✅ Assert<br/>Verify the<br/>output/behavior"]
    style A fill:#3498db,color:#fff
    style B fill:#9b59b6,color:#fff
    style C fill:#27ae60,color:#fff
```

---

### 3.1 Frontend Unit Test — Pure Function

**Use case**: A pricing utility used across an e-commerce site.

```javascript
// utils/pricing.js
export function applyDiscount(price, discountPercent) {
  if (discountPercent < 0 || discountPercent > 100) {
    throw new Error('Discount must be between 0 and 100');
  }
  return price - (price * discountPercent) / 100;
}

export function formatPrice(amount, currency = 'USD') {
  return new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency,
  }).format(amount);
}
```

```javascript
// utils/pricing.test.js  (using Vitest or Jest)
import { describe, it, expect } from 'vitest';
import { applyDiscount, formatPrice } from './pricing';

describe('applyDiscount', () => {
  it('applies a 10% discount correctly', () => {
    // Arrange
    const price = 100;
    const discount = 10;

    // Act
    const result = applyDiscount(price, discount);

    // Assert
    expect(result).toBe(90);
  });

  it('applies 0% discount and returns original price', () => {
    expect(applyDiscount(50, 0)).toBe(50);
  });

  it('applies 100% discount and returns 0', () => {
    expect(applyDiscount(200, 100)).toBe(0);
  });

  it('throws when discount is negative', () => {
    expect(() => applyDiscount(100, -5)).toThrow('Discount must be between 0 and 100');
  });

  it('throws when discount exceeds 100', () => {
    expect(() => applyDiscount(100, 110)).toThrow('Discount must be between 0 and 100');
  });
});

describe('formatPrice', () => {
  it('formats USD by default', () => {
    expect(formatPrice(1234.5)).toBe('$1,234.50');
  });

  it('formats EUR currency', () => {
    expect(formatPrice(99.99, 'EUR')).toContain('99.99');
  });
});
```

---

### 3.2 API Unit Test — Service Layer

**Use case**: A Node.js user service with a database dependency.

```javascript
// services/userService.js
export class UserService {
  constructor(db) {
    this.db = db;
  }

  async getUserById(id) {
    if (!id || typeof id !== 'string') {
      throw new Error('Invalid user ID');
    }
    const user = await this.db.findOne({ id });
    if (!user) throw new Error(`User ${id} not found`);
    return user;
  }

  async createUser(data) {
    const { email, name } = data;
    if (!email || !name) {
      throw new Error('email and name are required');
    }
    const existing = await this.db.findOne({ email });
    if (existing) throw new Error('Email already in use');
    return this.db.insert({ email, name, createdAt: new Date() });
  }
}
```

```javascript
// services/userService.test.js
import { describe, it, expect, vi, beforeEach } from 'vitest';
import { UserService } from './userService';

describe('UserService', () => {
  let db;
  let service;

  beforeEach(() => {
    // Arrange: create a fresh mock db before each test
    db = {
      findOne: vi.fn(),
      insert: vi.fn(),
    };
    service = new UserService(db);
  });

  describe('getUserById', () => {
    it('returns a user when found', async () => {
      const mockUser = { id: 'u1', name: 'Alice', email: 'alice@example.com' };
      db.findOne.mockResolvedValue(mockUser);

      const result = await service.getUserById('u1');

      expect(db.findOne).toHaveBeenCalledWith({ id: 'u1' });
      expect(result).toEqual(mockUser);
    });

    it('throws when user is not found', async () => {
      db.findOne.mockResolvedValue(null);

      await expect(service.getUserById('missing')).rejects.toThrow('User missing not found');
    });

    it('throws for invalid id types', async () => {
      await expect(service.getUserById(null)).rejects.toThrow('Invalid user ID');
      await expect(service.getUserById(123)).rejects.toThrow('Invalid user ID');
    });
  });

  describe('createUser', () => {
    it('creates a user with valid data', async () => {
      db.findOne.mockResolvedValue(null); // no existing user
      db.insert.mockResolvedValue({ id: 'u2', email: 'bob@example.com', name: 'Bob' });

      const result = await service.createUser({ email: 'bob@example.com', name: 'Bob' });

      expect(db.insert).toHaveBeenCalled();
      expect(result.email).toBe('bob@example.com');
    });

    it('throws when email is already in use', async () => {
      db.findOne.mockResolvedValue({ email: 'taken@example.com' });

      await expect(
        service.createUser({ email: 'taken@example.com', name: 'Carol' })
      ).rejects.toThrow('Email already in use');
    });

    it('throws when required fields are missing', async () => {
      await expect(service.createUser({ name: 'No Email' })).rejects.toThrow(
        'email and name are required'
      );
    });
  });
});
```

> **Key insight**: The database is mocked — the test runs in milliseconds and doesn't need a real DB connection. This is what makes unit tests fast and reliable.

---

### 3.3 Python Unit Test (Pytest)

**Use case**: A data transformation pipeline function.

```python
# transform.py
from typing import List, Dict

def normalize_user_records(records: List[Dict]) -> List[Dict]:
    """Normalize raw API records to internal schema."""
    result = []
    for r in records:
        if not r.get('email'):
            continue  # skip invalid records
        result.append({
            'id': str(r['id']),
            'email': r['email'].lower().strip(),
            'full_name': f"{r.get('first_name', '')} {r.get('last_name', '')}".strip(),
            'active': r.get('status') == 'active',
        })
    return result
```

```python
# test_transform.py
import pytest
from transform import normalize_user_records

class TestNormalizeUserRecords:
    def test_normalizes_basic_record(self):
        records = [{'id': 1, 'email': '  Alice@Example.com  ',
                    'first_name': 'Alice', 'last_name': 'Smith', 'status': 'active'}]
        result = normalize_user_records(records)
        assert result == [{'id': '1', 'email': 'alice@example.com',
                           'full_name': 'Alice Smith', 'active': True}]

    def test_skips_records_without_email(self):
        records = [{'id': 2, 'first_name': 'Bob', 'status': 'active'}]
        result = normalize_user_records(records)
        assert result == []

    def test_inactive_user_has_active_false(self):
        records = [{'id': 3, 'email': 'carol@example.com', 'status': 'inactive'}]
        result = normalize_user_records(records)
        assert result[0]['active'] is False

    def test_empty_input_returns_empty_list(self):
        assert normalize_user_records([]) == []

    def test_handles_missing_name_fields(self):
        records = [{'id': 4, 'email': 'noname@example.com'}]
        result = normalize_user_records(records)
        assert result[0]['full_name'] == ''

    @pytest.mark.parametrize("email,expected", [
        ('USER@DOMAIN.COM', 'user@domain.com'),
        ('  spaces@test.com  ', 'spaces@test.com'),
        ('Mixed.Case@Test.ORG', 'mixed.case@test.org'),
    ])
    def test_email_normalization(self, email, expected):
        records = [{'id': 1, 'email': email}]
        result = normalize_user_records(records)
        assert result[0]['email'] == expected
```

---

## 4. Integration Testing

> 🟡 **Intermediate**

Integration tests verify that **multiple units work together correctly** — typically a service + database, or multiple services communicating.

```mermaid
flowchart LR
    T["Test"] --> S["Service Layer"]
    S --> DB[("Real DB\n(Test Instance)")]
    S --> C["Cache\n(Redis Test)"]
    style T fill:#3498db,color:#fff
    style DB fill:#e67e22,color:#fff
    style C fill:#e74c3c,color:#fff
```

### 4.1 API Integration Test — Express + PostgreSQL

**Use case**: Testing the user registration endpoint end-to-end through the HTTP layer.

```javascript
// tests/integration/users.test.js
import request from 'supertest';
import { app } from '../../src/app';
import { db } from '../../src/db';

// This test hits a REAL test database
beforeAll(async () => {
  await db.migrate.latest(); // run migrations
});

afterAll(async () => {
  await db.destroy();
});

beforeEach(async () => {
  await db('users').truncate(); // clean state between tests
});

describe('POST /api/users', () => {
  it('creates a new user and returns 201', async () => {
    const response = await request(app)
      .post('/api/users')
      .send({ email: 'alice@example.com', name: 'Alice' })
      .expect(201);

    expect(response.body).toMatchObject({
      id: expect.any(String),
      email: 'alice@example.com',
      name: 'Alice',
    });

    // Verify it was actually saved to the DB
    const saved = await db('users').where({ email: 'alice@example.com' }).first();
    expect(saved).toBeTruthy();
  });

  it('returns 409 when email already exists', async () => {
    await db('users').insert({ email: 'taken@example.com', name: 'Bob' });

    const response = await request(app)
      .post('/api/users')
      .send({ email: 'taken@example.com', name: 'Carol' })
      .expect(409);

    expect(response.body.error).toMatch(/already in use/i);
  });

  it('returns 400 for missing required fields', async () => {
    await request(app)
      .post('/api/users')
      .send({ name: 'No Email' })
      .expect(400);
  });
});

describe('GET /api/users/:id', () => {
  it('returns the user when found', async () => {
    const [userId] = await db('users').insert(
      { email: 'dave@example.com', name: 'Dave' },
      ['id']
    );

    const response = await request(app)
      .get(`/api/users/${userId}`)
      .expect(200);

    expect(response.body.email).toBe('dave@example.com');
  });

  it('returns 404 when user does not exist', async () => {
    await request(app).get('/api/users/nonexistent-id').expect(404);
  });
});
```

### 4.2 Python Integration Test — FastAPI + SQLAlchemy

```python
# tests/integration/test_users_api.py
import pytest
from httpx import AsyncClient
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession

from app.main import app
from app.db import Base, get_session

TEST_DB_URL = "postgresql+asyncpg://user:pass@localhost/testdb"
engine = create_async_engine(TEST_DB_URL)

@pytest.fixture(autouse=True)
async def clean_db():
    async with engine.begin() as conn:
        await conn.run_sync(Base.metadata.create_all)
    yield
    async with engine.begin() as conn:
        await conn.run_sync(Base.metadata.drop_all)

@pytest.fixture
async def client():
    async with AsyncClient(app=app, base_url="http://test") as c:
        yield c

@pytest.mark.asyncio
async def test_create_user_success(client):
    response = await client.post("/users", json={"email": "alice@example.com", "name": "Alice"})
    assert response.status_code == 201
    data = response.json()
    assert data["email"] == "alice@example.com"
    assert "id" in data

@pytest.mark.asyncio
async def test_create_user_duplicate_email(client):
    payload = {"email": "bob@example.com", "name": "Bob"}
    await client.post("/users", json=payload)
    response = await client.post("/users", json=payload)
    assert response.status_code == 409

@pytest.mark.asyncio
async def test_list_users_pagination(client):
    for i in range(5):
        await client.post("/users", json={"email": f"user{i}@example.com", "name": f"User {i}"})

    response = await client.get("/users?limit=3&offset=0")
    assert response.status_code == 200
    assert len(response.json()["items"]) == 3
    assert response.json()["total"] == 5
```

---

## 5. End-to-End Testing

> 🟡 **Intermediate**

E2E tests simulate a **real user interacting with a real browser** against a fully running application. They catch issues that unit and integration tests miss — routing bugs, CSS layout breaks, auth flows, etc.

```mermaid
sequenceDiagram
    participant T as Test Runner
    participant B as Browser (Playwright)
    participant FE as Frontend App
    participant API as Backend API
    participant DB as Database

    T->>B: Launch browser
    B->>FE: Navigate to /login
    FE->>B: Render login form
    B->>FE: Fill email + password
    B->>FE: Click "Sign In"
    FE->>API: POST /auth/login
    API->>DB: Verify credentials
    DB-->>API: User found
    API-->>FE: JWT token + redirect
    FE->>B: Navigate to /dashboard
    B->>T: Assert dashboard visible ✅
```

### 5.1 Playwright E2E Test — Login Flow

**Use case**: Ensuring the login → dashboard flow works for real users.

```typescript
// tests/e2e/auth.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Authentication', () => {
  test.beforeEach(async ({ page }) => {
    // Seed a test user via API before each test
    await page.request.post('/api/test/seed', {
      data: { email: 'test@example.com', password: 'Password123!', name: 'Test User' },
    });
  });

  test('user can log in and sees the dashboard', async ({ page }) => {
    await page.goto('/login');

    // Interact like a real user
    await page.getByLabel('Email').fill('test@example.com');
    await page.getByLabel('Password').fill('Password123!');
    await page.getByRole('button', { name: 'Sign In' }).click();

    // After login, should land on dashboard
    await expect(page).toHaveURL('/dashboard');
    await expect(page.getByRole('heading', { name: 'Welcome, Test User' })).toBeVisible();
  });

  test('shows error message for invalid credentials', async ({ page }) => {
    await page.goto('/login');
    await page.getByLabel('Email').fill('test@example.com');
    await page.getByLabel('Password').fill('wrongpassword');
    await page.getByRole('button', { name: 'Sign In' }).click();

    await expect(page.getByText('Invalid email or password')).toBeVisible();
    await expect(page).toHaveURL('/login'); // didn't navigate away
  });

  test('redirect to login when accessing protected page unauthenticated', async ({ page }) => {
    await page.goto('/dashboard');
    await expect(page).toHaveURL('/login?redirect=/dashboard');
  });
});
```

### 5.2 Playwright E2E Test — Shopping Cart

**Use case**: A critical checkout flow on an e-commerce site.

```typescript
// tests/e2e/checkout.spec.ts
import { test, expect } from '@playwright/test';
import { loginAs } from './helpers/auth';

test.describe('Checkout flow', () => {
  test.beforeEach(async ({ page }) => {
    await loginAs(page, 'customer@example.com');
  });

  test('user can add item to cart and complete purchase', async ({ page }) => {
    // Browse to a product
    await page.goto('/products/awesome-widget');
    await expect(page.getByRole('heading', { name: 'Awesome Widget' })).toBeVisible();

    // Add to cart
    await page.getByRole('button', { name: 'Add to Cart' }).click();
    await expect(page.getByTestId('cart-count')).toHaveText('1');

    // Go to cart
    await page.getByTestId('cart-icon').click();
    await expect(page).toHaveURL('/cart');
    await expect(page.getByText('Awesome Widget')).toBeVisible();

    // Proceed to checkout
    await page.getByRole('button', { name: 'Proceed to Checkout' }).click();

    // Fill shipping info
    await page.getByLabel('Address').fill('123 Main St');
    await page.getByLabel('City').fill('Springfield');
    await page.getByLabel('ZIP').fill('12345');

    // Fill payment (use test card)
    await page.getByLabel('Card Number').fill('4242 4242 4242 4242');
    await page.getByLabel('Expiry').fill('12/28');
    await page.getByLabel('CVC').fill('123');

    await page.getByRole('button', { name: 'Place Order' }).click();

    // Verify success
    await expect(page.getByRole('heading', { name: 'Order Confirmed!' })).toBeVisible();
    await expect(page.getByTestId('order-id')).toBeVisible();
  });
});
```

---

## 6. API Testing

> 🟡 **Intermediate**

API testing validates that your HTTP endpoints behave correctly — right status codes, response shapes, error handling, auth, and edge cases.

### API Test Decision Tree

```mermaid
flowchart TD
    Start["🚀 What are you testing?"] --> Q1{"Is it a simple\nrequest/response?"}
    Q1 -->|Yes| UT["Unit Test\nMock the HTTP layer"]
    Q1 -->|No| Q2{"Does it need a\nreal database?"}
    Q2 -->|Yes| IT["Integration Test\nUse a test DB"]
    Q2 -->|No| Q3{"Is it a\ncritical flow?"}
    Q3 -->|Yes| E2E["E2E Test\nFull stack"]
    Q3 -->|No| CT["Contract Test\nPact or similar"]

    style Start fill:#3498db,color:#fff
    style UT fill:#27ae60,color:#fff
    style IT fill:#f39c12,color:#fff
    style E2E fill:#e74c3c,color:#fff
    style CT fill:#9b59b6,color:#fff
```

### 6.1 REST API Test Suite with Supertest

**Use case**: A complete CRUD API for blog posts.

```javascript
// tests/api/posts.test.js
import request from 'supertest';
import { app } from '../../src/app';
import { createToken } from '../helpers/auth';

const authToken = createToken({ userId: 'user-1', role: 'author' });
const adminToken = createToken({ userId: 'admin-1', role: 'admin' });

describe('Posts API', () => {
  describe('GET /api/posts', () => {
    it('returns paginated list of published posts', async () => {
      const res = await request(app).get('/api/posts?page=1&limit=10').expect(200);

      expect(res.body).toMatchObject({
        data: expect.arrayContaining([
          expect.objectContaining({
            id: expect.any(String),
            title: expect.any(String),
            status: 'published',
          }),
        ]),
        meta: {
          page: 1,
          limit: 10,
          total: expect.any(Number),
        },
      });
    });

    it('filters by category', async () => {
      const res = await request(app).get('/api/posts?category=tech').expect(200);
      res.body.data.forEach(post => expect(post.category).toBe('tech'));
    });

    it('returns empty array when no posts match', async () => {
      const res = await request(app).get('/api/posts?category=nonexistent').expect(200);
      expect(res.body.data).toEqual([]);
    });
  });

  describe('POST /api/posts', () => {
    it('creates a post when authenticated', async () => {
      const res = await request(app)
        .post('/api/posts')
        .set('Authorization', `Bearer ${authToken}`)
        .send({ title: 'Hello World', body: 'Content here', category: 'tech' })
        .expect(201);

      expect(res.body).toMatchObject({
        id: expect.any(String),
        title: 'Hello World',
        authorId: 'user-1',
        status: 'draft',
      });
    });

    it('returns 401 without authentication', async () => {
      await request(app)
        .post('/api/posts')
        .send({ title: 'Test', body: 'Content' })
        .expect(401);
    });

    it('returns 422 with validation errors', async () => {
      const res = await request(app)
        .post('/api/posts')
        .set('Authorization', `Bearer ${authToken}`)
        .send({ body: 'Missing title' })
        .expect(422);

      expect(res.body.errors).toContainEqual(
        expect.objectContaining({ field: 'title', message: expect.any(String) })
      );
    });
  });

  describe('DELETE /api/posts/:id', () => {
    it('allows admin to delete any post', async () => {
      await request(app)
        .delete('/api/posts/post-123')
        .set('Authorization', `Bearer ${adminToken}`)
        .expect(204);
    });

    it('forbids author from deleting another author\'s post', async () => {
      await request(app)
        .delete('/api/posts/post-by-other-author')
        .set('Authorization', `Bearer ${authToken}`)
        .expect(403);
    });
  });
});
```

### 6.2 GraphQL API Testing

```javascript
// tests/api/graphql.test.js
import request from 'supertest';
import { app } from '../../src/app';

const gql = (query, variables = {}) =>
  request(app)
    .post('/graphql')
    .send({ query, variables });

describe('GraphQL API', () => {
  describe('Query: user', () => {
    it('returns a user by id', async () => {
      const res = await gql(`
        query GetUser($id: ID!) {
          user(id: $id) {
            id
            name
            email
          }
        }
      `, { id: 'user-1' }).expect(200);

      expect(res.body.errors).toBeUndefined();
      expect(res.body.data.user).toMatchObject({
        id: 'user-1',
        name: expect.any(String),
        email: expect.any(String),
      });
    });

    it('returns null for nonexistent user', async () => {
      const res = await gql(`
        query { user(id: "nonexistent") { id name } }
      `).expect(200);

      expect(res.body.data.user).toBeNull();
    });
  });

  describe('Mutation: createPost', () => {
    it('creates a post successfully', async () => {
      const res = await gql(`
        mutation CreatePost($input: CreatePostInput!) {
          createPost(input: $input) {
            id
            title
            status
          }
        }
      `, { input: { title: 'GraphQL Post', body: 'Content', categoryId: 'cat-1' } })
        .set('Authorization', 'Bearer valid-token')
        .expect(200);

      expect(res.body.data.createPost.status).toBe('DRAFT');
    });
  });
});
```

---

## 7. Frontend Testing

> 🟡 **Intermediate**

Frontend testing covers components, user interactions, state management, and rendering behavior.

### Frontend Testing Stack Overview

```mermaid
graph LR
    subgraph "React Testing"
    RTL["React Testing Library\n(User-centric)"]
    JEST["Jest / Vitest\n(Runner + Assertions)"]
    JSDOM["jsdom\n(Virtual DOM)"]
    end

    subgraph "Vue Testing"
    VTU["Vue Test Utils\n(Component testing)"]
    end

    subgraph "E2E"
    PW["Playwright\n(Real browser)"]
    CY["Cypress\n(Real browser)"]
    end

    RTL --> JEST
    JEST --> JSDOM
    VTU --> JEST
```

### 7.1 React Component Testing

**Use case**: Testing a product search component with filtering.

```tsx
// components/ProductSearch.tsx
import { useState } from 'react';

interface Product {
  id: string;
  name: string;
  category: string;
  price: number;
}

interface Props {
  products: Product[];
  onAddToCart: (product: Product) => void;
}

export function ProductSearch({ products, onAddToCart }: Props) {
  const [query, setQuery] = useState('');
  const [category, setCategory] = useState('all');

  const filtered = products.filter(p => {
    const matchesQuery = p.name.toLowerCase().includes(query.toLowerCase());
    const matchesCategory = category === 'all' || p.category === category;
    return matchesQuery && matchesCategory;
  });

  return (
    <div>
      <input
        aria-label="Search products"
        value={query}
        onChange={e => setQuery(e.target.value)}
        placeholder="Search..."
      />
      <select
        aria-label="Filter by category"
        value={category}
        onChange={e => setCategory(e.target.value)}
      >
        <option value="all">All Categories</option>
        <option value="electronics">Electronics</option>
        <option value="clothing">Clothing</option>
      </select>

      {filtered.length === 0 ? (
        <p role="status">No products found</p>
      ) : (
        <ul>
          {filtered.map(product => (
            <li key={product.id}>
              <span>{product.name}</span>
              <span>${product.price}</span>
              <button onClick={() => onAddToCart(product)}>
                Add to Cart
              </button>
            </li>
          ))}
        </ul>
      )}
    </div>
  );
}
```

```tsx
// components/ProductSearch.test.tsx
import { render, screen, fireEvent } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { describe, it, expect, vi } from 'vitest';
import { ProductSearch } from './ProductSearch';

const mockProducts = [
  { id: '1', name: 'MacBook Pro', category: 'electronics', price: 1999 },
  { id: '2', name: 'iPhone 15', category: 'electronics', price: 999 },
  { id: '3', name: 'Blue T-Shirt', category: 'clothing', price: 29 },
  { id: '4', name: 'Running Shoes', category: 'clothing', price: 89 },
];

describe('ProductSearch', () => {
  it('renders all products initially', () => {
    render(<ProductSearch products={mockProducts} onAddToCart={vi.fn()} />);
    expect(screen.getAllByRole('listitem')).toHaveLength(4);
  });

  it('filters products by search query', async () => {
    const user = userEvent.setup();
    render(<ProductSearch products={mockProducts} onAddToCart={vi.fn()} />);

    await user.type(screen.getByLabelText('Search products'), 'iphone');

    expect(screen.getAllByRole('listitem')).toHaveLength(1);
    expect(screen.getByText('iPhone 15')).toBeInTheDocument();
  });

  it('filters products by category', async () => {
    const user = userEvent.setup();
    render(<ProductSearch products={mockProducts} onAddToCart={vi.fn()} />);

    await user.selectOptions(screen.getByLabelText('Filter by category'), 'clothing');

    const items = screen.getAllByRole('listitem');
    expect(items).toHaveLength(2);
    expect(screen.getByText('Blue T-Shirt')).toBeInTheDocument();
    expect(screen.queryByText('MacBook Pro')).not.toBeInTheDocument();
  });

  it('shows empty state when no products match', async () => {
    const user = userEvent.setup();
    render(<ProductSearch products={mockProducts} onAddToCart={vi.fn()} />);

    await user.type(screen.getByLabelText('Search products'), 'xyz-doesnt-exist');

    expect(screen.getByRole('status')).toHaveTextContent('No products found');
  });

  it('calls onAddToCart with correct product when button clicked', async () => {
    const user = userEvent.setup();
    const onAddToCart = vi.fn();
    render(<ProductSearch products={mockProducts} onAddToCart={onAddToCart} />);

    await user.type(screen.getByLabelText('Search products'), 'macbook');
    await user.click(screen.getByRole('button', { name: 'Add to Cart' }));

    expect(onAddToCart).toHaveBeenCalledOnce();
    expect(onAddToCart).toHaveBeenCalledWith(mockProducts[0]);
  });

  it('search is case-insensitive', async () => {
    const user = userEvent.setup();
    render(<ProductSearch products={mockProducts} onAddToCart={vi.fn()} />);

    await user.type(screen.getByLabelText('Search products'), 'MACBOOK');

    expect(screen.getByText('MacBook Pro')).toBeInTheDocument();
  });
});
```

### 7.2 Testing Async Components & API Calls

**Use case**: A component that fetches user data.

```tsx
// components/UserProfile.test.tsx
import { render, screen, waitFor } from '@testing-library/react';
import { vi, describe, it, expect, beforeEach } from 'vitest';
import { UserProfile } from './UserProfile';
import * as api from '../api/users';

vi.mock('../api/users');

describe('UserProfile', () => {
  beforeEach(() => {
    vi.clearAllMocks();
  });

  it('shows loading state initially', () => {
    vi.mocked(api.getUser).mockResolvedValue({ id: '1', name: 'Alice' });
    render(<UserProfile userId="1" />);
    expect(screen.getByRole('status', { name: /loading/i })).toBeInTheDocument();
  });

  it('displays user data after fetch', async () => {
    vi.mocked(api.getUser).mockResolvedValue({ id: '1', name: 'Alice', email: 'alice@example.com' });
    render(<UserProfile userId="1" />);

    await waitFor(() => {
      expect(screen.getByText('Alice')).toBeInTheDocument();
      expect(screen.getByText('alice@example.com')).toBeInTheDocument();
    });
  });

  it('shows error message when fetch fails', async () => {
    vi.mocked(api.getUser).mockRejectedValue(new Error('Network error'));
    render(<UserProfile userId="1" />);

    await waitFor(() => {
      expect(screen.getByRole('alert')).toHaveTextContent(/failed to load user/i);
    });
  });
});
```

---

## 8. Performance Testing

> 🔴 **Advanced**

Performance tests verify your application handles load, responds quickly, and doesn't degrade under stress.

### Types of Performance Testing

```mermaid
graph TD
    PERF["Performance Testing"]
    PERF --> LOAD["Load Testing\nNormal expected traffic\n(k6, Locust, JMeter)"]
    PERF --> STRESS["Stress Testing\nBeyond normal capacity\nFind breaking point"]
    PERF --> SOAK["Soak Testing\nSustained load over hours\nFind memory leaks"]
    PERF --> SPIKE["Spike Testing\nSudden traffic bursts\nFlash sales, viral events"]
    PERF --> BENCH["Benchmarking\nMeasure specific operations\n(autocannon, wrk)"]

    style PERF fill:#2c3e50,color:#fff
    style LOAD fill:#3498db,color:#fff
    style STRESS fill:#e74c3c,color:#fff
    style SOAK fill:#f39c12,color:#fff
    style SPIKE fill:#9b59b6,color:#fff
    style BENCH fill:#27ae60,color:#fff
```

### 8.1 Load Testing with k6

**Use case**: Load testing a user authentication endpoint before a product launch.

```javascript
// tests/performance/login.k6.js
import http from 'k6/http';
import { check, sleep } from 'k6';
import { Rate, Trend } from 'k6/metrics';

const errorRate = new Rate('errors');
const loginDuration = new Trend('login_duration', true);

export const options = {
  stages: [
    { duration: '30s', target: 10 },   // ramp up to 10 users
    { duration: '1m', target: 50 },    // ramp to 50 users
    { duration: '2m', target: 50 },    // hold at 50
    { duration: '30s', target: 0 },    // ramp down
  ],
  thresholds: {
    http_req_duration: ['p(95)<500'],   // 95% of requests must complete under 500ms
    errors: ['rate<0.01'],              // Error rate must stay below 1%
    login_duration: ['p(99)<1000'],     // 99th percentile under 1s
  },
};

export default function () {
  const payload = JSON.stringify({
    email: `user${Math.floor(Math.random() * 1000)}@example.com`,
    password: 'Password123!',
  });

  const params = {
    headers: { 'Content-Type': 'application/json' },
  };

  const res = http.post('http://api.example.com/auth/login', payload, params);

  const success = check(res, {
    'status is 200 or 401': (r) => r.status === 200 || r.status === 401,
    'response time OK': (r) => r.timings.duration < 500,
    'has auth token': (r) => r.status !== 200 || r.json('token') !== undefined,
  });

  errorRate.add(!success);
  loginDuration.add(res.timings.duration);

  sleep(1);
}
```

### 8.2 Performance Budget in CI

```yaml
# .github/workflows/perf.yml excerpt
- name: Run performance test
  run: k6 run --out json=results.json tests/performance/login.k6.js

- name: Assert performance budget
  run: |
    p95=$(cat results.json | jq '[.metrics.http_req_duration.values.p(95)] | .[0]')
    if (( $(echo "$p95 > 500" | bc -l) )); then
      echo "❌ p95 response time $p95ms exceeds 500ms budget"
      exit 1
    fi
    echo "✅ p95 response time: ${p95}ms"
```

---

## 9. Security Testing

> 🔴 **Advanced**

Security testing finds vulnerabilities before attackers do.

```mermaid
mindmap
  root((Security Testing))
    SAST
      Static code analysis
      Run in CI on every PR
      Tools: Semgrep, Bandit, ESLint security
    DAST
      Dynamic analysis
      Running application
      Tools: OWASP ZAP, Burp Suite
    Dependency Scanning
      Known CVEs
      Tools: npm audit, Snyk, Trivy
    Secrets Detection
      Leaked API keys
      Tools: truffleHog, detect-secrets
    Penetration Testing
      Manual expert testing
      Red team exercises
```

### 9.1 Input Validation Security Tests

**Use case**: Preventing SQL injection and XSS in user inputs.

```javascript
// tests/security/input-validation.test.js
import request from 'supertest';
import { app } from '../../src/app';

describe('Input Validation Security', () => {
  describe('SQL Injection Prevention', () => {
    const sqlPayloads = [
      "'; DROP TABLE users; --",
      "' OR '1'='1",
      "1; SELECT * FROM users",
      "admin'--",
      "' UNION SELECT * FROM users --",
    ];

    sqlPayloads.forEach(payload => {
      it(`rejects SQL injection payload: ${payload.substring(0, 20)}...`, async () => {
        const res = await request(app)
          .get(`/api/users?search=${encodeURIComponent(payload)}`);

        // Should either sanitize or return 400, never 500 (which suggests DB error)
        expect(res.status).not.toBe(500);
        if (res.status === 200) {
          // If it succeeded, it should have sanitized the input
          expect(res.body.data).toBeInstanceOf(Array);
        }
      });
    });
  });

  describe('XSS Prevention', () => {
    it('sanitizes HTML in user-submitted content', async () => {
      const xssPayload = '<script>alert("xss")</script>';

      const res = await request(app)
        .post('/api/posts')
        .set('Authorization', 'Bearer valid-token')
        .send({ title: 'Normal Title', body: xssPayload });

      if (res.status === 201) {
        expect(res.body.body).not.toContain('<script>');
        expect(res.body.body).not.toContain('alert');
      }
    });
  });

  describe('Rate Limiting', () => {
    it('blocks requests after exceeding rate limit', async () => {
      const requests = Array(101).fill(null).map(() =>
        request(app).post('/auth/login').send({
          email: 'brute@example.com',
          password: 'wrong',
        })
      );

      const responses = await Promise.all(requests);
      const tooManyRequests = responses.filter(r => r.status === 429);
      expect(tooManyRequests.length).toBeGreaterThan(0);
    });
  });

  describe('Authentication Headers', () => {
    it('returns proper security headers', async () => {
      const res = await request(app).get('/');
      expect(res.headers['x-content-type-options']).toBe('nosniff');
      expect(res.headers['x-frame-options']).toBeDefined();
      expect(res.headers['strict-transport-security']).toBeDefined();
    });
  });
});
```

---

## 10. Contract Testing

> 🔴 **Advanced**

Contract testing ensures that **services that communicate with each other maintain compatibility**, without needing to run both services simultaneously.

```mermaid
sequenceDiagram
    participant C as Consumer<br/>(Frontend/API Client)
    participant P as Pact Broker
    participant P2 as Provider<br/>(Backend API)

    C->>C: Write consumer tests<br/>defining expected API shape
    C->>P: Publish contract (pact file)
    P2->>P: Fetch contract
    P2->>P2: Verify contract against<br/>real implementation
    P2->>P: Publish verification result
    C->>P: Can I deploy?
    P-->>C: ✅ Yes, contract verified
```

### 10.1 Pact Consumer Test

```javascript
// tests/contract/userApi.consumer.test.js
import { Pact } from '@pact-foundation/pact';
import { UserApiClient } from '../../src/clients/userApiClient';

const provider = new Pact({
  consumer: 'FrontendApp',
  provider: 'UserService',
  port: 8080,
});

describe('UserService Contract - Consumer', () => {
  beforeAll(() => provider.setup());
  afterAll(() => provider.finalize());
  afterEach(() => provider.verify());

  describe('GET /users/:id', () => {
    it('returns user when found', async () => {
      // Define what we EXPECT the provider to return
      await provider.addInteraction({
        state: 'user u-123 exists',
        uponReceiving: 'a request for user u-123',
        withRequest: {
          method: 'GET',
          path: '/users/u-123',
          headers: { Accept: 'application/json' },
        },
        willRespondWith: {
          status: 200,
          headers: { 'Content-Type': 'application/json' },
          body: {
            id: 'u-123',
            name: 'Alice Smith',
            email: 'alice@example.com',
          },
        },
      });

      const client = new UserApiClient('http://localhost:8080');
      const user = await client.getUser('u-123');

      expect(user.id).toBe('u-123');
      expect(user.name).toBe('Alice Smith');
    });
  });
});
```

---

## 11. CI/CD Integration

> 🔴 **Advanced**

This is where testing becomes **a quality gate**, not just a developer tool.

### CI/CD Testing Pipeline

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

### 11.1 GitHub Actions — Full Test Pipeline

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
  cancel-in-progress: true  # Cancel old runs when new commits are pushed

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
      - name: Build application
        run: npm run build
      - name: Start application
        run: npm run start &
      - name: Wait for server
        run: npx wait-on http://localhost:3000 --timeout 30000
      - name: Run E2E tests
        run: npm run test:e2e
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

### 11.2 GitLab CI Pipeline

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

### 11.3 Makefile for Local Development

```makefile
# Makefile — run these locally before pushing

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

# Run all fast checks before pushing
check: test-unit
	@echo "✅ All checks passed"

# Full test suite (slow - use before merging)
test-all: test-unit test-integration test-e2e
	@echo "✅ Full test suite passed"
```

---

## 12. Test Strategy by Role

> 🔴 **Advanced**

Different roles have different responsibilities in a testing culture.

```mermaid
graph TD
    subgraph "🎨 Frontend Developer"
    F1["Component unit tests\n(React Testing Library)"]
    F2["Snapshot tests for\nUI regressions"]
    F3["E2E for critical\nuser journeys"]
    F4["Visual regression\n(Chromatic, Percy)"]
    end

    subgraph "⚙️ Backend / API Developer"
    B1["Unit tests for\nbusiness logic"]
    B2["Integration tests\nwith real DB"]
    B3["API contract tests\n(Pact)"]
    B4["Performance tests\nfor endpoints"]
    end

    subgraph "🏗️ Senior / Staff Engineer"
    S1["Define test strategy\nand tooling"]
    S2["Set coverage thresholds\nand quality gates"]
    S3["Design testing infra\n(test containers, seeds)"]
    S4["Conduct chaos/resilience\ntesting"]
    end

    subgraph "🚀 DevOps / Platform"
    D1["Build CI/CD pipelines"]
    D2["Manage test environments"]
    D3["Monitor flaky tests"]
    D4["Track coverage trends"]
    end
```

### Testing Maturity Model

```mermaid
graph LR
    L0["Level 0\n😰 No Tests\nFear of change"] -->
    L1["Level 1\n🌱 Some Unit Tests\nBasic confidence"] -->
    L2["Level 2\n🌿 Unit + Integration\nPR gates"] -->
    L3["Level 3\n🌳 Full Pyramid\nCI/CD quality gates"] -->
    L4["Level 4\n🏆 TDD + Chaos\nTest-driven culture"]

    style L0 fill:#e74c3c,color:#fff
    style L1 fill:#e67e22,color:#fff
    style L2 fill:#f39c12,color:#fff
    style L3 fill:#27ae60,color:#fff
    style L4 fill:#2980b9,color:#fff
```

### What to Test by Scenario

| Scenario | Unit | Integration | E2E | Performance | Security |
|----------|------|-------------|-----|-------------|----------|
| New utility function | ✅ Must | ❌ Skip | ❌ Skip | ❌ Skip | ❌ Skip |
| New API endpoint | ✅ Business logic | ✅ HTTP + DB | ⚠️ Critical only | ⚠️ High traffic | ✅ Auth/validation |
| New UI component | ✅ Logic/render | ❌ Skip | ⚠️ Critical flow | ❌ Skip | ❌ Skip |
| Auth flow change | ✅ Token logic | ✅ Full flow | ✅ Must | ✅ Load test | ✅ Security test |
| Database migration | ❌ Skip | ✅ Must | ❌ Skip | ⚠️ If large table | ❌ Skip |
| Payment integration | ✅ Calculation | ✅ With sandbox | ✅ Full checkout | ✅ Must | ✅ Must |

---

## 13. Quick Reference Cheatsheet

> All levels

### Test Naming Conventions

```
it('should [expected behavior] when [condition]')
it('[unit] [action] [expected result]')

✅  it('returns 404 when user does not exist')
✅  it('applyDiscount throws when discount exceeds 100')
✅  it('shows loading spinner while data is fetching')

❌  it('test 1')
❌  it('works correctly')
❌  it('handles edge cases')
```

### Mock vs Stub vs Spy vs Fake

```mermaid
graph LR
    subgraph "Test Double Types"
    STUB["Stub\nReturns hardcoded\nvalues"]
    MOCK["Mock\nVerifies it was\ncalled correctly"]
    SPY["Spy\nWraps real impl,\nrecords calls"]
    FAKE["Fake\nWorking simplified\nimplementation\n(e.g. in-memory DB)"]
    end
```

| Double | Use When |
|--------|----------|
| **Stub** | You need a dependency to return a specific value |
| **Mock** | You need to verify a function was called (with specific args) |
| **Spy** | You want to observe a real function without replacing it |
| **Fake** | You need a lightweight working version (e.g. in-memory database for tests) |

### Common Testing Pitfalls

```mermaid
mindmap
  root((⚠️ Pitfalls))
    Testing Implementation
      Not behavior
      Breaks on refactor
      Tests are useless
    Slow Tests
      Network calls in unit tests
      No test isolation
      Missing cleanup
    Flaky Tests
      Timing dependencies
      Shared global state
      Hardcoded ports
    Too Many Mocks
      Tests pass, reality fails
      Contract tests help here
    No Negative Tests
      Only test happy path
      Bugs hide in errors
```

### Assertions Cheatsheet (Vitest / Jest)

```javascript
// Value equality
expect(value).toBe(42);                  // strict equality (===)
expect(obj).toEqual({ a: 1 });           // deep equality
expect(obj).toMatchObject({ a: 1 });     // partial match (obj can have more keys)

// Truthiness
expect(value).toBeTruthy();
expect(value).toBeFalsy();
expect(value).toBeNull();
expect(value).toBeDefined();

// Numbers
expect(val).toBeGreaterThan(10);
expect(val).toBeCloseTo(3.14, 2);        // float comparison with precision

// Strings
expect(str).toContain('substring');
expect(str).toMatch(/regex/);

// Arrays
expect(arr).toHaveLength(3);
expect(arr).toContain('item');
expect(arr).toEqual(expect.arrayContaining(['a', 'b']));

// Async
await expect(promise).resolves.toBe('value');
await expect(promise).rejects.toThrow('error message');

// Functions & Mocks
expect(mockFn).toHaveBeenCalled();
expect(mockFn).toHaveBeenCalledTimes(2);
expect(mockFn).toHaveBeenCalledWith('arg1', 'arg2');
expect(mockFn).toHaveBeenLastCalledWith('last-arg');
```

### Tool Recommendations by Language

| Language | Unit | Integration | E2E | Performance | Security |
|----------|------|-------------|-----|-------------|----------|
| **JavaScript/TS** | Vitest / Jest | Supertest + Vitest | Playwright / Cypress | k6 | npm audit, Semgrep |
| **Python** | pytest | pytest + httpx | Playwright | Locust | bandit, safety |
| **Go** | `testing` pkg | `net/http/httptest` | Playwright | k6 | gosec |
| **Java** | JUnit 5 + Mockito | Spring Boot Test | Selenium / Playwright | Gatling | SpotBugs |
| **Ruby** | RSpec | RSpec + FactoryBot | Capybara | Gatling | Brakeman |

---

## Appendix: Setting Up Your Test Environment

### JavaScript / TypeScript Project

```bash
# Install Vitest (recommended over Jest for modern projects)
npm install -D vitest @vitest/coverage-v8

# Install React Testing Library
npm install -D @testing-library/react @testing-library/user-event @testing-library/jest-dom

# Install Playwright
npm install -D @playwright/test
npx playwright install

# Install Supertest for API testing
npm install -D supertest @types/supertest
```

```json
// vitest.config.ts (or vitest.config.js)
{
  "test": {
    "environment": "jsdom",
    "setupFiles": ["./src/test/setup.ts"],
    "coverage": {
      "provider": "v8",
      "thresholds": {
        "lines": 80,
        "functions": 80,
        "branches": 70
      }
    }
  }
}
```

### Python Project

```bash
# Install pytest and common plugins
pip install pytest pytest-asyncio pytest-cov pytest-mock httpx

# Run tests
pytest tests/ -v --cov=src --cov-report=html

# Run only unit tests
pytest tests/unit/ -v

# Run with coverage report
pytest --cov=. --cov-report=term-missing --cov-fail-under=80
```

```ini
# pytest.ini or pyproject.toml [tool.pytest.ini_options]
[pytest]
asyncio_mode = auto
testpaths = tests
python_files = test_*.py
python_classes = Test*
python_functions = test_*
```

---

> **Last updated**: 2026 · Maintained as a living document
>
> **Contributing**: Found a better pattern or a missing test type? Open a PR — the best test guides evolve with real-world experience.
