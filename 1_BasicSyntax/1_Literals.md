## 1. Basic Syntax

Ruby is known for being human-readable and expressive. Its syntax emphasizes clarity and simplicity — similar to Python, but with its own elegant flavor.

In this section, we’ll cover **Literals** and **Data Types**, which are the foundation of any Ruby program.

---

## Literals

Literals are the most direct way to create a value in Ruby.

They represent **fixed values** such as strings, numbers, symbols, and arrays that you can assign to variables.

For example:

```ruby
string = "string"
number = 20
```

Alternatively, you can instantiate objects using class constructors:

```ruby
string = String.new("string")
```

However, for scripting and debugging in Support Engineering, **literals** are preferred — they’re faster to write and easier to read.

> *When reviewing Ruby code in Veeqo applications, remember that both forms are valid — what matters is understanding what type of value you’re dealing with.*

---

## Data Types

As in most programming languages, Ruby provides several core data types. The most common ones you’ll encounter include:

* **String**
* **Integer**
* **Float**
* **Array**
* **Hash**
* **Symbol**
* **Boolean**

Let’s start with **Strings** — one of the most frequently used types in Ruby scripts and automation.

---

### Strings

Strings are sequences of characters enclosed in either double quotes (`" "`) or single quotes (`' '`).

Example:

```ruby
greet = "Hello"
```

Like arrays, strings can be accessed using **indexes**:

| Expression | Output |
| ---------- | ------ |
| `greet[0]` | `"H"`  |
| `greet[1]` | `"e"`  |

Strings also support **escape characters**, which modify how text is displayed or interpreted.

For instance:

```ruby
greet = "hell\no"
```

Output:

```
hell
o
```

Common escape characters include:

* `\n` — New line
* `\t` — Tab
* `\\` — Backslash

---

### HEREDOC Strings

For multi-line text, Ruby offers **HEREDOCs**, which make large blocks of text easier to manage and read:

```ruby
expected_result = <<HEREDOC
This would contain specially formatted text.

That might span many lines
HEREDOC
```

You can replace `HEREDOC` with any custom terminator word:

```ruby
long_string = <<END
Hello Trainees!
I hope you learn a lot.
END
```

---

### Other Multi-Line Options

