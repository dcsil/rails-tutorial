# Messages

Now that we have a layout, channels, and memberships all set up ... it's time to actually send messages!

Let's consider what a message is first:

- It's a string of text.
- It is sent from one user to either another user OR a channel.
- It is sent at a specific time
- It may have attached images (will ignore this for now)
- Should it be markdown? WYSIWYG (what you see is what you get, a visual text editor)?

Let's take a closer look at that last question.

Rails comes with a built-in text rendering engine called ActionText: https://edgeguides.rubyonrails.org/action_text_overview.html

ActionText can be integrated seamlessly to support [Trix](https://github.com/basecamp/trix), which is a rich text editor built by Basecamp.

While we could certainly use a different text editor or build a markdown rendering engine, we will use ActionText to familiarize ourselves with the process.

## ActionText

Run `bin/rails action_text:install` to add the JS dependency and copy over the necessary migration. This will create a migration to build out a separate `action_text_rich_texts` table. This will allow us to store rich text in our database.

Run `bin/rails db:migrate` to apply the migrations.

ActionText is now installed!

## The Message Model

ActionText can take care of the actual _message_ content of the message, but we will add it directly onto the message model itself.s

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
- The content is a rich text field. This is a special type of text field that can handle rich text via Action Text, we won't use that type otherwise.
- The sender is a reference to a user. This will generate a `sender_id` and not a `user_id`. This means we need to do a bit more work in the models to accommodate this, but we have a more accurate data model
- The recipient is a reference to a user or a channel. This will generate a `recipient_id` and a `recipeint_type` - not a `user_id` or `channel_id`. Rails will use the ID column to store the ID of the recipient, and the type column to store the class of the recipient. This will be almost transparent to us in the end.

This will generate some content we dont need:
- `app/assets/stylesheets/scaffolds.scss`
- `app/assets/stylesheets/messages.scss`

Run `rm app/assets/stylesheets/messages.scss app/assets/stylesheets/scaffolds.scss` to remove the scaffolds.scss and messages.scss files.

NOTE: Due to what seems to be a bug, we need to change the type of the generated references columns to bigint.

Open up the generated migration and remove `foreign_key: true` from the `channel` and `sender` columns, and add `type: :bigint`. This is due to the fact that our original table is a bigint, and the references migration is trying to use a smaller integer size. The migration will fail otherwise.

Now run `bin/rails db:migrate` to apply the migration.

That's it for now, we will work to integrate the model soon!

# Action Items

1. Run `bin/rails action_text:install`
1. Run `bin/rails g scaffold message content:rich_text channel:references sender:references recipient:references{polymorphic}`
1. In the generated migration, remove `foreign_key: true` from the `channel` and `sender` columns, and add `type: :bigint` to all the references columns.
1. Run `bin/rails db:migrate`
1. Run `rm app/assets/stylesheets/messages.scss app/assets/stylesheets/scaffolds.scss`
1. Done!

# Commit in the Example app

This is _all_ generated code. Nothing should have been modified outside of changing the types/foreign keys in that migration.

https://github.com/dcsil/rails-tutorial-example/commit/72ab2732544f4d3f7d6cb4b76af2eaf74c3e9473
