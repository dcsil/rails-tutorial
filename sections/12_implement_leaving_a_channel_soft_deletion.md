# Leaving a Channel: Soft Deletion (Part 1)

Right now we have a basic set up with a user and a channel. We can join a channel, but we can't leave it. Let's implement that.

When we enter a channel, we create a membership join table record. We could consider a few options to indicate a user has left a channel:
- We could simply delete this membership
- We could add an "active" column to the membership table and toggle this on/off
- We could soft delete the membership (mark it as deleted using a column) and create new one for new joins

Now let's consider what memberships can tell us:
- We may want to understand how many times the user has joined the channel
- We want to know when the user had last joined a channel, when they left the channel, and how long they have been in the channel
- Something else perhaps?

For the purposes of this exercise, we are going to implement soft deletion as this will help us answer questions like "how many times has this user joined this channel?"

# Soft Deletion

Soft deletion can be implemented in a few ways:
- We can use a column to mark a record as deleted
- We can use a timestamp to mark a record as deleted
- We can use a combination of the above

[Discard](https://github.com/jhawthorn/discard) is a library that can be used to implement soft deletion. It is implemented by @jhawthorn at GitHub and is available as a Gem. It provides some extra functionality for models such as:
- `kept` scope (equivalent to `where(deleted_at: nil)`)
- `discarded` scope (equivalent to `where.not(deleted_at: nil)`)
- And others:
  ```ruby
    post = Post.first   # => #<Post id: 1, ...>
    post.discard        # => true
    post.discard!       # => Discard::RecordNotDiscarded: Failed to discard the record
    post.discarded?     # => true
    post.undiscarded?   # => false
    post.kept?          # => false
    post.discarded_at   # => 2017-04-18 18:49:49 -0700
  ```

It is encouraged to read through that Gem's README to understand the nuance in soft deletion. For now, though, let's add the gem to our Gemfile and run `bundle install`.

```ruby
gem 'discard', '~> 1.2'
```

NOTE: To take advantage of the discarded column, you cannot call `destroy` or `delete` on the record. That will _actually_ delete the record (destroy initiates model callbacks, delete doesn't and just executes SQL). Instead you must call `discard` or `discard!` (the latter will raise an error if the record is not discarded).

## Adding a Discard Column

We now have the gem installed. Let's add a `discarded_at` column to our `memberships` table.

Run the following command to generate the migration:
```bash
bin/rails generate migration add_discarded_at_to_memberships discarded_at:datetime:index
```

Rails will parse this command and generate the following migration:
```ruby
class AddDiscardedAtToMemberships < ActiveRecord::Migration[6.1]
  def change
    add_column :memberships, :discarded_at, :datetime
    add_index :memberships, :discarded_at
  end
end

```

There are a few new things here:
- It knows how to parse the table/model name out of the command. In this case it parses `Memberships` out of `add_discarded_at_to_memberships`
- It can accept a `:index` option to add an index to the column

Now we need to add `include Discard::Model` to our `memberships` model. It should now look like this:
```ruby
class Membership < ApplicationRecord
  include Discard::Model
  belongs_to :user
  belongs_to :channel
end
```

## Scopes

### `current_user.memberships`

By default discarded records are included in calls like `Membership.all`, `current_user.memberships`, etc. This is great as it can be nuanced when you want to include everything, but you may want to exclude discarded records.

For example, when we join a channel we want to create a new record when no existing `undiscarded` record exists, but want to create one otherwise.

To accomplish this, we can open up our `ChannelsController` again and add a `kept` scope to the `current_user.memberships` cal which should now be:

```ruby
current_user.memberships.kept.find_or_create_by!(channel: @channel)
```

### `channel.members`

We also want to consider our calls to `channel.members.count` in the sidebar. We want to exclude discarded records when we count members.

`members` is an association on channel through `memberships`, so we can't add a `kept` scope to that members. There are a few options:
- We could add a separate `has_many` for all members and kept members
- We could add a separate `has_many` for kept memberships
- We could add a where clause to the members call (`channel.members.where(membership: { discarded_at: nil })`)

All things considered, we typically want to avoid having to add where clauses each time - we prefer to add a scope.

In reading the discard README, they also recommend against adding a default scope to your main scope, so in this case let's add a new scope.

Let's start by adding an `active_memberships` scope to `Channel`. We can use `has_many` like before, but give the name `active_memberships`. Because we are not using a name that matches a column, we also have to provide a `class_name`. If we added just a class_name, we would have an identical scope to `memberships`, so we need to add a constraint to that scope: `-> { kept }`. This will call our `kept` scope on the `memberships` table, which is available from the discard gem.

```ruby
has_many :active_memberships, -> { kept }, class_name: "Membership"
```

We can then add an `active_members` scope to mirror our members scope.

```ruby
has_many :active_members, through: :active_memberships, source: :user
```

Now we can change the call from `channel.members` to `channel.active_members` and we will get the correct result. We also want to change `memberships` to `active_memberships` in the `member?(user)` method we added before.

### `user.channels`

We also need to fix `channels` on the user model. Similar to channel, we get our association through a membership scope. We can add an `active_memberships` like before and change channels to use that:

```ruby
  has_many :active_memberships, -> { kept }, class_name: "Membership"
  has_many :channels, through: :active_memberships
```

We will want to change `current_user.channels` to `current_user.active_channels` in `app/views/layouts/application.html.erb`.

# Action Items

1. Add the `discard` gem to your Gemfile:
    ```ruby
    gem 'discard', '~> 1.2'
    ```
1. Run `bundle install`
1. Add a `discarded_at` column to your `memberships` table:
    ```bash
    bin/rails generate migration add_discarded_at_to_memberships discarded_at:datetime:index
    ```
1. Run `bin/rails db:migrate`
1. Add `include Discard::Model` to your `memberships` model:
    ```ruby
    class Membership < ApplicationRecord
      include Discard::Model
      belongs_to :user
      belongs_to :channel
    end
    ```
1. Add a `kept` scope to your `memberships` in the `ChannelsController#show`:
    ```ruby
    def show
      current_user.memberships.kept.find_or_create_by!(channel: @channel)
    end
    ```
1. Add `active_memberships` and `active_members` scopes to your `Channel`:
    ```ruby
    has_many :active_memberships, -> { kept }, class_name: "Membership"
    has_many :active_members, through: :active_memberships, source: :user
    ```
1. Add `active_memberships` and `active_channels` scopes to your `Channel`:
    ```ruby
    has_many :active_memberships, -> { kept }, class_name: "Membership"
    has_many :active_channels, through: :active_memberships, source: :channel
    ```
1. In `app/views/layouts/application.html.erb`, change `current_user.channels` to `current_user.active_channels` and `channel.members` to `channel.active_members`
1. Done!

# This section in the example app

https://github.com/dcsil/rails-tutorial-example/commit/22df515bce94c856fe485c0556429e2a160948ec

# Next Section
- [Implement leaving a channel: Controllers and routing](13_implement_leaving_a_channel_controllers_and_routing_p2.md)