## Using Concurrent::Promise while rescuing exceptions in Ruby

As I could not find a clear example about how to rescue exceptions from Concurrent::Promises (part of the  [Concurrent Ruby gem](https://github.com/ruby-concurrency/concurrent-ruby) ) I read through the documentation and here are two examples: one that documents success case and one that shows what is happening when there is an error. 

Here is a simple way to execute concurrent ruby and rescue exceptions. 

## Success Example

```ruby
require 'bundler/inline'

gemfile do
  ruby "3.0.1"
  source 'https://rubygems.org'
  gem "concurrent-ruby"
end

require "concurrent"
require "pp"

# Just a simple class that needs to execute something
class Load
  def initialize(id)
    @id = id
  end

  def work
    sleep(1)
    "Executing #{@id}"
  end
end

class ParallelExecution
  def call

    # Using here Concurrent::Array to be thread safe
    exceptions = Concurrent::Array.new
    promises = []
    10.times do |t|
      promises << Concurrent::Promises.future { Load.new(t).work }.rescue { exceptions.push(_1) }
    end

    # Calling .value on each job to wait for its execution
    values = Concurrent::Promises.zip(*promises).value

    # Raising exceptions if any of the jobs were returning an exception
    raise exceptions.first if exceptions.length > 0

    pp values
  end
end


ParallelExecution.new.call
``` 

This will print: 

```ruby
["Executing 0",
 "Executing 1",
 "Executing 2",
 "Executing 3",
 "Executing 4",
 "Executing 5",
 "Executing 6",
 "Executing 7",
 "Executing 8",
 "Executing 9"]
```

## Failure example

```ruby
require 'bundler/inline'

gemfile do
  ruby "3.0.1"
  source 'https://rubygems.org'
  gem "concurrent-ruby"
end

require "concurrent"
require "pp"

# Just a simple class that needs to execute something
class Load
  def initialize(id)
    @id = id
  end

  def work
    raise MyException.new
  end
end

class MyException < Exception ; end

class ParallelExecution
  def call

    # Using here Concurrent::Array to be thread safe
    exceptions = Concurrent::Array.new
    promises = []
    10.times do |t|
      promises << Concurrent::Promises.future { Load.new(t).work }.rescue { exceptions.push(_1) }
    end

    # Calling .value on each job to wait for its execution
    values = Concurrent::Promises.zip(*promises).value

    # Raising exceptions if any of the jobs were returning an exception
    raise exceptions.first if exceptions.length > 0

    pp values
  end
end


ParallelExecution.new.call
```

This code will raise an exception MyException.

## To notice in the code example

One can get all exceptions and do various stuff based on their type for example: 

```ruby
require 'bundler/inline'

gemfile do
  ruby "3.0.1"
  source 'https://rubygems.org'
  gem "concurrent-ruby"
end

require "concurrent"
require "pp"

# Just a simple class that needs to execute something
class Load
  def initialize(id)
    @id = id
  end

  def work
    exception = [MyException, SecondException].sample
    raise exception.send(:new)
  end
end

class MyException < Exception ; end
class SecondException < Exception ; end

class ParallelExecution
  def call

    # Using here Concurrent::Array to be thread safe
    exceptions = Concurrent::Array.new
    promises = []
    10.times do |t|
      promises << Concurrent::Promises.future { Load.new(t).work }.rescue { exceptions.push(_1) }
    end

    # Calling .value on each job to wait for its execution
    values = Concurrent::Promises.zip(*promises).value

    # Raising exceptions if any of the jobs were returning an exception
    if exceptions.length > 0
      count_my_exception = exceptions.count { _1.instance_of?(MyException) }
      count_second_exception = exceptions.count { _1.instance_of?(SecondException) }
      
      puts "MyException generated #{count_my_exception}"
      puts "SecondException generated #{count_second_exception}"
    end
  end
end


ParallelExecution.new.call
```



