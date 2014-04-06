# grape-swagger

[![Build Status](https://travis-ci.org/tim-vandecasteele/grape-swagger.svg?branch=master)](https://travis-ci.org/tim-vandecasteele/grape-swagger)

## What is grape-swagger?
grape-swagger provides an autogenerated documentation for your [grape](https://github.com/intridea/grape)-API. The generated documentation is Swagger-compliant, meaning it can easily be discovered in [Swagger UI](https://github.com/wordnik/swagger-ui)

## Related projects

* [Grape](https://github.com/intridea/grape)
* [Swagger UI](https://github.com/wordnik/swagger-ui)

## Installation

Add to your Gemfile:

```gem 'grape-swagger'```

## Usage

Once you have the gem installed, mount all your different APIs (with ```Grape::API``` superclass) on a root node. In the root class definition, also include ```add_swagger_documentation```, this sets up the system and registers the documentation on '/swagger_doc.json'. Setup done, you can restart your local server now.


``` ruby
require 'grape-swagger'

module API
  class Root < Grape::API
    mount API::Cats
    mount API::Dogs
    mount API::Pirates
    add_swagger_documentation
  end
end
```

To explore your API, either download [Swagger UI](https://github.com/wordnik/swagger-ui) and set it up yourself or go to the [online swagger demo](http://petstore.swagger.wordnik.com/) and enter your localhost url documentation root in the url field (probably something in the line of http://localhost:3000/swagger_doc.json).
If you use the online demo, make sure your API supports foreign requests by enabling CORS in grape, otherwise you'll see the API description, but requests on the API won't return. You can do this by putting below code in your Grape API definition:

```` ruby
before do
    header['Access-Control-Allow-Origin'] = '*'
    header['Access-Control-Request-Method'] = '*'
end
````

## Configure

You can pass a hash with some configuration possibilities to ```add_swagger_documentation```, all of these are optional:

* ```:target_class``` The API class to document, default `self`
* ```:mount_path``` The path where the API documentation is loaded, default '/swagger_doc'
* ```:class_name```
* ```:markdown``` Allow markdown in `notes`, default `false`
* ```:hide_format``` , Don't add '.(format)' to the end of URLs, default `false`
* ```:api_version``` Version of the API that's being exposed
* ```:base_path``` Basepath of the API that's being exposed, this configuration parameter accepts a Proc to evaluate base_path, useful when you need to use request attributes to determine the base_path.
* ```:authorizations``` Added to the `authorizations` key in the JSON documentation
* ```:include_base_url``` Add base path to the URLs, default `true`
* ```:root_base_path``` Add `basePath` key to the JSON documentation, default `true`
* ```:info``` Added to the `info` key in the JSON documentation
* ```:models``` Allows adds an array with the entities for build models specifications. You need to use grape-entity gem.
* ```:hide_documentation_path``` Don't show the '/swagger_doc' path in the generated swagger documentation
* ```:format```

## Swagger Header Parameters

Swagger also supports the documentation of parameters passed in the header. Since grape's ```params[]``` doesn't return header parameters we can specify header parameters seperately in a block after the description.

``` ruby
desc "Return super-secret information", {
  headers: {
    "XAuthToken" => {
      description: "Valdates your identity",
      required: true
    },
    "XOptionalHeader" => {
      description: "Not reallly needed",
      required: false
    }
  }
}
```

## Hiding an Endpoint

You can hide an endpoint by adding ```:hidden => true``` in the description of the endpoint:

``` ruby
desc 'Hide this endpoint', {
  :hidden => true
}
```


# Overriding auto generated nickname for an endpoint

You can specify a swagger nickname to use instead of the auto generated name by adding ```:nickname => 'string'``` in the description of the endpoint.

``` ruby
desc 'get a full list of Pets', {
  :nickname => 'getAllPets'
}
```


## Grape Entities

Add the [grape-entity](https://github.com/agileanimal/grape-entity) gem to our Gemfile.
Please refer to the [grape-entity documentation](https://github.com/intridea/grape-entity/blob/master/README.md)
for more details.

The following example exposes statuses. And exposes statuses documentation adding :type and :desc.

```ruby
module API

  module Entities
    class Status < Grape::Entity
      expose :text, :documentation => { :type => "string", :desc => "Status update text." }
    end
  end

  class Statuses < Grape::API
    version 'v1'

    desc 'Statuses index', {
      :entity => API::Entities::Status
    }
    get '/statuses' do
      statuses = Status.all
      type = current_user.admin? ? :full : :default
      present statuses, with: API::Entities::Status, :type => type
    end
  end
end
```

### Relationships

#### 1xN

```ruby
module API
  module Entities
    class Client < Grape::Entity
      expose :name, :documentation => { :type => "string", :desc => "Name" }
      expose :addresses, using: Entities::Address, documentation: {type: 'Address', desc: 'Addresses.', param_type: 'body', is_array: true}
    end

    class Address < Grape::Entity
      expose :street, :documentation => { :type => "string", :desc => "Street." }
    end

  end

  class Clients < Grape::API
    version 'v1'

    desc 'Clients index', {
      params: Entities::Client.documentation
    }
    get '/clients' do
      ...
    end
  end
  add_swagger_documentation models: [Entities::Client, Entities::Address]
end
```

#### 1x1

Note: `is_array` is `false` by default.

```ruby
module API
  module Entities
    class Client < Grape::Entity
      expose :name, :documentation => { :type => "string", :desc => "Name" }
      expose :address, using: Entities::Address, documentation: {type: 'Address', desc: 'Addresses.', param_type: 'body', is_array: false}
    end

    class Address < Grape::Entity
      expose :street, :documentation => { :type => "string", :desc => "Street" }
    end
  end

  class Clients < Grape::API
    version 'v1'

    desc 'Clients index', {
      params: Entities::Client.documentation
    }
    get '/clients' do
      ...
    end
  end
  add_swagger_documentation models: [Entities::Client, Entities::Address]
end
```

## Swagger Additions

The grape-swagger gem allows you to add an explanation in markdown in the notes field. Which would result in proper formatted markdown in Swagger UI. The default Swagger UI doesn't allow HTML in the notes field, so you need to use an adapted version of Swagger UI (you can find one at https://github.com/tim-vandecasteele/swagger-ui/tree/vasco).

We're using [kramdown](http://kramdown.rubyforge.org) for parsing the markdown, specific syntax can be found [here](http://kramdown.rubyforge.org/syntax.html).

Be sure to enable markdown in the `add_swagger_documentation` with 'markdown: true'.

``` ruby
desc "Reserve a virgin in heaven", {
  :notes => <<-NOTE
    Virgins in Heaven
    -----------------

    > A virgin doesn't come for free

    If you want to reserve a virgin in heaven, you have to do
    some crazy stuff on earth.

        def do_good
          puts 'help people'
        end

    * _Will go to Heaven:_ Probably
    * _Will go to Hell:_ Probably not
  NOTE
}
```

You can also document the HTTP status codes that your API returns with this syntax:

``` ruby
get '/', :http_codes => [
  [400, "Invalid parameter entry"],
] do
  ...
end
```

## Contributing to grape-swagger

See [CONTRIBUTING](CONTRIBUTING.md).

## Copyright and License

Copyright (c) 2012-2014 Tim Vandecasteele and contributors. See [LICENSE.txt](LICENSE.txt) for details.
