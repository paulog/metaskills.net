--- 
layout: post
title: Rails Button Links In Embedded Forms
disqus_id: /2010/01/05/rails-button-links-in-embedded-forms/
categories: 
  - heuristics
  - ruby-rails
---

<p>
  This is one I have had sitting around for almost 3 years now in my toolbox and thought I would share. Have you ever had complicated rails forms and needed simple form buttons that just took you to a simple link? Were you bitten by the <code>button_to</code> helper code because it generates another form inside of a form? If so, here is a simple rails view helper I made that creates simple button links for embedded forms by making an input with a javascript function. Tag soup you ask, hell yeah, but worth if if you need it.
</p>

```ruby
def button_to_link(name, link, options={})
  confirm_option = options.delete(:confirm)
  popup_option = options.delete(:popup)
  link_function = popup_option ? redirect_function(link,:new_window => true) : redirect_function(link)
  link_function = "if (confirm('#{escape_javascript(confirm_option)}')) { #{link_function}; }" if confirm_option
  button_to_function name, link_function, options
end

def redirect_function(location, options={})
  location = location.is_a?(String) ? location : url_for(location)
  if options[:new_window]
    %|window.open('#{location}')|
  else
    %|{window.location.href='#{location}'}|
  end
end
```

