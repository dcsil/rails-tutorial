# Add Primer

Up to this point, we've been using the default `scaffold.scss` file that was generated. This is fine for initial work, but it's not very useful for a real application.

We will use [Primer](https://primer.style/). Primer is a CSS framework that provides a set of default styles for a variety of elements. We will use Primer to create our own styles. It is developed by GitHub.

## Installing Primer

[Full instructions can be found here](https://primer.style/css/getting-started/).

Rails uses [Webpacker](https://github.com/rails/webpacker) to manage the assets. Webpacker is a wrapper around Webpack, a JS tool used to manage assets.

This means, however, that we can use the standard JS tooling to manage our assets.

Let's install primer with `npm install @primer/css --save`.

# Enabling Primer in our App

### JS and SCSS Changes

Create the following file: `app/javascript/packs/application.scss`. Inside add the following line:

```js
@import "@primer/css/index.scss";
```

Next, open `app/javascript/packs/application.js` and add the following line:
```js
import "./application.scss"
```

These will create a JS file that imports the `application.scss` file, which will import primer.

### HTML Changes

The application.js file is the entry point for our application's JS. Rails will require it using Webpacker in our layout `app/views/layouts/application.html.erb`.

You can see these lines in that file:
```erb
<%= stylesheet_link_tag 'application', media: 'all', 'data-turbolinks-track': 'reload' %>
<%= javascript_pack_tag 'application', 'data-turbolinks-track': 'reload' %>
```

The former will load the `stylesheets/application.scss` file, and the latter will load the `app/javascript/packs/application.js` file.

Add `<%= stylesheet_pack_tag 'application', 'data-turbolinks-track': 'reload' %>` after those lines. This will load CSS generated from the application.js file.

Note: There is an older method called `javascript_link_tag` to load in a specific JS file from under `app/assets/javascripts`, but the packs files generally replaced that.

### Run the Server

If you run `bin/rails s`, everything should load just the same. This is because we have only installed it and not started using Primer yet.

Now that we are using our assets, it is a good idea to also run the Webpacker server alongside Rails. Run `bin/webpack-dev-server`. This will start a server that will automatically reload whenever you make a change to your JS, and live reload your application.

## Primer Styles

First, we need to get rid of the old scaffolds.scss file. Run `rm app/assets/stylesheets/scaffolds.scss`, as well as the channels.scss and user.scss files (these are empty).

Next, our channels and users lists are being displayed in a table. We will use Primer instead, which has no Table styles. We will modify them to use a Box: https://primer.style/css/components/box#box-row.

### Table => Box

Open up `app/views/users/index.html.erb`. You should see a table in there:
```erb
<table>
  <thead>
    <tr>
      <th>Name</th>
      <th>Email</th>
      <th>pronouns</th>
      <th>Bio</th>
      <th colspan="3"></th>
    </tr>
  </thead>

  <tbody>
    <% @users.each do |user| %>
      <tr>
        <td><%= user.name %></td>
        <td><%= user.email %></td>
        <td><%= user.pronouns %></td>
        <td><%= user.bio %></td>
        <td><%= link_to 'Show', user %></td>
        <td><%= link_to 'Edit', edit_user_path(user) %></td>
        <td><%= link_to 'Destroy', user, method: :delete, data: { confirm: 'Are you sure?' } %></td>
      </tr>
    <% end %>
  </tbody>
</table>
```

This table has a static head, but a dynamic body. We will get rid of the table layout and make a list.

It should now look like this:

```erb
<p id="notice"><%= notice %></p>

<h1>Users</h1>

<div class="Box">
  <ul>
    <% @users.each do |user| %>
      <li class="Box-row">
        <div><%= user.name %></div>
        <div><%= user.email %></div>
        <div><%= user.pronouns %></div>
        <div><%= user.bio %></div>
        <div><%= link_to 'Show', user %></div>
        <div><%= link_to 'Edit', edit_user_path(user) %></div>
        <div><%= link_to 'Destroy', user, method: :delete, data: { confirm: 'Are you sure?' } %></div>
      </li>
    <% end %>
  </ul>
</div>

<br>

<%= link_to 'New User', new_user_path %>
```

Complete the same thing for Channels.

We have now enabled Primer and can start using it!

# Action Items

1. Run `npm install @primer/css --save`
1. Run `touch app/javascript/packs/application.scss`
1. Run `echo "@import '@primer/css/core/index.scss';" >> app/javascript/packs/application.scss`
1. Run `echo "import './application.scss';" >> app/javascript/packs/application.js`
1. Add `<%= stylesheet_pack_tag 'application', 'data-turbolinks-track': 'reload' %>` after the `javascript_pack_tag` in `app/views/layouts/application.html.erb`
1. Delete `app/assets/stylesheets/scaffolds.scss`, `app/assets/stylesheets/channels.scss`, and `app/assets/stylesheets/user.scss`
1. Open `app/views/users/index.html.erb` and replace the table with:
    ```erb
    <div class="Box">
      <ul>
        <% @users.each do |user| %>
          <li class="Box-row">
            <div><%= user.name %></div>
            <div><%= user.email %></div>
            <div><%= user.pronouns %></div>
            <div><%= user.bio %></div>
            <div><%= link_to 'Show', user %></div>
            <div><%= link_to 'Edit', edit_user_path(user) %></div>
            <div><%= link_to 'Destroy', user, method: :delete, data: { confirm: 'Are you sure?' } %></div>
          </li>
        <% end %>
      </ul>
    </div>
    ```
1. Open `app/views/channels/index.html.erb` and replace the table with:
    ```erb
    <div class="Box">
      <ul>
        <% @channels.each do |channel| %>
          <li class="Box-row">
            <div><%= channel.name %></div>
            <div><%= channel.description %></div>
            <div><%= channel.user_id %></div>
            <div><%= link_to 'Show', channel %></div>
            <div><%= link_to 'Edit', edit_channel_path(channel) %></div>
            <div><%= link_to 'Destroy', channel, method: :delete, data: { confirm: 'Are you sure?' } %></div>
          </li>
        <% end %>
      </ul>
    </div>
    ```
1. Run `bin/rails s` AND `bin/webpack-dev-server` (we run both now).
1. Open http://localhost:3000/users and see the new Primer styles.

# This section in the example app

https://github.com/dcsil/rails-tutorial-example/commit/4487da46996fe22d7750b52ffa396937418bc9b8

# Next Section
- [Layouts: Part 1](8_layouts_p1.md)