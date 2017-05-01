---
layout: dsp_post
title: How to delete record in Hanami
categories: dajsiepoznac2017 hanami shyshka
description: In this blog post we will learn how to delete records in Hanami framework.
---

# Deleting records in Hanami

The last basic functionality we need to add is the ability to delete words. Thanks to this we will have full CRUD in our app and we will be able to move to more complicated features

## Feature test for deleting words

As usually, we will start with adding a feature test. It's going to be fairly easy - we will assume that a single word is on the list of words, click "Delete" link and then we will make sure that the word is no longer on the list. Create **_spec/web/features/delete_word_spec.rb_** file and type in this code: 

```ruby
require 'features_helper'

describe 'Delete words' do
  before do
    WordRepository.new.clear
    @word = WordRepository.new.create(name: "lew", translation: "lion")
  end  

  it 'deletes the word' do
    visit '/'
    within '.word .actions .delete_word' do
     click_button "Delete"
    end

    current_path.must_equal('/')
    assert page.has_content?('All words')
    assert page.has_content?('There are no words yet.')    
  end
end
```

## Delete record

Our next task is to generate **_destroy_** action in the **_words controller_**. To do this type:

```
bundle exec hanami generate action web words#destroy --method=delete
```

As a result, we have an action that will be responsible for handling DELETE requests that will be used for deleting words. Let's change a little **_config/routes.rb_** file as well so we will have a named route:

```ruby
delete '/words/:id', to: 'words#destroy', as: :destroy_word
```

Let's implement the action now. It's simple action that will be very similar to the update action that we implemented in [this blog post]({{ site.url }}/update-record-in-hanami/). 

Open **_apps/web/controllers/words/destroy.rb_** and add this code:

```ruby
module Web::Controllers::Words
  class Destroy
    include Web::Action

    expose :word

    def call(params)
      @word = WordRepository.new.delete(params[:id])

      redirect_to '/'       
    end
  end
end
``` 

## Send delete request

Now we have to adjust our index view - next to the **_Edit_** link we want to have another link that will trigger the delete action. Since **_link_to_** helper works only with **_GET_** requests we need to figure out a different way of sending **_DELETE_** request.

There are 2 possible solutions here:
- we can add a class to our link and write some javascript so when we click this link we will send **_DELETE_** request using our javascript function
- instead of using links we can add very simlpe form and use this for sending **_DELETE_** requests

We will use the second approach. For each word on our list we want to render a form with "Delete" submit button. To avoid duplication we will use partials.

Our index page should look like this:

```ruby
<h1>All words</h1>

<% if words.any? %>
  <div id="words">
    <% words.each do |word| %>
      <div class="word row">
        <div class="col-xs-4"><%= word.name %></div>
        <div class="col-xs-4"><%= word.translation %></div>
        <div class="col-xs-4 actions">
          <%= link_to 'Edit', routes.edit_word_path(word.id) %> 
          <%= render partial: 'delete_word_button', locals: {word: word} %>


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

As we can see we are rendering a partial called **_delete_word_button_** and we are passing to it a word.

The last step is to add this partial.

Let's create **_apps/web/templates/words/_delete_word_button.html.erb_** file and add following code:

```ruby
<%= 
  form_for :word, routes.destroy_word_path(word.id), method: :delete, class: "delete_word" do
    submit 'Delete'
  end
%>
```

It's a very simple form - we specify the URL using previously added named route, specify that we want to send the form using **_DELETE_** method and we add "delete_word" CSS class so we can style form later on. The body of the form consists only of submit button.

After adding this our feature test should pass.

## Summary

Thanks to what we have achieved today we have a full CRUD application. We can add, edit and delete words, everything with a pretty good test coverage. Code can be found [here](https://github.com/detfis/shyshka). Now we are in the good position for adding more advanced functionalities, so stay tuned.