# User Authentication

Now that we have a Rails app with a User model, we want to be able to log in and out of the app. This is important to let us know who is currently using the application, and to make sure that we don't have any unauthorized access to the application.

User authentication can happen in various ways, the simplest of which is password authentication. While we can easily add an encrypted password to our User model, this is simply the data model (the M part of MVC). We still need views to be able to input an email and password, controllers and logic to authenticate, and other mechanisms like forgot password.

Writing your own authentication logic is far more involved and it's easy to miss something and leave your application vulnerable. For this reason we will not be writing our own.

Devise is a standard Ruby gem that provides a framework for authentication. It's a framework that provides a lot of the logic for authentication, including the ability to create a user, log in, and log out. It also provides a lot of the logic for password reset, email confirmation, and more. We will use this.

## Installation

Let's install Devise. [The full details are located here](https://github.com/heartcombo/devise), however let's jump right into it.

First, we need to add the gem to our Gemfile. Add `gem "devise"` to the Gemfile. You should not add it within a "group" block as we need it in all environments. Run `bundle install` to install the gem.

Second, Devise adds its own generator to the Rails application. Run `bin/rails generate devise:install` to install Devise.

Third, we need to install Devise to our User model. Run `bin/rails generate devise User` to install Devise to the User model. In a previous data model migration we added the email column. We need to comment that line out so we don't try to add it again. Open the generated migration and comment out the line `t.string :email`. Run `bin/rails db:migrate` to add the new columns.

When we added the email, however, we did not have the same contraints that Devise added. Namely we were missing `null: false` (disallow nulls) and `default: ""` (default value to ""). We can add these back byrunningRun `bin/rails g migration AddConstraintsToEmail` and adding this to the migration:
```ruby
 class AddConstraintsToEmail < ActiveRecord::Migration[6.1]
   def change
     change_column_null :users, :email, false
     change_column_default :users, :email, ''
   end
 end
```

Finally, we need to add `before_action :authenticate_user!` to the top of the ApplicationController class definition. This will ensure that all requests to the application will require authentication.

### Helpers

Devise, among other things, adds some helper methods to your Controllers and Views. For example:
- `user_signed_in?` returns true if the user is signed in, false otherwise.
- `current_user` returns the current signed in user.
- `user_session` returns the current user's session.

## Running the App

Let's start the rails server again with `bin/rails s`. When you navigate to http://localhost:3000/users, you should see a sign in page.

Our user from last time doesn't have a password, so we can't log in. Let's manually fix that for now in the _Rails console_.

Run `bin/rails console` in a new terminal tab. Run the following:

```ruby
user = User.first
user.password = '123456!'
user.password_confirmation = '123456!'
user.save!
```

Back in the server tab, try to log in. This should let you through to the users table.

# Action Items

1. Install the Devise gem by adding `gem "devise"` to the Gemfile.
2. Run `bundle install` to install the gem.
3. Run `bin/rails generate devise:install` to install Devise to the App.
4. Run `bin/rails generate devise User` to install Devise to the User model. **Note: We already had an email column, so remove that from the generated migration.** Comment out the line that starts with `t.string :email`
5. Run `bin/rails g migration AddConstraintsToEmail` and add:
   ```ruby
    class AddConstraintsToEmail < ActiveRecord::Migration[6.1]
      def change
        change_column_null :users, :email, false
        change_column_default :users, :email, ''
      end
    end
   ```
5. Run `bin/rails db:migrate` to add the new columns.
6. Add `before_action :authenticate_user!` to the top of the ApplicationController class definition:
   ```ruby
   class ApplicationController
     before_action :authenticate_user!
     ...
   end
   ```
7. Run `bin/rails console` and run the following:
    ```ruby
    user = User.first
    user.password = '123456!'
    user.password_confirmation = '123456!'
    user.save!
    ```
8. Run the server with `bin/rails s`.
9. Navigate to http://localhost:3000/users.
10. Try to log in. This should let you through to the users table once you use the correct email and password.

# This section in the example app

https://github.com/dcsil/rails-tutorial-example/commit/96839eb0e205401a750cb856fdccf5e856dac1ff

# Next Section
- [Data model: Part 2](6_data_model_p2.md)