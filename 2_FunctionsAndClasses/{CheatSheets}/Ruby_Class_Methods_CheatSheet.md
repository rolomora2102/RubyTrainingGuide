# Ruby Class & Method Cheat Sheet

Quick reference for Ruby classes, methods, and visibility.
Focused on what Support Engineers need most when reading or writing Ruby scripts in production or internal tooling.

---

## Class Basics

| Concept           | Syntax                 | Example                             |
| ----------------- | ---------------------- | ----------------------------------- |
| Define class      | `class MyClass; end`   | `class Ticket; end`                 |
| Create instance   | `.new`                 | `t = Ticket.new`                    |
| Constructor       | `def initialize(args)` | `def initialize(id); @id = id; end` |
| Instance variable | `@variable`            | `@name = "Rolo"`                    |
| Class variable    | `@@variable`           | `@@count = 0`                       |
| Constant          | `NAME = "Example"`     | `Config::VERSION`                   |

---

## Methods

| Type             | Syntax                              | Example                     |
| ---------------- | ----------------------------------- | --------------------------- |
| Instance method  | `def method_name; end`              | `def call; puts "run"; end` |
| Class method     | `def self.method_name; end`         | `def self.fetch_all; end`   |
| Arguments        | `def call(a, b)`                    | `def greet(name = "user")`  |
| Keyword args     | `def run(mode:, retries: 3)`        | `run(mode: "safe")`         |
| Varargs          | `def add(*args)`                    | `add(1, 2, 3)`              |
| Forward all args | `def wrapper(...); inner(...); end` | Ruby 3+ syntax              |
| Return value     | `return` or last expression         | `return if invalid?`        |
| Yield block      | `yield`                             | `def wrap; yield; end`      |

---

## Attribute Helpers

| Helper          | Description     | Example                |
| --------------- | --------------- | ---------------------- |
| `attr_reader`   | Getter only     | `attr_reader :name`    |
| `attr_writer`   | Setter only     | `attr_writer :name`    |
| `attr_accessor` | Getter + Setter | `attr_accessor :email` |

---

## Visibility Keywords

| Keyword     | Description                     | Accessible From            |
| ----------- | ------------------------------- | -------------------------- |
| `public`    | Default visibility              | Anywhere                   |
| `private`   | Internal only, no receiver      | Within same object         |
| `protected` | Between instances of same class | Other objects of same type |

---

## Common Control Patterns

| Pattern         | Purpose              | Example                |   |               |
| --------------- | -------------------- | ---------------------- | - | ------------- |
| Guard clause    | Early exit           | `return unless valid?` |   |               |
| Memoization     | Cache computed value | `@data                 |   | = fetch_data` |
| Immutability    | Lock object          | `freeze`               |   |               |
| Constant access | Namespace            | `ModuleName::CONSTANT` |   |               |

---

## Modules and Mixins

| Keyword   | Description             | Example                 |
| --------- | ----------------------- | ----------------------- |
| `module`  | Define reusable group   | `module Utils; end`     |
| `include` | Add as instance methods | `include Loggable`      |
| `extend`  | Add as class methods    | `extend Configurable`   |
| `prepend` | Override before lookup  | `prepend SafetyWrapper` |

---

## Delegation

| Feature           | Description             | Example                         |
| ----------------- | ----------------------- | ------------------------------- |
| Manual delegation | Call nested object      | `@user.name`                    |
| `Forwardable`     | Standard library helper | `def_delegators :@user, :email` |
| `delegate`        | Rails helper            | `delegate :name, to: :user`     |

---

## Useful Reflection

| Method                | Description            | Example                  |
| --------------------- | ---------------------- | ------------------------ |
| `.ancestors`          | Shows lookup chain     | `User.ancestors`         |
| `.methods`            | List of methods        | `obj.methods.sort`       |
| `.instance_variables` | List of instance vars  | `obj.instance_variables` |
| `.respond_to?`        | Check method existence | `obj.respond_to?(:call)` |

---

## Quick Tips

* Prefer **composition** over inheritance for small scripts.
* Use **keyword arguments** for clarity.
* Keep methods under ~10 lines for readability.
* Freeze constants and shared data.
* Wrap unsafe logic inside classes with clear public interfaces.
* Use `binding.irb` or `pp obj` for quick runtime inspection.

---
