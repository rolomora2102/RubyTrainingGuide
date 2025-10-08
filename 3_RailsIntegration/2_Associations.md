# **Associations.md**

## Why Associations Matter

ActiveRecord associations let you connect models the same way tables relate in SQL — but with friendlier syntax.

They’re what make `user.orders` or `order.user` just *work* without writing a single join.
They turn foreign keys into relationships you can traverse, query, and preload efficiently.

---

## Core Macros — The Relationship Builders

| Macro               | Description                                               | Example                                    |
| ------------------- | --------------------------------------------------------- | ------------------------------------------ |
| `belongs_to`        | Adds a single reference (foreign key lives on this model) | `belongs_to :user`                         |
| `has_one`           | One-to-one; foreign key lives on the other model          | `has_one :profile`                         |
| `has_many`          | One-to-many                                               | `has_many :orders`                         |
| `has_many :through` | Many-to-many through a join model                         | `has_many :products, through: :line_items` |

You’ll also see modifiers like:

```ruby
has_many :orders, dependent: :destroy
belongs_to :account, optional: true
has_one :invoice, inverse_of: :order, touch: true
```

Each option changes behavior during deletes, validations, and callbacks.

---

## Traversing Associations

You can chain associations to navigate data naturally:

```ruby
user.orders.pending
order.user.email
company.employees.where(role: "manager")
```

### Querying Through Associations

You can join or eager load related records to avoid repeated queries:

```ruby
Order.joins(:user).where(users: { active: true })
Order.includes(:user).each { |o| puts o.user.email }
```

| Method       | What it does                                         | When to use                                 |
| ------------ | ---------------------------------------------------- | ------------------------------------------- |
| `joins`      | INNER JOIN — filters by related table, no preloading | Filtering or aggregating                    |
| `includes`   | Loads related objects in separate queries            | Avoid N+1 queries                           |
| `preload`    | Forces separate SELECTs only                         | Safer in read-heavy endpoints               |
| `eager_load` | Forces one big LEFT OUTER JOIN                       | When you need conditions on the association |

---

## Discovering Relationships Dynamically (Reflection APIs)

When exploring an unknown model, you can ask Rails what associations it has:

```ruby
User.reflect_on_all_associations.map(&:name)
# => [:orders, :address, :profile]

User.reflect_on_all_associations(:has_many).map(&:name)
# => [:orders]

assoc = User.reflect_on_association(:orders)
assoc.macro         # => :has_many
assoc.class_name    # => "Order"
assoc.foreign_key   # => "user_id"
assoc.options       # => { dependent: :destroy }
```

This helps in debugging or writing scripts that adapt to changing schema.

---

## Practical Discovery Workflow (Rails Console)

When you’re not sure how two models relate:

1. **Start with reflection**

   ```ruby
   Model.reflect_on_all_associations.map(&:name)
   ```
2. **Pick one and inspect**

   ```ruby
   Model.reflect_on_association(:foo).macro
   ```
3. **Check the schema or migration**

   ```ruby
   Model.column_names
   ```
4. **Verify with data**

   ```ruby
   Model.first.foo
   ```

---

## Understanding `dependent:` Options

| Option                 | What it does                             | Use with caution                    |
| ---------------------- | ---------------------------------------- | ----------------------------------- |
| `:destroy`             | Deletes associated records via callbacks | Triggers validations; slow but safe |
| `:delete_all`          | Direct SQL delete, skips callbacks       | Dangerous; no `before_destroy`      |
| `:nullify`             | Sets foreign keys to NULL                | Good for optional references        |
| `:restrict_with_error` | Prevents delete if children exist        | Safe in prod environments           |

In production scripts, **never** mass-delete without understanding the `dependent:` cascade.

---

## Eliminating N+1 Queries

The “N+1” problem happens when you fetch N records and each triggers its own query for associations.

```ruby
# N+1 problem
User.all.each { |u| puts u.profile.name }

# fixed
User.includes(:profile).each { |u| puts u.profile.name }
```

Rails log tip:

```
User Load (0.5ms)
Profile Load (0.4ms)  SELECT ... WHERE "profile"."user_id" = ?
```

If you see the same SQL repeating — preload.

---

## Example: Traversing + Preloading Safely

```ruby
# show all users with their last order date
users = User.includes(:orders)
users.each do |u|
  puts "#{u.name}: #{u.orders.last&.created_at}"
end
```

### Generated SQL

```sql
SELECT * FROM users;
SELECT * FROM orders WHERE user_id IN (...);
```

Efficient and predictable.

---

## Side-by-Side: `joins` vs `includes`

| Code                                                       | SQL Behavior             | Result                                                  |
| ---------------------------------------------------------- | ------------------------ | ------------------------------------------------------- |
| `User.joins(:orders).where(orders: { status: "paid" })`    | Single INNER JOIN        | Returns only users with matching orders                 |
| `User.includes(:orders).where(orders: { status: "paid" })` | LEFT JOIN via eager load | Returns users, possibly duplicating rows if not careful |

Rule of thumb:
Use `joins` for filters, `includes` for eager loading, and combine cautiously.

---

## Quick Reflection Cheats

```ruby
# list association names and macros
User.reflect_on_all_associations.map { |a| [a.name, a.macro] }.to_h
# => { orders: :has_many, profile: :has_one }

# foreign keys
User.reflect_on_all_associations.map { |a| [a.name, a.foreign_key] }
# => [[:orders, "user_id"], [:profile, "user_id"]]
```

These one-liners are gold when you’re debugging a production issue and need to know “what’s linked to what.”

---

## Production Safety Reminders

* Don’t destroy cascades blindly; confirm `dependent:` behavior first.
* Avoid modifying large associations directly in console (`user.orders.destroy_all`).
* Scope narrowly and verify `.count` before `.update_all` or `.delete_all`.
* When exploring, prefer **read-only** console sessions or wrap with `transaction { raise Rollback }`.

---

## Summary

Associations are what make Rails models *feel alive*.
They let you:

* Traverse relationships naturally (`user.orders`)
* Query efficiently (`includes`, `joins`)
* Inspect structure dynamically (`reflect_on_all_associations`)

The more you understand them, the faster you can debug and the safer you’ll be when touching production data.

---
