# Rack::Authorize

Rack::Authorize is a Rack middleware to authorize api access. We know there is Cancan which is a great authorization library for Ruby on Rails. But the rules of Cancan are defined for Ruby Class, when we create web services, it's a common task to restrict the access to api endpoints. That's what Rack::Authorize focus on, it's only used for api stuff. Rack::Authorize can be used in any Ruby web framework since it's a Rack middleware. Thanks Rack.

## Installation

Add this line to your application's Gemfile:

```ruby
gem 'rack-authorize', :git => 'git@github.com:yanguango/rack-authorize.git'
```

This gem is still in development, so it's not posted to rubygems.org

And then execute:

    $ bundle

Or install it yourself as:

    $ gem install rack-authorize

## Usage

### Define Abilities
You can define ability with `can` macros, which inspired by Cancan. Also you can use `can_get`, `can_post` helpers to define conveniently.

```ruby
class Ability
  include Rack::Authorize::Ability

  def initialize(user)
    can :get, "/api/articles" 
    # user can access "/api/things" with GET method
    
    can :post, "/api/articles" if user.role == 'writer' 
    # user can post article if he is a writer
    
    can :all, "/api/comments" 
    # user can access "/api/comments" with any methods
    
    can_get, :all 
    # user can access all api endpoints with GET method
    
    can_all, :all if user.is_super? 
    # super user has no restriction
  end
end
```

### Use Rack middleware
As a Rack middleware, you can use Rack::Authorize in any Rack compatible framework, what you need to do is just use this middleware at appropriate position.

```ruby
use Rack::Authorize do |method, path|
  ability = Ability.new(@current_user)
  ability.can?(method, path)
end
```

### Framework Integration
### Sinatra
```ruby
class App < Sinatra::Base
  use Rack::Auth::Basic do |username, password|
    @current_user ||= User.authenticate!(username, password)
  end

  use Rack::Authorize do |method, path|
    ability = Ability.new(@current_user)
    ability.can?(method, path)
  end

  get '/api' do
    "Hello World!"
  end
end
```

### Grape with Rails
```ruby
class AdminAPI < Grape::API

  version 'v1', using: :header
  format :json
  prefix 'api'
  
  namespace :admin do
    use Rack::Auth::Basic do |username, password|
      @current_use ||= AdminUser.authenticate!(username, password)
    end

    use Rack::Authorize do |method, path|
      ability = Ability.new(@current_user)
      ability.can?(method, path)
    end
    
    resources :users do
      get '/' do
        "Hello World!"
      end
    end
  end
end
```

## Todo
* Path prefix
* Api endpoint whitelist
* ... 

## Development

After checking out the repo, run `bin/setup` to install dependencies. Then, run `rake test` to run the tests. You can also run `bin/console` for an interactive prompt that will allow you to experiment.

To install this gem onto your local machine, run `bundle exec rake install`. To release a new version, update the version number in `version.rb`, and then run `bundle exec rake release`, which will create a git tag for the version, push git commits and tags, and push the `.gem` file to [rubygems.org](https://rubygems.org).

## Contributing

Bug reports and pull requests are welcome on GitHub at https://github.com/yanguango/rack-authorize. This project is intended to be a safe, welcoming space for collaboration, and contributors are expected to adhere to the [Contributor Covenant](contributor-covenant.org) code of conduct.


## License

The gem is available as open source under the terms of the [MIT License](http://opensource.org/licenses/MIT).

