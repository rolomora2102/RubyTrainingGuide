# Flow & Logic

> **Purpose:**
> Understand how Ruby decides *what to run next* — including conditionals, iterators, blocks, and error handling.
> These are the foundations of reading, debugging, and writing Ruby code effectively in Support Engineering contexts.

---

## Truthiness and Guards

Ruby has a very simple truth system:
Only `false` and `nil` are **falsy**.
Everything else — even `0`, `""`, and `[]` — is **truthy**.

That’s why you’ll often see patterns like this in Ruby code:

```ruby
puts "value exists" if value
```

Even an empty string (`""`) will make that print, because it’s truthy.

---

### Guard Clauses

Guards help keep methods short and readable.
Instead of deep nested conditionals, Ruby developers often *return early* when something is missing or invalid.

```ruby
def process_order(order)
  return unless order  # guard clause — exit early if nil
  puts "Processing #{order.id}"
end
```

They make code cleaner and easier to read — something especially important in debugging logs or customer-impact code.

---

### Safe Navigation Operator

The safe navigation operator (`&.`) prevents your code from crashing when a variable might be `nil`.
Instead of throwing a `NoMethodError`, it simply returns `nil`.

```ruby
user&.account&.name
```

If `user` or `account` is `nil`, Ruby won’t crash — it’ll just skip gracefully.

> *You’ll use this constantly when exploring relationships in Rails models (for example, `order.customer&.email`).*

---

## Conditionals

Ruby conditionals are designed to read like natural language.
They use `if`, `elsif`, and `else`, always ending with `end`.

```ruby
if temperature > 30
  puts "It's hot"
elsif temperature > 20
  puts "Nice weather"
else
  puts "A bit cold"
end
```

For short checks, Ruby supports **inline conditionals**:

```ruby
puts "Warning!" if errors.any?
puts "All good!" unless errors.any?
```

Or use the **ternary operator** for simple one-line decisions:

```ruby
status = active ? "Active" : "Inactive"
```

---

## Case Statements

When you need to compare one value against several possibilities, `case` is cleaner than multiple `if` statements:

```ruby
case response.status
when 200 then puts "OK"
when 404 then puts "Not Found"
else          puts "Unexpected status"
end
```

You’ll often see this in error handling or HTTP response checks.

---

## Loops and Iterators

### Iterators

Iterators are Ruby’s elegant way of repeating actions — without the clutter of manual counters.
They’re everywhere in Rails because ActiveRecord collections behave just like arrays.

Example with an array:

```ruby
users = ["Rolo", "Danii", "Caro"]

users.each do |user|
  puts "Hello #{user}"
end
```

Or inline:

```ruby
users.each { |user| puts user }
```

---

### Common Iterator Methods

Ruby collections (arrays, hashes, ActiveRecord results, etc.) include many built-in iterator methods that you’ll use constantly as a Support Engineer.

| Method                 | Description                                                                            | Example                        |      |                        |
| ---------------------- | -------------------------------------------------------------------------------------- | ------------------------------ | ---- | ---------------------- |
| **`.each`**            | Runs a block for every element (no return value).                                      | `orders.each {                 | o    | puts o.id }`           |
| **`.each_with_index`** | Same as `.each`, but also provides an index counter.                                   | `users.each_with_index {       | u, i | puts "#{i+1}. #{u}" }` |
| **`.map`**             | Creates a **new array** with the results of the block — perfect for transforming data. | `emails = users.map(&:email)`  |      |                        |
| **`.select`**          | Returns only elements matching a condition. Doesn’t modify the original array.         | `active_users = users.select { | u    | u.active? }`           |
| **`.find`**            | Returns the **first** element matching a condition. Stops after it finds one.          | `user = users.find {           | u    | u.id == 5 }`           |

---

### Iterator Behavior

