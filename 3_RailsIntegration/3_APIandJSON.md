# **APIandJSON.md**

## Why This Matters

Rails makes it incredibly easy to build APIs — sometimes *too* easy.
A single `render json: @object` can expose every column in your table, including ones that should never leave your system.

This guide teaches how to **produce clean, predictable JSON**, **accept parameters safely**, and **build stable endpoints** for both internal and external consumers.

---

## Rendering JSON in Controllers

The simplest form:

```ruby
def show
  user = User.find(params[:id])
  render json: user, status: :ok
end
```

That’s enough for small scripts — but in production you’ll want to:

* Use **strong parameters** to whitelist inputs.
* Control **what fields** are exposed.
* Add consistent **status codes** and **error shapes**.

---

## Strong Parameters

Rails controllers protect against mass assignment by requiring explicit parameter whitelisting.

```ruby
def user_params
  params.require(:user).permit(:email, :name, :role)
end

def create
  user = User.new(user_params)
  if user.save
    render json: user, status: :created
  else
    render json: { errors: user.errors.full_messages }, status: :unprocessable_entity
  end
end
```

Without strong params, someone could POST arbitrary attributes like `admin: true` — and that’s how production incidents happen.

---

## Shaping JSON Output

You can customize what gets sent by using `as_json` or `to_json` with options:

```ruby
user.as_json(only: [:id, :email], methods: :full_name)
# => { "id": 1, "email": "a@b.c", "full_name": "Alice Brown" }
```

| Option     | What it does              |
| ---------- | ------------------------- |
| `only:`    | Whitelist specific fields |
| `except:`  | Exclude fields            |
| `methods:` | Include computed methods  |
| `include:` | Nest associations         |

```ruby
order.as_json(
  only: [:id, :status],
  include: { user: { only: [:id, :email] } }
)
```

For full APIs, teams often wrap this logic in serializers (e.g. **ActiveModelSerializers**, **Blueprinter**, **JBuilder**).
Even if your project doesn’t use them, you’ll recognize patterns like:

```ruby
render json: OrderSerializer.new(order)
```

---

## Building Predictable, Stable APIs

### 1. Whitelist Everything

Never send full objects by default. Explicitly select which attributes are safe for clients.

### 2. Separate Read and Write Shapes

Input payloads (`POST`, `PATCH`) shouldn’t mirror output payloads (`GET`).
They often differ in structure and allowed fields.

### 3. Include Metadata

For lists or large responses, include `count`, `page`, or `has_more` flags.

```ruby
render json: {
  data: orders.as_json(only: %i[id status total_cents]),
  meta: { count: orders.count }
}
```

---

## Pagination and Filtering

Avoid dumping thousands of records. Use pagination helpers:

```ruby
Order.limit(50).offset(100)
```

Or with gems like **kaminari** / **will_paginate**:

```ruby
Order.page(params[:page]).per(25)
```

Filtering example:

```ruby
Order.where(status: params[:status])
```

Always sanitize parameters — never directly interpolate SQL.

---

## Idempotent Write Endpoints

When external clients or background jobs retry a request, you don’t want duplicates.

```ruby
order = Order.find_or_initialize_by(external_id: params[:idempotency_key])
order.update!(status: params[:status])
render json: order, status: :ok
```

Use unique identifiers or database constraints to make requests repeatable without double-writing.

---

## Handling Errors Gracefully

Use consistent JSON error shapes and HTTP codes:

```ruby
rescue_from ActiveRecord::RecordNotFound do |e|
  render json: { error: e.message }, status: :not_found
end

rescue_from ActiveRecord::RecordInvalid do |e|
  render json: { error: e.record.errors.full_messages }, status: :unprocessable_entity
end
```

You can define these once in `ApplicationController` to standardize across the app.

---

## Quick `curl` Testing Recipes

```bash
curl -i http://localhost:3000/users/1
curl -X POST http://localhost:3000/users \
  -H "Content-Type: application/json" \
  -d '{"user": {"email": "a@b.c", "name": "Alice"}}'
```

Combine with `jq` to pretty-print responses:

```bash
curl -s http://localhost:3000/orders | jq '.data[] | {id, status}'
```

---

## Performance and Security Considerations

| Concern                    | Mitigation                                     |
| -------------------------- | ---------------------------------------------- |
| **N+1 queries**            | Use `.includes` or `.preload` for associations |
| **Large payloads**         | Use `select` / `pluck`, paginate results       |
| **Sensitive data leakage** | Never render entire models directly            |
| **Slow serializations**    | Cache computed fields or partial JSON          |
| **Untrusted inputs**       | Always use strong parameters                   |

---

## Minimal, Safe JSON Controller Example

```ruby
class OrdersController < ApplicationController
  def index
    orders = Order.includes(:user).where(status: "active")
    render json: {
      data: orders.as_json(
        only: [:id, :status, :total_cents],
        include: { user: { only: [:id, :email] } }
      ),
      meta: { count: orders.count }
    }, status: :ok
  end

  def create
    order = Order.new(order_params)
    if order.save
      render json: order.as_json(only: [:id, :status]), status: :created
    else
      render json: { errors: order.errors.full_messages }, status: :unprocessable_entity
    end
  end

  private

  def order_params
    params.require(:order).permit(:status, :user_id, :total_cents)
  end
end
```

This structure:

* Uses strong params
* Returns predictable JSON
* Avoids internal column leaks
* Scales with minimal SQL

---

## Key Takeaways

* Explicitly control every field that leaves your system.
* Validate, sanitize, and permit everything that comes in.
* Keep endpoints idempotent and safe for retries.
* Paginate and preload for performance.
* Log what was written — especially in production.

Clean JSON isn’t just about readability; it’s about **trust** between your API and its consumers.

---
