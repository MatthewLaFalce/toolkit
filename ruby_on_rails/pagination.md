# Modal Forms

## Steps
<!--ts-->
* [Prepare Your Bundle](#step-0---prepare-your-bundle)
* [Creating your Custom Stylesheet](#step-1---creating-your-custom-stylesheet)
* [Importing your Custom Stylesheet](#step-2---importing-your-custom-stylesheet)
* [Use It!](#step-3---use-it)
<!--te-->

## Step 0 - Prepare Your Bundle

Make sure that your project is using at least `rails 5.2`.

We will use the `will-paginate` gem to handle all of the leg work in actually paginating the views. This document shows how to create a custom sytle sheet and us it in conjuction with  `will-paginate`.

```Gemfile
ruby '2.6.3'
gem 'rails', '~> 5.2'
gem 'will_paginate'
```

### Other Dependencies

`bootstrap` is also required for this, so it is necessary to add it to your asset pipeline.

```Gemfile
gem 'bootstrap'
```

## Step 1 - Creating your Custom Stylesheet

```scss
/* app/assets/stylesheets/custom/pagination.scss */

.pear_pagination {
  text-align: center;
  padding: 1em;
  cursor: default;
}
.pear_pagination a,
.pear_pagination span {
  padding: 0.2em 0.3em;
}
.pear_pagination .disabled {
  color: #aaaaaa;
}
.pear_pagination .current {
  font-style: normal;
  font-weight: bold;
  background-color: #bebebe;
  display: inline-block;
  width: 1.4em;
  height: 1.4em;
  line-height: 1.5;
  -moz-border-radius: 1em;
  -webkit-border-radius: 1em;
  border-radius: 1em;
  text-shadow: rgba(255, 255, 255, 0.8) 1px 1px 1px;
}
.pear_pagination a {
  text-decoration: none;
}
.pear_pagination a:hover,
.pear_pagination a:focus {
  text-decoration: underline;
}
```

## Step 2 - Importing your Custom Stylesheet

```scss
/* app/assets/stylesheets/application.scss */

@import "custom/pagination";
```

## Step 3 - Use It!

``` erb

<div class="pear_pagination">
  <div class="page_info">
    <%= page_entries_info @posts %>
  </div>
  <%= will_paginate @posts, :container => false %>
</div>
```
