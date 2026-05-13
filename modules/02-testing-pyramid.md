# 02 — The Testing Pyramid

> 🟢 **Beginner**

[← Back to Index](../README.md)

---

The testing pyramid describes the **ideal ratio** of test types in a healthy codebase.

## The Pyramid

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

## The Anti-Pattern: Ice Cream Cone

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

## Practical Target Ratios

| Project Size | Unit | Integration | E2E |
|-------------|------|-------------|-----|
| Small / startup | 70% | 20% | 10% |
| Medium / growing | 60% | 30% | 10% |
| Large / enterprise | 55% | 35% | 10% |

The exact numbers matter less than the **direction**: prioritise fast tests, keep E2E lean and focused on critical user journeys only.

---

**← Previous:** [Why Testing Matters](./01-why-testing-matters.md) · **Next →** [Unit Testing](./03-unit-testing.md)
