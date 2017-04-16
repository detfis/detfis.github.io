---
layout: dsp_post
title: Hanami 1.0.0 released
categories: dajsiepoznac2017 hanami shyshka
description: How to upgrade Hanami app to version 1.0.0
---

# HANAMI 1.0.0 has been released!

06.04.2017 is a big date for [Hanami](http://hanamirb.org/) framework - version 1.0.0 has been released! Since version 0.9.2 (which we were using) there should not be any breaking changes in the API, so the upgrade of the [Shyshka](https://github.com/detfis/shyshka) project should go seamlessly. 

## How to upgrade to version 1.0?

There is not that much we need to do. [Hanami](http://hanamirb.org/) maintainers provide description what should be changed [here](http://hanamirb.org/guides/upgrade-notes/v100/).

So, the first thing we need to do is to update the *Gemfile*

```ruby
gem 'hanami',       '~> 1.0'
gem 'hanami-model', '~> 1.0'
```

Next udpate the gems by running:

```ruby
bundle update hanami hanami-model
```

One of the new things in [Hanami](http://hanamirb.org/) framework is the **_config/boot.rb_**, which should look like this:

```ruby
require_relative './environment'
Hanami.boot
```

This file will be used for booting our project from external commands, for instance to use it with Sidekiq.

Another thing that has slightly changed is where we setup the logger for the application. We used to do this in **_apps/web/application.rb_** and we have to set this up in *config/environment.rb*.

Delete then all **_logger_** refences from the **_apps/web/application.rb_** file and add this code to the **_config/environment.rb_**: 

```ruby
  environment :development do
    # See: http://hanamirb.org/guides/projects/logging
    logger level: :info
  end

  environment :production do
    logger level: :info, formatter: :json

    mailer do
      delivery :smtp, address: ENV['SMTP_HOST'], port: ENV['SMTP_PORT']
    end
  end
```

Note, that this should be added after the general settings like **_mount_**, **_model_** or **_mailer_**.

The last step is to add **_lib/shyshka.rb_** file that will define our main module. This file should look like this:

```ruby
module Shyshka
end
```

Summary

That's all we need to do to upgrade [Hanami](http://hanamirb.org/) application from version 0.9.2 to 1.0. After adding this our tests should still pass and application should work the same as previously.





