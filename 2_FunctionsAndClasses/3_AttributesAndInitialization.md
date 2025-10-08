# Attributes and Initialization in Ruby

Attributes define how data is stored and accessed within a class.
Initialization sets up that data safely and predictably.
Together, they form the foundation of well-structured Ruby objects.

---

## Why This Matters for SEs

In support engineering, it’s common to inspect or extend Ruby objects during triage.
Understanding how instance variables, accessors, and initialization behave helps you:

* Trace where data is coming from.
* Avoid breaking encapsulation.
* Safely modify or mock objects for testing or automation.
* Keep scripts deterministic and maintainable.

---

## Instance Variables

An instance variable belongs to a single object.
It’s defined with an `@` and initialized inside the constructor.

```ruby
class User
  def initialize(name)
    @name = name
  end
end
```

Each instance has its own copy of `@name` — changing one doesn’t affect another.

---

## Attribute Accessors

Ruby provides shortcuts for reading and writing these variables.

| Method          | Purpose      | Example               |
| --------------- | ------------ | --------------------- |
| `attr_reader`   | Read-only    | `attr_reader :name`   |
| `attr_writer`   | Write-only   | `attr_writer :name`   |
| `attr_accessor` | Read + Write | `attr_accessor :name` |

Example:

```ruby
class User
  attr_accessor :name
end

user = User.new
user.name = "Carla"
puts user.name # "Carla"
```

This replaces the need for manual getter/setter methods and keeps code minimal.

---

## Default Values and Keyword Initialization

You can set defaults directly in `initialize`:

```ruby
class Config
  attr_reader :env, :timeout

  def initialize(env: "prod", timeout: 30)
    @env = env
    @timeout = timeout
  end
end
```

Keyword arguments make it explicit which settings are expected — safer for scripts that depend on structured input.

---

## Memoization

Memoization means caching a computed value so it’s only calculated once.
It’s common when fetching data or performing heavy operations.

```ruby
def connection
  @connection ||= Database.connect
end
```

If `@connection` is already set, Ruby skips the call to `Database.connect`.
This pattern improves performance and ensures consistent state across a single object’s lifecycle.

---

## Immutability with `freeze`

Once initialized, some objects should never change.
`freeze` locks the object, raising an error if you attempt to modify it.

```ruby
class Settings
  attr_reader :data

  def initialize(data)
    @data = data.freeze
  end
end
```

This prevents accidental mutations during runtime, especially useful for configuration objects or constants.

---

## Using `self` During Initialization

Sometimes you’ll see `self` used in initialization to call other setters or ensure internal consistency.

```ruby
class Order
  attr_accessor :status

  def initialize(status:)
    self.status = status.strip.downcase
  end
end
```

This ensures the initialization uses the same logic as manual assignments.

---

## The `pattr_initialize` Helper

In some codebases, you’ll find helpers like `pattr_initialize` or `dry-initializer`.
These come from external gems (such as ActiveSupport or dry-rb) and simplify initialization.

```ruby
class TicketProcessor
  pattr_initialize :ticket_id, :priority
end
```

This automatically defines an `initialize` method that assigns `@ticket_id` and `@priority`.
It’s not part of Ruby itself — if you don’t see it working, the gem may not be loaded.
In general, use explicit `initialize` unless you’re in a Rails app where this helper is common.

---

## Freezing and Constants Together

Constants are uppercase by convention and should not change.
Freezing them reinforces that guarantee.

```ruby
class Config
  DEFAULTS = { retries: 3, timeout: 30 }.freeze
end
```

When triaging Rails apps, constants like these often live in initializers or module definitions.

---

## Practical Example

```ruby
class Report
  attr_reader :title, :author, :data

  def initialize(title:, author:, data: [])
    @title  = title
    @author = author
    @data   = data.freeze
  end

  def summary
    "#{title} by #{author} — #{data.count} entries"
  end
end

r = Report.new(title: "Billing Summary", author: "Rolando", data: [1, 2, 3])
puts r.summary
```

This example combines safe initialization, keyword defaults, and immutability — a reliable pattern for automation and internal tooling.

---

## Key Takeaways

| Concept            | Description           | Example                 |   |            |
| ------------------ | --------------------- | ----------------------- | - | ---------- |
| `attr_reader`      | Getter only           | `attr_reader :name`     |   |            |
| `attr_accessor`    | Getter + Setter       | `attr_accessor :status` |   |            |
| Default args       | Fallback values       | `timeout: 30`           |   |            |
| Memoization        | Cache result          | `@conn                  |   | = connect` |
| Freeze             | Prevent modification  | `data.freeze`           |   |            |
| `pattr_initialize` | Gem shortcut for init | `pattr_initialize :id`  |   |            |

---

### SE Perspective

When building or reading scripts:

* Always use keyword initialization for clarity and safety.
* Prefer `attr_reader` unless mutation is required.
* Use memoization carefully — only for values that truly don’t change.
* Freeze data when consistency matters.
* Avoid depending on non-core helpers unless you confirm the gem is loaded in the environment.

---
