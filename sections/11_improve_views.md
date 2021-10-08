# Improve Views

We are at the point where we have basic data models and decent views for the most part. Before we continue, let's make sure our existing views are all decent.

Channels Index, Users Index, and the New/Edit page of both are all outdated.

We can use Primer to update these. This can be a more creative endeavour, so I'll list what I have here but it's not the only way to do it.

## Channels Index

First, we will add a helper method to `channel.rb`. Add the following method:

```ruby
  def member?(user)
    memberships.exists?(user_id: user.id)
  end
```

This will let us easily check is a user is in a channel without worrying about SQL calls.

Next, we'll change the views.

```erb
<p id="notice"><%= notice %></p>

<div class="d-flex flex-justify-between flex-items-center">
  <h1>Channel List</h1>
  <div><%= link_to 'New Channel', new_channel_path, class: "btn btn-sm btn-primary" %></div>
</div>

<div class="Box">
  <ul>
    <% @channels.each do |channel| %>
      <li class="Box-row d-flex flex-justify-between">
        <div>
          <h2><%= channel.name %></h2>
          <p><%= channel.description %></p>
          <%= link_to "Created by #{channel.creator.name}", user_path(channel.creator), class: "Link--muted" %>
        </div>
        <div class="d-flex flex-column">
          <%= link_to channel.member?(current_user) ? 'Show Channel' : 'Join Channel', channel, class: "btn btn-primary" %>
          <% if channel.creator == current_user %>
            <div><%= link_to 'Edit', edit_channel_path(channel), class: "btn mt-1" %></div>
            <div><%= link_to 'Destroy', channel, method: :delete, data: { confirm: "Are you sure you want to destroy #{channel.name}?" }, class: "btn mt-1 btn-danger" %></div>
          <% end %>
        </div>
      </li>
    <% end %>
  </ul>
</div>
```

## Users Index

```erb
<p id="notice"><%= notice %></p>

<div class="d-flex flex-justify-between flex-items-center">
  <h1>Users</h1>
  <div><%= link_to 'New user', new_user_path, class: "btn btn-sm btn-primary" %></div>
</div>

<div class="Box">
  <ul>
    <% @users.each do |user| %>
      <li class="Box-row d-flex flex-justify-between">
        <div>
          <h2><%= user.name %><% if user.pronouns.present? %> <small><%= user.pronouns %></small><% end %></h2>
          <p><%= user.bio %></p>
        </div>
        <div class="d-flex flex-column">
          <%= link_to 'See profile', user, class: "btn btn-primary" %>
          <% if user == current_user %>
            <div><%= link_to 'Edit', edit_user_path(user), class: "btn mt-1" %></div>
            <div><%= link_to 'Destroy', user, method: :delete, data: { confirm: "Are you sure you want to delete your account?" }, class: "btn mt-1 btn-danger" %></div>
          <% end %>
        </div>
      </li>
    <% end %>
  </ul>
</div>
```

## Edit/New

```erb
<div class="container">
  <h1>Editing Channel</h1>
  <%= render 'form', channel: @channel %>
</div>
```

All of these are basically the same, changing out the variable and heading.

# Action Items

1. Add a `member?` method to `channel.rb`
    ```ruby
      def member?(user)
        memberships.exists?(user_id: user.id)
      end
    ```
1. Update Channels Index (See above)
2. Update Users Index (See above)
3. Update Edit/New for Channels/Users (See above)

# This section in the example app

https://github.com/dcsil/rails-tutorial-example/commit/380b9ce7e4a1e06f0be2e2d2d7229adb8eda2ea4

# Next Section
- [Implement leaving a channel: Soft deletion](12_implement_leaving_a_channel_soft_deletion.md)