---
layout: dsp_post
title: How to update record in Hanami
categories: dajsiepoznac2017 hanami shyshka
description: In this blog post we will learn how to update records
---

-- nie zapomniec o walidacji!!!

# Updating records in Hanami

In [link]one of the previous posts we've learned how to create records in [link]Hanami framework. Today we will learn how to update the records.

## Editing words in the system

- new feature spec

```
bundle exec hanami generate action web words#edit edit_path
```

add named route

*apps/web/templates/words/edit.html.erb*

reuse form code -> partials, update new

## Submit the form to update the word

```
bundle exec hanami generate action web words#update --method=patch
```

*spec/web/controllers/words/update_spec.rb*

```ruby
module Web::Controllers::Words
  class Update
    include Web::Action

    expose :word

    def call(params)
      @word = WordRepository.new.find(params[:id])
      WordRepository.new.update(@word.id, params[:word])  
      redirect_to '/'      
    end
  end
end
```

## Add link to the index page

```
<h1>All words</h1>

<% if words.any? %>
  <div id="words">
    <% words.each do |word| %>
      <div class="word row">
        <div class="col-xs-4"><%= word.name %></div>
        <div class="col-xs-4"><%= word.translation %></div>
        <div class="col-xs-4 actions">
          <%= link_to 'Edit', routes.edit_word_path(word.id) %>
        </div>
      </div>
    <% end %>
  </div>
<% else %>
  <p class="placeholder">There are no words yet.</p>
<% end %>

<div class="buttons">
  <%= link_to 'New Word', routes.new_word_path, class: 'btn btn-success', title: 'New Word' %>
</div>
```