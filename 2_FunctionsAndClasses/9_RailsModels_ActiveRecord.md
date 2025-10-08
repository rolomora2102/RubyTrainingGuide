# Rails Models and ActiveRecord

Rails models represent **the data layer** of an application.
They map Ruby objects to database tables using **ActiveRecord**, allowing you to query, validate, and manage relationships declaratively.

For Support Engineers, understanding how models work is essential to trace issues, debug data behavior, and safely test or extend automation jobs.

---

## Why This Matters for SEs

When triaging or reviewing code, you’ll often interact with models to:

* Reproduce or debug production issues locally.
* Understand where associations and callbacks are triggered.
* Validate data integrity and constraints.
* Track how a single change might cascade through related records.

---

## Model Basics

A Rails model inherits from `ApplicationRecord`, which extends `ActiveRecord::Base`.

```ruby
class User < ApplicationRecord
end
```

This connects the class to a database table named `users`.

---

## Associations

Associations define relationships between tables.
They make it easy to navigate related data without writing SQL directly.

### `belongs_to`

Links a record to another single record.

```ruby
class Order < ApplicationRecord
  belongs_to :user
end
```

This expects a `user_id` column in the `orders` table.

You can access it directly:

```ruby
order.user
order.user.email
```

### `has_one`

Defines a one-to-one relationship.

```ruby
class User < ApplicationRecord
  has_one :profile
end
```

### `has_many`

Defines a one-to-many relationship.

```ruby
class User < ApplicationRecord
  has_many :orders
end
```

You can chain queries:

```ruby
user.orders.where(status: "pending")
```

---

## Dependent Options

You can control what happens to associated records when the parent is deleted.

```ruby
has_many :orders, dependent: :destroy
```

Other options include `:nullify` or `:restrict_with_error`.
Always confirm dependencies before deleting records in production.

---

## Validations

Validations ensure data consistency before saving to the database.

```ruby
class User < ApplicationRecord
  validates :email, presence: true, uniqueness: true
  validates :age, numericality: { greater_than: 0 }
end
```

If validations fail, the record won’t save:

```ruby
user.save # => false
user.errors.full_messages
```

### Common Validators

| Validator      | Example                                    | Description     |
| -------------- | ------------------------------------------ | --------------- |
| `presence`     | `validates :name, presence: true`          | Must be present |
| `uniqueness`   | `validates :email, uniqueness: true`       | Must be unique  |
| `length`       | `validates :bio, length: { maximum: 200 }` | Character limit |
| `numericality` | `validates :price, numericality: true`     | Must be numeric |
| `format`       | `validates :email, format: /@/`            | Matches regex   |

---

## Scopes

Scopes define reusable queries.

```ruby
class Order < ApplicationRecord
  scope :recent, -> { where("created_at > ?", 7.days.ago) }
  scope :pending, -> { where(status: "pending") }
end

Order.recent.pending
```

Scopes return `ActiveRecord::Relation` objects, which can be chained or further filtered.

---

## Callbacks

Callbacks are hooks that run at specific points in a model’s lifecycle.
They automate actions like logging, normalization, or background job triggers.

```ruby
class Order < ApplicationRecord
  before_validation :normalize_status
  after_commit :notify_user

  private

  def normalize_status
    self.status = status.downcase
  end

  def notify_user
    NotificationService.call(user, "Order processed")
  end
end
```

### Common Callback Types

| Callback            | Triggered                 | Example          |
| ------------------- | ------------------------- | ---------------- |
| `before_validation` | Before validation runs    | Normalization    |
| `after_validation`  | After validation passes   | Logging          |
| `before_save`       | Before saving             | Data adjustments |
| `after_commit`      | After transaction commits | Async actions    |

Use callbacks sparingly. Overuse can make debugging difficult when multiple side effects occur invisibly.

---

## Default Values and Enums

Rails allows you to define enumerated attributes for readability.

```ruby
class Ticket < ApplicationRecord
  enum status: { open: 0, in_progress: 1, closed: 2 }
end

ticket.open!
ticket.closed? # => true
```

Enums are stored as integers but accessed as descriptive symbols.

---

## Query Methods

ActiveRecord provides readable query methods:

```ruby
User.find(1)
User.where(active: true)
User.order(created_at: :desc)
User.find_by(email: "rolo@example.com")
```

These methods return objects, not raw SQL.
To see the SQL query, use `.to_sql`.

```ruby
User.where(active: true).to_sql
```

---

## Validations vs Database Constraints

Always remember that model validations live in Ruby, not the database.
They can be bypassed if someone writes directly to the database.
Critical integrity rules should also exist at the DB level (via constraints or triggers).

---

## Practical Example

```ruby
class User < ApplicationRecord
  has_many :orders, dependent: :destroy

  validates :email, presence: true, uniqueness: true
  validates :name, length: { minimum: 3 }

  scope :active, -> { where(active: true) }

  before_validation :normalize_email

  private

  def normalize_email
    self.email = email.downcase.strip
  end
end
```

Readable, safe, and consistent — this structure is easy to debug and extend.

---

## Key Takeaways

| Concept      | Description                   | Example                               |
| ------------ | ----------------------------- | ------------------------------------- |
| `belongs_to` | One record references another | `belongs_to :user`                    |
| `has_many`   | Parent with multiple children | `has_many :orders`                    |
| `validates`  | Data validation               | `validates :email, presence: true`    |
| `scope`      | Reusable queries              | `scope :recent, -> { where(...) }`    |
| `callback`   | Lifecycle hooks               | `before_save :normalize_data`         |
| `enum`       | Named states                  | `enum status: { open: 0, closed: 1 }` |

---

### SE Perspective

When reading Rails code:

* Identify associations first — they define how data connects.
* Check validations to understand business rules.
* Use scopes to reproduce or filter data in console sessions.
* Watch for callbacks that might modify or trigger actions automatically.
* Always confirm whether a behavior is model-level or database-level before making assumptions in triage.

---
