# **DebuggingInRails.md**

## Why Debugging in Rails Is Different

Rails hides a lot of complexity under conventions.
That’s great when everything works — but when something breaks, it helps to know how to *see what Rails is actually doing.*

Debugging in Rails means:

* Watching the SQL that ActiveRecord runs
* Inspecting callbacks and lifecycle hooks
* Using the console effectively
* Understanding how associations and queries behave in real time

---

## Console Setup and Safety

Start the console for an environment:

```bash
rails console
rails console production
```

If you just want to inspect data safely:

```bash
rails console --sandbox
```

Everything you do will be rolled back when you exit — perfect for tests or exploratory queries.

Always confirm your environment before making changes:

```ruby
Rails.env
ActiveRecord::Base.connection_db_config.database
```

If you’re in `production`, use **read-only DB roles** or wrap experiments in a rollback transaction.

---

## Seeing the SQL Behind the Scenes

Every ActiveRecord query can be previewed before execution:

```ruby
User.where(active: true).to_sql
# => "SELECT \"users\".* FROM \"users\" WHERE \"users\".\"active\" = 1"
```

To see how the database executes it:

```ruby
User.where(active: true).explain
```

For example:

```
EXPLAIN for: SELECT * FROM users WHERE active = 1
Seq Scan on users  (cost=0.00..12.70 rows=3 width=...)
```

### Temporarily Enable SQL Logging

```ruby
ActiveRecord::Base.logger.level = :debug
```

Now your console will show the exact SQL Rails runs for every call.

---

## Tracing the Model Lifecycle

Rails models run through a chain of callbacks during create/update/destroy.

Common ones:

```
before_validation
after_validation
before_save
after_save
after_commit
```

You can see where they’re defined:

```bash
grep -R "before_save" app/models
```

Or add a quick trace:

```ruby
puts "Saving #{self.inspect}" if Rails.env.development?
```

If a record isn’t saving, check:

1. Validations: `record.errors.full_messages`
2. Callbacks: look for `before_save` that halts with `throw(:abort)`

---

## Diffing Record States

Inspecting changes helps understand what Rails *thinks* changed.

```ruby
user.changed?         # true if unsaved changes exist
user.changes          # pending changes before save
user.saved_changes    # changes after save
user.previous_changes # previous transaction
user.reload           # re-fetch from DB
```

Example:

```ruby
user.update!(name: "New Name")
user.previous_changes
# => { "name" => ["Old Name", "New Name"], "updated_at" => [...] }
```

---

## Debugging Associations

Associations sometimes don’t behave as expected (e.g. missing preloads, nil values).

Check what’s loaded:

```ruby
user.association(:orders).loaded?   # => true or false
user.association(:orders).preload?  # => whether it's preloaded
```

See what’s available via reflection:

```ruby
User.reflect_on_all_associations.map(&:name)
```

If something is unexpectedly `nil`, ensure you didn’t forget `includes` or a proper foreign key.

---

## Spotting and Fixing N+1 Queries

Look for repeated SELECTs in logs:

```
User Load (0.5ms)
Profile Load (0.4ms)  SELECT "profiles".* WHERE "profiles"."user_id" = ?
```

Fix by eager loading:

```ruby
User.includes(:profile).each { |u| puts u.profile.name }
```

Validate with:

```ruby
User.includes(:profile).to_sql
```

and confirm the generated SQL doesn’t loop endlessly.

---

## Safe Sandboxes for Write Tests

If you *must* simulate writes in prod console, wrap them safely:

```ruby
ActiveRecord::Base.transaction do
  user = User.find_by(email: "a@b.c")
  user.update!(status: "active")
  raise ActiveRecord::Rollback
end
```

This ensures changes are rolled back automatically.

You can also simulate logic:

```ruby
ActiveRecord::Base.transaction do
  puts User.where(active: false).count
  raise ActiveRecord::Rollback
end
```

Great for “dry-run” counts or controlled scripts.

---

## Using `ActiveSupport::Notifications`

Rails instruments a ton of internal events — you can subscribe and print them live:

```ruby
ActiveSupport::Notifications.subscribe("sql.active_record") do |_, start, finish, _, data|
  puts "#{(finish - start).round(3)}s #{data[:sql]}"
end
```

You’ll see how long each SQL call takes, right in your console.

This also works for other events, like:

* `process_action.action_controller`
* `render_template.action_view`

Perfect for debugging slow endpoints.

---

## Log Tailing and Context

In another terminal:

```bash
tail -f log/development.log
```

Combine with **tagged logging** for better visibility:

```ruby
class ApplicationController < ActionController::Base
  around_action :add_context

  def add_context
    Rails.logger.tagged(request.uuid) { yield }
  end
end
```

Now every log line is traceable per request — incredibly useful during incident reviews.

---

## Minimal Debugging Toolkit Summary

| Goal                   | Command/Technique                        |
| ---------------------- | ---------------------------------------- |
| Preview SQL            | `.to_sql` / `.explain`                   |
| Watch logs             | `tail -f log/development.log`            |
| Inspect callbacks      | `grep -R "before_" app/models`           |
| See changed attributes | `.changes`, `.saved_changes`             |
| Check associations     | `.association(:foo).loaded?`             |
| Sandbox console        | `rails console --sandbox`                |
| Time code blocks       | `ActiveSupport::Notifications.subscribe` |

---

## Production Debugging Etiquette

1. Always start read-only when possible.
2. Avoid `.destroy_all` and `.update_all` without filters.
3. Wrap exploratory writes in transactions with `raise Rollback`.
4. Log what you do and why.
5. Verify with `.count`, `.limit(5)`, or `.pluck(:id)` before changing anything.

---

## Final Thoughts

Rails makes development fast — but debugging is where you really *understand* your application.
Being fluent in `to_sql`, callbacks, associations, and notifications turns Rails from a “magic box” into a transparent, predictable system.

When you can read your logs like a story and replicate production behavior safely in console, you’ve reached the level of confidence engineers trust for real-world operations.

---
