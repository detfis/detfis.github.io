---
layout: dsp_post
title: Haml and Hanami
categories: dajsiepoznac2017 hanami shyshka
description: What is Haml and how to use it in Hanami project
---

# What is Haml?

[Haml](http://haml.info/) (HTML abstraction markup language) is a markup language designed to make template created simpler, quicker and more beautiful. Its core principles are:
* markup should be beautiful
* markup should be DRY
* markup should be well-intended
* HTML structure should be clear

The promise of using [Haml](http://haml.info/) is that you will gain productivity and your templates will be cleaner and more readable.

Those are exactly the reason why I decide to move out of the default templating engine in [Hanami](http://hanamirb.org/), erb(which stands for embedded ruby).

ERB is more verbose and I find it less readable. 

## Small refactor first

Before we switch [Shyshka](https://github.com/detfis/shyshka) to [Haml](http://haml.info/), let's clean up some code first. [In the post on deleting records in Hanami]({{ site.url }}/delete-records-in-hanami/) we created template **_apps/web/templates/words/_delete_word_button.html.erb_** which had this code in it:

```ruby
<%= 
  form_for :word, routes.destroy_word_path(word.id), method: :delete, class: "delete_word" do
    submit 'Delete'
  end
%>
```

Then, next to each word we had to render this template by calling

```ruby
<%= render partial: 'delete_word_button', locals: {word: word} %>
```

If we take a closer look at the **_apps/web/templates/words/_delete_word_button.html.erb_** we will notice, that there aren't many template-related things in it - it has no HTML tags and one CSS class assigned to a form. It contains only ruby code responsible for generating the form.

I decided that view is more suitable for this kind of code. Let's open **_apps/web/views/words/index.rb_** and add the following code there:

```ruby
def delete_word_form(word)
  form_for :word, routes.destroy_word_path(word.id), method: :delete, class: "delete_word" do
    submit 'Delete'
  end
end
```

Now we have to call this funtion in the **_apps/web/templates/words/index.html.erb_** file. Delete the line that renders **_delete_word_button_** partial and replace it with this code:

```ruby
<%= delete_word_form(word) %>
``` 

Everything should work as expected and now we can delete **_apps/web/templates/words/_delete_word_button.html.erb_** template, which is not needed anymore.

## Adding Haml to Hanami

Adding [Haml](http://haml.info/) to [Hanami](http://hanamirb.org/) is super easy. The first thing you need to do is add **_haml_** gem to **_Gemfile_**: 

```ruby
gem 'haml'
```

Now run this command in the console to install gem:

```
bundle install
```

Next step is to change all the views to use [Haml](http://haml.info/). Firstly, we need to change the file extension from **_.erb_** to **_.haml_**. Secondly, we need to update every single view. Unfortunately, there is no automatic tool for it (at least I'm not aware of any of them), so we have to do it manually. This is how those templates should look like:

**_apps/web/templates/words/_form.html.haml_**:

```ruby
= form_for form, class: 'form' do
  - div class: 'input form-group' do
    - label      :name
    - text_field :name, class: "form-control"

  - div class: 'input form-group' do
    - label      :translation
    - text_field :translation, class: "form-control"

  - div class: 'controls' do
    - submit submit_label, class: "btn btn-success"

- unless params.valid?
  .errors
    %h3 There was a problem with your submission
    %ul
      - params.error_messages.each do |message|
        %li= message
```

**_apps/web/templates/words/edit.html.haml_**:

```ruby
%h2 Edit word

= render partial: 'form'
```

**_apps/web/templates/words/index.html.haml_**:

```ruby
%h1 All words

- if words.any?
  #words
    - words.each do |word|
      .word.row
        .col-xs-4= word.name
        .col-xs-4= word.translation
        .col-xs-4.actions
          = link_to 'Edit', routes.edit_word_path(word.id)
          = delete_word_form(word)

- else
  %p.placeholder There are no words yet.

.buttons
  = link_to 'New Word', routes.new_word_path, class: 'btn btn-success', title: 'New Word'
```

**_apps/web/templates/words/new.html.haml_**:

```ruby
%h2 Add word

= render partial: 'form'
```

## Generating new actions with Haml templates

When we create a new action, [Hanami](http://hanamirb.org/) by default will create ERB templates for use. It's cumbersome to manually change the extension of the template file if we want to use [Haml](http://haml.info/). Fortunately, we can specify templating engine we would like to use:

```
bundle exec hanami generate action web examples#show --template=haml
```

## Summary

Today we've made a fairly simple thing that will make our life easier in the future. As we will be adding new features to our application we will have to create more and more templates, so it's important to make this process as seamless as possible.