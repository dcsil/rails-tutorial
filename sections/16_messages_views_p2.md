# Messages: Views (Part 2)

Now that we have our models, and a controller (with views), we need to think a bit more about how these will be made.

For example, we have a `index`, `show`, `new`, and `edit` action for messages, but messages are never made independently or shown through this controller. They are always made and edited on the channel show page, so let's get rid of them.

## Remove some actions

In the `messages_controller.rb` file, we can see that we have an `index`, `show`, `new`, and `edit` action. We can remove them by deleting the lines.

```ruby
# Delete these
def index
  @messages = Message.all
end

def show
end

def new
  @message = Message.new
end

def edit
end
```

Next, we need to restrict the routes to not include these actions. Open `config/routes.rb` change `resources :messages` to `resources :messages, only: [:create, :update, :destroy]`.

Let's also take this time to nest messages under channels. Your full routes should look something like this:
```ruby
# config/routes.rb
Rails.application.routes.draw do
  resources :channels do
    resources :messages, except: [:create, :update, :destroy]
    resources :memberships, only: :destroy
  end

  devise_for :users
  resources :users
end
```

Next, delete the views for the ERB templates as we wont use them. Run: `rm app/views/messages/*.html.erb`.

Finally, we won't be hitting this controller for HTML. We will be exlusively using AJAX to make requests to the server. Rails tries to be helpful by providing html and json responses, but we can get rid of the HTML ones. For example this snippet:

```ruby
def update
  respond_to do |format|
    if @message.update(message_params)
      format.html { redirect_to @channel }
      format.json { render :show, status: :ok, location: @message }
    else
      format.html { render :edit, status: :unprocessable_entity }
      format.json { render json: @message.errors, status: :unprocessable_entity }
    end
  end
end
```

Can become:

```ruby
def update
  if @message.update(message_params)
    render :show, status: :ok, location: [@channel, @message], formats: :json
  else
    render json: @message.errors.full_messages.to_sentence, status: :unprocessable_entity
  end
end
```

You'll notice we got rid of the `respond_to` block which is now unnecessary. We kept the json responses. Any of the `render :show` blocks need `formats: :json` to be added, otherwise they may try to render non-json responses

`location` determines the URL location of the object. In this case, `@message` is not enough as message is nested under channel, so we should use `@channel` as well.

We will also make a change to `@message.errors` and make it `@message.errors.full_messages.to_sentence`. This will give us a more readable error message without any work on the front end.


## Updating Messages Controller with Channel

We just nested messages under channels, but we also need to update the messages controller to use the channel.

Just like we did with memberships, we can add `before_action :set_channel` to the top of the controller. Make sure this is the first line as `before_action`s are called in order, and we will want to use the channel in `set_message`

Next, add our `set_channel` method to the bottom of the controller in the private section.

```ruby
def set_channel
  @channel = Channel.find(params[:channel_id])
end
```

Now we can change the `set_message` to use the channel.

```ruby
def set_message
  @message = @channel.messages.find(params[:id])
end
```

This covers most of the endpoints, except for `create`. We can change `Message.new` to `@channel.messages.build` to instantiate a message on the channel:

```ruby
def create
  @message = @channel.messages.build(message_params)
  ...
end
```

If we wanted to save it immediately you could call `channel.messages.create(message_params)` instead, but that would return true/false and we need the message object for error handling later.

## Adding more to the controller

We need to assign the channel and current_user to the message as the recipient and sender respectively. This will apply in the create method:

```ruby
def create
  @message = @channel.messages.build(message_params)
  @message.sender = current_user
  @message.recipient = @channel

  if @message.save
    render :show, status: :created, location: @message, formats: :json
  else
    render json: @message.errors, status: :unprocessable_entity
  end
end
```

We also need to pare down the message_params method. Rails includes something called "Strong Params" which allows us to restrict _which parameters_ are allowed. In this case, the model could accept the sender_id, but we never want a request to be able to change it to something other than the current user. So let's remove everything the user shouldn't be able to change.

```ruby
def message_params
  params.require(:message).permit(:content)
end
```

## Channel Show

A standard chat application usually has a side bar of channels, and a main chat area with a chat input at the bottom. It looks something like this:
```
_________________________________________________________
|                 |                                     |
|                 |                                     |
|                 |                                     |
|                 |                                     |
|                 |                                     |
|                 |                                     |
|                 |_____________________________________|
|                 |                                     |
|                 |                                     |
_________________________________________________________
```

We have that sidebar right now, but we need to change the chat show page to have a chat box at the bottom. We can leverage Primer and Flexbox to get this layout. In show.html.erb, add the following:

