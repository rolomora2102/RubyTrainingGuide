# Rails Model DSL Cheat Sheet

Quick reference for **ActiveRecord** macros and patterns used in Rails models.
Essential for Support Engineers who need to understand model behavior, data relationships, and lifecycle events at a glance.

---

## Core Association Macros

| Macro                     | Purpose                                                   | Example                                     |
| ------------------------- | --------------------------------------------------------- | ------------------------------------------- |
| `belongs_to`              | One record references another (foreign key on this model) | `belongs_to :user`                          |
| `has_one`                 | One-to-one relationship                                   | `has_one :profile`                          |
| `has_many`                | One-to-many relationship                                  | `has_many :orders`                          |
| `has_many :through`       | Many-to-many via join model                               | `has_many :products, through: :order_items` |
| `has_and_belongs_to_many` | Direct many-to-many (legacy)                              | `has_and_belongs_to_many :tags`             |
| `dependent:`              | Define behavior on delete                                 | `has_many :orders, dependent: :destroy`     |

---

## Association Options

| Option         | Description                    | Example                                     |
| -------------- | ------------------------------ | ------------------------------------------- |
| `class_name:`  | Custom target model name       | `belongs_to :author, class_name: "User"`    |
| `foreign_key:` | Custom key column              | `belongs_to :owner, foreign_key: "user_id"` |
| `inverse_of:`  | Optimize bi-directional access | `has_many :orders, inverse_of: :user`       |
| `optional:`    | Skip presence validation       | `belongs_to :user, optional: true`          |

---

## Validations

| Macro                       | Description        | Example                                            |
| --------------------------- | ------------------ | -------------------------------------------------- |
| `validates`                 | Generic validation | `validates :email, presence: true`                 |
| `validates_presence_of`     | Must be present    | `validates_presence_of :name`                      |
| `validates_uniqueness_of`   | Must be unique     | `validates_uniqueness_of :email`                   |
| `validates_length_of`       | Character limit    | `validates_length_of :bio, maximum: 160`           |
| `validates_numericality_of` | Numeric only       | `validates_numericality_of :price`                 |
| `validates_format_of`       | Match regex        | `validates_format_of :email, with: /@/`            |
| `validates_inclusion_of`    | Must be in set     | `validates_inclusion_of :role, in: %w[user admin]` |

Rails 6+ also supports **`validates`** with multiple options in one call:

```ruby
validates :email, presence: true, uniqueness: true, format: /@/
```

---

## Scopes

Scopes define reusable, chainable queries.

| Macro         | Description      | Example                                                     |
| ------------- | ---------------- | ----------------------------------------------------------- |
| `scope`       | Custom query     | `scope :recent, -> { where("created_at > ?", 7.days.ago) }` |
| Chain scopes  | Combine filters  | `User.active.recent`                                        |
| Dynamic scope | Accept arguments | `scope :by_status, ->(s) { where(status: s) }`              |

---

## Enums

Maps symbols to integers for readable state machines.

| Macro             | Description         | Example                               |
| ----------------- | ------------------- | ------------------------------------- |
| `enum`            | Define named states | `enum status: { open: 0, closed: 1 }` |
| Methods generated | Predicate + bang    | `ticket.closed?`, `ticket.open!`      |

---

## Callbacks

| Callback            | Trigger                       | Common Use              |
| ------------------- | ----------------------------- | ----------------------- |
| `before_validation` | Before validation runs        | Normalize data          |
| `after_validation`  | After validation passes       | Logging                 |
| `before_save`       | Before saving (create/update) | Format data             |
| `after_save`        | After saving                  | Audit or sync           |
| `before_create`     | Before first save only        | Generate defaults       |
| `after_commit`      | After DB transaction          | Trigger jobs            |
| `around_*`          | Wraps operation               | Timing, instrumentation |

**Example:**

```ruby
before_save :normalize_email

def normalize_email
  self.email = email.downcase.strip
end
```

---

## Class Methods

| Macro                | Description           | Example                          |
| -------------------- | --------------------- | -------------------------------- |
| `.find`              | By primary key        | `User.find(1)`                   |
| `.find_by`           | By attribute          | `User.find_by(email: "a@b.com")` |
| `.where`             | Filter query          | `Order.where(status: "open")`    |
| `.order`             | Sort results          | `User.order(created_at: :desc)`  |
| `.limit` / `.offset` | Pagination            | `User.limit(10).offset(20)`      |
| `.pluck`             | Extract column values | `User.pluck(:email)`             |

---

## Transactions

Ensure atomic database operations:

```ruby
ActiveRecord::Base.transaction do
  order.save!
  payment.process!
end
```

If any step raises an exception, everything rolls back.

---

## Touch and Timestamps

| Feature      | Description              | Example                            |
| ------------ | ------------------------ | ---------------------------------- |
| `touch:`     | Updates parent timestamp | `belongs_to :project, touch: true` |
| `updated_at` | Auto-managed field       | `record.touch` manually updates it |

---

## Default Scopes

Applies filters automatically to all queries.

```ruby
default_scope { where(active: true) }
```

Use carefully — they persist across all model queries and can cause confusion during debugging.

---

## Common Shortcuts

| Action          | Code                  |
| --------------- | --------------------- |
| Create record   | `User.create!(attrs)` |
| Update record   | `user.update!(attrs)` |
| Delete record   | `user.destroy`        |
| Reload data     | `user.reload`         |
| Existence check | `User.exists?(id)`    |
| Count records   | `Order.count`         |

---

## Debugging Tips for SEs

* Use `.to_sql` to inspect what a scope or query actually runs.
* Check **callbacks** if unexpected actions happen on save or destroy.
* Review **validations** when a record silently fails to save.
* Inspect `association.loaded?` to see if data was lazy-loaded.
* Use `.unscope` to remove `default_scope` for debugging:

  ```ruby
  User.unscoped.where(admin: true)
  ```

---

## Key Takeaways

| Concept            | Purpose                | Example                            |
| ------------------ | ---------------------- | ---------------------------------- |
| Association macros | Define relationships   | `has_many :orders`                 |
| Validation macros  | Enforce data integrity | `validates :email, presence: true` |
| Scopes             | Reusable filters       | `scope :recent, -> { ... }`        |
| Callbacks          | Lifecycle hooks        | `before_save :normalize`           |
| Enums              | Named states           | `enum status: { open: 0 }`         |
| Transactions       | Atomic ops             | `ActiveRecord::Base.transaction`   |

---
