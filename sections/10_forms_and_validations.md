# Forms and Validations

At this point we have a decent layout for our application, but we need to add some functionality to it. We can add users and forms, but we have no validations and the forms are default.

Let's update the forms, then add proper validations.

# Forms

The forms are located in `app/views/users/_form.html.erb` and `app/views/channels/_form.html.erb`. These are used for the new/edit views for each respective model.

Take a look at the Primer form CSS: https://primer.style/css/components/forms

If we use the form-group example, we can translate the labels and form to:
```erb
<div class="form-group">
  <div class="form-group-header">
    <%= form.label :description %>
  </div>
  <div class="form-group-body">
    <%= form.text_area :description, class: "form-control" %>
  </div>
</div>
```

As you can see, we can add HTML classes. Convert all of the form over using Primer.

# Controller Validations

Right now we are able to update other users and channels created by other users, this should not be allowed. We can fix this with some before actions.

In `app/controllers/channels_controller.rb` we can add the following after the `before_action :set_channel` call as these are called in order and we need the channel's assignment:

```ruby
before_action :ensure_creator!, only: [:edit, :update, :destroy]
```

This will call a method we will define called `ensure_creator!`. The `!` is a convention to indicate it could block the action. This will also only be run on the edit, update, and destroy actions (aka the modifying actions of a channel).

The content of the method is:

```ruby
def ensure_creator!
  return if @channel.creator == current_user
  redirect_to channels_url, notice: "You can only edit or delete your own channels."
  false # Prevent the action from continuing
end
```

We can do a similar thing for the users controller.

# Model Validations

Now that we have some validation on controller actions, it makes sense to add some validation to the models.

If we consider the `Channel` model, there are a number of validations we can add. You can see the full list [here](https://guides.rubyonrails.org/active_record_validations.html#validation-helpers).

For now, let's validate that a channel's name is under 80 characters (arbitrary), is present, and is unique. We will also add a basic description validation.

We can add the following to the `Channel` model:
```ruby
validates :name, length: { maximum: 80 }, presence: true, uniqueness: true
validates :description, length: { maximum: 240 }
```

Note: These validations _don't_ add database level constraints. You can consider adding these separately if you wish via migrations but that won't be covered in this tutorial.

Now if you try to add a duplicate channel name, it will fail.

As an exercise, add additional validations you think are necessary to both Channel and User.

Note 2: You cannot go to /users/new to create a user as Devise blocks this. Log out and register a new user instead.

# Forms and Model Validations

We now need to nicely show the model validations in the form.

Rails, when a validation error occurs, is nice and will give us human readable error messages in the errors attribute.

For example, errors on a `Channel` model will look like this:
```ruby
channel.errors.full_messages #=> ["Name can't be blank", "Name is too long (maximum is 80 characters)"]
channel.errors.messages #=> {"name"=>["can't be blank", "is too long (maximum is 80 characters)"], "description"=>["is too long (maximum is 240 characters)"]}
channel.errors[:name] #=> ["can't be blank", "is too long (maximum is 80 characters)"]
channel.errors[:name].to_sentence #=> "Name can't be blank and is too long (maximum is 80 characters)"
```

We can use this to our advantage to show the user what they need to fix.

We can change this:
```erb
<div class="form-group">
  <div class="form-group-header">
    <%= form.label :description %>
  </div>
  <div class="form-group-body">
    <%= form.text_area :description, class: "form-control" %>
  </div>
</div>
```

to this:

```erb
<div class="form-group <% if user.errors[:name].any? %>errored<% end %>">
  <div class="form-group-header">
    <%= form.label :name %>
  </div>
  <div class="form-group-body">
    <%= form.text_field :name, class: "form-control", "aria-describedby": user.errors[:name].any? ? "name-input-validation" : nil %>
    <% if user.errors[:name].any? %>
      <p class="note error" id="name-input-validation">Name <%= user.errors[:name].to_sentence %>.</p>
    <% end %>
  </div>
</div>
```

At the same time, remove the unstyled error_explanation divs:

```erb
<div id="error_explanation">
  <h2><%= pluralize(channel.errors.count, "error") %> prohibited this channel from being saved:</h2>
  <ul>
    <% channel.errors.each do |error| %>
      <li><%= error.full_message %></li>
    <% end %>
  </ul>
</div>
```

As an exercise, convert both the `Channel` and `User` forms to use the same approach.

# Action Items

1. Update forms to use Primer's Form Group. Here is an example:
    ```erb
    <div class="form-group">
      <div class="form-group-header">
        <%= form.label :description %>
      </div>
      <div class="form-group-body">
        <%= form.text_area :description, class: "form-control" %>
      </div>
    </div>
    ```
2. In `app/controllers/channels_controller.rb`, after the `before_action :set_channel` call, add the `ensure_creator!` call:
    ```ruby
    before_action :ensure_creator!, only: [:edit, :update, :destroy]
    ```
3. In the same controller, add the ensure_creator! method:
    ```ruby
    def ensure_creator!
      return if @channel.creator == current_user
      redirect_to channels_url, notice: "You can only edit or delete your own channels."
      false # Prevent the action from continuing
    end
    ```
4. In `app/controllers/users_controller.rb`, after the `before_action :set_user` call, add the `ensure_user!` call:
    ```ruby
    before_action :ensure_user!, only: [:edit, :update, :destroy]
    ```
5. In the same controller, add the ensure_user! method:
    ```ruby
    def ensure_user!
      return if @user == current_user
      redirect_to channels_url, notice: "You can only edit or delete yourself."
      false # Prevent the action from continuing
    end
    ```
6. Add the following validations to the `Channel` model:
    ```ruby
    validates :name, length: { maximum: 80 }, presence: true, uniqueness: true
    validates :description, length: { maximum: 240 }
    ```
7. Add appropriate validations to the `User` model. NOTE: You cannot go to /users/new to create a user as Devise blocks this. Log out and register a new user instead.
8. Update the User and Channel forms to show error messages by using the format:
    ```erb
    <div class="form-group <% if user.errors[:name].any? %>errored<% end %>">
      <div class="form-group-header">
        <%= form.label :name %>
      </div>
      <div class="form-group-body">
        <%= form.text_field :name, class: "form-control", "aria-describedby": user.errors[:name].any? ? "name-input-validation" : nil %>
        <% if user.errors[:name].any? %>
          <p class="note error" id="name-input-validation">Name <%= user.errors[:name].to_sentence %>.</p>
        <% end %>
      </div>
    </div>
    ```
9. Remove the unstyled `error_explanation` divs:
    ```erb
    <div id="error_explanation">
      <h2><%= pluralize(channel.errors.count, "error") %> prohibited this channel from being saved:</h2>
      <ul>
        <% channel.errors.each do |error| %>
          <li><%= error.full_message %></li>
        <% end %>
      </ul>
    </div>
    ```
10. Run the server and try to create a channel.

# This section in the example app

https://github.com/dcsil/rails-tutorial-example/commit/77dc2f4783b1b38c7791d227978e0a7ecc0ab1c8

# Next Section
- [Improve views](11_improve_views.md)