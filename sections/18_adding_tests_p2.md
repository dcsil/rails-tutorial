# Adding Tests (Part 2)

In the last section we got some basic issues fixed, and fixed our users controller tests. We will focus on `test/controllers/channels_controller_test.rb` now.

## ChannelsControllerTest#test_should_destroy_channel

```
Error:
ChannelsControllerTest#test_should_destroy_channel:
ActiveRecord::InvalidForeignKey: Mysql2::Error: Cannot delete or update a parent row: a foreign key constraint fails (`chat_app_test-0`.`memberships`, CONSTRAINT `fk_rails_49d43d6efb` FOREIGN KEY (`channel_id`) REFERENCES `channels` (`id`))
    app/controllers/channels_controller.rb:54:in `destroy'
    test/controllers/channels_controller_test.rb:46:in `block (2 levels) in <class:ChannelsControllerTest>'
    test/controllers/channels_controller_test.rb:45:in `block in <class:ChannelsControllerTest>'


rails test test/controllers/channels_controller_test.rb:44
```

This is very similar to the issue with foreign keys in the last section. We can fix this the same way by adding a `dependent: :destroy` to memberships and messages.

## ChannelsControllerTest#test_should_update_channel

```
Failure:
ChannelsControllerTest#test_should_update_channel [/Users/juliannadeau/src/github.com/dcsil/chat_app/test/controllers/channels_controller_test.rb:41]:
Expected response to be a <3XX: redirect>, but was a <422: Unprocessable Entity>


rails test test/controllers/channels_controller_test.rb:39
```

So this test shows us that we should be able to update a channel, but it is returning a 422 unprocessable entity. Let's add a debugger in the channel controller update action and scope down to that test for our next run: `rails test test/controllers/channels_controller_test.rb:39`

Note: Make sure to put the debugger _after_ the call to update, otherwise you wont have any errors to see. In this case, right after `else` is a great spot.

```
   45:       else
   46:         debugger
=> 47:         format.html { render :edit, status: :unprocessable_entity }
   48:         format.json { render json: @channel.errors, status: :unprocessable_entity }
   49:       end
   50:     end
   51:   end
(byebug) @channel.valid?
false
(byebug) @channel.errors.full_messages
["Name has already been taken"]
```

As we can see our validation for the channel name is causing issues. This is a bit confusing though because we are updating the channel to have the same name as before, so what's going on?

Remember our fixtures? Those are injected by SQL into the database, and not through Rails. These fixtures don't have to adhere to validations we add on the Rails side as they are being pushed directly into the database.

If we open `test/fixtures/channels.yml` we can change the names to be unique.

This test now passes!

## ChannelsControllerTest#test_should_create_channel

Our last issue is with the create channel test. We can take the same approach and put a debugger in the `create` action after `Channel.new`. Let's scope down to this test: `rails test test/controllers/channels_controller_test.rb:21`

We put the debugger before the save call, so errors will be empty:
```
(byebug) @channel.errors.full_messages
[]
```

We can force validation by calling `.valid?` on the channel object.

```
(byebug) @channel.valid?
false
(byebug) @channel.errors.full_messages
["Name has already been taken"]
```

This is a bit confusing, but we can see that the channel name is already taken. Didn't we just solve this?

We fixed the fixtures, but our call to this endpoint duplicated the name of an existing channel. Let's go back to the channels controller test and make that a unique name.

All Channels Controller tests should pass now!

# Action Items

1. Add a `dependent: :destroy` to memberships and messages in `app/models/channel.rb`
1. Change the names in `test/fixtures/channels.yml` to be unique
1. Fix the create channel test by changing the name to be unique from an existing channel

# This section in the example app

https://github.com/dcsil/rails-tutorial-example/commit/02e6885462341c3afd0511e851590de5245e8540

# Next Section
- [Adding tests: Part 3](19_adding_tests_p3.md)