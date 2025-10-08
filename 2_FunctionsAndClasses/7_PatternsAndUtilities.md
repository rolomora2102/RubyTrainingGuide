# Patterns and Utilities in Ruby and Rails

Ruby’s flexibility allows developers to create lightweight, expressive patterns without complex frameworks.
In large systems, patterns like **Service Objects**, **Form Objects**, and **Policy Objects** provide structure, clarity, and reusability — all crucial for safe and maintainable code.

---

## Why This Matters for SEs

Support Engineers benefit from recognizing common Ruby patterns because they:

* Help you understand **how logic is separated** across files.
* Clarify **where to look** when tracing code execution.
* Prevent confusion between controller, model, and service responsibilities.
* Simplify debugging — you can quickly locate which class “owns” a piece of behavior.

---

## 1. Service Objects

Service objects wrap a **single, well-defined operation** into its own class.
They help move business logic out of controllers and models.

**Structure**

```ruby
class SendInvoice
  def self.call(order_id)
    new(order_id).call
  end

  def initialize(order_id)
    @order = Order.find(order_id)
  end

  def call
    return failure("Order already invoiced") if @order.invoiced?

    Invoice.create!(order: @order)
    success("Invoice created successfully")
  rescue => e
    failure(e.message)
  end

  private

  def success(message)
    { ok: true, message: message }
  end

  def failure(message)
    { ok: false, message: message }
  end
end
```

**Why it works**

* Explicit entry point (`.call`).
* Simple testing and reuse.
* No dependency on Rails internals — a plain Ruby object (PORO).

In Amazon-style systems, this pattern improves **traceability** and makes automation safer to reason about.

---

## 2. Form Objects

Form objects handle complex input that updates **multiple models** at once.
They centralize validation and submission logic.

```ruby
class RegistrationForm
  include ActiveModel::Model

  attr_accessor :user_name, :email, :plan_id

  validates :email, presence: true
  validates :plan_id, presence: true

  def save
    return false unless valid?
    User.create!(name: user_name, email: email, plan_id: plan_id)
  end
end
```

**Why it matters**

Form objects keep controllers clean and validation rules consistent across multiple models.
They’re commonly used in multi-step forms, signup flows, and admin dashboards.

---

## 3. Policy Objects

Policy objects define authorization rules — who can perform what action.
They separate access control logic from business logic.

```ruby
class OrderPolicy
  attr_reader :user, :order

  def initialize(user, order)
    @user = user
    @order = order
  end

  def view?
    user.admin? || order.user_id == user.id
  end

  def cancel?
    user.admin? || (order.user_id == user.id && order.pending?)
  end
end
```

This pattern is often combined with libraries like **Pundit**, but can exist standalone in internal tools.

---

## 4. Query Objects

Query objects encapsulate complex database logic in dedicated classes.
They prevent models from becoming overloaded with query methods.

```ruby
class RecentOrdersQuery
  def initialize(relation = Order.all)
    @relation = relation
  end

  def call
    @relation.where("created_at > ?", 7.days.ago)
  end
end
```

This pattern improves testability and reusability of data retrieval logic.

---

## 5. Decorators / Presenters

Decorators (or presenters) wrap objects to modify or format output, especially in views or APIs.

```ruby
class UserPresenter
  def initialize(user)
    @user = user
  end

  def display_name
    "#{@user.name} (#{@user.email})"
  end
end
```

This avoids cluttering models with presentation logic.

---

## 6. Command Pattern (Action Classes)

Commands are similar to service objects but emphasize **intent** and **side effects** — one command per action.

```ruby
class CloseTicketCommand
  def initialize(ticket)
    @ticket = ticket
  end

  def execute
    @ticket.update!(status: "closed")
    LogEvent.call("Ticket closed", @ticket.id)
  end
end
```

Commands are ideal for operations that trigger multiple steps or events, such as closing tickets, billing users, or syncing data.

---

## 7. Value Objects

Value objects represent simple, immutable concepts with equality based on value, not identity.

```ruby
class Money
  attr_reader :amount, :currency

  def initialize(amount, currency)
    @amount = amount
    @currency = currency
    freeze
  end

  def ==(other)
    amount == other.amount && currency == other.currency
  end
end
```

They improve correctness and prevent subtle bugs caused by mutating shared data.

---

## 8. Utility Modules

Utility modules collect stateless, reusable methods that can be included where needed.

```ruby
module TimeHelper
  def self.pretty_duration(seconds)
    "#{seconds / 60}m #{seconds % 60}s"
  end
end

TimeHelper.pretty_duration(125)
# => "2m 5s"
```

Utilities are most useful for formatting, parsing, or cross-cutting helpers.

---

## Putting It All Together

Example of how these patterns interact in a clean architecture:

```ruby
class UserRegistration
  def self.call(params)
    form = RegistrationForm.new(params)
    return { ok: false, errors: form.errors.full_messages } unless form.valid?

    user = form.save
    SendWelcomeEmail.call(user.id)
    { ok: true, user: user }
  end
end
```

Each component handles one responsibility:

* `RegistrationForm` validates input.
* `UserRegistration` orchestrates flow.
* `SendWelcomeEmail` handles the side effect.

This separation makes triage, testing, and rollback significantly easier.

---

## Key Takeaways

| Pattern        | Purpose                   | Example                                  |
| -------------- | ------------------------- | ---------------------------------------- |
| Service Object | Encapsulate one operation | `MyService.call(params)`                 |
| Form Object    | Manage complex input      | `RegistrationForm.new(params)`           |
| Policy Object  | Handle permissions        | `OrderPolicy.new(user, order)`           |
| Query Object   | Encapsulate DB logic      | `RecentOrdersQuery.call`                 |
| Decorator      | Format output             | `UserPresenter.new(user)`                |
| Command        | Explicit action           | `CloseTicketCommand.new(ticket).execute` |
| Value Object   | Immutable concept         | `Money.new(10, "USD")`                   |
| Utility Module | Reusable helpers          | `TimeHelper.pretty_duration`             |

---

### SE Perspective

When reading or debugging Ruby code:

* Identify the **pattern type** first — it reveals the file’s role.
* Service, Command, and Query objects often sit between controllers and models.
* Policy or Form objects usually handle authorization or validation.
* Value objects and utilities improve reliability and test coverage.
* Recognizing these patterns speeds up incident resolution and safe code changes.

---
