# Rails 6 Web App Quick Start Guide

> Some useful tools to help most projects

## TOC


## Prerequisites

- Node.js (for webpacker to run)
- Yarn (used by webpacker to install dependencies)
- Visual Studio Code (I’ll explain how to set up debugging in it)
- Ruby (does this really need to be said?)
- Rails installed globally (in order to use the “rails new” generator)

## Create a homepage
First we ned to create a new controller for our static pages.

```ruby
# app/controllers/pages_controller.rb
class PagesController < ApplicationController

  include ShowStatic

end
```

And create this module to support our pages controller.
```ruby
# lib/show_static.rb
module ShowStatic

  def show
    render(params[:id] || 'home')
  end

  # The homepage is authorized
  def authorized?
    params[:id].blank? ? true : super
  end

end
```

Then let's create a route to point to our new controller.
```ruby
# config/routes.rb
Rails.application.routes.draw do
  root to: 'pages#home'
end
```

