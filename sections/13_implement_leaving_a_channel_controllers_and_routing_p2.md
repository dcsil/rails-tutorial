# Leaving a Channel: Controllers and Routing (Part 2)

Now that memberships can be soft deleted, we have the underlying architecture to leave a channel, but we need to actually implement the views and endpoints to allow this.

We allow users to join a channel by creating a membership when they look at the channel's `show` action (e.g. `/channel/1`). However, there is no equivalent mechanism to indicate a user wishes to leave a channel. We must implement our own.

There are 2 ways to do this:
- We could add a new `leave` action to the channel controller, and then handle it there
- We could add a new `memberships` controller and have only a `destroy` action

The former is easiest, while the latter adheres more strictly to REST principles (CRUD actions only: e.g. index, show, update, create, destroy, edit, new).

Let's take the latter approach since it adheres to REST and provides us an opportunity to learn more of Rails' built-in functionality.

## Creating the memberships controller

To create a controller, you could manually build one like so:
```
class MembershipsController < ApplicationController
end
```
Rails requires this to be at `app/controllers/memberships_controller.rb`.

However, like everything else with Rails, there is a generator:

```bash
bin/rails generate controller memberships
```

This is great because it will create a controller, but also create a few other files:
```
create  app/controllers/memberships_controller.rb
invoke  erb
create    app/views/memberships
invoke  test_unit
create    test/controllers/memberships_controller_test.rb
invoke  helper
create    app/helpers/memberships_helper.rb
invoke    test_unit
invoke  assets
invoke    scss
create      app/assets/stylesheets/memberships.scss
```

We dont need all of this though (we wont have views, so we need none of the helpers/views/scss), so we can remove `app/views/memberships`, `app/assets/stylesheets/memberships.scss`, and `app/helpers/memberships_helper.rb`.

Next, we need to add the `destroy` action to the controller. We will need to:
- Find the membership based on the current user
- Soft delete the membership
- Redirect to something other channel's `show` action (this would rejoin)
- Add a notice for success or error

```ruby
class MembershipsController < ApplicationController
  def destroy
    # Find the user's current_membership
    membership = current_user.memberships.find(params[:id])
    if membership.discard # Returns true if successful
      redirect_to channels_path, notice: "You have left the channel."
    else
      redirect_to channels_path, notice: "Failed to leave the channel: #{membership.errors.full_messages.to_sentence}"
    end
  end
end
```

The `notice:` key you see above, and in other places, is a Rails convention. It is used to pass a message to the next request. You will see `<p id="notice"><%= notice %></p>` in various spots in the application. We will work to improve this view later on, and consolidate it to a single location.

## Routing

Right now we have a controller, but we can't reach it from the browser. We need to tell Rails how to reach it.

Open up `config/routes.rb` and add the following:
```ruby
resources :memberships, only: :destroy
```

This will add routes for the memberships controller, but only for the destroy action. If you run `bin/rails routes` you can see it (protip: add `--grep memberships` to scope down the response)

```bash
$ bin/rails routes --grep memberships --expanded
--[ Route 1 ]--
Prefix            | membership
Verb              | DELETE
URI               | /memberships/:id(.:format)
Controller#Action | memberships#destroy
```

## The Views

Now that we have a controller and routing, we need to implement the view to leave the channel. We will implement a "Leave button" in the channel's `show` action.

First things first, we need to know the active membership for the current user and channel. Luckily we already find_or_create a membership, so we just need to assign it to an instance variable to use in the view.

Change `current_user.memberships.kept.find_or_create_by!(channel: @channel)` to `@active_membership = current_user.memberships.kept.find_or_create_by!(channel: @channel)`.

Then in the `app/views/channels/show.html.erb` file, add the following:
```erb
<%= link_to "Leave Channel", membership_path(@active_membership), class: "btn btn-danger", method: :delete, data: { confirm: "Are you sure you want to leave #{@channel.name}?" } %>
```

What this does:

- This will create a link with the text "Leave Channel".
- It will like to the membership path, which is a helper added by our `config/routes.rb` file. 
- This button will have the classes `btn btn-danger`.
- The `data: { confirm: "..." }` part will be translated into a `data-confirm="..."` html attribute. This will use JS to show a confirm prompt when the user clicks this destructive action.
- The `method` will be added as a `data-method="delete"` attribute. It is important that we send using an HTTP delete rather than a standard GET as that is what our route is set up for.

Those last 2 parts normally would do nothing in a standard HTML application, but Rails adds some Javascript in the front end to handle these attributes. In `apps/javascripts/packs/application.js`, you can see the line `import Rails from "@rails/ujs"`. Rails-UJS is a library that adds some Javascript to the front end to handle these attributes.

Now let's try it out!

# Action Items

1. Run `bin/rails generate controller memberships`
1. Run `rm -rf app/views/memberships app/assets/stylesheets/memberships.scss app/helpers/memberships_helper.rb`
1. Open up `config/routes.rb` and add the following:
    ```ruby
    resources :memberships, only: :destroy
    ```
1. Open the ChannelsController and assign the membership to an instance variable.
    ```ruby
    @active_membership = current_user.memberships.kept.find_or_create_by!(channel: @channel).
    ```
1. Open up `app/views/channels/show.html.erb` and add the following:
    ```erb
    <%= link_to "Leave Channel", membership_path(@active_membership), class: "btn btn-danger", method: :delete, data: { confirm: "Are you sure you want to leave #{@channel.name}?" } %>
    ```
1. Try it out!

# This section in the example app

https://github.com/dcsil/rails-tutorial-example/commit/a209b0183211e49b3eb5beee22271612a2355331

# Next Section
- [Nested routes](14_nested_routes.md)