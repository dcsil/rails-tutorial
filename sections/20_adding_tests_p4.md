# Adding Tests (Part 4)

We have gotten all existings tests to pass, but we have not yet written any new tests. Let's work on that.

We have added some functionality in various places and will write tests for that in this section:

- `channel.member?` method
- Memberships controller
- Redirects to sign in page when not logged in

# Channel Member? method

This is a method that checks if a given channel has a given member. In the `test/models/channel_test.rb` file, let's add 2 tests for this method:

  ```ruby
  setup do
    @channel = channels(:one)
  end

  test "should return true if the channel has the given member" do
    member = @channel.members.first
    refute_nil member
    assert @channel.member?(member)
  end

  test "should return false if the channel does not have the given member" do
    member = User.where.not(id: @channel.members.pluck(:id)).first
    refute_nil member
    refute @channel.member?(member)
  end
  ```

# Memberships Controller

We only have one method in the `MembershipsController`: `destroy`. Let's add a test for that. First, let's set up the controller:

```ruby
setup do
  @membership = memberships(:one)
  @user = @membership.user
  @channel = @membership.channel
  sign_in(@user)
end
```

Rails adds some helpers, like `memberships` that refer to the test fixture files. `:one` refers to the YAML key in that file, which could be any string you want.

Let's add a fake test to make sure our set up works:
```ruby
  test "destroy discards the membership" do
    assert true
  end
```

It does! Let's add a test for the `destroy` method:

Normally we could just write this:
```ruby
assert_difference('Membership.count', -1) do
  delete channel_membership_url(@channel, @membership)
end
```
which would assert that we have one less membership after calling it, but we soft delete it. So we need to make sure we use `Membership.kept.count`!

If you don't like using a string in the assert_difference call, it also takes a proc.

Let's also assert we are redirected to the channels url since the channel url would cause a new membership to be created.

On that note, let's also `assert_no_difference('Membership.count')` to make sure we don't create a new membership by mistake by redirecting to the channel. We can ensure we follow redirects with `follow_redirect!` just to make sure we don't create a new membership by mistake.

The whole test should be:

```ruby
test "destroy discards the membership" do
  assert_no_difference("Membership.count") do
    assert_difference('Membership.kept.count', -1) do
      delete channel_membership_url(@channel, @membership)
    end
    assert_redirected_to channels_url
    follow_redirect!
  end
end
```

# Not Logged In Redirect

We want to make sure we dont allow anyone to access any endpoint without being logged in. We can accomplish this by asserting we are redirected to the sign in page when we are not logged in.

Let's use the memberships controllr endpoint as an example:

```ruby
test "destroy redirects to sign in with not logged in" do
  sign_out(@user) # Another devise helper. We are signed in in the setup, so we need to sign out

  # Assert no differnece in the membership count or the kept count (eg nothing changed)
  assert_no_difference("Membership.count") do
    assert_no_difference('Membership.kept.count') do
      delete channel_membership_url(@channel, @membership)
    end

    # Assert redirected to /users/sign_in
    assert_redirected_to new_user_session_path
  end
end
```

You can repeat this for other endpoints as well if you want.

# Action Items

1. Add tests for the `channel.member?` method in `test/models/channel_test.rb`
2. Add tests for the `MembershipsController` in `test/controllers/memberships_controller_test.rb`
3. Add tests to check that if we are not logged in, we redirect to sign in page in `test/controllers/memberships_controller_test.rb` (and the others if you want)
4. Done!

# This section in the example app

https://github.com/dcsil/rails-tutorial-example/commit/2e0f15836163badafad63afa050cc6f475061bb3
