# Adding Tests (Part 1)

At this point we have a working application. Before we continue addinging functionality, we need to add some tests to ensure that our application is working as expected.

Rails uses Minitest to run tests by default. Minitest is a framework that allows you to write tests in Ruby. Some applications will use Rspec, which is a Ruby testing framework that is similar to Minitest.

For this tutorial, we will use Minitest as it is the default Rails testing framework.

We actually have a bunch of tests already! When we started building the application we used some `bin/rails generate` commands to create some files, which included some tests by default.

Let's start by running `bin/rails test` to run the tests and see what is and isn't working.

> Note: `bin/rails test` is a Rails command that runs all the tests in the `test` directory.
> If you want to run a specific test, you can use `bin/rails test <path>`. And you can run a specific test in that file by appending a colon and line number to the method definition: `bin/rails test <path>:<lineno>`.

# Rexml

This may or may not happen depending on when you run through this tutorial.

In Ruby 3.0, there is a new gem called [Rexml](https://github.com/ruby/rexml). This gem handles XML. Prior to Ruby 3.0, rexml was in the standard Ruby Library so a dependency of Rails (Selenium) did not need to manually include it.

Selenium is a testing framework that allows you to write integrations tests for web applications. It is a wrapper around the browser and allows you to interact with a running test-managed version of the web application in the test process.

You'll know you need to do this because you get an error message when you try to run the tests:
```
[...]/.rbenv/versions/3.0.2/lib/ruby/gems/3.0.0/gems/bootsnap-1.8.1/lib/bootsnap/load_path_cache/core_ext/kernel_require.rb:34:in `require': cannot load such file -- rexml/document (LoadError)
```

How do we fix this? Selenium has a fix for this, but it is not released yet and won't be for version 3.x. We need to use the beta version of Selenium.

In the `group :test` block, add `gem 'rexml'`:
```ruby
group :test do
  # Adds support for Capybara system testing and selenium driver
  gem 'capybara', '>= 3.26'
  gem 'selenium-webdriver', '~> 4.0.0.beta4'
  # Easy installation and use of web drivers to run system tests with browsers
  gem 'webdrivers'
end
```

Now, we can't run just `bundle install` as we are updating a gem, so we need to run `bundle update selenium-webdriver` to update the specific gem. DO NOT RUN `bundle update` as it will update all the gems, which could be disastrous in an already deployed application as many things could break with dozens of version updates.

Tests should now start!

# Fixtures

Fixtures are the default Rails way to inject information into the database so there are instances of all the various objects that we need to test.

Rails uses a special directory called `test/fixtures` to store the fixtures in YAML files. These YAML files are then loaded into the database as SQL. This is an important distinction as the objects in the fixture files _will not run any ActiveRecord callbacks on creation_.

Right now, our fixtures are failing to be loaded. We added some changes to the user model's email column, and the default value of `MyString` is invalid as it is not unique.

```
Error:
ChannelsControllerTest#test_should_destroy_channel:
Mysql2::Error: Duplicate entry 'MyString' for key 'index_users_on_email'
```

Let's open `test/fixtures/users.yml` and change the emails to be unique. I used `one@example.com` and `two@example.com`

# Devise

All of our tests were built without the knowledge of our user authentication system. However, we added a before_action to our `ApplicationController` to authenticate the user. All of our controller tests are failing because of this.

There's not a ton of information about this issue as the response is valid and Rails doesn't know about it. We know, however, that this is an issue because we see errors like this:

```
Failure:
UsersControllerTest#test_should_get_edit [~/test/controllers/users_controller_test.rb:33]:
Expected response to be a <2XX: success>, but was a <302: Found> redirect to <http://www.example.com/users/sign_in>
Response body: <html><body>You are being <a href="http://www.example.com/users/sign_in">redirected</a>.</body></html>
```

There is a whole section in Devise's README on this topic: https://github.com/heartcombo/devise#test-helpers

The gist is that we need to add `include Devise::Test::IntegrationHelpers` to all of our controller tests and integration tests.

In `test/controllers/*_controller_test.rb` add `include Devise::Test::IntegrationHelpers` to the top of the class:

```ruby
require "test_helper"

class ChannelsControllerTest < ActionDispatch::IntegrationTest
  include Devise::Test::IntegrationHelpers
  ...
end
```

Next, we need to log into the application using the newly included helper: `sign_in @user`

Within the setup block in each controller test, let's add `sign_in(something_that_makes_sense)`

```ruby
  # messages_controller_test.rb
  setup do
    @message = messages(:one)
    sign_in(@message.sender)
  end
```

```ruby
  # channels_controller_test.rb
  setup do
    @channel = channels(:one)
    sign_in(@channel.creator)
  end
```

```ruby
  # users_controller_test.rb
  setup do
    @user = users(:one)
    sign_in(@user)
  end
```

Let's run the tests again and see if they work.

# Wall of output

Right now if we run tests we will see a wall of errors, particularly ones like `NameError: undefined local variable or method 'new_message_url'`. This is because we changed the nesting of some of the models. If we run a single controller for a model that we didn't nest, we can see it works a bit better. So let's start there:

`bin/rails test test/controllers/users_controller_test.rb`

We have 2 failures.
Let's focus on each individually.

### UsersControllerTest#test_should_create_user

```
Failure:
UsersControllerTest#test_should_create_user [~/src/github.com/dcsil/chat_app/test/controllers/users_controller_test.rb:21]:
"User.count" didn't change by 1.
Expected: 3
  Actual: 2


rails test test/controllers/users_controller_test.rb:20
```

In this case we can see that we expected a new user to be created, but it wasn't. To fix this, I'd throw a debugger into the controller and see what the problem is.

```ruby
  def create
    @user = User.new(user_params)
    debugger
```

Then I would rerun the test: `rails test test/controllers/users_controller_test.rb:20`

In this case, we are still failing AND we aren't hitting the debugger statement. What's going on?

Remember how we added devise? Devise handles registrations itself, so we have a conflicting route.

```
$ bin/rails routes --grep user
...
POST   /users(.:format)               devise/registrations#create
POST   /users(.:format)               users#create
```

The solution here is to remove the create action and associated tests.

In `config/routes.rb`, change `resources :users` to `resources :users, except: [:create]`

In `app/controllers/user_controller.rb`, remove the create action.

In `test/controllers/users_controller_test.rb`, remove the create test.

Great, now let's run user tests again.

`bin/rails test test/controllers/users_controller_test.rb`. Our only failure is now `UsersControllerTest#test_should_destroy_user`

### UsersControllerTest#test_should_destroy_user

```
Error:
UsersControllerTest#test_should_destroy_user:
ActiveRecord::InvalidForeignKey: Mysql2::Error: Cannot delete or update a parent row: a foreign key constraint fails (`chat_app_test-0`.`memberships`, CONSTRAINT `fk_rails_99326fb65d` FOREIGN KEY (`user_id`) REFERENCES `users` (`id`))
    app/controllers/users_controller.rb:53:in `destroy'
    test/controllers/users_controller_test.rb:45:in `block (2 levels) in <class:UsersControllerTest>'
    test/controllers/users_controller_test.rb:44:in `block in <class:UsersControllerTest>'


rails test test/controllers/users_controller_test.rb:35
```

This error is because we have a foreign key constraint. When we delete a user, we end up with a bunch of violated foreign keys in other tables like memberships.

We have a few options here:
- Change the foreign key to allow nil (could be useful for channel creator)
- Cascade destroy all associated objects
- Change the user to be soft deleted

Let's use a mixture to fix this. A user, essentially, has a few associated objects:

- Memberships
- Messages
- Created Channels

We don't want to treat these equally - for example we don't want a whole channel to go away when a user goes away, but we don't care about the user's memberships.

- For memberships, we can add `dependent: :destroy` to the `has_many :memberships` in `app/models/user.rb`
- For channels we can add `dependent: :nullify` to the `has_many :channels` in `app/models/user.rb`
  - We also need to change the database underneath this column.
  - Create a migration with `bin/rails g migration NAME`
  - In the migration change method, add:
    ```ruby
    def change
      remove_foreign_key :channels, :users
      change_column :channels, :user_id, :bigint, null: true
    end
    ```
  - Migrate with `bin/rails db:migrate`
  - Finally, in channels.rb add `optional: true` to the `belongs_to :creator` line

All user conrroller tests now pass.

```
$ bin/rails test test/controllers/users_controller_test.rb
Running via Spring preloader in process 30512
Run options: --seed 18902

# Running:

......

Finished in 16.897910s, 0.3551 runs/s, 0.4143 assertions/s.
6 runs, 7 assertions, 0 failures, 0 errors, 0 skips
```

Let's take a break at this point :)

# Action Items

1. Change `gem 'selenium-webdriver'` in the Gemfile to `gem 'selenium-webdriver', '~> 4.0.0.beta4'` and run `bundle update selenium-webdriver`
1. Open `test/fixtures/users.yml` and change emails to be unique
1. Add `include Devise::Test::IntegrationHelpers` to all of our controller tests and integration tests.
1. In the setup method in each of these test files, add `sign_in(@user)` where @user is replaced with something that makes sense for that file. E.g. `@channel.creator` for channels controller or `@message.sender` for messages controller.
1. In `config/routes.rb`, change `resources :users` to `resources :users, except: [:create]`
1. In `app/controllers/user_controller.rb`, remove the create action.
1. In `test/controllers/users_controller_test.rb`, remove the create test.
1. In `app/models/user.rb`, change the memberships line to `has_many :memberships, dependent: :destroy`
1. In `app/models/user.rb`, change the channels line to `has_many :channels, dependent: :nullify`
1. Run `bin/rails g migration RemoveForeignKeyAndNullConstraintFromChannels`
1. Open the migration and change the method to:
    ```ruby
    def change
      remove_foreign_key :channels, :users
      change_column :channels, :user_id, :bigint, null: true
    end
    ```
1. Run `bin/rails db:migrate`
1. Open `app/models/channel.rb` and add `optional: true` to the `belongs_to :creator` line
1. Done! `bin/rails test test/controllers/users_controller_test.rb` should now pass.

# This section in the example app

https://github.com/dcsil/rails-tutorial-example/commit/69c6d1eb57e2cb2c748a5cdc3fe5582bdd0bd735

# Next Section
- [Adding tests: Part 2](18_adding_tests_p2.md)