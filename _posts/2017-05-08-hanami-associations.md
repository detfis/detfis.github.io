---
layout: dsp_post
title: Associations between models in Hanami
categories: dajsiepoznac2017 hanami shyshka
description: How to create associations between models in Hanami
---

# Adding new entity

Today we will work on extending [Shyshka](https://github.com/detfis/shyshka) application with the possibility of adding examples to the words. To each word we will be able to add as many examples of usages as we wish.

## What are associations?

In simple words, associations model relationships between entities. In our case, our one entity will be **_Word_** and the second one will be **_Example_**. Each **_Word_** can have many **_Examples_** but every **_Example_** can have only one **_Word_** associated to it. 

## Associations in Hanami

Since associations represents how data is linked in the database, we define associations in [repositories](http://hanamirb.org/guides/models/repositories/). 

Unlike associations in [Ruby on Rails](http://rubyonrails.org/) framework, associations does not add any public method to the repository. Instead, [Hanami](http://hanamirb.org/) prefers **_explicit interface_** - which means that if we want to perform specific action, like adding word with examples, we need to explicitly add method to perform this operation. This design choice is supposed to prevent repositories from having extensive and mostly unused public API.

Another difference between [Ruby on Rails](http://rubyonrails.org/) and [Hanami](http://hanamirb.org/) associations is lacking of **_eager loading_** - if we want to load a **_Word_** with associated **_Examples_** we have to add additional method to do it. If we don't add this method, then the result of the association will be `nil`.

## Adding `has_many` association

The first we need to do is creating a new entity called **_Example_**.

```ruby
bundle exec hanami generate migration create_examples
``` 

Open newly created migration file and add this code:

```ruby
Hanami::Model.migration do
  change do
    create_table :examples do
      primary_key :id
      foreign_key :word_id, :words, on_delete: :cascade, null: false

      column :text, String, null: false

      column :created_at, DateTime, null: false
      column :updated_at, DateTime, null: false
    end    
  end
end
```

Most of the code above is familiar - we define a new table called **_examples_**, set **_primary_key_** and some columns. The new thing here is **_foreign_key_** - we specify here a column that will link **_Example_** with a **_Word_**. By setting `on_delete: :cascade` we define how this entry should behave when associated **_Word_** will be deleted - in our case we want to delete **_Examples_** as well.
 
To run the migration type in the console:

```
bundle exec hanami db prepare
```

Now, let's generate **_Example_** model. We will do this by typing:

```
bundle exec hanami generate model example
```

Next, let's open `WordRepository` and define the association with **_Example_** here:

```ruby
class WordRepository < Hanami::Repository
  associations do
    has_many :examples
  end

  def create_with_examples(data)
    assoc(:examples).create(data)
  end

  def add_example(word, data)
    assoc(:examples, word).add(data)
  end  

  def find_with_examples(id)
    aggregate(:examples).where(id: id).as(Word).one
  end
end
```

We have defined a `has_many` association between **_Word_** and **_Example_** models and we have added three new methods:
- `create_with_examples` for creating **_Word_** with **_Example_** at one go
- `add_example` for adding **_Example_** to an existing **_Word_**
- `find_with_examples` for retrieving **_Word_** with all **_Examples_** preloaded

## Summary

Today we've stepped out of basic CRUD app and we've extended our app with the first association. In the next post, we will prepare our frontend for adding and displaying words with associated examples.
