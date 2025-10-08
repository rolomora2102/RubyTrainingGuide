# Rails Controller DSL Cheat Sheet

Quick reference for **Rails controllers**, filters, rendering, and request handling.
Designed for Support Engineers who need to understand the controller flow quickly when debugging API behavior or customer-facing issues.

---

## Core Controller Structure

| Concept           | Syntax                                               | Example                                               |
| ----------------- | ---------------------------------------------------- | ----------------------------------------------------- |
| Define controller | `class UsersController < ApplicationController; end` | `class OrdersController < ApplicationController; end` |
| Action method     | `def action_name; end`                               | `def show; end`                                       |
| Route link        | Maps via `config/routes.rb`                          | `get "/users/:id", to: "users#show"`                  |

Each **public method** inside a controller becomes an action accessible by route.

---

## Common Filters

Filters run before, after, or around actions to handle cross-cutting concerns like authentication or logging.

| Filter          | Timing                 | Example                            |
| --------------- | ---------------------- | ---------------------------------- |
| `before_action` | Runs before action     | `before_action :authenticate_user` |
| `after_action`  | Runs after action      | `after_action :log_request`        |
| `around_action` | Wraps around execution | `around_action :measure_runtime`   |

**Example:**

```ruby
class OrdersController < ApplicationController
  before_action :set_order
  after_action  :audit_access

  private

  def set_order
    @order = Order.find(params[:id])
  end

  def audit_access
    Rails.logger.info("Order accessed: #{@order.id}")
  end
end
```

---

## Rendering

Render decides what to return as a response.
By default, Rails renders a view matching the action name, but you can override it.

| Type          | Example                                                   | Description                         |
| ------------- | --------------------------------------------------------- | ----------------------------------- |
| View template | `render "users/show"`                                     | Renders `.html.erb` or `.html.haml` |
| JSON          | `render json: @user`                                      | API-style response                  |
| Inline        | `render plain: "OK"`                                      | Quick text response                 |
| Status        | `render json: { error: "Not found" }, status: :not_found` | Include HTTP code                   |
| Nothing       | `head :no_content`                                        | Send only HTTP status code          |

---

## Redirecting

Redirects send users or clients to a different route or URL.

| Type        | Example                                 | Description         |
| ----------- | --------------------------------------- | ------------------- |
| Internal    | `redirect_to users_path`                | Redirect within app |
| With params | `redirect_to user_path(@user)`          | Redirect with ID    |
| With notice | `redirect_to root_path, notice: "Done"` | Adds flash message  |

---

## Strong Parameters

Strong parameters protect models from unpermitted mass assignment.
They are a core Rails security feature.

**Example:**

```ruby
def user_params
  params.require(:user).permit(:name, :email, :role)
end
```

If an attribute isn’t listed, Rails will ignore it — or raise an error if `config.action_controller.action_on_unpermitted_parameters` is set.

---

## Common Controller Helpers

| Helper         | Purpose                     | Example                        |
| -------------- | --------------------------- | ------------------------------ |
| `params`       | Access request parameters   | `params[:id]`                  |
| `session`      | Store data between requests | `session[:user_id] = @user.id` |
| `flash`        | Temporary user messages     | `flash[:notice] = "Updated"`   |
| `current_user` | Usually set by auth logic   | `current_user.email`           |
| `cookies`      | Manage browser cookies      | `cookies[:token] = "abc"`      |

---

## Error Handling

Use `rescue_from` in `ApplicationController` to handle exceptions gracefully.

```ruby
class ApplicationController < ActionController::Base
  rescue_from ActiveRecord::RecordNotFound, with: :not_found

  private

  def not_found
    render json: { error: "Resource not found" }, status: :not_found
  end
end
```

This prevents 500s and provides consistent error responses.

---

## JSON APIs

For API-only apps, controllers usually inherit from `ActionController::API`, omitting session/cookie support but keeping filters, rendering, and rescue behavior.

**Example:**

```ruby
class Api::V1::TicketsController < ActionController::API
  before_action :find_ticket

  def show
    render json: @ticket
  end

  private

  def find_ticket
    @ticket = Ticket.find(params[:id])
  end
end
```

---

## Status Codes (Symbols)

| Symbol                   | Code | Description          |
| ------------------------ | ---- | -------------------- |
| `:ok`                    | 200  | Successful response  |
| `:created`               | 201  | Resource created     |
| `:no_content`            | 204  | Success, no body     |
| `:bad_request`           | 400  | Invalid request      |
| `:unauthorized`          | 401  | Auth required        |
| `:forbidden`             | 403  | Permission denied    |
| `:not_found`             | 404  | Record missing       |
| `:unprocessable_entity`  | 422  | Validation failed    |
| `:internal_server_error` | 500  | General server error |

---

## Logging and Metrics

Use structured logs to make controller activity traceable.

```ruby
Rails.logger.info(
  controller: controller_name,
  action: action_name,
  params: params.to_unsafe_h.slice(:id, :user_id)
)
```

This helps correlate user actions with background jobs or database operations.

---

## Example Controller: Full Lifecycle

```ruby
class TicketsController < ApplicationController
  before_action :authenticate_user
  before_action :find_ticket, only: [:show, :update, :destroy]

  def index
    render json: Ticket.all
  end

  def show
    render json: @ticket
  end

  def update
    if @ticket.update(ticket_params)
      render json: @ticket
    else
      render json: { errors: @ticket.errors.full_messages }, status: :unprocessable_entity
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

---

## Key Takeaways

| Concept           | Description             | Example                                |
| ----------------- | ----------------------- | -------------------------------------- |
| `before_action`   | Run setup or auth       | `before_action :set_user`              |
| `render`          | Send response           | `render json: @data`                   |
| `redirect_to`     | Navigate elsewhere      | `redirect_to root_path`                |
| Strong parameters | Whitelist input         | `params.require(:user).permit(:email)` |
| `rescue_from`     | Handle exceptions       | `rescue_from RecordNotFound`           |
| API controllers   | Lightweight controllers | `< ActionController::API`              |

---

### SE Perspective

When debugging controller issues:

* Always check **filters** — authentication or parameter setup may block the action.
* Review **render** vs **redirect_to** — responses may be redirecting silently.
* Confirm **strong parameters** include all required fields.
* Trace **exception handling** — `rescue_from` may suppress stack traces.
* Use logs or `rails routes` to validate request flow and controller mapping.
* For JSON APIs, check headers (`Content-Type` and `Accept`) to ensure expected responses.

---
