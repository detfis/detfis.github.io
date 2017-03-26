---
layout: dsp_post
title: Add Bootstrap to Hanami
categories: dajsiepoznac2017 hanami
description: Easy way to add Bootstrap to Hanami project
---

# Add Bootstrap to Hanami project #

In [previous post]({{ site.url }}/first-feature-in-hanami/) on [Shyshka](https://github.com/detfis/shyshka) web app we ended up having list of words with their translations. This list has no CSS styles and we will add a tool that is going to improve an appearance of the project. My choice is to add [Bootstrap](http://getbootstrap.com/). 

## What is Bootstrap? ##

[Bootstrap](http://getbootstrap.com/) is an open-source JS and CSS framework developed at Twitter. Its goal is to develop responsive, mobile first projects on the web. It contains HTML and CSS-based design templates for typography, forms, buttons, navigation and other interface components, as well as optional JavaScript extensions.

## How to add Bootstrap to Hanami? ##

When I wanted to add [Bootstrap](http://getbootstrap.com/) to my project I did some googling first. I found [hanami-bootstrap gem](https://github.com/davydovanton/hanami-bootstrap) and this is definitely an easy option to add it to the project.

After looking at the code of this gem I decided I will add it directly. There are 2 reasons to do this:
- it's simple enough and adding it through a gem which I'm not sure if it's going to be maintained might not be a good idea
- I want to take this opportunity to learn a little more about how [Hanami](http://hanamirb.org/) handles assets.

You can find information on assets in Hanami [here](http://hanamirb.org/guides/assets/overview/).

One of the solutions is to add the additional folder to assets path and add [Bootstrap](http://getbootstrap.com/) there. We will add **vendor** folder in which we will keep third-party libraries.

Open **apps/web/application.rb** file and change **asset** section:

```ruby
assets do

  # Specify sources for assets
  #
  sources << [
    'assets',
    'vendor/assets'
  ]
end
```

It tells assets management tool in [Hanami](http://hanamirb.org/) - which is separated to its own [gem](https://github.com/hanami/assets) - to lazy copy files from those paths to the **public/assets** folder. 

After that we should create **vendor/assets** folders under **apps/web** directory. 

Our folder structure is set correctly now.

The next step is to download [jQuery](https://jquery.com/) library, which is a dependency for [Bootstrap](http://getbootstrap.com/). Download [jQuery](https://jquery.com/) and copy it to the **apps/web/vendor/assets/jquery** folder.

After that we can download [Bootstrap](http://getbootstrap.com/) library and copy it under **apps/web/vendor/assets/bootstrap**. 

All files are placed correctly and now we can actually use those libraries in our project.

First, require them in the **apps/web/templates/application.html.erb**

```
    <%= javascript 'jquery-3.2.0.min' %>
    <%= javascript 'bootstrap.min' %>
    <%= stylesheet 'bootstrap' %>
```

We use here *assets helpers* from [Hanami](http://hanamirb.org/) framework. You can read more about them [here](http://hanamirb.org/guides/helpers/assets/).

We will also add *container* CSS class from [Bootstrap](http://getbootstrap.com/) framework. So **apps/web/templates/application.html.erb** should look like this:

```html 
<!DOCTYPE html>
<html>
  <head>
    <%= javascript 'jquery-3.2.0.min' %>
    <%= javascript 'bootstrap.min' %>
    <%= stylesheet 'bootstrap' %>
    <title>Web</title>
    <%= favicon %>
  </head>
  <body class="container">
    <%= yield %>
  </body>
</html>
``` 

The last step is to add [Bootstrap](http://getbootstrap.com/) styles to the list of words. We will start very basic and for now, the file responsible for displaying this list should look like this:

```html 
<h1>All words</h1>

<div id="words">
  <div class="word row">
    <div class="col-xs-6">kot</div>
    <div class="col-xs-6">cat</div>
  </div>

  <div class="word row">
    <div class="col-xs-6">pies</div>
    <div class="col-xs-6">dog</div>
  </div>
</div>
```

## Summary ##

That's all we had to do to add [Bootstrap](http://getbootstrap.com/) to [Hanami](http://hanamirb.org/) project. When we refresh page we should see, that page looks a little better.

![hanami_bootstrap_basic]({{ site.url }}/assets/images/hanami_bootstrap_basic.png)
