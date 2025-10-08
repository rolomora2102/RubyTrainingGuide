# Methods in Ruby

Methods are the foundation of Ruby’s expressiveness.  
They define how a class or script performs specific actions, handle data, and maintain readability.  
Understanding how to define, call, and control methods is what makes Ruby powerful for both automation and debugging.

---

## Why Methods Matter for SEs

In SE workflows, methods let you:

- Encapsulate logic in small, testable units.  
- Make scripts safer and easier to reason about.  
- Reuse behavior without repeating code.  
- Debug failures faster by isolating functionality.

When you’re reading Ruby or Rails code, identifying the method flow often reveals why something is failing.

---

## Defining a Method

A simple method looks like this:

```ruby
def greet
  "Hello"
end
```

When called, Ruby returns the value of the last evaluated expression automatically:

```ruby
greet # => "Hello"
```

You can also use `return` explicitly, but it’s usually not required unless you want to exit early.

---

## Arguments and Defaults

Methods can accept parameters to make them dynamic.

```ruby
def greet(name)
  "Hello, #{name}"
end
```

You can provide default values:

```ruby
def greet(name = "there")
  "Hello, #{name}"
end
```

---

## Keyword Arguments

Keyword arguments make your code more readable and safer, especially in large systems.

```ruby
def log_event(type:, user:, critical: false)
  puts "[#{type.upcase}] Event for #{user} (critical: #{critical})"
end

log_event(type: "update", user: "Alice")
```

If a keyword argument is required, Ruby will raise an error if it’s missing — this is a good practice for scripts that must receive specific data.

---

## Variable Arguments (`*args`, `**kwargs`)

Use these when you don’t know in advance how many arguments will be passed.

```ruby
def report(*lines)
  lines.each { |line| puts line }
end
```

`**kwargs` collects keyword arguments into a hash.

```ruby
def configure(**options)
  puts options.inspect
end

configure(retries: 3, timeout: 10)
```

This pattern is common in Rails helpers and ActiveSupport utilities.

---

## Return Behavior

Ruby automatically returns the last evaluated value.

```ruby
def add(a, b)
  a + b
end
```

You can still use `return` to exit early or make intent explicit.

```ruby
def find_user(id)
  return nil unless id
  User.find(id)
end
```

`return` is especially useful when combined with **guard clauses**, helping keep code clean and predictable.

---

## Methods and Control Flow

You can use `if`, `unless`, and early returns inside methods to make them safer.

```ruby
def process(order)
  return unless order.valid?

  order.submit
  "Processed"
end
```

Guard clauses like `return unless condition` are widely used across Amazon’s internal scripts to prevent unnecessary actions or unexpected crashes.

---

## Blocks and Yield

Methods can receive blocks of code using `yield`.

```ruby
def with_logging
  puts "Start"
  yield if block_given?
  puts "End"
end

with_logging { puts "Inside block" }
```

This is a common Ruby pattern for wrapping actions with before/after logic — similar to how Rails handles filters or transactions.

---

## Method Chaining

Since every method returns a value, you can chain them.

```ruby
result = "   hello  ".strip.upcase.reverse
```

When designing methods, returning `self` allows chaining on custom classes:

```ruby
class Builder
  def set_name(name)
    @name = name
    self
  end
end

Builder.new.set_name("App")
```

---

## Private and Protected Methods

As covered in the previous file, visibility defines who can call a method.

Private methods are not accessible from outside the class:

```ruby
class User
  def full_name
    "#{first_name} #{last_name}"
  end

  private

  def first_name
    "Jane"
  end

  def last_name
    "Doe"
  end
end
```

Protected methods can be called by other instances of the same class, though this is less common.

---

## Keyword Rest Parameters and Forwarding

In modern Ruby, you can easily forward all arguments to another method:

```ruby
def wrapper(...)
  inner_call(...)
end
```

The `...` syntax captures both positional and keyword arguments, a clean way to delegate calls without redefining the signature.

---

## Practical Example

```ruby
class OrderValidator
  def initialize(order)
    @order = order
  end

  def call
    return error("Invalid order") unless valid_order?
    return error("Missing customer") unless @order.customer

    puts "Order #{@order.id} is valid"
    true
  end

  private

  def valid_order?
    @order.respond_to?(:id)
  end

  def error(message)
    puts "[Error] #{message}"
    false
  end
end
```

This pattern — a small, focused method with clear return values and guard clauses — is reliable, testable, and easy to debug.

---

## Key Takeaways

| Concept | Description | Example |
|----------|--------------|----------|
| Method definition | Defines reusable logic | `def process; end` |
| Default arguments | Provides fallbacks | `def call(id = nil)` |
| Keyword args | Safer, more readable | `def run(user:, env:)` |
| Return values | Last line or explicit | `return if invalid` |
| Blocks & yield | Execute external logic | `yield if block_given?` |
| Forwarding (`...`) | Pass everything along | `def wrapper(...); call(...); end` |

---

## SE Perspective

When writing or reading Ruby:

- Keep methods small and predictable.  
- Use keyword arguments for clarity.  
- Prefer early returns over nested `if` statements.  
- Wrap reusable logic into clear methods rather than inline scripts.  
- Always think of safety: what happens if the method receives unexpected input?
