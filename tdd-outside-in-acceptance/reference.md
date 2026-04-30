# TDD Outside-In – Reference Patterns

## Test File Layout

```
test/
  [resource].acceptance.spec.ts   # One file per aggregate/resource
```

## Endpoint Coverage

| HTTP Method | Typical happy path |
|-------------|-------------------|
| POST /resource | 201, body contains id |
| GET /resource/:id | 200, body matches expected shape |
| PUT /resource/:id | 200/204, GET returns updated data |
| DELETE /resource/:id | 200/204, subsequent GET returns 404 |

## Fixture Pattern

```typescript
const resourceDtoFixture = (data: Partial<ResourceDto> = {}): ResourceDto => ({
  requiredField: 'default value',
  nested: { ... },
  ...data,
});
```

Use `Partial<T>` so tests override only what matters. Avoid hardcoding IDs in fixtures.

## Response Assertions

- **Create (POST)**: `expect(response.status).toEqual(201)`; assert `response.body.id` exists.
- **Read (GET)**: Use `expect.objectContaining({ ... })` for partial match; use `expect.any(String)` for IDs.
- **Update (PUT)**: Assert status; optionally GET and assert updated fields.
- **Delete (DELETE)**: Assert status; GET same resource expects 404.

## Nested Resources

For `/resource/:parentId/child`:

```typescript
describe('Calling POST /resource/:parentId/child', () => {
  let parentId: string;

  beforeEach(async () => {
    const res = await request(app).post('/resource').send(parentFixture());
    parentId = res.body.id;
  });

  it('should respond with child id', async () => {
    const response = await request(app)
      .post(`/resource/${parentId}/child`)
      .send(childDtoFixture());
    expect(response.status).toEqual(201);
    expect(response.body.id).toBeDefined();
  });
});
```

## Controller Stub

Until implementation, controllers can:

- Return `throw new NotImplementedException()` to keep tests red but compiling.
- Return minimal placeholders (e.g. `{ id: 'stub' }`) if needed for test setup – but prefer red.

## ID Format

Do not bake a specific ID format into the skill. Project may use:

- MongoDB ObjectId
- UUID
- Prefixed strings (e.g. `product_xyz`, `variant_abc`)

Use `expect.any(String)` or `toMatch(/expected-pattern/)` based on project convention.
