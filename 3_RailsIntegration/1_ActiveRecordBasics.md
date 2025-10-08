# **ActiveRecordBasics.md**

## What ActiveRecord Is — and Why It Feels “Magical”

ActiveRecord is Rails’ **Object-Relational Mapper (ORM)**.
It’s the bridge that turns database rows into Ruby objects and vice versa.

By convention:

* Every table maps to a **class** (`users` → `User`).
* Every column becomes an **attribute**.
* `id` is assumed as the **primary key**.
* `created_at` and `updated_at` are tracked automatically.

You almost never write raw SQL. Instead, you build *relations* that ActiveRecord translates into SQL for you.
When you call `User.find(1)`, you’re not retrieving a hash—you’re instantiating a full Ruby object tied to a record in the DB.

---

## Querying 101 — Relations, Laziness, and Chaining

### Basic Finders

```ruby
User.find(1)                # by ID, raises if not found
User.find_by(email: "a@b.c") # returns nil if not found
User.where(active: true)     # returns an ActiveRecord::Relation
```

### Relations Are Lazy

`User.where(active: true)` builds a query but doesn’t hit the database until you *need* the data:

```ruby
users = User.where(active: true) # no SQL yet
users.count                      # now runs SELECT COUNT(*)
users.to_a                       # executes and returns an array
```

That laziness is what makes `.where(...).order(...).limit(...)` chains so powerful.

### Common Filters and Helpers

| Method                 | What it does                      |
| ---------------------- | --------------------------------- |
| `.where`               | Filter by conditions              |
| `.order(:created_at)`  | Sort                              |
| `.limit(10)`           | Restrict rows                     |
| `.pluck(:id)`          | Return plain Ruby array of values |
| `.select(:id, :email)` | Load only specific columns        |
| `.exists?`             | Quick boolean check               |
| `.count`               | Aggregate count                   |

Use `pluck` or `select` when you only need specific fields — they’re faster and consume less memory than loading full objects.

---

## Loading Columns vs. Loading Objects

`User.all` loads every column into full model objects.
If you only need a subset of data:

```ruby
User.pluck(:id, :email) # returns arrays like [1, "a@b.c"]
User.select(:id, :email) # returns lightweight model instances
```

**Rule of thumb:**
Use `pluck` for reporting or lightweight operations; use real objects only when you need callbacks, validations, or methods.

---

## Writing Safely — Updates, Transactions, Concurrency

### Update Methods (and When to Use Them)

| Method               | Runs validations? | Runs callbacks? | Notes                                                             |
| -------------------- | ----------------- | --------------- | ----------------------------------------------------------------- |
| `update` / `update!` | ✅                 | ✅               | Normal path (`!` raises on failure)                               |
| `update_columns`     | ❌                 | ❌               | Direct SQL; skips everything — **avoid** unless absolutely needed |
| `update_attribute`   | ⚠️                | ✅ (partial)     | Legacy — **avoid**                                                |

### Transactions

Group multiple changes atomically:

```ruby
ActiveRecord::Base.transaction do
  order.update!(status: "processing")
  Payment.create!(order: order)
  raise ActiveRecord::Rollback if something_bad_happened
end
```

If anything fails, the DB rolls back automatically.
Always wrap bulk or multi-model updates this way.

### Concurrency and Idempotency

Multiple jobs might update the same record at once.
Use `with_lock` to prevent race conditions:

```ruby
order.with_lock do
  order.update!(status: "processing")
end
```

For recurring jobs or retries, make writes **idempotent**:

* Use `find_or_create_by`
* Check state before changing (`return if order.completed?`)
* Use unique keys in background jobs

---

## Introspection and Reflection — Seeing What’s Inside

Useful when exploring an unfamiliar model in console:

```ruby
User.column_names
User.attribute_names
user = User.first
user.attributes
user.changed?            # boolean
user.saved_changes       # after save
user.previous_changes    # previous transaction
user.reload              # refresh from DB
```

To see what methods exist:

```ruby
user.methods.sort
user.public_methods(false)
user.respond_to?(:email)
```

These tricks help you understand what data and behaviors a model actually exposes.

---

## Production Safety Checklist

Before running any manual script or console fix:

1. **Verify environment:**

   ```ruby
   Rails.env.production? # => true or false
   ```

   If true, pause and double-check.

2. **Scope narrowly:**
   Avoid `User.update_all(...)` unless you *really* intend to hit all rows.

3. **Dry-run first:**

   ```ruby
   ActiveRecord::Base.transaction do
     users = User.where(active: false)
     users.update_all(active: true)
     raise ActiveRecord::Rollback
   end
   ```

   Confirm the SQL and number of affected rows before removing the rollback.

4. **Log context:**
   Leave breadcrumbs — who ran it, why, and what was changed.

---

## Common Patterns for Scale

### Batch Processing

```ruby
User.find_each(batch_size: 1000) do |user|
  user.process!
end
```

Keeps memory low, loading records in chunks instead of all at once.

### Controlled Updates Template

```ruby
def activate_pending_orders
  ActiveRecord::Base.transaction do
    orders = Order.where(status: "pending")
    puts "Updating #{orders.count} orders"
    orders.find_each do |o|
      o.update!(status: "active") if should_activate?(o)
    end
  end
end
```

Safe, idempotent, and auditable.

---

## Quick Reference

| Task           | Snippet                                         |
| -------------- | ----------------------------------------------- |
| List columns   | `Model.column_names`                            |
| Get attributes | `record.attributes.slice("id", "email")`        |
| Batch process  | `find_each(batch_size: 1000)`                   |
| Preview SQL    | `relation.to_sql`                               |
| Safe update    | Transaction + validations + rollback on failure |

---

**Key takeaway:**
ActiveRecord gives you incredible power — but with that power comes the ability to change production data instantly. Always **scope narrowly, wrap writes, and double-check before commit**.

---
