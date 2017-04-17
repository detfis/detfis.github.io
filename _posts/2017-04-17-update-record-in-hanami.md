---
layout: dsp_post
title: How to update record in Hanami
categories: dajsiepoznac2017 hanami shyshka
description: In this blog post we will learn how to update records, use partials for shared templates and what views or named routes are for.
---

# Updating records in Hanami

In [one of the previous posts]({{ site.url }}/create-record-in-hanami/) we've learned how to create records in [Hanami](http://hanamirb.org/) framework. Today we will learn how to update the records.

## Editing words in the system

Editing records is the a feature, so we will start again with the [BDD](https://en.wikipedia.org/wiki/Behavior-driven_development) approach and our first step will be creating the feature test. In this test we will create a record, update it and make sure that the updated value is present on the list of words. Create **_spec/web/features/edit_word_spec.rb_** file and add this code:

```ruby
require 'features_helper'

describe 'Edit a word' do
  before do
    WordRepository.new.clear
    @word = WordRepository.new.create(name: "lew", translation: "lion")
  end

  it 'can edit a word' do
    visit "/words/#{@word.id}/edit"

    within 'form#word-form' do
      fill_in 'Name',  with: 'lew_2'
      fill_in 'Translation', with: 'lion_2'

      click_button 'Update'
    end

    current_path.must_equal('/')
    assert page.has_content?('All words')
    assert page.has_content?('lion_2')
  end
  
end
``` 

Take a look at the **_before_** block, where we create word that will be used in any further test cases.

## Update record

The next step is to generate **_edit_** action in the **_words controller_**. To do this type:

```
bundle exec hanami generate action web words#edit
```

As you already know, this will create and new entry in the **_config/routes.rb_** file. 

Now lets go to the template. In **_apps/web/templates/words/edit.html.erb_** file we want to display the form to edit the word. This file should look like this:

```ruby
<%=
  form_for :word, "/words/word.id", {word: word}, {method: :patch} do
    div class: 'input' do
      label      :name
      text_field :name
    end

    div class: 'input' do
      label      :translation
      text_field :translation
    end

    div class: 'controls' do
      submit 'Update Word'
    end
  end
%>
```

Our next task is to create action that will handle the submitted form. Type:

```
bundle exec hanami generate action web words#update --method=patch
```

As we see - we have created **_update_** action that will handle PATCH requests. PATCH and PUT are the HTTP methods that are reserved for updating existing records. The difference between PUT and PATCH is quite subtle (PUT should be used when you update all attributes, PATCH when only a subset of those attributes) and [Hanami](http://hanamirb.org/) lets you use both for updating records.

Last step before making our feature test pass is adding code to the update action. Add the following code to the **_apps/web/controllers/words/update.rb**:

```ruby
module Web::Controllers::Words
  class Update
    include Web::Action

    expose :word

    def call(params)
      @word = WordRepository.new.update(params[:id], params[:word])

      redirect_to '/'      
    end
  end
end
```

After running the test we should see that all the tests pass. 

## Introduce partials

As we can see, files responsible for displaying the form for adding and editing a word are very similar  

It's a duplication and keeping the same code in 2 places is not optimal - if we decided to make a change in the form we would have to remember to change both files. 

There is a way to factor out common code, though. We will have to combine two concepts that we did not use yet - **_partials_** and **_views_**. 

Partials are just parts of the html that live in seperate file and are "injected" into the parent template. Their names usually start with "_". 

To meet the needs of our case we have to create file **_apps/web/templates/words/_form.html.erb_** and paste the content of the edit template there.

To call a partial change both templates:

**_apps/web/templates/words/new.html.erb_**:

```ruby
<h2>Add word</h2>

<%= render partial: 'form' %>
```

**_apps/web/templates/words/edit.html.erb_**:

```ruby
<h2>Edit word</h2>

<%= render partial: 'form' %>
```

We still have one problem to solve - our new and edit forms are very similar, but not identical - they have different arguments passed to **_form_for_** function and have different text on the submit button.

## Introduce views

For exactly this reason views have been invented. They are the layer that is responsible for preparing data to our templates. So we need to change each view class in that way that corresponding template will receive the correct data.

We have to change 4 view classes - **_new_**, **_create_**, **_edit_** and **_update_**.

In those files, we have to add 2 methods - one for indicating what text should be on the submit button, the second for setting arguments for the **_form_for_** method.

Those files should look like this:

**_apps/web/views/words/new.rb_**:

```ruby
module Web::Views::Words
  class New
    include Web::View

    def form
      Form.new(:word, '/words')
    end

    def submit_label
      'Create'
    end
  end
end
```

**_apps/web/views/words/create.rb_**:

```ruby
module Web::Views::Words
  class Create
    include Web::View

    def form
      Form.new(:word, '/words')
    end

    def submit_label
      'Create'
    end  
  end
end
```

**_apps/web/views/words/edit.rb_**:

```ruby
module Web::Views::Words
  class Edit
    include Web::View

    def form
      Form.new(:word, "/words/#{word.id}", {word: word}, {method: :patch})
    end

    def submit_label
      'Update'
    end    
  end
end
```

**_apps/web/views/words/update.rb_**:

```ruby
module Web::Views::Words
  class Update
    include Web::View

    def form
      Form.new(:word, "/words/#{word.id}", {word: word}, {method: :patch})
    end

    def submit_label
      'Update'
    end     
  end
end
```

The last step is to change our form template to actually use those methods provided by the views. To do so change the **_apps/web/templates/words/_form.html.erb_**:

```ruby
<%=
  form_for form, class: 'form' do
    div class: 'input form-group' do
      label      :name
      text_field :name, class: "form-control"
    end

    div class: 'input form-group' do
      label      :translation
      text_field :translation, class: "form-control"
    end

    div class: 'controls' do
      submit submit_label, class: "btn btn-success"
    end
  end
%>
```

After those changes our tests should still be passing and we still should be able to add and edit new words.

## Introduce named routes

The last thing we need to take care of is adding the possibility to access the edit page. To do this we will add link "Edit" next to each word on the index page. 

To achieve this, add this code after the translation:

```html
<div class="col-xs-4 actions">
  <%= link_to 'Edit', "words/#{word.id}/edit" %>
</div>
```

This would do the trick but we can do better. Generating the path by string interpolation is not the recommended way of doing this. Much better approach is to use [**_named routes_**](http://hanamirb.org/guides/routing/basic-usage/#named-routes). They specify the name for a given route that can be used later in templates, views or in tests.

To make use of them alter **_config/routes.rb_** file to look like this:

```ruby
get '/words/:id/edit', to: 'words#edit', as: :edit_word
patch '/words/:id', to: 'words#update'
post '/words', to: 'words#create'
get '/words/new', to: 'words#new', as: :new_word
root to: 'words#index'
``` 

As you can see we added 2 named routes: 
- edit_word
- new_word

Now we can change the index view to look like this:

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

## Summary

Today we've covered a lot of ground:
- we've added the functionality of updating a record
- we've refactored our code to use partials
- we've used views for the first time
- we've made use of named routes

In the next post, we will add some data validation and I'll show how elegantly this problem is solved in [Hanami](http://hanamirb.org/)