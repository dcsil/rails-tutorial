# Data Model (Part 2)

Previously we looked at the following tables:

- Users
- Channels
- Messages
- Memberships
- Notifications

We have already managed the User table, and we will now focus on some of the other tables. Let's consider Channels and Memberships.

## Channels

We know based on the requirements that Channels can be created by Users but the other information is vague. Let's start with a name, description, and creator (user).

Run the following command to create the table, model, controller, views, etc for Channels:

`bin/rails g scaffold Channel name:string description:text user:references`

You may notice that there's a special `user` column of type `references`. This is because we want to associate a User with a Channel. This special type will add the associations for us.

You should have a migration file called `db/migrate/TIMESTAMP_create_channels.rb`.

```ruby
class CreateChannels < ActiveRecord::Migration[6.1]
  def change
    create_table :channels do |t|
      t.string :name
      t.text :description
      t.references :user, null: false, foreign_key: true

      t.timestamps
    end
  end
end
```

We will also have a model with the following:
```ruby
class Channel < ApplicationRecord
  belongs_to :user
end
```

The `belongs_to` method is a special method that will add an association to the Channel model. This will let us find the creator of a Channel by calling `user` on a channel instance. Keep in mind that this executes another SQL query to find the user. We will go over a way to optimize this in the next section.

First, we want to have the user be called `creator`. To do this we can modify the `belongs_to` call to:
`belongs_to :creator, class_name: "User", foreign_key: :user_id`.

Now if you run `bin/rails db:migrate` and `bin/rails s`, you should be able to go to http://localhost:3000/channels and create a channel. The UX here isn't optimal yet (e.g. you have to select a user by inputting a user ID when it should auto use the current user), but it's a start. We'll optimize it later.

## Memberships

Users can be members of many Channels. We will create a table to represent this relationship. Now, it's important to note that we don't need additional views or anything for this table because it is a simple join table and we don't need forms or listings of memberships right now. So instead of running a scaffold generate, we will run a model generate.

`bin/rails g model Membership user:references channel:references`

Now we can run the migration with `bin/rails db:migrate`. We can't actually use the table yet, but it is there!

## User Changes

We have just created a table for Channels and a table for Memberships. These both have associations into the user, but we have no way of referencing those. We can easily fix that with a few changes to the User model.

To the top of the User model add the following:
```ruby
has_many :memberships
has_many :channels, through: :memberships
has_many :created_channels, class_name: "Channel", foreign_key: :user_id
```

These add a couple of associations:

- `has_many :memberships`: This says "we have many memberships". Rails knows to use the `user_id` column by convention to figure out which memberships are ours
- `has_many :channels, through: :memberships`: This says "we have many channels through memberships". This will find all the channels that we are a member of by looking through the memberships table.
- `has_many :created_channels, class_name: "Channel", foreign_key: :user_id`: This says "we have many channels that we created". This will find all the channels that we created by looking at the table associated with the "Channel" model, and use the `user_id` column to find the channels that we created.

## Channel Changes

Add the following to the Channel model:

```ruby
has_many :memberships
has_many :members, through: :memberships, source: :user
```

These add a couple of associations:

- `has_many :memberships`: This says "we have many memberships". Rails knows to use the `channel_id` column by convention to figure out which memberships are ours
- `has_many :members, through: :memberships, source: :user`: This says "we have many members through memberships". We want to call them members and not users, so we can add the `source: :user` to tell it to use the user_id/User model
# Action Items

1. Run `bin/rails g scaffold Channel name:string description:text user:references`
1. Run `bin/rails g model Membership user:references channel:references`
1. Run `bin/rails db:migrate`
1. Change the belongs_to in `app/models/channel.rb` to `belongs_to :creator, class_name: "User", foreign_key: :user_id`
1. Add the following to the User model:
    ```ruby
    has_many :memberships
    has_many :channels, through: :memberships
    has_many :created_channels, class_name: "Channel", foreign_key: :user_id
    ```
1. Add the following to the Channel model:
    ```ruby
    has_many :memberships
    has_many :members, through: :memberships, source: :user
    ```
1. Run `bin/rails s`
1. Go to http://localhost:3000/channels and create a channel
1. Done!

# This section in the example app

https://github.com/dcsil/rails-tutorial-example/commit/1f9621cf79c5e06b3ce217f22b37889f3e6d0fd6
