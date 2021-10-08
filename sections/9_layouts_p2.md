# Layouts (Part 2)

In the last section we added a basic layout. However, if you now log out and load the sign in page, it will use this layout too!

In this section we will make the layout more specific to logged in users, and add the user's channel memberships to the sidebar.

## Making the layout specific

If you remember from the previous section, we added Devise to the application. This also added the `user_signed_in?` we can use.

To fix the layout for logged out users, let's make use of that.

Change the body content of the `app/views/layouts/application.html.erb` file to:
```erb
<% if user_signed_in? %>
  <div class="d-flex height-full">
    <div class="col-2 p-2 color-bg-canvas-inverse color-text-inverse">
      CHANNELS GO HERE
    </div>
    <div class="col-10 p-2">
      <%= yield %>
    </div>
  </div>
<% else %>
  <%= yield %>
<% end %>
```

Now the logged out user will not have any special layouts.

## Adding the user's channel memberships to the sidebar (and a log out button)

In the sidebar we want to show a list of the user's channel memberships. Let's leverage the `current_user` method to get the user, and ask for the channels association we added before.

Then we'll put that in a [Primer `Box Box--condensed`](https://primer.style/css/components/box#box-padding-density) in the sidebar.

The layout's body content should be something like:
```erb
 <% if user_signed_in? %>
  <div class="d-flex height-full">
    <nav class="col-2 pt-5 color-bg-canvas-inverse color-text-inverse menu" aria-label="Person settings">
      <%= link_to "Log out", destroy_user_session_path, method: :delete, class: "menu-item" %>
      <% current_user.channels.each do |channel| %>
        <a class="menu-item color-text-inverse d-flex flex-justify-between" href="<%= channel_path(channel) %>">
          <span><%= channel.name %></span>
          <span class="Counter color-bg-canvas"><%= channel.members.count %></span>
        </a>
      <% end %>
    </nav>
    <div class="col-10 p-2">
      <%= yield %>
    </div>
  </div>
<% else %>
  <%= yield %>
<% end %>
```

You'll see we use the `current_user` method to get the user, and then use the `channels` association to get the channels. We don't have to check if the `current_user` is nil, because we're using `user_signed_in?` to check if the user is logged in.

For each channel we will link to the channel's show page in a `Box-row`.

If you reloaded the page now, you should see the user's channel memberships in the sidebar but we don't currently have any. Let's fix that.

# Channel Memberships

A channel membership is what determined if a user is in a channel. We have no specific controller or endpoint to determine this, but we can consider the act of showing a channel "joining" it, at least for now.

Upon viewing a channel, we could add the user to the channel's memberships.

Open up `app/controllers/channels_controller.rb` and add the following to the `show` method:

```ruby
current_user.memberships.find_or_create_by!(channel: @channel)
```

This will take the `current_user` (guaranteed to exist because the `before_action` in the `application_controller` ensures that we are logged in) and create a membership with the channel we found.

We found a channel in a `before_action`. At the top of the `ChannelsController` you will find the following:
```ruby
before_action :set_channel, only: %i[ show edit update destroy ]
```

This calls that method and sets the `@channel` variable.

Lastly, `find_or_create_by!` will try to find or create a membership based on the parameters passed in. If it can't do either, it will raise an error (that's what the `!` means). When you do `MODEL.ASSOCIATIONS.CREATE/UPDATE_METHOD(PARAMS)`, the model is automatically added to the params as needed (as a user_id in this case).

Now when you view a channel you will be automatically added to the channel's memberships. We can implement "leave" functionality later.

# Action Items

1. Change the body content of the `app/views/layouts/application.html.erb` file to:
    ```erb
    <% if user_signed_in? %>
      <div class="d-flex height-full">
        <nav class="col-2 pt-5 color-bg-canvas-inverse color-text-inverse menu" aria-label="Person settings">
          <% current_user.channels.each do |channel| %>
            <a class="menu-item color-text-inverse d-flex flex-justify-between" href="<%= channel_path(channel) %>">
              <span><%= channel.name %></span>
              <span class="Counter color-bg-canvas"><%= channel.members.count %></span>
            </a>
          <% end %>
        </nav>
        <div class="col-10 p-2">
          <%= yield %>
        </div>
      </div>
    <% else %>
      <%= yield %>
    <% end %>
    ```
2. Open up `app/controllers/channels_controller.rb` and add the following to the `show` method:

    ```ruby
    current_user.memberships.find_or_create_by!(channel: @channel)
    ```
3. Go to a channel (create one if you have none) and check the sidebar, you should see that channel show up there.

# This section in the example app

https://github.com/dcsil/rails-tutorial-example/commit/465a698563c82a6001fd9f3bc09717d46ced242b

# Next Section
- [Forms and Validations](10_forms_and_validations.md)