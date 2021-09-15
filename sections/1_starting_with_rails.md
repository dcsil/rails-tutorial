# Starting with Rails

Rails is a web application framework written in Ruby under the MIT license. It was created by David Heinemeier Hansson in January 2006 and released as open source on January 21, 2008. It is used to create dynamic web sites, with a focus on simplicity and productivity. It is the most popular web framework for Ruby.

It is based on the concept of a model–view–controller (MVC) architecture, which is a design pattern for separating the logic of a program into separate components. Models hold the data, views display the data, and controllers handle the data and the user interaction.

<img alt="MVC Architecture Diagram" src="../images/mvc.png" height="400"><br>
(From freecodecamp.org)

Ruby on Rails is distributed as a RubyGem. You can install it with the command: `gem install rails`. This will install the latest version. We do not want to use Bundler here because we would like to use an executable `rails` command to generate a new Rails application, including the Gemfile. Everything else we use will be installed by the Gemfile.

We are now going to generate a new Rails application. We will use the command `rails new my_app`. This will create a new directory called `my_app` in the current directory with a default rails app inside. NOTE: we do not want to run this command by itself. Rails uses a lot of settings and we want to set the database as we generate our app. Make sure to run this command with the flag `--database=mysql`.

# Action Items

1. Install Rails: `gem install rails`
1. Generate a new Rails application:
   `rails new chat_app --datbabase=mysql`
1. Bundle install: `bundle install`. This may take some time as it has to install some gems, compile some C extensions and install JavaScript dependencies.
1. Run Rails: `bin/rails s`
1. Hit the root URL: `http://localhost:3000`
1. Done! You should see a welcome message.

# Troubleshooting

### MySQL Problems

You may see the following error:

```
ld: library not found for -lssl
...
An error occurred while installing mysql2 (VERSION), and Bundler cannot continue.
Make sure that `gem install mysql2 -v 'VERSION' --source 'https://rubygems.org/'` succeeds before bundling.
```

This is failing due to OpenSSL. You can fix this by running `gem install mysql2 --source 'https://rubygems.org/' -- --with-ldflags=-L/usr/local/opt/openssl/lib --with-cppflags=-I/usr/local/opt/openssl/include`, then rerunning `rails new chat_app --datbabase=mysql`. Select `Y` to override conflicts as we haven't made any changes yet.
