# ScopedSerializer

[![Gem Version](http://img.shields.io/gem/v/scoped_serializer.svg?style=flat)][gem]
[![Build Status](http://img.shields.io/travis/booqable/scoped_serializer.svg?style=flat)][travis]
[![Coverage Status](http://img.shields.io/coveralls/booqable/scoped_serializer.svg?style=flat)][coveralls]
[![Code Climate](http://img.shields.io/codeclimate/github/booqable/scoped_serializer.svg?style=flat)][codeclimate]
[![Dependency Status](http://img.shields.io/gemnasium/booqable/scoped_serializer.svg?style=flat)][gemnasium]

[gem]: https://rubygems.org/gems/scoped_serializer
[travis]: http://travis-ci.org/booqable/scoped_serializer
[coveralls]: https://coveralls.io/r/booqable/scoped_serializer
[codeclimate]: https://codeclimate.com/github/booqable/scoped_serializer
[gemnasium]: https://gemnasium.com/booqable/scoped_serializer

ScopedSerializer serializes your models based on context.

## Installation

```
gem 'scoped_serializer'
```

## Quick summary

Setup serialization scopes and render different output based on context. Use it when you're developing an API and return different data per action (collections/resources).

__Quick example__
```ruby
class OrderSerializer < ScopedSerializer::Serializer
  attributes :id, :status
  has_one :customer

  scope :resource do
    has_many :products => :stock_items
  end

  scope :collection do
    has_many :notes
  end
end

# ScopedSerializer.render(@order) # Default output
# ScopedSerializer.render(@order, :scope => :resource) # Includes products and stock_items
# ScopedSerializer.render(Order.all, :scope => :collection) # Array of orders, each includes notes
```

It supports associations and eager loading out-of-the-box.

## The problem

While developing the API for [Booqable Rental Software](http://booqable.com/ "Booqable Rental Software"), we ran into problems where we would return different data based on context. For example we would list orders but not nested associations. When the user requests a single order, we do want to render nested associations. This resulted in overly complicated serializers with a lot of conditional rendering.

The alternative would be to create different serializer based on context, such as `OrderCollectionSerializer` and `OrderResourceSerializer`.

## The solution

Using ScopedSerializer you can create one serializer with different configuration based on scope.

### Example

```ruby
class OrderSerializer < ScopedSerializer::Serializer
  attributes :id, :status
  has_one :customer

  scope :resource do
    has_many :products => :stock_items
  end
end
```

The above example will render the following outputs:

__Example for `ScopedSerializer.render(@order)`__
```json
{
  "order": {
    "id": 1,
    "status": "reserved",
    "customer": { ... }
  }
}
```

__Example for `ScopedSerializer.render(@order, :scope => :resource)`__
```json
{
  "order": {
    "id": 1,
    "status": "reserved",
    "customer": { ... },
    "products": [
      {
        "id": 1,
        "name": "Sony NX5",
        "stock_items": [...]
      }
    ]
  }
}
```

## Scopes

Scopes are not pre-defined and thus can be anything you want. Whatever you need. In our application we render the scope `:collection` for index actions and `:resource` for resource actions. This entirely depends on your implementation. If a scope does not exist it will simply use the default settings.

## Associations

All associations are supported;

- has_many
- has_one (or belongs_to if you prefer it)

ScopedSerializer takes care of eager loading associations. Need custom eager loading? No problem, just override it.

```ruby
class OrderSerializer < ScopedSerializer::Serializer
  scope :resource do
    has_many :products => :stock_items, :preload => false
  end
  scope :something_else do
    has_many :products => :stock_items, :preload => { :stock_items => :item }
  end
end
```

ScopedSerializer __does not eager load collections__. You will need to manually eager load the nested associations. For example:

```ruby
# No eager loading at all
ScopedSerializer.render(Order.all, :scope => :resource)
# Manually eager load
ScopedSerializer.render(Order.includes(:products => :stock_items), :scope => :resource)
```

__More options__

```ruby
class OrderSerializer < ScopedSerializer::Serializer
  scope :resource do
    # Render a specific scope
    has_many :products => :stock_items, :scope => :resource

    # Render a specific serializer
    has_many :products => :stock_items, :serializer => ItemSerializer
  end
end
```

## Contributing

- Fork it
- Create your feature branch (git checkout -b my-new-feature)
- Commit your changes (git commit -am 'Add some feature')
- Push to the branch (git push origin my-new-feature)
- Create a new Pull Request
