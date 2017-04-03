---
layout: dsp_post
title: How to submit form in Hanami
categories: dajsiepoznac2017 hanami shyshka
description: In this blog post we will learn how to create words in our system
---

# Creating records in Hanami

We've made good progress in the [last post]({{ site.url }}/hanami-entities-and-repositories/) but we are still missing one of the key features - the ability to create and update words. In this post, we will create forms to do this for us. 

## Adding words to the system

Since we are adding a new feature to the system, we will start with feature test first. In our feature test we will specify that after filling certain fields in the form and clicking `Create` button we will end up having new word on the list.

Let's create *spec/web/features/add_word_spec.rb* file and fill it with this code:

```ruby
require 'features_helper'

describe 'Add a word' do
  after do
    WordRepository.new.clear
  end

  it 'can create a new word' do
    visit '/words/new'

    within 'form#word-form' do
      fill_in 'Name',  with: 'lew'
      fill_in 'Translation', with: 'lion'

      click_button 'Create'
    end

    current_path.must_equal('/')
    assert page.has_content?('All words')
    assert page.has_content?('lion')
  end
end
```

One thing worth mentioning is the *after* block before the definitions of the tests. It makes sure that we will delete content of our test database after each test, so next test will start with the clean plate.

Our next step is to generate action in the *words controller*. We have already done similar thing when we were working on the list of the words. All we need to do is to type:

```
bundle exec hanami generate action web words#new
```

Among others, this will add entry in the *config/routes.rb* file for handling *new* action.

Another generated file is *apps/web/templates/words/new.html.erb*. This is where our form will live, so add this code there:

```ruby
<h2>Add word</h2>

<%=
  form_for :word, '/words' do
    div class: 'input form-group' do
      label      :name
      text_field :name, class: "form-control"
    end

    div class: 'input form-group' do
      label      :translation
      text_field :translation, class: "form-control"
    end

    div class: 'controls' do
      submit 'Create Word', class: "btn btn-success"
    end
  end
%>
```

As you see, we are using here some helper methods, like *form_for*, *text_field* or *submit* to create our form. Those methods are provided by [Hanami](http://hanamirb.org/) and you can read about them more [here](http://hanamirb.org/guides/helpers/forms/).


## Submit the form

We have the form to collect the data, but what we are still missing is the action that will accept submitted data and store them in the DB. Let's start with creating a new action.

```
bundle exec hanami generate action web words#create --method=post
```

As you probably noticed - we passed additional option the generator, *--method=post*. It indicates that this action will handle HTTP POST requests and not GET requests as previous ones. 

GET requests are used when we only fetch data from the server. When we create new resources, we should use POST action.

Our *config/routes.rb* file has been updated with this entry `post '/words', to: 'words#create'`.

Next step is to implement the create action. We want this action to do 2 things:
- save the new word in the database
- redirect to the list of words after saving record

Let's express that with test. Type following code to the *spec/web/controllers/words/create_spec.rb* file:

```ruby
require 'spec_helper'
require_relative '../../../../apps/web/controllers/words/create'

describe Web::Controllers::Words::Create do
  let(:action) { Web::Controllers::Words::Create.new }
  let(:params) { Hash[word: { name: 'lew', translation: 'lion' }] }

  before do
    WordRepository.new.clear
  end

  it 'creates a new word' do
    action.call(params)

    action.word.id.wont_be_nil
    action.word.name.must_equal params[:word][:name]
  end

  it 'redirects the user to the words listing' do
    response = action.call(params)

    response[0].must_equal 302
    response[1]['Location'].must_equal '/'
  end
end
```

Next thing we need to do is to implement the *create* action. This is pretty straightforward and should look like this:

```ruby
module Web::Controllers::Words
  class Create
    include Web::Action

    expose :word

    def call(params)
      @word = WordRepository.new.create(params[:word])

      redirect_to '/'      
    end
  end
end
```

## Add link and named route

The very last thing we need to do is to update words template with the link to our newly created form. Open *apps/web/templates/words/index.html.erb* file and add this code at the bottom of the file:

```
<div class="buttons">
  <%= link_to 'New Word', routes.new_word_path, class: 'btn btn-success', title: 'New Word' %>
</div>
```

We can see some code that we are not familiar with yet:
- `link_to` method
- `routes.new_word_path`

`link_to` method is a [helper](http://hanamirb.org/guides/helpers/links/) provided by [Hanami](http://hanamirb.org/) for generation links in our app. One of its parameters is the url that it should direct to and this is provided by the second helper - `routes.new_word_path`. To make this work we have to change a little *config/routes.rb* file:

```ruby
get '/words/new', to: 'words#new', as: :new_word
``` 

Thanks to above line we have generated named route for a new_word and now we can use it in our application. You can learn more about routing in Hanami [here](http://hanamirb.org/guides/routing/overview/).

## Summary

In this post, we managed to display the form, collect input from the user, pass it to the *create* action and save the resource in the database. Pretty nice. In the next blog post, we will learn how to validate the form and display error messages.