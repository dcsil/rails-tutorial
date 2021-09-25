# Adding Tests (Part 3)

Let's focus on `test/controllers/messages_controller_test.rb` now.

`bin/rails t test/controllers/messages_controller_test.rb`

## NameErrors

This outputs a lot of NameErrors like this:

```
NameError: undefined local variable or method `new_message_url' for #<MessagesControllerTest:0x00007fb6bcbf85e0 @_routes=nil, @NAME="test_shoul....
```

Remember how we nested our messages under channels, we need to update the urls to reflect this.

Change all the `message_path` calls to `channel_message_path` calls, `message_url` calls to `channel_message_url` calls, `new_message_url` calls to `new_channel_message_url` calls, etc.

We also need to add a channel to the urls. So `channel_message_path(@message)` would become `channel_message_path(@channel, @message)`.

We can get @channel in the setup method by adding `@channel = @message.recipient` after the @message initialization.

Let's run our the tests again.

## Fixtures

Now we see errors that look like this:

```
MessagesControllerTest#test_should_show_message:
NameError: uninitialized constant Recipient
    test/controllers/messages_controller_test.rb:8:in `block in <class:MessagesControllerTest>'
```

This took me a moment to realize, however our fixtures file `test/fixtures/messages.yml` has set all the recipient_type to `Recipient`s, which doesn't exist. Remove the `recipient_type` rows and add suffix the `recipient` line with ` (Channel)` which hints to Rails that we want to use a channel.

## Non-existant routes

Remember that we disabled a lot of the messages routes? We need to reflect that in the tests, otherwise we'll get a `No route matches` errors.

Delete everything except the create, update, and destroy tests.

## Changed functionality

We changed a few things in our messages controller:

- We only respond to JSON requests
- We don't allow recipient or sender to be passed in

Let's accommodate those!

### JSON requests

We don't redirect anymore, so let's remove those `assert_redirected_to` calls and replace them with `assert_response :success` calls (or `assert_response :created` for the create action).

### No recipient or sender

Change params to include only content. Both create and update should have `params: { message: { content: "content" } }`

Let's make sure things are assigned properly as well. In create's test add:
```ruby
message = Message.last
assert_equal @channel, message.recipient
assert_equal @message.sender, message.sender
```

In update's test add:
```ruby
assert_equal @channel, @message.reload.recipient
assert_equal @message.sender, message.reload.sender
```

.reload is necessary because we changed the message and we must tell Rails to reload it all.

Tests should now pass for everything!

# Action Items

1. Update the URLs to reflect that we nested messages under channels
1. Remove `recipient_type` lines from `test/fixtures/messages.yml`
1. Add ` (Channel)` suffices to recipient lines in `test/fixtures/messages.yml`
1. In `test/controllers/messages_controller_test.rb`, delete everything except the create, update, and destroy tests.
1. In `test/controllers/messages_controller_test.rb`, add `params: { message: { content: "content" } }` to create and update tests, removing the existing params
1. In `test/controllers/messages_controller_test.rb`, remove `assert_redirected_to` calls and replace them with `assert_response :success` calls (or `assert_response :created` for the create action).
1. Add this after the create POST action in the create test:
    ```ruby
    message = Message.last
    assert_equal @channel, message.recipient
    assert_equal @message.sender, message.sender
    ```
1. In update's test add:
    ```ruby
    assert_equal @channel, @message.reload.recipient
    assert_equal @message.sender, message.reload.sender
    ```
1. All tests now pass!

# This section in the example app

https://github.com/dcsil/rails-tutorial-example/commit/55531e954be32c9ba2701f920bb3ae8081ddd213

# Next Section
- [Adding tests: Part 4](20_adding_tests_p4.md)