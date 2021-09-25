# Messages

Now that we have a layout, channels, and memberships all set up ... it's time to actually send messages!

Let's consider what a message is first:

- It's a string of text.
- It is sent from one user to either another user OR a channel.
- It is sent at a specific time
- It may have attached images (will ignore this for now)
- Should it be markdown? WYSIWYG (what you see is what you get, a visual text editor)?

Let's take a closer look at that last question.

Rails comes with a built-in text rendering engine called ActiveText: https://edgeguides.rubyonrails.org/action_text_overview.html

ActiveText can be integrated seamlessly to support [Trix](https://github.com/basecamp/trix), which is a rich text editor built by Basecamp.

While we could certainly use a different text editor or build a markdown rendering engine, we will use ActiveText to familiarize ourselves with the process.

## ActiveText

Run `bin/rails action_text:install` to add the JS dependency and copy over the necessary migration. This will create a migration to build out a separate `action_text_rich_texts` table. This will allow us to store rich text in our database.

Run `bin/rails db:migrate` to apply the migrations.

ActiveText is now installed!

## The Message Model

ActiveText can take care of the actual _message_ content of the message, but we will add it directly onto the message model itself.s

Otherwise, we need to know:
- Who sent the mesage
- Who/Where it was sent

That second question is a bit more complicated as we can send messages to channels and users. If we used a simple "references" column, then we would end up with a `*_id` column, which is not enough information to determine if we are talking about a channel or a user.

Rails comes with a built in "polymorphic" association that can handle multiple classes.

Let's run:
```bash
bin/rails g scaffold message content:rich_text channel:references sender:references recipient:references{polymorphic}
```

There are a few new concepts here:
- The content is a rich text field. This is a special type of text field that can handle rich text via ActiveText, we won't use that type otherwise.
- The sender is a reference to a user. This will generate a `sender_id` and not a `user_id`. This means we need to do a bit more work in the models to accommodate this, but we have a more accurate data model
- The recipient is a reference to a user or a channel. This will generate a `recipient_id` and a `recipeint_type` - not a `user_id` or `channel_id`. Rails will use the ID column to store the ID of the recipient, and the type column to store the class of the recipient. This will be almost transparent to us in the end.

This will generate some content we dont need:
- `app/assets/stylesheets/scaffolds.scss`
- `app/assets/stylesheets/messages.scss`

Run `rm app/assets/stylesheets/messages.scss app/assets/stylesheets/scaffolds.scss` to remove the scaffolds.scss and messages.scss files.

NOTE: Due to what seems to be a bug, we need to change the type of the generated references columns to bigint.

Open up the generated migration and remove `foreign_key: true` from the `recipient` and `sender` columns, and add `type: :bigint`. This is due to the fact that our original table is a bigint, and the references migration is trying to use a smaller integer size. The migration will fail otherwise.

Now run `bin/rails db:migrate` to apply the migration.

Next, we need to change the `belongs_to :sender` in the `Message` model to `belongs_to :sender, class_name: "User"` so that Rails knows where to find the sender.

## Channel and User Model

We just added messages with a recipient, which can be either a user or a channel. Now we will likely want to call `channel.messages` at some point, so we need to add that scope.

If it was a normal (non polymorphic) column, we could just add `has_many :messages`, but since it is a polymorphic column, we need to add a `has_many :messages, as: :recipient`. This will tell rails to use the `recipient_type` and `recipient_id` columns to determine the class and ID of the recipient.

In the user model we can also add `has_many :sent_messages, as: :sender, class_name: "Message"` to get all the messages sent by the user.

That's it for now, we will work to integrate the model soon!

# Action Items

1. Run `bin/rails action_text:install`
1. Run `bin/rails g scaffold message content:rich_text sender:references recipient:references{polymorphic}`
1. In the generated migration, remove `foreign_key: true` from the `recipient` and `sender` columns, and add `type: :bigint` to all the references columns.
1. Run `bin/rails db:migrate`
1. Run `rm app/assets/stylesheets/messages.scss app/assets/stylesheets/scaffolds.scss`
1. Add `has_many :messages, as: :recipient` to the channel and user models
1. Add `has_many :sent_messages, as: :sender, class_name: "Message"` to the user model
1. Change `belongs_to :sender` in the `Message` model to `belongs_to :sender, class_name: "User"`
1. Done!

# This section in the example app

https://github.com/dcsil/rails-tutorial-example/commit/705bf962012b0311ed6bf5e9d292d1a513e18057

# Next Section
- [Messages views: Part 2](16_messages_views_p2.md)