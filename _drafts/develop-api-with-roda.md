---
layout: post
title: "Building your API with roda and rails"
description: ""
tags:
  - ruby
  - rails
  - roda
  - api
---

## Introduction

Sometimes you need to build API serveice in your rails application.
Of course you can use popular solutions as grape or rails-api gems.
And also you can see real examples in gitlab or rubygems projects. 

But today I want to talk about other framework. It's roda.
Roda was created by jeremyevans and it's fast and simple ruby routing framework.
Why you should a look on it?

I have a list with advantages of this framework:
1. Fast;
2. Simple;
3. This framework provide simple way to working with big count of different plugins;
4. You can use any architecture with it;

So I told that roda is fast. If you want to see this you can check this benchmark repository and you'll see that roda is really fast.
For example, I run benchmarks on localy and get this result:

```
Framework            Requests/sec  % from best
----------------------------------------------
mustermann                9389.60       100.0%
roda                      9252.03       98.53%
rack                      9246.16       98.47%
hanami-router             6240.81       66.47%
sinatra                   2935.63       31.26%
grape                     2512.17       26.75%
```

## Integration

Roda based on rack. That's why that integration with rails will be very simple.
Firstly you need to add roda gem to your gemfile:

```ruby
gem 'roda'
```

Secondly you need to create simple roda application.
I use `json` and `all_verbs` plugins.
First needs to JSON responce and second provides all REST methods for your application (like `put`, `patch`, etc).
I puted my roda application in to `lib/api/base.rb` path but you can use any place what you waht.

```ruby
# lib/api/base.rb
require 'roda'
require 'json'

module API
  class Base < Roda
    plugin :json
    plugin :all_verbs

    route do |r|
      r.on "v1" do
        r.get 'hello' do
          { hello: :world }
        end
      end
    end
  end
end
```

After that you need to mount your application in `config/routes.rb`.
This is done as well as any other rack application:

```ruby
# config/routes.rb
require './lib/api/base'
mount API::Base => '/api'
```

And the last part, we need to split our application to different modules with different logic and routes.
For this we need to create required module. In example I created `Users` module with all REST routes:

```ruby
# lib/api/users.rb
module API
  class Users < Roda
    plugin :json
    plugin :all_verbs

    route do |r|
      r.is 'users' do
        r.get do
          { users: User.last(10) }
        end

        r.post do
          { user: User.new }
        end
      end

      r.is 'users/:id' do |id|
        user = User.find(id)

        r.get do
          { user: user}
        end

        r.patch do
          { user: 'updated' }
        end

        r.delete do
          { user: 'deleted' }
        end
      end
    end
  end
end
```

Finnaly you need to mount our module to roda application. It is very simple too.

```ruby
# lib/api/base.rb
require 'roda'
require 'json'
require 'users'

module API
  class Base < Roda
    plugin :json
    plugin :all_verbs

    route do |r|
      r.on "v1" do
        r.run API::Users
      end
    end
  end
end
```

That's all. Now we have a simple roda API application which integrated to our rails app.

## Problems / future

In the last part I want to list problems and ideas what I want to solve in future.

1. Roda don't have a swagger integrating from the box. Now I'm thinking about using swagger-block gem for it.
2. Also roda don't typecast your params. I know that grape use virtus gem for this. And this feature you need realize by self too.

## Conclusion 

On this blog post I wanted to show that you're not limited only in popular API server gems.
As you can see roda have amazing ideas and properties like modulatiry, simplicity, speed and stability.
Also this framework have a simple way to integration in to your rails application.

Happy hacking! emoji


[gitlab-api]: https://github.com/gitlabhq/gitlabhq/tree/master/lib/api
[rubygems-api]: https://github.com/rubygems/rubygems.org/tree/master/app/controllers/api
[jeremyevans]: https://github.com/jeremyevans
[roda]: http://roda.jeremyevans.net
[grape]: http://www.ruby-grape.org
[benchmarks]: https://github.com/luislavena/bench-micro
[rodauth]: http://rodauth.jeremyevans.net
[why-roda]: http://roda.jeremyevans.net/why.html
