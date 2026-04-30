---
name: tdd-outside-in-acceptance
description: Enables TDD outside-in approach by writing acceptance tests for backend HTTP endpoints (happy path only) and designing external components (DTOs, controllers, routes) before implementation. Use when starting a new feature, designing backend endpoints, outside-in TDD, or acceptance test-driven API design.
---

# TDD Outside-In Acceptance

Design backend features from the HTTP contract inward. Write acceptance tests first, design the external shell (DTOs, controllers), and leave tests red until implementation.

## Core Principle

**Tests stay red by design.** Do not iterate on acceptance tests or implement internal logic in the first phase. The acceptance tests define the contract; the external components (API layer) are scaffolded; the internals come later.

## Workflow

1. **Define the HTTP contract** – List endpoints (method, path, request/response bodies).
2. **Write acceptance tests** – Happy path only, one per endpoint.
3. **Design external components** – DTOs, controller routes, module wiring.
4. **Stop** – Leave tests red. Do not implement handlers, services, or persistence yet.

## Acceptance Test Structure

Organize by resource, then by endpoint:

```
describe('[Resource] Acceptance', () => {
  describe('Calling POST /[resource]', () => {
    it('should respond with [resource] id', async () => { ... });
    it('should create [resource] with [nested] when passed', async () => { ... });
  });

  describe('Calling GET /[resource]/:id', () => {
    it('should respond with [resource] when existing', async () => { ... });
  });

  describe('Calling PUT /[resource]/:id', () => { ... });
  describe('Calling DELETE /[resource]/:id', () => { ... });
});
```

## Test Guidelines

- **Happy path only** – No error cases, validation edges, or 404 scenarios in this phase.
- **Use fixtures** – `resourceDtoFixture(partial?: Partial<ResourceDto>)` for request bodies. Keeps tests readable.
- **Assert contract** – Status code, response body shape, ID presence/format. Use `expect.objectContaining()` or `expect.any(String)` for IDs (format is project-specific: ObjectId, UUID, prefixed strings).
- **Setup via API** – Use `beforeEach` to create data via other endpoints when a test needs existing resources.
- **No iteration** – Do not fix or refine acceptance tests initially. They document expected behavior.

## External Components to Design

| Component | Purpose |
|-----------|---------|
| Request DTOs | Validate incoming bodies (class-validator, zod, etc.) |
| Response shape | Document expected response structure; may align with DTOs |
| Controller | Define routes, accept DTOs; handlers can throw `NotImplemented` or return placeholders |
| Module wiring | Register controller, any minimal stubs needed to bootstrap tests |

## What NOT to Do

- Do not implement services, repositories, or business logic.
- Do not make acceptance tests pass.
- Do not add error-path or edge-case tests yet.
- Do not over-specify ID formats; keep assertions flexible (e.g. `expect.any(String)` or project convention).

## NestJS Example

```typescript
// Fixture
const productDtoFixture = (data: Partial<ProductDto> = {}): ProductDto => ({
  name: 'product name',
  type: ProductType.event,
  period: { start: '2026-02-19T00:00:00.000Z', end: '2026-02-20T00:00:00.000Z' },
  ...data,
});

// Test
it('should respond with product id', async () => {
  const response = await request(app.getHttpServer())
    .post('/products')
    .send(productDtoFixture());

  expect(response.status).toEqual(201);
  expect(response.body.id).toBeDefined();
  // Prefer expect.any(String) or project-specific format (ObjectId, UUID)
});
```

## Checklist

- [ ] HTTP contract defined (method, path, bodies)
- [ ] Acceptance tests written for each endpoint (happy path)
- [ ] DTOs created for requests
- [ ] Controller with routes scaffolded
- [ ] Tests run and are **red**
- [ ] No internal logic implemented

## Additional Resources

- For detailed test structure and assertion patterns, see [reference.md](reference.md)
