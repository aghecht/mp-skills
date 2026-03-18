# When to Mock

Mock at **system boundaries** only:

- External APIs (payment, email, etc.)
- Databases (sometimes - prefer test DB)
- Time/randomness
- File system (sometimes)

Don't mock:

- Your own classes/modules
- Internal collaborators
- Anything you control

## Designing for Mockability

At system boundaries, design interfaces that are easy to mock:

**1. Use dependency injection**

Pass external dependencies in rather than creating them internally:

```typescript
// Easy to mock
function processPayment(order, paymentClient) {
  return paymentClient.charge(order.total);
}

// Hard to mock
function processPayment(order) {
  const client = new StripeClient(process.env.STRIPE_KEY);
  return client.charge(order.total);
}
```

```ruby
# Easy to mock
def process_payment(order, payment_client)
  payment_client.charge(order.total)
end

# Hard to mock
def process_payment(order)
  client = StripeClient.new(ENV['STRIPE_KEY'])
  client.charge(order.total)
end
```

**2. Prefer SDK-style interfaces over generic fetchers**

Create specific functions for each external operation instead of one generic function with conditional logic:

```typescript
// GOOD: Each function is independently mockable
const api = {
  getUser: (id) => fetch(`/users/${id}`),
  getOrders: (userId) => fetch(`/users/${userId}/orders`),
  createOrder: (data) => fetch('/orders', { method: 'POST', body: data }),
};

// BAD: Mocking requires conditional logic inside the mock
const api = {
  fetch: (endpoint, options) => fetch(endpoint, options),
};
```

```ruby
# GOOD: Each method is independently mockable
module Api
  def self.get_user(id)
    Faraday.get("/users/#{id}")
  end

  def self.get_orders(user_id)
    Faraday.get("/users/#{user_id}/orders")
  end

  def self.create_order(data)
    Faraday.post('/orders', data)
  end
end

# BAD: Mocking requires conditional logic inside the mock
module Api
  def self.fetch(endpoint, options = {})
    Faraday.run_request(options[:method] || :get, endpoint, options[:body], {})
  end
end
```

The SDK approach means:
- Each mock returns one specific shape
- No conditional logic in test setup
- Easier to see which endpoints a test exercises
- Type safety per endpoint
