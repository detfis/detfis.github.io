---
layout: dsp_post
title: Hanami - Entities and Repositories
categories: dajsiepoznac2017 hanami shyshka
description: Introduction of data modeling tools in Hanami - Entities and Repositories
---

# Hanami - Entities and Repositories

In [one of the previous posts]({{ site.url }}/first-feature-in-hanami/) we have created a list of the words. This list has one big flaw, though - it's hard-coded. We would like to display dynamic data there. To achieve this we will introduce two parts of [Hanami](http://hanamirb.org/) ecosystem responsible for dealing with data - entities and repositories. 

## What are Entities?

[Entity](http://hanamirb.org/guides/models/entities/) is the core of the application. This is where business logic lives. They are defined by their identity. They are decoupled from the persistence layer - database - which makes them lightweight and easy to test. Every entity has a schema associated with it. It can be either schema derived automatically from the database table definition (if we are using SQL database), or custom schema defined inside entity class. If we are not using SQL database we are obliged to define a custom schema. 

The role of the schema is to:
- whitelist the data used during initialization
- enforce data integrity  

When we are defining custom schemas [Hanami](http://hanamirb.org/) provides data types, which are based on the [dry-types gem](http://dry-rb.org/gems/dry-types/). 

## What are Repositories

[Repository](http://hanamirb.org/guides/models/repositories/) is an object that lies between [entity](http://hanamirb.org/guides/models/entities/) and the persistence layer. It provides an API to query and execute commands on the database. It's storage independent and uses specific adapters that implement low-level code that deals with the database. 

Repository is quite a common pattern in software development world. It provides many advantages:
- applications depend on a stable API
- many data sources can live inside one application
- data sources can be changed easily

By design, all query methods in repositories are private. The goal is to create intention revealing APIs. So, instead of creating this method:

```ruby
SomeRepository.new.where(reference_id: 123).order(:created_at).limit(2)
```

this method is much preferred:

```ruby
def fetch_recent_records(reference, limit: 2)
  objects.where(reference_id: reference.id).order(:created_at).limit(2)
end
```

## Create first model

Entities and repositories are tightly connected with each other, so [Hanami](http://hanamirb.org/) provides a generator to create both at one go:

```
bundle exec hanami generate model word
```

This will generate following output:

```
      create  lib/shyshka/entities/word.rb
      create  lib/shyshka/repositories/word_repository.rb
      create  spec/shyshka/entities/word_spec.rb
      create  spec/shyshka/repositories/word_repository_spec.rb
```

Notice that files are placed to the *lib/project_name* folder, not to the *apps/web* folder. This is because they define our domain model and this will be shared across multiple application that we could create under one [Hanami](http://hanamirb.org/) project.

## Generate migration

Our next task is to alter the database structure, so it will be able to store word records. [Hanami](http://hanamirb.org/) way of managing database schema is [migration](http://hanamirb.org/guides/migrations/overview/). So, in order to prepare database we need to use another generator:

```
bundle exec hanami generate migration add_words
```

This will generate migration file and store it inside *db/migrations* folder.

Name of the migration file is prefixed with timestamp. 

Change migration file so it will look like this:

```ruby

Hanami::Model.migration do
  change do
    create_table :words do
      primary_key :id

      column :name, String, null: false
      column :translation, String, null: false

      column :created_at, DateTime, null: false
      column :updated_at, DateTime, null: false
    end
  end
end

```

We've created table named *words* with primary key *id*, string columns *name* and *translation*, and two columns storing info about creation and update time of the record.

To run this migration type:

```
bundle exec hanami db prepare
```

## How to use entities and repositories?

Since we are using SQL database - our schema will be derived from the db structure, so there is no need to change *lib/shyshka/entities/word.rb* file. Let's write a test which will confirm that we can initialize our entity with expected arguments. Fill *spec/shyshka/entities/word_spec.rb* with this code:

```ruby
require 'spec_helper'

describe Word do
  it 'can be initialized with attributes' do
    word = Word.new(name: 'pies', translation: 'dog')
    word.name.must_equal 'pies'
    word.translation.must_equal 'dog'
  end
end
```

When we run our test suite we will see the following error:

```
PG::UndefinedTable: ERROR:  relation "words" does not exist (Sequel::DatabaseError)
```

This is because we migrated only our development database and not the one that is used during tests. To fix this type this command:

```
HANAMI_ENV=test bundle exec hanami db prepare
```

As you see, *HANAMI_ENV* env variable defines which environment should be used. By default it's set to *development*.

After running tests now with the command:

```
bundle exec rake test
``` 

we should see, that all the tests are passing.

Now we can test our repository as well. To do so we need to access the hanami console. We can do this with this command:

```
bundle exec hanami console
```

We can initialize repository by typing:

```
repository = WordRepository.new
```

We can check if we have any words stored in the db:

```
repository.all
=> []
```

To create a new word you type:

```
word = repository.create(name: "kot", translation: "cat")
=>  => #<Word:0x007ffdb6369c58 @attributes={:id=>1, :name=>"kot", :translation=>"cat", :created_at=>2017-04-01 10:36:28 UTC, :updated_at=>2017-04-01 10:36:28 UTC}>
```

We can verify that the word has been stored by typing:

```
repository.all
=> [#<Word:0x007ffdb6351a40 @attributes={:id=>1, :name=>"kot", :translation=>"cat", :created_at=>2017-04-01 10:36:28 UTC, :updated_at=>2017-04-01 10:36:28 UTC}>]
```

or:

```
repository.find(word.id)
 => #<Word:0x007ffdb6329860 @attributes={:id=>1, :name=>"kot", :translation=>"cat", :created_at=>2017-04-01 10:36:28 UTC, :updated_at=>2017-04-01 10:36:28 UTC}>
```

You can find more information about repository API [here](http://hanamirb.org/guides/models/repositories/).

## Displaying data from the database

Now we have tools to deal with the database, so it's the highest time to get rid of the hard-coded data from our list of words.

Let's start with the feature spec we've created to test list of words. Open *spec/web/features/list_words_spec.rb* and let's use repository to add some words to the test database there:

```ruby
require 'features_helper'

describe 'List words' do
  let(:repository) { WordRepository.new }
  before do
    repository.clear

    repository.create(name: 'pies', translation: 'dog')
    repository.create(name: 'kot', translation: 'cat')
  end  

  it 'displays list of the words' do
    visit '/'

    within '#words' do
      assert page.has_css?('.word', count: 2), 'Expected to find 2 words'
    end
  end
end
```

Run test and make sure that all the tests are passing.

Next step is to change the view as well. We will stop here for a second to think what we actually want to achieve.
We want to handle 2 scenarios:
- list all the words we have in our database
- display message that we don't have any words yet if we just starting using our system

We also want to make sure that our template will have all the data accessible that it needs to render the view. In our case it's the list of words.

Following the philosophy of [BDD](https://en.wikipedia.org/wiki/Behavior-driven_development) let's add some tests first. Change *spec/web/controllers/words/index_spec.rb* to look like this:

```ruby
require 'spec_helper'
require_relative '../../../../apps/web/views/words/index'

describe Web::Views::Words::Index do
  let(:exposures) { Hash[words: []] }
  let(:template)  { Hanami::View::Template.new('apps/web/templates/words/index.html.erb') }
  let(:view)      { Web::Views::Words::Index.new(template, exposures) }
  let(:rendered)  { view.render }

  it 'exposes #words' do
    view.words.must_equal exposures.fetch(:words)
  end

  describe 'when there are no words' do
    it 'shows a placeholder message' do
      rendered.must_include('<p class="placeholder">There are no words yet.</p>')
    end
  end

  describe 'when there are words' do
    let(:word1)     { Word.new(name: 'pies', translation: 'dog') }
    let(:word2)     { Word.new(name: 'kot', translation: 'cat') }
    let(:exposures) { Hash[words: [word1, word2]] }

    it 'lists them all' do
      rendered.must_include('pies')
      rendered.must_include('kot')
    end

    it 'hides the placeholder message' do
      rendered.wont_include('<p class="placeholder">There are no words yet.</p>')
    end
  end  
end
```

Our test should fail now because we still have the words hard-coded in the template.

Let's fix it now.

```ruby
<h1>All words</h1>

<% if words.any? %>
  <div id="words">
    <% words.each do |word| %>
      <div class="word row">
        <div class="col-xs-6"><%= word.name %></div>
        <div class="col-xs-6"><%= word.translation %></div>
      </div>
    <% end %>
  </div>
<% else %>
  <p class="placeholder">There are no words yet.</p>
<% end %>
```

When we run our tests now we will see that it still fails. 

```
NameError: undefined local variable or method `words' for #<Hanami::View::Rendering::Scope:0x007fe6b71cef30>
```

It complains that it does not know what *words* are. This is expected since we did not expose the variable from the controller, where it mend to be created to the view. To fix this make your *apps/web/contollers/words/index.rb* file look like this:

```ruby
module Web::Controllers::Words
  class Index
    include Web::Action

    expose :words

    def call(params)
      @words = WordRepository.new.all
    end
  end
end
``` 

After this change we should see that all the tests are passing. 

## How can we test controllers?

Last thing is making sure that our controller will expose the variables we expect. Let's add out first controller test then.

Controllers are supposed to be a thin layer that gets the request params, calls repositories to fetch the necessary data and then exposes those data to the view so it can handle rendering properly. So to test controller sufficiently all we need to do is to make sure it returns correct status code and exposes correct variables. Our *spec/web/controllers/words/index_spec.rb* file should look like this:

```ruby
require 'spec_helper'
require_relative '../../../../apps/web/controllers/words/index'

describe Web::Controllers::Words::Index do
  let(:action) { Web::Controllers::Words::Index.new }
  let(:params) { Hash[] }
  let(:repository) { WordRepository.new }

  before do
    repository.clear

    @word = repository.create(name: 'pies', translation: 'dog')
  end

  it 'is successful' do
    response = action.call(params)
    response[0].must_equal 200
  end

  it 'exposes all words' do
    action.call(params)
    action.exposures[:words].must_equal [@word]
  end
end
```

Now all our tests should pass.

## Summary

We achieved quite a lot here:
- we added some dynamic data to our list of words through entities and repositories
- we learned how to test views
- what is the purpose of the controller and how to test it

Code for this project is open sourced and can be found [here](https://github.com/detfis/shyshka) 

Our next task will be adding a form to create new words, so stay tuned!