```erb
<div class="d-flex flex-column height-full">
  <div class="flex-shrink-0 color-bg-canvas-inverse color-text-inverse p-2 d-flex flex-justify-between flex-items-center">
    <div>
      <h1><%= @channel.name %></h1>
      <p><%= @channel.description %></p>
    </div>
    <div>
      <% if current_user == @channel.creator %>
        <%= link_to 'Edit', edit_channel_path(@channel), class: "btn btn-sm" %>
      <% end %>
      <%= link_to "Leave Channel", channel_membership_path(@channel, @active_membership), class: "btn btn-sm btn-danger", method: :delete, data: { confirm: "Are you sure you want to leave #{@channel.name}?" } %>
    </div>
  </div>
  <div style="overflow: scroll;" class="flex-auto p-3">
    <p id="notice"><%= notice %></p>
    <% @channel.messages.each do |message| %>
      <p><%= message.content %>
    <% end %>
  </div>
  <div style="height: 175px" class="color-bg-canvas-inverse color-text-inverse p-3">
   MESSAGE BOX
  </div>
</div>
```

In `app/views/layouts/application.html.erb` we will touch up the CSS to work better with the channels show page. Change the `nav` to:
```erb
<nav class="col-2 pt-5 m-0 color-bg-canvas-inverse color-text-inverse menu border-0 rounded-0" aria-label="Channels">
```

And remove the padding on the `col-10`:
```erb
<div class="col-10">
  <%= yield %>
</div>
```

# Message Jbuilder

Due to our nesting of messages under channels, we need to update the `app/views/messages/_message.json.jbuilder`. This is the file we will render for JSON responses.

Change the message_url to the channel_message_url:

```ruby
json.url channel_message_url(message.recipient, message, format: :json)
```

# The message form

Now that we have a layout, we need to add a form to the chat show page. We can do this by adding a form to the bottom of the page where we have `MESSAGE BOX`.

