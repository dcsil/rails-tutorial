# Nested Routes

In the last section we generated some routes for the memberships destroy action. If you noticed, the URL it generated was `/memberships/:id`.

Memberships, however, are a part of a user or a channel. It makes more sense to nest them under one of those to have the URL be something like `/channels/:channel_id/memberships/:id`.

In this section, we will do just that.

## Routes

Open up `config/routes.rb`. We are going to nest memberships under the `channels` resource (we chose channels since we will only act on memberships under current user, so the user nesting didn't make as much sense). The resource method accepts a block, and everything inside of that block will be nested under the that resource.

So this:
```ruby
resources :channels
resources :memberships
```

will become this:

```ruby
resources :channels do
  resources :memberships
end
```

> An aside: You can add arbitrary routes as well under resources by using a `collection` and `member` block.
> The `collection` block will nest it under the namespace, but not an instance.
> The `member` block will nest it under the instance.
> For example:
> ```ruby
> resources :channels do
>   collection do
>     get 'search'
>   end
>   member do
>     get 'info'
>   end
> end
> ```
> This will generate the following routes:
> - The default CRUD routes of the resource
> - `/channels/search` as `channels_search_path` (note the pluralization of channel)
> - `/channels/:id/info` as `channel_info_path`

Now when you run `bin/rails routes --grep memberships`, you will see the change:

```bash
$ bin/rails routes --grep memberships --expanded
--[ Route 1 ]------------------------------------------------------------------------------------------------------------------------------------
Prefix            | channel_membership
Verb              | DELETE
URI               | /channels/:channel_id/memberships/:id(.:format)
Controller#Action | memberships#destroy
```

The important thing to note is that the `prefix` also changed. Where we had `membership_path` before, we must replace it with `channel_membership_path` and also pass in the channel (or channel id).

The Leave Channel button should now be:

```erb
<%= link_to "Leave Channel", channel_membership_path(@channel, @active_membership), class: "btn btn-danger", method: :delete, data: { confirm: "Are you sure you want to leave #{@channel.name}?" } %>
```

## The controller changes

In Memberships Controller, we are now nested under Channels. That means that `channel_id` will always be available and we should only act on memberships in that channel.

By convention, we can add a `before_action :set_channel` to the top of the file. This is an action that will run before any action. Inside we will write code to set the `@channel` variable to the channel that is passed in the URL.

```ruby
before_action :set_channel

...

private

def set_channel
  @channel = Channel.find(params[:channel_id])
end
```

Now that we have that set up, we can scope the memberships call to the channel. Change this:

```ruby
# Find the user's current_membership
membership = current_user.memberships.find(params[:id])
```

to this:

```ruby
# Find the user's current_membership
membership = @channel.memberships.find_by(id: params[:id], user: current_user)
```

This changes the initial scope to use the channel. We also changed from using `find` which only accepts the ID, to using `find_by` which accepts keywords/hash of the attributes on which you want to search. In find_by, we include the current_user so it is still scoped to the current user.

Try it out!

# Action Items

1. Nest the memberships under the channels resource in `config/routes.rb`:
   ```ruby
   resources :channels do
     resources :memberships
   end
   ```
1. Update the URL for the Leave Channel Button in `app/views/channels/show.html.erb`:
   ```erb
    <%= link_to "Leave Channel", channel_membership_path(@channel, @active_membership), class: "btn btn-danger", method: :delete, data: { confirm: "Are you sure you want to leave #{@channel.name}?" } %>
    ```
1. Add `before_action :set_channel` to the top of `app/controllers/memberships_controller.rb`:
1. Add the set_channel method to the bottom of `app/controllers/memberships_controller.rb` under `private`:
   ```ruby
   private
   def set_channel
     @channel = Channel.find(params[:channel_id])
   end
   ```
1. Change the scope of the `memberships` call to the channel in `app/controllers/memberships_controller.rb`:
   ```ruby
   def destroy
     @membership = @channel.memberships.find_by(id: params[:id], user: current_user)
     ...
   end
   ```
1. Try it out!

# This section in the example app

https://github.com/dcsil/rails-tutorial-example/commit/f1b086c43cc437e0bfd5cc9c5f08b6f623660b47

# Next Section
- [Action text and messages: Part 1](15_action_text_and_messages_p1.md)