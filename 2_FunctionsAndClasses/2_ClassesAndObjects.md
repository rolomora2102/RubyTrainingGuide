# Classes and Objects in Ruby

Classes are the *core building blocks* of Ruby.
They define what an **object** is, how it behaves, and what data it holds.
In Ruby, almost *everything* is an object — numbers, strings, arrays, even classes themselves.

---

## Why Classes Matter for SEs

As an SE, classes give you **structure** and **control**:

* They let you organize scripts into safe, reusable components.
* You can separate logic into **methods**, add **guardrails** (like private visibility), and define clear entry points (like `.call` or `.perform`).
* In triage, reading a class helps you understand how data flows through the system — from initialization to output — and where failures might occur.

---

## Defining a Class

A class starts with the `class` keyword and ends with `end`.

```ruby
class TicketProcessor
end
```

This doesn’t do anything yet — but it defines a blueprint from which we can create *instances* (objects).

---

## The `initialize` Constructor

When you call `.new`, Ruby automatically looks for an `initialize` method.
It’s the constructor — it runs once when you create the object.

```ruby
class TicketProcessor
  def initialize(ticket_id, priority = "normal")
    @ticket_id = ticket_id       # instance variable
    @priority = priority
  end
end

processor = TicketProcessor.new(1234, "high")
```

The `@` prefix creates **instance variables**, unique to each object.

`initialize` can also use default or keyword arguments — ideal for safe scripting.

```ruby
def initialize(ticket_id:, priority: "normal")
  @ticket_id = ticket_id
  @priority = priority
end
```

---

## `self`: Knowing Who You Are

`self` refers to *the current object* — context matters.

Inside instance methods, `self` is the instance:

```ruby
def log_id
  puts "Processing ticket ##{@ticket_id}"
end
```

Inside a **class method**, `self` refers to the *class itself*:

```ruby
class TicketProcessor
  def self.supported_priorities
    %w[low normal high critical]
  end
end

TicketProcessor.supported_priorities
```

Use `self.` when defining class-level utilities, like `.from_json`, `.fetch_all`, or `.perform`.

---

## Instance vs Class Methods

* **Instance methods** act on *a single object* (`processor.process`).
* **Class methods** act on *the class itself* (`TicketProcessor.perform_all`).

```ruby
class TicketProcessor
  def process
    puts "Processing #{@ticket_id}"
  end

  def self.perform_all
    puts "Running all queued processors"
  end
end
```

---

## Visibility: Public, Private, Protected

You can control which methods are exposed.

```ruby
class TicketProcessor
  def process
    validate!
    puts "Ticket #{@ticket_id} processed"
  end

  private

  def validate!
    raise "Missing ticket_id" unless @ticket_id
  end
end
```

* `public`: accessible everywhere (default)
* `private`: only within the same object
* `protected`: similar to private, but allows access from other instances of the same class

Guardrails like this are important in SE scripts — they prevent unsafe calls and reduce the blast radius of mistakes.

---

## `freeze`: Locking an Object

Once you’ve set up your object, you can **freeze** it to make it immutable.

```ruby
config = { retries: 3, env: "prod" }.freeze
config[:env] = "dev"  # => RuntimeError: can't modify frozen Hash
```

This is common in initializers or configuration classes to ensure consistency.

---

## Putting It All Together

```ruby
class TicketProcessor
  def initialize(ticket_id:, priority: "normal")
    @ticket_id = ticket_id
    @priority = priority
    freeze
  end

  def call
    return unless valid_ticket?

    puts "Processing ticket ##{@ticket_id} with #{@priority} priority"
  end

  private

  def valid_ticket?
    @ticket_id.to_i > 0
  end
end

TicketProcessor.new(ticket_id: 123).call
```

Clear constructor
Safe defaults
Guard clauses (`return unless`)
Private logic
Immutable state

---

## Key Takeaways

| Concept      | Purpose                            | Quick Example           |
| ------------ | ---------------------------------- | ----------------------- |
| `initialize` | Sets up your object                | `def initialize(name)`  |
| `self`       | Refers to the current object/class | `def self.build`        |
| `@variable`  | Instance data                      | `@user_id = id`         |
| Visibility   | Guardrails                         | `private def validate!` |
| `freeze`     | Prevents changes                   | `config.freeze`         |

---

### SE Perspective

Use classes to:

* Wrap risky scripts in clear, testable logic (`MyJob.call` instead of long scripts).
* Use private methods and guard clauses to enforce safety.
* Rely on immutability (`freeze`) for predictable behavior during automation.