* `.each` is for **actions** — printing, logging, or side effects.
* `.map` is for **transformation** — generating new arrays or lists.
* `.select` is for **filtering** — narrowing results.
* `.find` is for **retrieving one** specific item.
* `.each_with_index` helps when you need an index counter in your output.

Example:

```ruby
numbers = [1, 2, 3, 4]

doubles = numbers.map { |n| n * 2 }   # => [2, 4, 6, 8]
evens   = numbers.select { |n| n.even? } # => [2, 4]
```

> *These same iterators work on ActiveRecord queries in Rails. For example:*
>
> ```ruby
> User.active.map(&:email)
> Order.pending.each { |o| puts o.id }
> ```

---

## Ranges

Ranges define a continuous sequence of values, useful for comparisons or queries.

```ruby
1..5   # includes 5
1...5  # excludes 5
```

They can contain numbers, characters, or even dates:

```ruby
('a'..'c') # => a, b, c
```

> *In Rails, you’ll often see date ranges used for reports or filtering logs:*
>
> ```ruby
> Order.where(created_at: 7.days.ago..Time.current)
> ```

---

## Blocks and `yield`

A **block** is a reusable chunk of code passed into a method — like a mini method without a name.
You’ll often see them in iterators, but they’re everywhere in Ruby.

```ruby
3.times do
  puts "Hello!"
end
```

You can also define blocks inline:

```ruby
3.times { puts "Hi again!" }
```

When a method includes `yield`, it can **execute the block** you pass to it:

```ruby
def process_user
  puts "Fetching user..."
  yield
  puts "Done!"
end

process_user { puts "Processing complete!" }
```

Output:

```
Fetching user...
Processing complete!
Done!
```

> *Rails uses this concept in callbacks and migrations. When you define `before_save` or `change` in migrations, Rails internally uses `yield` to run your code at the right time.*

---

## Error Handling

In Ruby, errors are handled using `begin...rescue...ensure`.
It’s similar to `try...catch`, but more expressive and flexible.

```ruby
begin
  result = 10 / 0
rescue ZeroDivisionError => e
  puts "Error: #{e.message}"
ensure
  puts "Execution finished."
end
```

Output:

```
Error: divided by 0
Execution finished.
```

### Multiple Rescue Blocks

You can handle specific error types separately:

```ruby
begin
  File.open("data.txt")
rescue Errno::ENOENT
  puts "File not found."
rescue StandardError => e
  puts "General error: #{e.message}"
end
```

---

### Raising Errors

You can raise your own errors to enforce logic or surface issues:

```ruby
raise "Something went wrong"
raise ArgumentError, "Invalid parameter"
```

---

### Error Handling in Rails

You’ll frequently see errors come from API calls, DB operations, or background jobs.
Use `rescue` blocks to catch and handle these safely:

```ruby
begin
  response = ExternalApi.get_data
rescue Net::OpenTimeout, SocketError
  puts "Network issue detected"
end
```

> *Understanding where Ruby raises exceptions helps you debug job failures, request errors, and model validations quickly.*

---

## Memoization and Early Returns

Sometimes you’ll want to **cache** a value that doesn’t change often — like a list of records.
Ruby makes this easy with the memoization pattern using `||=`:

```ruby
@orders ||= Order.pending.to_a
```

If `@orders` is already set, it won’t re-query the database.

Early returns also make code easier to follow — for example:

```ruby
def process(order)
  return unless order.active?
  puts "Processing #{order.id}"
end
```

> *You’ll see this style in many Rails jobs and service classes — it keeps logic predictable and efficient.*

---

**Summary**

* Only `false` and `nil` are falsy.
* Use guard clauses to keep code readable.
* `&.` avoids `nil` errors in nested objects.
* Iterators (`each`, `map`, `select`, `find`) are Ruby’s way to loop cleanly.
* Ranges simplify queries and filters.
* `yield` enables flexible method behavior.
* `begin/rescue` helps catch and explain errors gracefully.
* Memoization (`||=`) avoids redundant queries in jobs or debugging scripts.

---

