# Introduction: Ruby, Bundler, RubyGems, and rbenv

Ruby on Rails is a framework based on the Ruby programming language. To truly understand how Rails works, it's important to understand how Ruby works, particularly on your system.

This first lesson is going to cover Ruby, Bundler, and Ruby version managers.

## Ruby

Ruby is a programming language developed in the 1990s by Yukihiro Matsumoto. It is a dynamic, interpreted, object-oriented, general-purpose programming language. It was designed to make programmers happy.

That last sentence is the most important thing to understand about Ruby. Every decision that goes into the Ruby language- the syntax, standard library, etc.- is decided with the developer in mind. It is designed to be first and foremost enjoyable to use.

Ruby is a dynamic language, which means that it is able to interpret code at runtime rather than beforehand. This means that it is able to change itself to fit the needs of the program, such as introspecting (looking at) itself. This can also make it difficult to know what the code will do before it is run if you are not careful.

## Bundler

Dependencies are the packages of code that are required to run a program and provide functionality bundled up in a nice little package. Bundler is a tool that helps you manage your dependencies. With Bundler you can specify which dependencies you want to use, and it will install them for you and make sure they are available for you to use. We call these dependencies gems.

Gems are installed from a file called `Gemfile`. This is a Ruby file that tells Bundler what gems and version ranges you want to use, while `Gemfile.lock` tells Bundler what specific versions of those gems you want to use.

## RubyGems

Bundler is a part of RubyGems, while Rubygems is a part of Ruby.

RubyeGems is an index of packages that are available for you to use. You can see these at rubygems.org. You can install gems using Bundler, or individually using `gem install`.

## Ruby Version Managers

Ruby version managers are tools that help you manage and install your Ruby versions. There are a lot of them, but the most common ones are rbenv, chruby, and rvm.

# Node and Nodenv

Node is a JavaScript runtime. It allows us to run JS code, and work with it in our app. Nodenv was created to help manage Node versions, just like Rbenv for Ruby.

We will use both for the JS in our app.

Yarn is an alternative package manager for JavaScript. It is a bit like NPM. Rails uses this.

# Action Items

1. If you are on Mac, run `brew list ruby`. If Ruby is installed via Homebrew, run `brew uninstall ruby`. This will conflict with Rbenv.
1. [Install Rbenv](https://github.com/rbenv/rbenv#installation)
1. Install Ruby 3.0.2: Assuming you have Rbenv installed, run `rbenv install 3.0.2`.
1. Run `rbenv rehash` to make sure the new version is loaded.
1. Run `rbenv global 3.0.2` to set the version of Ruby to use. This will set the default Ruby version to 3.0.2 and will be the default version used from now on unless overridden with a local `.ruby-version` file.
1. Install Bundler: Assuming you have Ruby 3.0.2 installed, run `gem install bundler`.
1. Install [Nodenv](https://github.com/nodenv/nodenv#installation)
1. Install a recent version of Node.js: Run `nodenv install 16.6.1`.
1. Set it globally: Run `nodenv global 16.6.1`.
1. We also need to install [Yarn](https://yarnpkg.com/en/docs/install). Run `npm install -g yarn`.
1. Done!

# Next Section
- [Starting with Rails](1_starting_with_rails.md)