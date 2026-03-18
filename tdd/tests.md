# Good and Bad Tests

## Good Tests

**Integration-style**: Test through real interfaces, not mocks of internal parts.

```typescript
// GOOD: Tests observable behavior
test("user can checkout with valid cart", async () => {
  const cart = createCart();
  cart.add(product);
  const result = await checkout(cart, paymentMethod);
  expect(result.status).toBe("confirmed");
});
```

```ruby
# GOOD: Tests observable behavior
RSpec.describe "checkout" do
  it "can checkout with valid cart" do
    cart = create_cart
    cart.add(product)
    result = checkout(cart, payment_method)
    expect(result.status).to eq("confirmed")
  end
end
```

Characteristics:

- Tests behavior users/callers care about
- Uses public API only
- Survives internal refactors
- Describes WHAT, not HOW
- One logical assertion per test

## Bad Tests

**Implementation-detail tests**: Coupled to internal structure.

```typescript
// BAD: Tests implementation details
test("checkout calls paymentService.process", async () => {
  const mockPayment = jest.mock(paymentService);
  await checkout(cart, payment);
  expect(mockPayment.process).toHaveBeenCalledWith(cart.total);
});
```

```ruby
# BAD: Tests implementation details
RSpec.describe "checkout" do
  it "calls payment_service.process during checkout" do
    mock_payment = instance_double(PaymentService)
    expect(mock_payment).to receive(:process).with(cart.total)
    checkout(cart, mock_payment)
  end
end
```

Red flags:

- Mocking internal collaborators
- Testing private methods
- Asserting on call counts/order
- Test breaks when refactoring without behavior change
- Test name describes HOW not WHAT
- Verifying through external means instead of interface

```typescript
// BAD: Bypasses interface to verify
test("createUser saves to database", async () => {
  await createUser({ name: "Alice" });
  const row = await db.query("SELECT * FROM users WHERE name = ?", ["Alice"]);
  expect(row).toBeDefined();
});

// GOOD: Verifies through interface
test("createUser makes user retrievable", async () => {
  const user = await createUser({ name: "Alice" });
  const retrieved = await getUser(user.id);
  expect(retrieved.name).toBe("Alice");
});
```

```ruby
# BAD: Bypasses interface to verify
RSpec.describe "createUser" do
  it "saves to database" do
    create_user(name: "Alice")
    row = db.query("SELECT * FROM users WHERE name = ?", ["Alice"])
    expect(row).not_to be_nil
  end
end

# GOOD: Verifies through interface
RSpec.describe "createUser" do
  it "makes user retrievable" do
    user = create_user(name: "Alice")
    retrieved = get_user(user.id)
    expect(retrieved.name).to eq("Alice")
  end
end
```
