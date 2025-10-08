# Modules and Mixins in Ruby

Modules are Ruby’s way of grouping related methods, constants, and logic without creating a class hierarchy.
They’re reusable, lightweight, and essential for organizing behavior across multiple classes.

---

## Why This Matters for SEs

For Support Engineers, understanding modules helps you:

* Identify **where behavior comes from** in complex Ruby or Rails systems.
* Understand **namespacing** and avoid confusion when multiple classes share similar names.
* Recognize when code is using **mixins** (`include`, `extend`, `prepend`) instead of inheritance.
* Debug method lookup and conflicts more confidently.

---

## Defining a Module

A module looks similar to a class, but it cannot be instantiated.

```ruby
module LoggerHelper
  def log_info(message)
    puts "[INFO] #{message}"
  end
end
```

Modules are ideal for shared functionality that doesn’t belong to a single class.

---

## Including a Module

`include` mixes a module’s methods as **instance methods** into a class.

```ruby
class Processor
  include LoggerHelper
end

Processor.new.log_info("Running job")
```

This pattern is common in Rails — for example, `ActiveModel::Validations` adds validation methods into model classes.

---

## Extending a Module

`extend` adds module methods as **class methods**.

```ruby
module Audit
  def log_class_action
    puts "Action logged at class level"
  end
end

class Report
  extend Audit
end

Report.log_class_action
```

Use `extend` when the functionality applies to the class itself, not its instances.

---

## Prepending a Module

`prepend` is similar to `include`, but it changes the **method lookup order**.
Methods in the prepended module take priority over those defined in the class.

```ruby
module SafeRun
  def perform
    puts "Running safely"
    super
  end
end

class Job
  prepend SafeRun

  def perform
    puts "Executing job"
  end
end

Job.new.perform
# Output:
# Running safely
# Executing job
```

This is a common Ruby metaprogramming trick — it allows intercepting method calls without editing the original class directly.

---

## Namespacing with Modules

Modules are also used as namespaces to organize classes and prevent naming conflicts.

```ruby
module Billing
  class Invoice
    def process
      puts "Processing invoice"
    end
  end
end

Billing::Invoice.new.process
```

Here, `Billing::Invoice` clearly belongs to the `Billing` context, avoiding collisions with any other `Invoice` class.

---

## The `::` Operator

The double colon (`::`) is the **scope resolution operator**.
It’s used to access constants or nested classes/modules.

```ruby
Math::PI     # => 3.14159
Billing::Invoice.new
```

You’ll often see it in Rails for referencing modules like `ActiveRecord::Base` or `ActionController::API`.

---

## Constants Inside Modules

Modules can define constants that are shared by all classes that include them.

```ruby
module Config
  VERSION = "1.2.0"
  DEFAULT_TIMEOUT = 30
end

puts Config::VERSION
```

Constants defined in a module follow Ruby’s standard constant resolution rules — if a class includes the module, it can access them directly.

---

## Combining Multiple Mixins

Ruby allows mixing several modules into one class.
This supports **composition over inheritance** — combining small, focused behaviors.

```ruby
module Logging
  def log(msg) = puts "[LOG] #{msg}"
end

module Retryable
  def retry_job
    puts "Retrying job..."
  end
end

class Worker
  include Logging
  include Retryable
end

Worker.new.log("Starting")
Worker.new.retry_job
```

Each module brings its own capabilities without creating deep inheritance chains.

---

## Practical Example

```ruby
module Loggable
  def log(message)
    puts "[#{Time.now}] #{message}"
  end
end

module Retryable
  def with_retries(limit: 3)
    attempts = 0
    begin
      yield
    rescue StandardError => e
      attempts += 1
      retry if attempts < limit
      log("Failed after #{limit} attempts: #{e.message}") if respond_to?(:log)
    end
  end
end

class Job
  include Loggable
  include Retryable

  def perform
    with_retries do
      log("Performing job...")
      raise "Simulated error" if rand < 0.3
      log("Job completed")
    end
  end
end
```

Here, the `Job` class gains both logging and retry logic — no inheritance, no duplication.

---

## Key Takeaways

| Concept   | Description                           | Example             |
| --------- | ------------------------------------- | ------------------- |
| `module`  | Defines reusable logic                | `module Utils`      |
| `include` | Adds methods as instance methods      | `include Formatter` |
| `extend`  | Adds methods as class methods         | `extend Audit`      |
| `prepend` | Overrides class methods before lookup | `prepend SafeRun`   |
| `::`      | Access constants or nested classes    | `Billing::Invoice`  |
| Constants | Fixed shared values                   | `VERSION = "1.0"`   |

---

### SE Perspective

When reading or debugging Ruby:

* Look for `include`, `extend`, or `prepend` to understand where methods come from.
* Remember that modules can change behavior without visible inheritance.
* Use namespacing to keep scripts organized and avoid conflicts.
* Favor small, composable modules over large, multipurpose classes.
* Trace method resolution with `ancestors` if unsure where a method is defined:

```ruby
SomeClass.ancestors
```

---
