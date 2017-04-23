---
layout: dsp_post
title: Validation in Hanami
categories: dajsiepoznac2017 hanami shyshka
description: How to validate records in Hanami framework
---

# Validating records in Hanami

Now we have the ability to add and edit new words in our application. In this blog post we will learn how to validate records we want to update or create.

## Why we have to add validation?

Now it's possible to add a new record without a name or translation - maybe in the future we will add the possibility for adding words without translations, but for now, we want to make sure that both fields are filled.

## How we validate records in Hanami

Code which responsibility is to perform validations has been factored out and lives in the gem called [hanami-validations](https://github.com/hanami/validations). Under the hood, it uses a great gem called [dry-validation](https://github.com/dry-rb/dry-validation). I highly recommend to check this out - it's one of my favorite gems and I use it even in my rails apps. 

[link]Hanami proposes a little different approach for validation than for example Rails. In Rails you initialize the object and then you check if this object is valid. In [link]Hanami you validate the params first and only if the params are valid you create a new object. Thanks to that you avoid having objects that are in the invalid state (at least during initialization). You can read why this is a good thing in the Piotr Solnica [blog post](http://solnic.eu/2015/12/28/invalid-object-is-an-anti-pattern.html) who is also the creator of the [dry-validation](https://github.com/dry-rb/dry-validation) gem.

## Adding validation in Hanami

In [link]Hanami we are validating params, so a controller is a place where we do this. To add validation to our project add this code:

```ruby
params do
  required(:word).schema do
    required(:name).filled(:str?)
    required(:translation).filled(:str?)
  end
end
```

to both **_Create_** and **_Update_** actions.

In the above code, we define a schema and we expect that parameters passed to those actions will have key _word_ and the key _word_ will have keys _nane_ and _translation_, which both will be strings.

Now we need to update the actions, to actually trigger the validations. Our **_apps/web/controllers/words/create.rb_** file should look like this:

```ruby
module Web::Controllers::Words
  class Create
    include Web::Action

    expose :word

    params do
      required(:word).schema do
        required(:name).filled(:str?)
        required(:translation).filled(:str?)
      end
    end

    def call(params)
      if params.valid?
        @word = WordRepository.new.create(params[:word])

        redirect_to '/'
      else
        @word = Word.new(params[:word])
        self.status = 422
      end
    end
  end
end
```

As we see, to validate params we need to call _valid?_ method on params. If validation fails, we will return status code _422_, which indicates that the record was incorrect and we will initialize a word object again, because we want to display the form once more with error messages.

**_Update_** action should look very similar: 

```ruby
module Web::Controllers::Words
  class Update
    include Web::Action

    expose :word

    params do
      required(:word).schema do
        required(:name).filled(:str?)
        required(:translation).filled(:str?)
      end
    end

    def call(params)  
      if params.valid?
        @word = WordRepository.new.update(params[:id], params[:word])

        redirect_to '/'
      else
        @word = WordRepository.new.find(params[:id])
        self.status = 422
      end
    end
  end
end
```

We have to also update slightly our **_Create_** and **_Update_** views - since we want to display the form again if the validation failed, we need to instruct the view which template it should use.

To do this add 

```ruby
template 'words/new'
```

to **_Create_** view and 

```ruby
template 'words/edit'
```

to **_Update_** view.

Next step will be adding the test that will reflect newly added logic. To do this lets open **_spec/web/controllers/words/create_spec.rb_** and add scenario where we pass invalid params.

```ruby
require 'spec_helper'
require_relative '../../../../apps/web/controllers/words/create'

describe Web::Controllers::Words::Create do
  let(:action) { Web::Controllers::Words::Create.new }

  before do
    WordRepository.new.clear
  end

  describe 'with valid params' do
    let(:params) { Hash[word: { name: 'lew', translation: 'lion' }] }

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

  describe 'with invalid params' do
    let(:params) { Hash[word: {}] }

    it 're-renders the words#new view' do
      response = action.call(params)
      response[0].must_equal 422
    end

    it 'sets errors attribute accordingly' do
      response = action.call(params)
      response[0].must_equal 422

      action.params.errors[:word][:name].must_equal  ['is missing']
      action.params.errors[:word][:translation].must_equal ['is missing']
    end
  end
end
```

We've placed our previous tests inside _with valid params_ _descibe_ block and we've added _with invalid params_ block.

All the test should pass at this stage.

## Displaying error messages

Our validation works great now, but we would like to show to the user why adding or editing a word failed and what should be fixed. 
To do this we will update the partial with our form to include this code:

```html
<% unless params.valid? %>
  <div class="errors">
    <h3>There was a problem with your submission</h3>
    <ul>
      <% params.error_messages.each do |message| %>
        <li><%= message %></li>
      <% end %>
    </ul>
  </div>
<% end %>
```

The last part is to update our feature tests. We want to make sure that if the user will not be able to save a word he would see the error messages. 

We have to update 2 files - **_spec/web/features/add_word_spec.rb_** and **_spec/web/feature/edit_word_spec.rb_**.

The first file should look like this:

```ruby
require 'features_helper'

describe 'Add a word' do
  before do
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

  it 'displays list of errors when params contains errors' do
    visit '/words/new'

    within 'form#word-form' do
      click_button 'Create'
    end

    current_path.must_equal('/words')

    assert page.has_content?('There was a problem with your submission')
    assert page.has_content?('Name must be filled')
    assert page.has_content?('Translation must be filled')
  end  
end
```

and the second like this:

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

  it 'displays list of errors when params contains errors' do
    visit "/words/#{@word.id}/edit"

    within 'form#word-form' do
      fill_in 'Name',  with: ''
      fill_in 'Translation', with: ''
      click_button 'Update'
    end

    current_path.must_equal("/words/#{@word.id}")

    assert page.has_content?('There was a problem with your submission')
    assert page.has_content?('Name must be filled')
    assert page.has_content?('Translation must be filled')
  end  
end
```

As we can see, we've added to both files the test where parameters are invalid and we've made some assertion on what is visible on the page when validation failed.

## Summary

Today we've added an important part of the application and now we can say that adding and editing words is completely finished. The last missing part from the basic functionalities we want to provide is deleting the records and this what we will be working on next. 
