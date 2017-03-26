---
layout: dsp_post
title: Shyshka - first feature
categories: dajsiepoznac2017 hanami shyshka
description: We will add first feature to Shyshka in a BDD way.
---

# First feature in Hanami #

In the [last blog post]({{ site.url }}/generate-project-in-hanami-with-postgresql/) we generated [Hanami](http://hanamirb.org/) project with [PostgreSQL](https://www.postgresql.org/) as a database. Now we will add our first feature - list of all words.

[Hanami](http://hanamirb.org/) encourages [Behavioral Driven Architecture](https://en.wikipedia.org/wiki/Behavior-driven_development) as a way of creating web applications. I'm a big fan of it so we will follow the suggested path.

## First feature spec ##

The main idea behind [BDD](https://en.wikipedia.org/wiki/Behavior-driven_development) is that you first create high-level test and then add the minimal amount of code that will make the code pass.

In our case, we want to see the list of words when we visit the main page of the application. To create our first feature spec we need to create new file **spec/web/features/visit_home_page_spec.rb**. Our spec should look like this:

```ruby
require 'features_helper'

describe 'Visit home page' do
  it 'is successful' do
    visit '/'

    page.body.must_include('All words')
  end
end
```

When we type `bundle exec rake test` - which is the command to run all the test in our application - we should see the following error:

```
Run options: --seed 47149

# Running:

F

Finished in 0.018929s, 52.8290 runs/s, 105.6580 assertions/s.

  1) Failure:
Visit home page#test_0001_is successful [/Users/kamil/projects/shyshka/spec/web/features/list_words_spec.rb:7]:
Expected "<!DOCTYPE html>\n<html>\n  <head>\n    <title>404 - Not Found</title>
```

This error shows us that we don't have the root path. Let's fix this.

## First feature in Hanami ##

The first thing we will do is to create a route. To do this let's open **apps/web/config/routes.rb** and this line **root to: 'words#index'**.
It means that whenever someone visits root page of our application, action **index** from the **words** controller should be executed.

To create this action we will use [generators](http://hanamirb.org/guides/command-line/generators/) which will create all the files that we need.

```
bundle exec hanami generate action web words#index
```

We should see the following output:

```
      create  spec/web/controllers/words/index_spec.rb
      create  apps/web/controllers/words/index.rb
      create  apps/web/views/words/index.rb
      create  apps/web/templates/words/index.html.erb
      create  spec/web/views/words/index_spec.rb
      insert  apps/web/config/routes.rb
```

To make our feature test pass, the last thing is to change the **apps/web/templates/words/index.html.erb** file. 

```html
<h1>All words</h1>
```

Add the line above to this file and run tests again.

This is what you should see: 

```
Run options: --seed 19286

# Running:

.

Finished in 0.011854s, 84.3600 runs/s, 168.7200 assertions/s.

1 runs, 2 assertions, 0 failures, 0 errors, 0 skips
```

Great! we have our first feature test passing!

## Let's add more behavior ##

OK, so we tested the scenario of visiting the homepage. In our case, the homepage is also the page where all the words will be listed. To implement this scenario let's create second feature test **spec/web/features/list_words_spec.rb**. This file should look like this:

```ruby
require 'features_helper'

describe 'List words' do
  it 'displays list of the words' do
    visit '/'

    within '#words' do
      assert page.has_css?('.word', count: 2), 'Expected to find 2 words'
    end
  end
end
```

After running tests we should see this error:

```
  1) Error:
List words#test_0001_displays list of the words:
Capybara::ElementNotFound: Unable to find css "#words"
```

The philosophy of BDD is to create the least amount of code to make tests pass, so all we need to do is to alter again the **apps/web/templates/words/index.html.erb** file:

```html
<h1>All words</h1>

<div id="words">
  <div class="word">
    <h2>kot</h2>
    <p>cat</p>
  </div>

  <div class="word">
    <h2>pies</h2>
    <p>dog</p>
  </div>
</div>

```

After running our tests again we can confirm that everything passes.

## Summary ##

We have the test passing, but we are not finished yet. Hard-coding words in the template is definitely not how we want our dictionary to work. 
The goal of our application is to be a dictionary of words and phrases. We need to add a way of saving words to the database. We will work on this in the next post. 
