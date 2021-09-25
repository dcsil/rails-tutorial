# Navigating your Rails App

Rails applications follow convention over configuration. This means that certain things are expected to always be done a certain way. While you can configure it to do something else, you should have a good reason to do so as it makes it more difficult for others to start working on your application.

One of these conventions has to do with the folder structure. Rails applications are organized into the same general top level folders, as well as the following subfolders. This will help you navigate your application and be true for all Rails applications that follow the standard convention.

- `app/`
  - This is the main folder for your application. It contains all of the code for your application.
  - `controllers/`
  - `helpers/`
  - `models/`
  - `views/`
  - `mailers/`
- `config/`
  - This is the main folder for your application's configuration. It contains all of the code for your application's configuration, credentials (encrpypted!), routes, etc
  - `environments/`
  - `credentials/`
  - `initializers/`
  - `routes.rb`
  - `application.rb`
  - `database.yml`
- `db/`
  - This folder contains your application's database setup, including the current schema and migrations.
  - `schema.rb` OR `structure.sql`
  - `seeds.rb`
  - `migrate/`
- `lib/`
  - This folder contains code that will not be reloaded automatically. This is a good spot to put things loaded before your app is booted, like middleware (code that runs in the webserver) and Rake tasks
  - `tasks/`
  - `middleware/`
- `log/`
  - Development and Production logs are stored here.
- `public/`
  - Public files, such as images, stylesheets, error pages, and javascripts, can be stored here.
  - `assets/`
- `test/`
  - This folder contains your application's test code. By default Rails uses Minitest, though many apps use Rspec. We will use Minitest in this tutorial.
  - `controllers/`
  - `helpers/`
  - `models/`
  - `mailers/`
  - `integration/`
  - `test_helper.rb`
- `vendor/`
  - Vendored code will be in here. We won't be using this folder in this tutorial, but some apps will install and version control their gems in this folder.
- `tmp/`
  - This folder will contain temporary files that may be deleted at any time. The most important subfolder is the `pids` folder which will contain the PIDs of the processes that are running your application.
  - `pids/`

# Action Items

There are no action items in this section other than reading the structure above and understanding it.

# Next Section
- [Project Introduction](3_project_introduction.md)