# Rails Controllers and Actions

Controllers are the **entry point** for requests in a Rails application.
They handle input, trigger business logic, and render responses.
Each action inside a controller represents an endpoint that can respond with HTML, JSON, or other formats.

---

## Why This Matters for SEs

Support Engineers often investigate issues related to:

* Incorrect request handling or routing.
* Missing or unexpected parameters.
* Unexpected callbacks or filters.
* Rendered responses or redirections.

Understanding how controllers work helps you trace the full request flow — from route to model to response.

---

## Basic Structure

Controllers inherit from `ApplicationController`, which itself inherits from `ActionController::Base` (or `ActionController::API` for APIs).

```ruby
class UsersController < ApplicationController
  def show
    @user = User.find(params[:id])
    render json: @user
  end
end
```

Here, `params` holds all incoming request parameters, and `render` sends the response.

---

## Actions

An **action** is any public method inside a controller.
Each action corresponds to a route.

```ruby
class OrdersController < ApplicationController
  def index
    @orders = Order.all
  end

  def create
    @order = Order.create(order_params)
  end
end
```

In Rails, the convention is simple:

| HTTP Verb | Action          | Purpose        |
| --------- | --------------- | -------------- |
| GET       | `index`, `show` | Retrieve data  |
| POST      | `create`        | Create records |
| PUT/PATCH | `update`        | Modify records |
| DELETE    | `destroy`       | Remove records |

---

## Filters: `before_action` and `after_action`

Filters run before or after controller actions.
They’re often used for authentication, authorization, or setup tasks.

```ruby
class OrdersController < ApplicationController
  before_action :authenticate_user
  before_action :set_order, only: [:show, :update, :destroy]
  after_action :log_access

  private

  def authenticate_user
    redirect_to login_path unless current_user
  end

  def set_order
    @order = Order.find(params[:id])
  end

  def log_access
    Rails.logger.info("Accessed Order #{@order&.id}")
  end
end
```

Filters keep controllers organized, but excessive use can hide important behavior, so always trace them during triage.

---

## Rendering Responses

Rails automatically renders views by convention, but you can control rendering explicitly.

### Render Templates

```ruby
render "orders/show"
```

### Render JSON

Common in APIs and automation tools.

```ruby
render json: { status: "ok", data: @order }
```

### Render with Status Code

```ruby
render json: { error: "Not found" }, status: :not_found
```

### Render Nothing (Head Responses)

For status-only responses, use `head`.

```ruby
head :no_content
```

---

## Redirecting

Use `redirect_to` to send users to another route or URL.

```ruby
redirect_to orders_path
```

You can include flash messages for user feedback.

```ruby
redirect_to root_path, notice: "Order created successfully"
```

---

## Strong Parameters

Strong parameters protect against mass-assignment vulnerabilities.
They define exactly which parameters are permitted for a model.

```ruby
private

def order_params
  params.require(:order).permit(:user_id, :status, :total)
end
```

Rails will raise an error if unpermitted parameters are passed.
This is critical for security — never allow unchecked input to reach models.

---

## Common Controller Helpers

| Method         | Description                                 |
| -------------- | ------------------------------------------- |
| `params`       | Access request parameters                   |
| `session`      | Store temporary data between requests       |
| `flash`        | Display messages after redirects            |
| `current_user` | Typically defined in authentication filters |
| `render`       | Send data or views as response              |
| `redirect_to`  | Redirect to another path                    |

---

## Example: JSON API Controller

```ruby
class TicketsController < ApplicationController
  before_action :find_ticket, only: [:show, :update, :destroy]

  def index
    render json: Ticket.all
  end

  def show
    render json: @ticket
  end

  def create
    ticket = Ticket.new(ticket_params)
    if ticket.save
      render json: ticket, status: :created
    else
      render json: { errors: ticket.errors.full_messages }, status: :unprocessable_entity
    end
  end

  private

  def find_ticket
    @ticket = Ticket.find(params[:id])
  end

  def ticket_params
    params.require(:ticket).permit(:title, :description, :priority)
  end
end
```

This pattern is standard in modern Rails APIs: filters for setup, strong params for security, and explicit rendering for clarity.

---

## Error Handling

Controllers often use `rescue_from` to handle specific exceptions gracefully.

```ruby
class ApplicationController < ActionController::Base
  rescue_from ActiveRecord::RecordNotFound, with: :not_found

  private

  def not_found
    render json: { error: "Resource not found" }, status: :not_found
  end
end
```

This prevents unexpected 500 errors from surfacing to users or systems.

---

## Key Takeaways

| Concept         | Description                               | Example                               |
| --------------- | ----------------------------------------- | ------------------------------------- |
| Action          | Controller method responding to a request | `def show; end`                       |
| `before_action` | Run code before specific actions          | `before_action :set_user`             |
| `after_action`  | Run code after actions                    | `after_action :log_access`            |
| `render`        | Return data or views                      | `render json: @data`                  |
| `redirect_to`   | Navigate to another page                  | `redirect_to root_path`               |
| Strong params   | Secure parameter filtering                | `params.require(:user).permit(:name)` |

---

### SE Perspective

When debugging controller issues:

* Start by checking **filters** — they often explain unexpected redirects or missing data.
* Look at the **action flow** and whether the response is rendered or redirected.
* Confirm that **strong parameters** allow the necessary fields.
* Review **error handlers** (`rescue_from`) for customized behavior.
* Use `rails routes` to verify which action handles the request.
* Always test JSON responses using `curl` or Postman to validate expected output.

---
