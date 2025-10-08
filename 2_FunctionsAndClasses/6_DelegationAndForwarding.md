# Delegation and Forwarding in Ruby

Delegation is about sending work from one object to another.
Instead of duplicating logic or building deep inheritance hierarchies, delegation lets objects cooperate — each doing what it’s best at.

This approach keeps your code clean, testable, and easier to maintain, especially in large systems or tools with multiple components.

---

## Why This Matters for SEs

Support Engineers frequently work with service layers, helpers, and wrapper objects.
Delegation helps you:

* Understand why a method call resolves elsewhere.
* Track where data or behavior originates.
* Avoid changing core logic unnecessarily.
* Extend code safely without breaking its structure.

---

## Simple Delegation Pattern

You can delegate behavior manually using method calls:

```ruby
class UserPresenter
  def initialize(user)
    @user = user
  end

  def name
    @user.name
  end
end
```

This avoids inheritance while still exposing only what’s needed from the inner object.

---

## Using `Forwardable`

Ruby’s standard library includes `Forwardable`, which simplifies this pattern:

```ruby
require "forwardable"

class UserPresenter
  extend Forwardable

  def_delegators :@user, :name, :email

  def initialize(user)
    @user = user
  end
end
```

`def_delegators` automatically defines methods (`name`, `email`) that forward to `@user`.

This keeps your code DRY while preserving encapsulation.

---

## ActiveSupport’s `delegate`

In Rails, `delegate` from ActiveSupport provides a clean syntax for the same concept:

```ruby
class Order
  delegate :email, :name, to: :customer

  def initialize(customer)
    @customer = customer
  end
end
```

When you call `order.email`, Ruby internally forwards the call to `order.customer.email`.

Optional arguments make it even more expressive:

```ruby
delegate :address, to: :customer, prefix: true
```

Now `order.customer_address` calls `customer.address`.

---

## Why Delegation Beats Inheritance

Delegation favors **composition** — combining objects — over deep inheritance.
It makes relationships explicit and reduces tight coupling.

Inheritance example:

```ruby
class ApiLogger < Logger
end
```

If `Logger` changes internally, `ApiLogger` might break.

Delegation alternative:

```ruby
class ApiLogger
  def initialize(logger)
    @logger = logger
  end

  def log(message)
    @logger.info("[API] #{message}")
  end
end
```

This approach is explicit, predictable, and easier to test or replace.

---

## The `alias` and `alias_method` Keywords

`alias` creates a new name for an existing method.
It’s often used for backward compatibility or slight variations in naming.

```ruby
class Job
  def execute
    puts "Running job"
  end

  alias run execute
end

Job.new.run
```

`alias_method` does the same but is defined at runtime and can be used inside methods or loops.

```ruby
class Job
  alias_method :perform, :execute
end
```

Use these cautiously — they can make method tracing harder if overused.

---

## Method Forwarding (`...` Operator)

Ruby 3 introduced a modern, concise way to forward all arguments to another method using `...`:

```ruby
def wrapper(...)
  original_method(...)
end
```

This forwards positional, keyword, and block arguments seamlessly.
It’s perfect for writing wrapper methods or decorators without re-declaring parameters.

---

## Composition in Practice

Example of combining delegation, forwarding, and composition in a simple service:

```ruby
class Notifier
  def initialize(sender)
    @sender = sender
  end

  def notify(...)
    @sender.deliver(...)
  end
end

class EmailSender
  def deliver(to:, message:)
    puts "Email sent to #{to}: #{message}"
  end
end

sender = EmailSender.new
notifier = Notifier.new(sender)
notifier.notify(to: "user@example.com", message: "Backup completed")
```

Here, `Notifier` doesn’t inherit from `EmailSender`.
It simply delegates the work — clearer, safer, and easier to replace with another sender later.

---

## Practical Example with ActiveSupport

```ruby
class Ticket
  attr_reader :customer

  delegate :email, :name, to: :customer, prefix: true

  def initialize(customer)
    @customer = customer
  end

  def notify
    puts "Notifying #{customer_name} at #{customer_email}"
  end
end
```

Delegation allows this object to stay focused on its purpose while reusing the customer’s existing logic.

---

## Key Takeaways

| Concept            | Description                   | Example                          |
| ------------------ | ----------------------------- | -------------------------------- |
| Manual delegation  | Forward calls manually        | `@user.name`                     |
| `Forwardable`      | Standard Ruby delegation      | `def_delegators :@user, :name`   |
| `delegate`         | Rails shortcut for delegation | `delegate :email, to: :customer` |
| `alias`            | Create method alias           | `alias run execute`              |
| Forwarding (`...`) | Pass all args                 | `def call(...); run(...); end`   |
| Composition        | Combine objects safely        | `Notifier.new(sender)`           |

---

### SE Perspective

When analyzing or modifying code:

* Check for `delegate` or `def_delegators` to see where behavior originates.
* Use delegation to expose only what’s needed, not the full object.
* Prefer composition over inheritance — it keeps systems modular.
* Use `alias` carefully, and document its purpose.
* Apply the forwarding operator (`...`) for modern, concise wrappers.

---
