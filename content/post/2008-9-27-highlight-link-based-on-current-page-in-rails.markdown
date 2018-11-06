---
layout: post
title: "Highlight link based on current page in rails"
date: 2008-09-27T19:47:00Z
comments: false
permalink: /post/52081481/highlight-link-based-on-current-page-in-rails
categories:
---



This is common pattern in website navigation, where it highlights the link (usually by setting `class=”active”`) that took you to the current page while you are on that page.

First, define a helper:

```ruby
def is_active?(page_name)
  "active" if params[:action] == page_name
end

```

Then call it in your link_to’s in your layout as such:

```ruby
link_to 'Home', '/', :class => is_active?("index")
link_to 'About', '/about', :class => is_active?("about")
link_to 'contact', '/contact', :class => is_active?("contact")
```

This effect is achieved due to how link_to handles being passed `nil` for its `:class`, so when `is_active?` returns `nil` (because its not the current page), `link_to` outputs nothing as its class (not `class=””` as you might expect).