Rails comes with form helpers built in, and we will use those. [Documentation](https://guides.rubyonrails.org/form_helpers.html).

Our form is going to be very basic. It will have the trix editor for the rich text and a submit button.

`form_with` can take a variety of options, but we will only use the `:model` option. Rails will handle the rest.

```erb
<%= form_with model: [@channel, @channel.messages.build] do |form| %>
<% end %>
```

By providing the model option with an array `[@channel, @channel.messages.build]`, we are telling Rails that we want to use the `@channel` and a newly built message. Because the message is newly built, Rails will know to send this to the create method.

We can use the `form.rich_text_area` helper to create a rich text area. We can also use the `form.submit` helper to create a submit button:

```erb
<%= form_with model: [@channel, @channel.messages.build] do |form| %>
  <%= form.rich_text_area :content, class: "color-bg-canvas color-text-primary p-1" %>
  <%= form.submit "Send Message", class: "btn btn-sm mt-2" %>
<% end %>
```

This will know to submit the form to the create option and redirect/render the response.

# AJAX

Rendering the response isn't a great solution. We don't want to see some JSON response, we want to see the page update. We can use AJAX to do this.

First, add `local: false` to the form_with:
```erb
<%= form_with model: [@channel, @channel.messages.build], local: false do |form| %>
  <%= form.rich_text_area :content, class: "color-bg-canvas color-text-primary p-1" %>
  <%= form.submit "Send Message", class: "btn btn-sm mt-2" %>
<% end %>
```

Next, if you submit a message like this - nothing will happen but you will see the changes in the console.

Rails has some built in AJAX helpers we can use `form_with` and `local: false`. On success, the form will get the `ajax:success` event and on failure, the `ajax:error` event. We can listen to these events and do something with the response.

Open `app/javascript/packs/application.js` and add the following:

```js
document.addEventListener("turbolinks:load", () => {
  const forms = document.querySelectorAll('form[data-remote="true"]');
  forms.forEach(form => {
    form.addEventListener("ajax:success", (event) => {
      window.location.reload();
    });
    form.addEventListener("ajax:error", (event) => {
      alert(event.detail[0]);
    });
  });
});
```

`document.addEventListener("turbolinks:load", () => {...})` is an event to listen for the document to be finished loading. In other apps you may listen to the `ready` event, but Rails uses Turbolinks. Turbolinks is a library that makes pages snappier by doing some HTML diffing and rerendering only parts of the page that have changed. What this means is that we can't listen to the ready event and must use the `turbolinks:load` event.

Then we simply go through all the forms with `data-remote="true"` and add the event listeners. Forms will have `data-remote="true"` if they are using `local: false`

This simple JS will reload the page on success, and otherwise show an alert if there's an error. It's basic, but the first step to getting AJAX to work.

# Submitting a message

At this point you shoudl be able to submit a message! You can even add text styling and upload attachments (thanks ActiveText!). The images won't load properly though unless you add a gem first, so let's do that!

Add `gem 'image_processing', '~> 1.2'` to your Gemfile and run `bundle install`.

Now submit a message and let it load!

# Action Items

1. In the `messages_controller.rb` file, delete the `index`, `show`, `new`, and `edit` actions
1. Open `config/routes.rb` change `resources :messages` to `resources :messages, only: [:create, :update, :destroy]`.
1. Delete views we dont need with `rm app/views/messages/*.html.erb`
1. In the `messages_controller.rb` file, remove the `respond_to` blocks - keeping the json responses only. Here's an example:
    ```ruby
    def update
      respond_to do |format|
        if @message.update(message_params)
          format.html { redirect_to @channel }
          format.json { render :show, status: :ok, location: @message }
        else
          format.html { render :edit, status: :unprocessable_entity }
          format.json { render json: @message.errors, status: :unprocessable_entity }
        end
      end
    end
    
    # Can become

    def update
      if @message.update(message_params)
        render :show, status: :ok, location: [@channel, @message], formats: :json
      else
        render json: @message.errors.full_messages.to_sentence, status: :unprocessable_entity
      end
    end
    ```
1. In `messages_controller.rb`, change `@message.errors` to `@message.errors.full_messages.to_sentence`
1. Add `before_action :set_channel` to `messages_controller.rb`
1. Add the `set_channel` method to `messages_controller.rb`
    ```ruby
    def set_channel
      @channel = Channel.find(params[:channel_id])
    end
    ```
1. Update create to use `@channel.messages` instead of `Message`:
    ```ruby
    def create
      @message = @channel.messages.build(message_params)
      ...
    end
    ```
1. Update index to use `@channel.messages` instead of `Message.all`:
    ```ruby
    def index
      @messages = @channel.messages
    end
    ```
1. Update `message_params` to:
    ```ruby
    def message_params
      params.require(:message).permit(:content)
    end
    ```
1. Explicitly set recipient and sender to `@channel` and `current_user` in create:
    ```ruby
    def create
      @message = @channel.messages.build(message_params)
      @message.sender = current_user
      @message.recipient = @channel
      ...
    end
    ```
1. Change `app/views/channels/show.html.erb` to:
    ```erb
    <div class="d-flex flex-column height-full">
      <div class="flex-shrink-0 color-bg-canvas-inverse color-text-inverse p-2 d-flex flex-justify-between flex-items-center">
        <div>
          <h1><%= @channel.name %></h1>
          <p><%= @channel.description %></p>
        </div>
        <div>
          <% if current_user == @channel.creator %>
            <%= link_to 'Edit', edit_channel_path(@channel), class: "btn btn-sm" %>
          <% end %>
          <%= link_to "Leave Channel", channel_membership_path(@channel, @active_membership), class: "btn btn-sm btn-danger", method: :delete, data: { confirm: "Are you sure you want to leave #{@channel.name}?" } %>
        </div>
      </div>
      <div style="overflow: scroll;" class="flex-auto p-3">
        <p id="notice"><%= notice %></p>
        <% @channel.messages.each do |message| %>
          <p><%= message.content %>
        <% end %>
      </div>
      <div style="height: 175px" class="color-bg-canvas-inverse color-text-inverse p-3">
      MESSAGE BOX
      </div>
    </div>
    ```
1. Update `app/views/messages/_message.json.jbuilder`. Change the message_url to the channel_message_url:

    ```ruby
    json.url channel_message_url(message.recipient, message, format: :json)
    ```
1. In `app/views/layouts/application.html.erb` change the `nav` to:
    ```erb
    <nav class="col-2 pt-5 m-0 color-bg-canvas-inverse color-text-inverse menu border-0 rounded-0" aria-label="Channels">
    ```
1. And remove the padding on the `col-10`:
    ```erb
    <div class="col-10">
      <%= yield %>
    </div>
    ```
1. In `app/views/channels/shoiw.html.erb` replace `MESSAGE BOX` with:
    ```erb
    <%= form_with model: [@channel, @channel.messages.build], local: false do |form| %>
      <%= form.rich_text_area :content, class: "color-bg-canvas color-text-primary p-1" %>
      <%= form.submit "Send Message", class: "btn btn-sm mt-2" %>
    <% end %>
    ```
1. Open `app/javascript/packs/application.js` and add the following:

    ```js
    document.addEventListener("turbolinks:load", () => {
      const forms = document.querySelectorAll('form[data-remote="true"]');
      forms.forEach(form => {
        form.addEventListener("ajax:success", (event) => {
          window.location.reload();
        });
        form.addEventListener("ajax:error", (event) => {
          alert(event.detail[0]);
        });
      });
    });
    ```
1. Add `gem 'image_processing', '~> 1.2'` to your Gemfile and run `bundle install`.

# This section in the example app

https://github.com/dcsil/rails-tutorial-example/commit/e68e4579c7b291e18013b907d86f18f20be1a9c0

# Next Section
- [Adding tests: Part 1](17_adding_tests_p1.md)