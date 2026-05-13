# 04 — Integration Testing

> 🟡 **Intermediate**

[← Back to Index](../README.md)

---

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

## When to Write Integration Tests

| Scenario | Write one? |
|----------|-----------|
| Service reads/writes to a real DB | ✅ Yes |
| HTTP endpoint with auth + validation | ✅ Yes |
| Two services communicating over a queue | ✅ Yes |
| Pure function with no I/O | ❌ Unit test instead |
| Full browser user flow | ❌ E2E test instead |

---

## 4.1 API Integration Test — Express + PostgreSQL

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

---

## 4.2 Python Integration Test — FastAPI + SQLAlchemy

```python
# tests/integration/test_users_api.py
import pytest
from httpx import AsyncClient
from sqlalchemy.ext.asyncio import create_async_engine

from app.main import app
from app.db import Base

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

## Running Integration Tests in CI

Use Docker service containers to spin up real databases:

```yaml
# GitHub Actions example
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
    ports:
      - 5432:5432
```

See the full pipeline in [CI/CD Integration](./11-cicd-integration.md).

---

**← Previous:** [Unit Testing](./03-unit-testing.md) · **Next →** [End-to-End Testing](./05-end-to-end-testing.md)
