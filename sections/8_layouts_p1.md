# Layouts

Right about now you should have a basic layout that isn't really very interesting. It also doesn't work too well for a chat application.

This section will cover how to make a better layout. We will introduce a sidebar for channels and leave the main content for messages.

This section will be focused mostly on HTML and not a lot of Rails/Ruby work, but it is necessary to continue.

# Layouts

Rails uses a "layout" to lay the foundations for each page view. If you look at any of our views, say `app/views/users/show.html.erb`, you'll see that it is an incomplete HTML page.

By default Rails will lay everything out in `app/views/layouts/application.html.erb`. This is the layout that will be used for all pages unless otherwise overridden. The content that was in `app/views/users/show.html.erb` will be placed in the yield tag (`<%= yield %>`) in the layout.

We can therefore modify `app/views/layouts/application.html.erb` to change the layout for all pages.

# The changes

We want a sidebar on the left and a main content area on the right. Let's look to the Primer Grid: [Primer Grid](https://primer.style/css/objects/grid).

First, let's make sure the html and body elements use 100% of the height of the screen. Open `app/javascript/packs/application.scss` and add the following:

```css
html, body {
  height: 100%;
}
```

Next, open `app/views/layouts/application.html.erb` and replace the body contents with:
```erb
<div class="d-flex height-full">
  <div class="col-2 p-2 color-bg-canvas-inverse color-text-inverse">
    CHANNELS GO HERE
  </div>
  <div class="col-10 p-2">
    <%= yield %>
  </div>
</div>
```

When you reload, you should have something like this:

<img src="../images/8_primer_layout.png" alt="Layout you should expect when you reload. The dark sidebar is on the left, the light main content is on the right" height="500">

# Action Items

1. Add the following to `app/javascript/packs/application.scss`:
    ```css
    html, body {
      height: 100%;
    }
    ```
1. In `app/views/layouts/application.html.erb`, replace the body contents with:
    ```erb
    <div class="d-flex height-full">
      <div class="col-2 p-2 color-bg-canvas-inverse color-text-inverse">
        CHANNELS GO HERE
      </div>
      <div class="col-10 p-2">
        <%= yield %>
      </div>
    </div>
    ```
1. Run `bin/rails s` and `bin/webpack-dev-server`, then reload the page.

# This section in the example app

https://github.com/dcsil/rails-tutorial-example/commit/67cc3ec561e83b23e3df4b4cad448819392af83d

# Next Section
- [Layouts: Part 2](9_layouts_p2.md)