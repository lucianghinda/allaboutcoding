## How to assign multiple instance variables to a class in Ruby

I was inspired to write this article by a [question](https://twitter.com/adrianthedev/status/1499342584912699393) asked by @[Adrian Marin](@adrianthedev) in the Twitter Ruby on Rails community and the responses I read there.

%[https://twitter.com/adrianthedev/status/1499342584912699393]

# Context

Let's say there you want to create a class that can accept in initializer a number of keyword arguments that will be used to set defaults for instance variables with the same name as the keyword arguments. 

Example:

```ruby
class Receipe
  attr_accessor  :flour_type
  attr_accessor  :number_of_people

  def initialize(flour_type:, number_of_people: )
    @flour_type = flour_type
    @number_of_people = number_of_people
  end
end


```

# Problem

As described by Adrian, you notice that if you add more variables, the number of lines in the initializer will increase with each variable you add there. 

Let's say you want to add 4 more variables: 

```ruby
class BreadReceipe
  attr_accessor  :flour_type
  attr_accessor  :number_of_people
  attr_accessor  :salt
  attr_accessor  :water
  attr_accessor  :yeast
  attr_accessor  :oven_time

  def initialize(flour_type:, number_of_people:, salt: , water: , yeast: , oven_time: )
    @flour_type = flour_type
    @number_of_people = number_of_people
    @salt = salt
    @water = water
    @yeast = yeast
    @oven_time = oven_time
  end
end
```

# Solutions

You can find all solutions on [Replit.com](https://replit.com/@lucianghinda/AssignInstanceVariables).

## Refactor to use hash

Solution idea proposed by [@sairam](https://twitter.com/sairam) [here](https://twitter.com/sairam/status/1499348766540767232).

One option is to refactor this to accept a hash as an argument and use a little bit of meta-programming: 

```ruby
class BreadReceipe
  attr_accessor  :flour_type
  attr_accessor  :number_of_people

  def initialize(options) 
    options.each do |key, value|
      next unless self.respond_to?(key) # verify that the accessor exists
      self.__send__("#{key}=", value)
    end
  end
end

my_receipe = BreadReceipe.new({
  flour_type: "All purpose", 
  number_of_people: 3
})
```

A possible problem (or feature) with writing the initializer like this is that it will initialize instance variables even if you define them as attr_readers.

If you want to restrict the initializer to only initialize attributes defined with attr_writer, replace `self.__send__` with `self.public_send` and add one more check inside the loop:

```ruby
 options.each do |key, value|
      next unless self.respond_to?(":#{key}=") # verify that the writer exists
      self.public_send("#{key}=", value)
 end
```

For a more example of robust code, I recommend you check how the [ActiveModel::AttributeAssignment](https://github.com/rails/rails/blob/3ec6b6e80bfc0d5ad729e0a14ca98f62775e099d/activemodel/lib/active_model/attribute_assignment.rb#L28) is doing the assignment. I will also talk about this a little bit below in the ActiveModel example.

## Tapping into object initializer

Another option is not to initialize the instance variables in the initializer and assign them when creating a new object:

```ruby
class BreadReceipe
  attr_accessor  :flour_type
  attr_accessor  :number_of_people
end
```

And in this case it matters how we initialize the object:

```ruby
my_receipe = BreadReceipe.new.tap do
  _1.flour_type = "All Purpose"
  _1.number_of_people = 3
end
```

## Using Dry gem

You can take a look at the [Dry::Initializer gem](https://dry-rb.org/gems/dry-initializer/3.0/) and use that to create your class.

```ruby
require 'dry-initializer'

class BreadReceipe
  extend Dry::Initializer
  option :flour_type
  option :number_of_people
end

my_receipe = BreadReceipe.new(flour_type: "All purpose", number_of_people: 3)
```
This had the advantage that you can coerce or type constrain the accessors if you like. 

```ruby
require 'dry-initializer'
require 'dry-types'

class BreadReceipeWithCoerce
  extend Dry::Initializer

  option :flour_type, Dry::Types['strict.string']
  option :number_of_people, Dry::Types['strict.integer']
end

begin
  my_receipe = BreadReceipeWithCoerce.new(flour_type: "All Purpose", number_of_people: "13")
rescue Dry::Types::ConstraintError => e 
  puts e
end
```

## Using ActiveModel::API

Solution idea proposed by [@JureCindro](https://twitter.com/JureCindro) [here](https://twitter.com/JureCindro/status/1499370686548217858).

In case you are working in Rails, then you can use ActiveModel::API

```ruby
require "active_model"
class BreadReceipe
  include ActiveModel::API
  attr_accessor :flour_type, :number_of_people
end

my_receipe = BreadReceipe.new(flour_type: "All purpose", number_of_people: 3)
```

If you want to see how this works, take a look at [active_model/api.rb](https://github.com/rails/rails/blob/main/activemodel/lib/active_model/api.rb) [line 61](https://github.com/rails/rails/blob/3ec6b6e80bfc0d5ad729e0a14ca98f62775e099d/activemodel/lib/active_model/api.rb#L61) where it includes `include ActiveModel::AttributeAssignment` and then take a look of how the [`assign_attributes` ](https://github.com/rails/rails/blob/3ec6b6e80bfc0d5ad729e0a14ca98f62775e099d/activemodel/lib/active_model/attribute_assignment.rb#L28) method works. You will notice that it does something similar with the first solution proposed here but with some validations. For example using ActiveModel::API you will not be able to instantiate from `new` an `attr_reader` accessor.

Example:
```ruby
class BreadReceipeWithAttrReader
    include ActiveModel::API
    attr_accessor :flour_type
    attr_reader :number_of_people
  end

begin
  with_attr_reader = BreadReceipeWithAttrReader.new(
    flour_type: "All purpose", 
    number_of_people: 3
  )
rescue ActiveModel::UnknownAttributeError => e
  puts e
end
```

## Inherit from OpenStruct

Solution proposed by [@aantix](https://twitter.com/aantix) [here](https://twitter.com/aantix/status/1499478380487532546). 

If you are not required to inherit your class from another specific class, you can inherit your class from OpenStruct, and then this will accept any keyword argument when initializing the object.

```ruby
class BreadReceipe < ::OpenStruct
  def initialize(args)
    super(args)
  end
end

my_receipe = BreadReceipe.new(one: "argument one", two: "argument two", flour_type: "all purpose")
```
The problem with this is that you can add any instance variable and by default it will be accessible for reading and writing. 

You can restrict the setting but not the retrieving of a variable: 

```ruby
class BreadReceipeWithAttrWriter < ::OpenStruct
  attr_writer :internal
  attr_reader :steps

  def initialize(args)
    super(args)
    # do some other initializing stuff
  end
end

with_attr_writer = BreadReceipeWithAttrWriter.new(
  flour_type: "All purpose",
  number_of_people: 3,
  private_variable: "payload",
  steps: "cannot initialize this"
)

puts "It has steps: ", with_attr_writer.steps
puts "It can read private", with_attr_writer.private_variable

```

## Using a class level builder method

Solution proposed by @[Adrian Marin](@adrianthedev) [here](https://twitter.com/adrianthedev/status/1499675526763630596). 

It might be that none of the solutions presented below work for you: either because you need to inherit from some specific class, or because you want to control explicitly what is happening. 

In this case, a solution could be to build your own builder as a class method:

```ruby
class BreadReceipe
  class << self
    def build(**args)
      new_object = new # creates a new object

      args.each do |key, value|
        setter_method = :"#{key}="
        next unless new_object.respond_to?(setter_method)

        new_object.__send__(setter_method, value)
      end

      new_object
    end
  end

  attr_accessor :flour_type
  attr_accessor :number_of_people
  attr_reader :steps

  def initialize
    @steps = load_steps
  end

  private

  def load_steps
    @steps = "Initial steps"
  end
end

```

With this solution the `steps` variable that was declared only as the reader will not be overwritten by the `.build` 

```ruby
my_receipe = BreadReceipe.build(
  flour_type: "All purpose",
  number_of_people: 3,
  steps: "Trying to set new steps"
)

puts my_receipe.steps # => Initial steps
```



