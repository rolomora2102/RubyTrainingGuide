# Background Jobs and Queues in Rails

Background jobs allow Ruby and Rails applications to perform long-running or asynchronous work outside of the main web request cycle.
This keeps systems responsive while processing tasks like email delivery, data imports, or API integrations.

Rails provides a unified interface for background work through **ActiveJob**, which can run on adapters like Sidekiq, Resque, or DelayedJob.

---

## Why This Matters for SEs

Support Engineers often investigate or monitor job systems for:

* Missing or stuck jobs.
* Duplicate processing or retries.
* Misconfigured queues or adapters.
* Logging and performance issues.

Understanding job flow helps you quickly trace how asynchronous work is triggered, executed, and logged.

---

## ActiveJob Basics

All jobs inherit from `ApplicationJob`, which extends `ActiveJob::Base`.

```ruby
class ReportJob < ApplicationJob
  def perform(report_id)
    report = Report.find(report_id)
    report.generate
  end
end
```

This defines the work the job should execute.

You enqueue a job like this:

```ruby
ReportJob.perform_later(42)
```

`perform_later` places it in the background queue, while `perform_now` runs it immediately in the same process.

---

## Queues

Each job can define its own queue using `queue_as`.

```ruby
class ReportJob < ApplicationJob
  queue_as :reports
end
```

You can view or filter jobs by queue name in dashboards like Sidekiq Web UI or Resque.

Queues help separate workloads — for example, `emails`, `reports`, or `critical`.

---

## Job Uniqueness

Job uniqueness is **not native to Rails**, but supported by certain queue adapters or gems (e.g., `sidekiq-unique-jobs`).
It prevents the same job from being enqueued multiple times simultaneously.

```ruby
class SyncInventoryJob < ApplicationJob
  unique :until_executed

  def perform(store_id)
    InventorySyncService.call(store_id)
  end
end
```

When debugging, verify if your environment uses a uniqueness gem and whether jobs are being deduplicated or blocked.

---

## Retries and Error Handling

ActiveJob includes basic retry behavior, depending on the adapter.
You can customize retry logic manually.

```ruby
class SyncJob < ApplicationJob
  retry_on Net::TimeoutError, wait: 10.seconds, attempts: 3
  discard_on ActiveRecord::RecordNotFound

  def perform
    # job logic
  end
end
```

* `retry_on` retries jobs when certain exceptions occur.
* `discard_on` prevents endless retry loops for unrecoverable errors.

This is key for stability in distributed systems — you can control failure handling precisely.

---

## Logging and Observability

Logging jobs helps with triage and post-incident analysis.
Use structured logs with context identifiers.

```ruby
class PaymentJob < ApplicationJob
  queue_as :payments

  def perform(payment_id)
    Rails.logger.info("Starting PaymentJob for #{payment_id}")
    PaymentService.call(payment_id)
    Rails.logger.info("PaymentJob completed for #{payment_id}")
  rescue => e
    Rails.logger.error("PaymentJob failed for #{payment_id}: #{e.message}")
    raise
  end
end
```

In large systems, logs from jobs may go to centralized monitoring tools (e.g., CloudWatch, Datadog, Kibana).
Understanding how to filter logs by job ID or queue name accelerates root cause analysis.

---

## Callbacks in Jobs

ActiveJob provides lifecycle hooks similar to models and controllers:

```ruby
class NotifyUserJob < ApplicationJob
  before_perform { Rails.logger.info("Starting job...") }
  after_perform  { Rails.logger.info("Job done!") }
end
```

These hooks are useful for metrics, telemetry, or structured logging.

---

## Idempotency

Idempotency ensures that running a job multiple times produces the same result.
This is vital for reliability in distributed environments.

For example, check if a record already exists before creating it again:

```ruby
def perform(order_id)
  return if Order.exists?(order_id)
  create_order(order_id)
end
```

Jobs should always be written in a way that allows safe re-execution.

---

## Example: End-to-End Background Job

```ruby
class DataExportJob < ApplicationJob
  queue_as :exports
  retry_on Timeout::Error, wait: 30.seconds, attempts: 5

  def perform(user_id)
    user = User.find(user_id)
    data = ExportService.generate_for(user)

    File.write("exports/#{user.id}.json", data.to_json)
    Rails.logger.info("Export completed for user #{user.email}")
  rescue => e
    Rails.logger.error("Export failed for #{user_id}: #{e.message}")
    raise
  end
end
```

This example demonstrates queue assignment, retries, and logging — a complete, production-safe structure.

---

## Testing and Running Jobs

You can run jobs synchronously in Rails console or tests:

```ruby
ReportJob.perform_now(1)
```

For background execution, check the adapter:

```ruby
Rails.application.config.active_job.queue_adapter
```

Common adapters include:

* `:async` (default for small apps)
* `:sidekiq`
* `:resque`
* `:delayed_job`

Ensure production environments use reliable queue systems like Sidekiq for scalability.

---

## Key Takeaways

| Concept         | Description                | Example                        |
| --------------- | -------------------------- | ------------------------------ |
| `perform`       | Job logic                  | `def perform(args)`            |
| `perform_later` | Async execution            | `Job.perform_later(id)`        |
| `queue_as`      | Assigns queue name         | `queue_as :critical`           |
| Uniqueness      | Avoids duplicates          | `unique :until_executed`       |
| Retries         | Handles transient failures | `retry_on Timeout::Error`      |
| Callbacks       | Run before/after job       | `before_perform {}`            |
| Idempotency     | Safe re-execution          | `return if already_processed?` |

---

### SE Perspective

When investigating background jobs:

* Check **queue adapter configuration** — the job may never run if it’s misconfigured.
* Inspect **job logs** using the job ID or queue name.
* Confirm **retries and uniqueness** settings to avoid duplicates or floods.
* Validate **idempotency** — jobs should handle re-execution safely.
* For failed jobs, check the queue backend (e.g., Sidekiq Web UI) for stack traces and retry counts.

---
