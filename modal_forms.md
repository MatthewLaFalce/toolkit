# Modal Forms

## Steps
<!--ts-->
* [Prepare Your Bundle](#step-0---prepare-your-bundle)
* [Modifying your layout files](#step-1---modifying-your-layout-files)
* [Creating modal.js](#step-2---creating-modaljs)
* [Creating the Modal Responder](#step-3---creating-the-modal-responder)
* [Modifying the Application Controller](#step-4---modifying-the-application-controller)
* [Time to Use It!](#step-5---time-to-use-it)
<!--te-->

## Step 0 - Prepare Your Bundle

Make sure that your project is using at least `rails 4.2` and at least `responders 2.0`.

The `responders` gem adds the methods `respond_with` and `respond_to`.

```Gemfile
gem 'rails', '~> 4.2'
gem 'responders', '~> 2.0'
```

### Other Dependencies

`bootstrap` and `jquery` are also required for this, so it is necessary to add them to your asset pipeline.

```Gemfile
gem 'bootstrap'
gem 'jquery-rails'
```

## Step 1 - Modifying your layout files

**First we need to define the layout for our modal.**

```erb
<%# app/views/layouts/modal.html.erb %>

<div class="modal" id="mainModal" tabindex="-1" role="dialog" aria-labelledby="mainModalLabel" aria-hidden="true">
  <div class="modal-dialog">
    <div class="modal-content">
      <div class="modal-header">
        <h5 class="modal-title" id="mainModalLabel">
          <%= yield :title if content_for? :title %>&nbsp;
        </h5>
        <button type="button" class="close" data-dismiss="modal"><span aria-hidden="true">&times;</span><span class="sr-only">Close</span></button>
      </div>

      <%= yield %>

    </div>
  </div>
</div>
```

**We also need to define where our modal will be rendered. For that we will place this in our `application` layout.**
```erb
<%# app/views/layouts/application.html.erb %>

<div id="modal-holder"></div>
```

## Step 2 - Creating modal.js

Now we want to create the javascript that will render our links that have the `data-modal` attribute in our modal layout.

As well we will need to create the functionality that will allow our remote forms to submit data back to the appropriate controller. Our application should be able to properly handle redirects to the given page and form re-displays with errors just as our normal forms would. We will assume that if the response has the `location` header set, that we will redirect the user to the specified location. Otherwise we will redisplay the form.

```javascript
// app/assets/javascripts/modals.js

$(function () {
  const modal_holder_selector = "#modal-holder";
  const modal_selector = ".modal";

  $(document).on("click", "a[data-modal]", function () {
    const location = $(this).attr("href");
    // Load modal dialog from server
    $.get(location, (data) => {
      $(modal_holder_selector).html(data).find(modal_selector).modal();
    });
    return false;
  });

  $(document).on("ajax:success", "form[data-modal]", function (event) {
    const [data, _status, xhr] = event.detail;
    const url = xhr.getResponseHeader("Location");
    if (url) {
      // Redirect to url
      window.location = url;
    } else {
      // Remove old modal backdrop
      $(".modal-backdrop").remove();
      // Update modal content
      const modal = $(data).find("body").html();
      $(modal_holder_selector).html(modal).find(modal_selector).modal();
    }

    return false;
  });
});
```

## Step 3 - Creating the Modal Responder

In this class we will inherit from `ActionController::Responder` since that is what `respond_with` uses for its rendering. We will make our own custom class to handle rendering for our modal, which we will call `ModalResponder`.

We will override the `render` and `redirect_to` methods in order to modify the behavior when a request is made using XHR.

Now if a request is made via AJAX we will `render` using out custom `modal` layout and instead of redirecting we want `redirect_to` to return only headers with the `location` header set which will handle our javascript logic.

```ruby
# app/services/modal_responder.rb

class ModalResponder < ActionController::Responder
    cattr_accessor :modal_layout
    self.modal_layout = 'modal'

    def render(*args)
      options = args.extract_options!
      if request.xhr?
        options.merge! layout: modal_layout
      end
      controller.render *args, options
    end

    def default_render(*args)
      render(*args)
    end

    def redirect_to(options)
      if request.xhr?
        head :ok, location: controller.url_for(options)
      else
        controller.redirect_to(options)
      end
    end
  end
```

## Step 4 - Modifying the Application Controller

Now we just need to add a helper method, `respond_modal_with`, that will use our `ModalResponder`.

```ruby
class ApplicationController < ActionController::Base
  respond_to :html

  # Prevent CSRF attacks by raising an exception.
  # For APIs, you may want to use :null_session instead.
  protect_from_forgery with: :exception

  def respond_modal_with(*args, &blk)
    options = args.extract_options!
    options[:responder] = ModalResponder
    respond_with *args, options, &blk
  end

end
```

## Step 5 - Time to Use It!

First we need to add a link to the modal:

```erb
<%# app/views/layouts/_header.html.erb %>

<%= link_to 'Add Message', new_message_path, class: 'btn btn-outline-success my-2 my-sm-0', data: { modal: true } %>
```

Now, we need to modify our controller using our new `respond_modal_with` method instead of `respond_with`:

```ruby
# app/controllers/messages_controller.rb

class MessagesController < ApplicationController

  respond_to :html, :json

  def new
    @message = Message.new
    respond_modal_with @message
  end

  def create
    @message = Message.create(message_params)
    respond_modal_with @message, location: messages_path
  end

  private
    def set_message
      @message = Message.find(params[:id])
    end

    def message_params
      params.require(:message).permit(:name, :body)
    end
end
```

And, finally, you should add two attributes to your form:

```erb
<%# app/views/messages/_form.html.erb %>

<%= simple_form_for(@message, remote: request.xhr?, html: { data: { modal: true } }) %>
```

`remote` tells `jquery_ujs` to submit this form with AJAX. We use `request.xhr?` because we want this form to be functional when we display it in a modal and separately.

`data-modal` is used to tell our script to handle this form as the modal form.

