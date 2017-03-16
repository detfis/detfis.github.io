---
layout: dsp_post
title: Generate project in Hanami with PostgreSQL
categories: dajsiepoznac2017 hanami shyshka
description: Generate Hanami project with PostgreSQL
---

# Generate project in Hanami with PostgreSQL

In this blog post, I'll show you how to setup a [Hanami](http://hanamirb.org/) project with [PostgreSQL](https://www.postgresql.org/) as a database. 

## Setup

I'll assume that you have [ruby](https://www.ruby-lang.org/). If not - [here](https://www.ruby-lang.org/en/documentation/installation/) you can find detailed instruction how to do it. I suggest to use ruby version manager, my favorite one is [RVM](https://rvm.io/). 

Another prerequisite is gem dependency manager called [Bundler](http://bundler.io/).

```ruby
gem install bundler
```

In this project I will also use [PostgreSQL](https://www.postgresql.org/), so make sure you have it on your computer as well. 

The next step is to install [Hanami](http://hanamirb.org/) gem. You can do it by running this command: 

```ruby
gem install hanami
```

If this run successfully, we can generate our first [Hanami](http://hanamirb.org/) project: 

```ruby
hanami new shyshka --database=postgres
```

Now let's go to the project folder and install gem dependencies using [Bundler](http://bundler.io/):

```ruby
cd shyshka
bundle install
```

Our next step will be to generate the database. Database configuration can be found in `.env.development` file and for test environment in `.env.test`. After setting up correct credentials we can run this command:

```ruby
bundle exec hanami db prepare 
```

and this one to setup test environment database: 

```ruby
HANAMI_ENV=test bundle exec hanami db prepare
```

After that we can launch development server: 

```ruby
bundle exec hanami server
```

Now your web app is running and after typing `http://localhost:2300` in your browser you should be able to see this:

![hanami_default_app]({{ site.url }}/assets/images/hanami_default.png)

## Sum up

OK, that's it for today. We managed to setup development environment, generate and run our first [Hanami](http://hanamirb.org/) project with [PostgreSQL](https://www.postgresql.org/) as a database. In next post, we will learn how to add resources to the project and how to write tests. Code for [Shyshka](https://github.com/detfis/shyshka) is open sourced and can be found [here](https://github.com/detfis/shyshka). 

Thanks and see you soon!