You can also use backslashes (`\`) to continue text on the next line:

```ruby
long_string = "Hello Trainees! \
I hope you learn a lot."
```

Or use **`%Q()` syntax**, which behaves like double-quoted strings and supports interpolation:

```ruby
long_greet = %Q(
Hello Trainees!
I hope you learn a lot.
)
```

> *In most day-to-day tasks, you’ll work with simple single-line strings. However, knowing HEREDOCs is useful when logging long responses or storing templates.*

---


## Numbers

In Ruby, numbers are objects that can represent integers, decimals, or even values written in hexadecimal, binary, or scientific notation.
They’re part of the `Numeric` class family, with the most common being `Integer` and `Float`.

---

### Integer

Integers are whole numbers — positive, negative, or zero.
They’re written in decimal form by default:

```ruby
count = 42
```

Ruby also supports other numeric bases, which are often seen in backend data or ActiveRecord fields:

| Type        | Example  | Value |
| ----------- | -------- | ----- |
| Decimal     | `25`     | 25    |
| Binary      | `0b1101` | 13    |
| Octal       | `0o17`   | 15    |
| Hexadecimal | `0xFF`   | 255   |

---

### Float

Floats are numbers with decimal points.

```ruby
price = 19.99
```

They can also use **scientific notation**, which is common in data processing and math operations:

```ruby
distance = 1.2e3  # 1.2 × 10³ = 1200.0
```

---

### Basic Operations

Ruby supports standard arithmetic operators for all numeric types:

`+`, `-`, `*`, `/`, `%`, and `**`

Example:

```ruby
5 / 2.0  # => 2.5
```

If both operands are integers, the result will be an integer. To get a float, make at least one operand a float.

---

### Type Conversion

You’ll often need to convert values, especially when parsing from databases or logs:

```ruby
"10".to_i    # => 10
"3.14".to_f  # => 3.14
```

---

### Quick Notes

* Use underscores for readability: `1_000_000`
* Random values can be generated with `rand(100)`
* Numbers can be checked or compared directly (`x.positive?`, `x.zero?`, etc.)

---

## Booleans and Conditionals

Booleans in Ruby are simple but powerful — they represent truthy or falsy values used for logic, flow control, and validation.

---

### Boolean Values

Ruby has two main boolean values:

```ruby
true
false
```

And one special value often used in conditionals:

```ruby
nil  # represents “nothing” or “no value”
```

In Ruby, **only `false` and `nil` are considered falsy**.
Everything else (including `0`, empty strings `""`, and empty arrays `[]`) is truthy.
This is different from many other languages like JavaScript or Python.

---

### Conditional Statements

Ruby’s conditionals follow a natural, readable flow.
They start with `if`, can include `elsif` and `else`, and always end with `end`.

Example:

```ruby
if temperature > 30
  puts "It's hot outside"
elsif temperature > 20
  puts "Nice weather"
else
  puts "A bit cold today"
end
```

---

### One-Line Conditionals

For quick checks or inline logic, Ruby allows concise one-liners:

```ruby
puts "Warning!" if errors.any?
```

You can also reverse the order for clarity:

```ruby
puts "All good!" unless errors.any?
```

---

### Logical Operators

Ruby supports the common logical operators used in conditionals:

| Operator | Meaning     | Example                   |            |       |   |               |
| -------- | ----------- | ------------------------- | ---------- | ----- | - | ------------- |
| `&&`     | Logical AND | `true && false` → `false` |            |       |   |               |
| `        |             | `                         | Logical OR | `true |   | false`→`true` |
| `!`      | Logical NOT | `!true` → `false`         |            |       |   |               |

You’ll also see the English-style versions (`and`, `or`, `not`) — these behave slightly differently in precedence, but for scripts, both are acceptable as long as they’re consistent.

---

### Case Statement

When comparing a value against multiple possibilities, `case` makes code cleaner:

```ruby
case status
when 200
  puts "OK"
when 404
  puts "Not Found"
else
  puts "Unexpected status"
end
```

This pattern appears often when handling API responses or internal job results.

---


## Symbols

Symbols in Ruby can feel a bit abstract at first. They look similar to strings, but they behave very differently under the hood.

A **symbol** is a lightweight, immutable label that points to a single memory location.
This means that every time you reference the same symbol, Ruby uses the *exact same object in memory* — no matter where it appears in your code.

---

### Understanding Symbols

When you create a symbol like this:

```ruby
symbol = :status
```

Ruby stores `:status` once and reuses it everywhere.
You can confirm this by checking its `object_id`:

```ruby
:symbol.object_id
# => 870108

:symbol.object_id
# => 870108  (same ID every time)
```

Now compare that to strings:

```ruby
"name".object_id
# => 340

"name".object_id
# => 360  (different each time)
```

This shows the main difference:

* **Symbols** are *unique and immutable*.
* **Strings** are *recreated each time* they’re called.

---

### Why This Matters

Since a symbol always refers to the same memory space, it’s perfect when you need a consistent, unchanging identifier — like a key, flag, or label that represents something fixed across the application.

This is why symbols are widely used in **Rails**, especially inside **ActiveRecord models**, **params**, or **hash keys**:

```ruby
user = { name: "Rolo", status: :active }
```

Here, `:name` and `:status` are symbols that serve as identifiers for the values they hold.

---

### Symbols vs. Constants

At first glance, symbols might look like constants, but they’re not.

| Concept      | Description                                                                                      |
| ------------ | ------------------------------------------------------------------------------------------------ |
| **Symbol**   | Immutable label that always points to the same memory location. Does not hold a value by itself. |
| **Constant** | A named container that *stores* a value. The name doesn’t change, but the stored value can.      |

Example:

```ruby
PI = 3.14        # Constant
:pi               # Symbol
```

---

### Key Takeaways

* Symbols are **immutable** names or labels.
* They use **one memory reference** no matter how many times they appear.
* They’re **not strings**, even though they look similar.
* They’re used as **identifiers**, often in hashes or method names.
* The **same symbol always has the same `object_id`**.

---

> 💡 *Think of symbols as "labels for meaning," not data.
> You don’t change them, you refer to them — and that’s what makes them so efficient and reliable in backend code.*

---


## Arrays

In Ruby, **arrays** are ordered collections used to store multiple values — similar to *lists* in other languages like Python.

You can think of an array as a container that keeps elements in sequence, accessible by their numeric position (index).

---

### Arrays vs. Lists

If you’ve worked with lists in Python or JavaScript, Ruby arrays will feel familiar.
Technically, Ruby doesn’t have a separate “list” type — the `Array` class does all that work.

So while other languages distinguish between lists, tuples, or sequences, in Ruby **it’s all just arrays**, and that’s perfectly fine for us.

For Support Engineers, this distinction doesn’t matter much.
When we run Rails queries like `.where`, `.pluck`, or `.ids`, the data we get back is almost always an **array**.

We will see this in detail later on.

Example:

```ruby
User.where(active: true)
# => [#<User id: 1>, #<User id: 2>, #<User id: 3>]
```

Or if we pluck specific fields:

```ruby
User.pluck(:email)
# => ["rolo@example.com", "sebvargs@example.com", "moracald@example.com"]
```

So whenever you see brackets `[]`, think **“array of results”** — even if the data comes directly from ActiveRecord or an API response.

---

### Creating Arrays

Arrays are created with square brackets `[]` or by using `Array.new`:

```ruby
colors = ["red", "blue", "green"]
numbers = Array.new([1, 2, 3])
```

Both produce the same structure.

---

### Accessing Elements

Ruby arrays are zero-indexed — meaning the first element is `[0]`:

```ruby
colors[0]  # => "red"
colors[-1] # => "green" (last element)
```

You can also slice arrays:

```ruby
colors[0..1]  # => ["red", "blue"]
```

---

### Common Array Methods

Ruby provides simple and intuitive methods for working with arrays:

| Method               | Description                 | Example                              |   |           |
| -------------------- | --------------------------- | ------------------------------------ | - | --------- |
| `.first`             | Returns the first element   | `users.first`                        |   |           |
| `.last`              | Returns the last element    | `users.last`                         |   |           |
| `.length` or `.size` | Count of elements           | `users.size`                         |   |           |
| `.include?`          | Checks if an element exists | `users.include?(“rolo@example.com”)` |   |           |
| `.each`              | Iterates through elements   | `users.each {                        | u | puts u }` |

---

### Arrays in Rails

In day-to-day support work, you’ll often inspect arrays in logs, Rails console, or API payloads.
Here’s an example from a real-world scenario:

```ruby
Order.where(status: :pending).pluck(:id)
# => [1021, 1022, 1023]
```

This returns an array of IDs that you can quickly check, count, or use to debug a process.

> 💡 *In short: arrays are your best friends when exploring or validating backend data.
> You don’t need to worry about whether they’re “lists” or “arrays” — they behave the same way in Ruby.*

---

## Hashes

A **Hash** in Ruby is a collection of **key-value pairs**.
If an array is like a list of items, a hash is more like a labeled box — each piece of data has a name (the key) and a corresponding value.

Hashes are extremely common in Ruby and Rails, especially when working with **ActiveRecord data**, **JSON payloads**, or **API responses**.

---

### Basic Syntax

You can create a hash in two main ways:

```ruby
# Old style
user = { :name => "Rolo", :role => "Support Engineer" }

# Modern style (preferred)
user = { name: "Rolo", role: "Support Engineer" }
```

Both are valid, but the second style is cleaner and used everywhere in Rails.
The keys here (`:name`, `:role`) are **symbols**, not strings — which keeps them consistent and memory-efficient.

---

### Accessing Values

To get a value, call its key inside brackets:

```ruby
user[:name]  # => "Rolo"
```

If you try to access a key that doesn’t exist, Ruby returns `nil` instead of raising an error.

```ruby
user[:email]  # => nil
```

This behavior is helpful when inspecting data in logs or debugging nil-related issues.

---

### Updating Hashes

You can easily add or update key-value pairs:

```ruby
user[:team] = "Veeqo Support"
user[:role] = "SSE"  # updates existing value
```

---

### Common Hash Methods

| Method    | Description                   | Example                                            |      |                      |
| --------- | ----------------------------- | -------------------------------------------------- | ---- | -------------------- |
| `.keys`   | Returns all keys              | `user.keys` → `[:name, :role, :team]`              |      |                      |
| `.values` | Returns all values            | `user.values` → `["Rolo", "SSE", "Veeqo Support"]` |      |                      |
| `.each`   | Iterates over keys and values | `user.each {                                       | k, v | puts "#{k}: #{v}" }` |
| `.merge`  | Combines two hashes           | `user.merge({ location: "CR" })`                   |      |                      |

---

### Hashes in Rails

Rails uses hashes everywhere. When you see data like this:

```ruby
params = { user: { name: "Rolo", email: "rolo@example.com" } }
```

Or when ActiveRecord returns structured data:

```ruby
User.find_by(id: 1).attributes
# => { "id" => 1, "name" => "Rolo", "email" => "rolo@example.com" }
```

That’s all hash data — keys and values representing column names and their corresponding data.

---

### Nested Hashes

Hashes can contain other hashes, allowing complex structured data:

```ruby
ticket = {
  id: 1021,
  customer: { name: "Carlos", email: "moracald@example.com" },
  status: :open
}
```

You can access nested values using chained keys:

```ruby
ticket[:customer][:email]  # => "moracald@example.com"
```

> 💡 *This structure appears often when reading payloads from APIs or ActiveRecord JSON fields — so being comfortable navigating nested hashes is key.*

---

### Symbols as Keys

You’ll notice that most Rails hashes use **symbols** as keys instead of strings.
This is because symbols are faster, take up less memory, and remain consistent throughout the app.

If you receive a hash with string keys (for example, from JSON), you can easily convert them:

```ruby
hash.transform_keys(&:to_sym)
```

---

### Accessing Data from API Responses

When working with API or Rails responses, remember that **the keys are usually strings**, not symbols.
This means you’ll need to use **string keys** when accessing data directly.

Example:

```ruby
response.payload["orders"]
```

If you try using a symbol here (e.g., `response.payload[:orders]`), it will likely return `nil`, because the actual key in the hash is `"orders"` — not `:orders`.

You can always check the key type like this:

```ruby
response.payload.keys
# => ["orders", "status", "timestamp"]
```

> 💡 *Tip:* If you prefer working with symbols, you can safely convert the keys once using:
>
> ```ruby
> response.payload.transform_keys(&:to_sym)
> ```
>
> Then you can use symbol access like `response[:orders]`.

This small detail avoids a lot of confusion when debugging JSON or inspecting API payloads in the Rails console.

---

## Ranges and Iterators

In Ruby, **ranges** represent a sequence of values with a defined start and end.
They’re simple, powerful, and often used in loops, queries, and conditions.

---

### Ranges

A range is created using two or three dots:

```ruby
1..5   # includes 5 → (1, 2, 3, 4, 5)
1...5  # excludes 5 → (1, 2, 3, 4)
```

You can use them with numbers, characters, or even dates:

```ruby
('a'..'d')  # => a, b, c, d
```

They’re especially useful when validating or filtering data:

```ruby
if (1..10).include?(value)
  puts "Value is within range"
end
```

> 💡 *In Rails, ranges are often used for date-based queries:*
>
> ```ruby
> Order.where(created_at: (Date.today - 7.days)..Date.today)
> ```

---


