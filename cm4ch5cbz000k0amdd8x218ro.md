---
title: "How to Return Multiple Values from a Method in Ruby Using Data.define"
seoTitle: "Return Multiple Values in Ruby with Data.define"
seoDescription: "Learn how to return multiple values from a Ruby method using the Data.define class for improved code clarity and maintainability"
datePublished: Fri Dec 06 2024 08:18:56 GMT+0000 (Coordinated Universal Time)
cuid: cm4ch5cbz000k0amdd8x218ro
slug: how-to-return-multiple-values-from-a-method-in-ruby-using-datadefine
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1733472914778/25d070f7-64d2-4e28-a6e6-51c911e6e433.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1733473031088/8d5a2eb1-284f-4100-bb48-7ddc5fd437e9.png
tags: ruby, patterns, ruby-on-rails, pattern-matching

---

I read a question on [Bluesky](https://bsky.app/profile/pacumo.bsky.social/post/3lc3wcqayiq2d) and [X](https://x.com/pabcumo/status/1862424013084999999) that I quickly replied to, but I wanted to expand on my recommendation.

## The context

This is the code that was originally shared and we were asked for our opinion on it.

```ruby
def something_with_two_outputs
  a = parse_data
  b = process_result
  
  return a,b 
end

parsed_data, result = something_with_two_outputs
```

I think it's a valid question. I've seen code in real production apps that returns not just 2, but 6 values, which are then passed to other methods that use one or more of these values.

For my purpose I want to use something else to have some names for what is happening inside. But it reflects the same question or patternȘ

```ruby
def parse(response)
  headers = parse_headers(response),
  body = parse_body(response)

  return headers, body
end
```

## The problem with this pattern

There are two problems with this pattern in an OOP language like Ruby:

1. It creates **a dependency on the position of the arguments**, which is the main issue.
    
2. It is difficult to understand **what each return value represents** when calling the method without looking inside it. This is a secondary effect.
    

### The problem with the dependency on the position of the arguments

You might call this method like this:

```ruby
headers, parsed_body = parse(response)
```

or you might call it like this:

```ruby
parsed_body, headers = parse(response)
```

Which one is the right one? Well you have to read the entire body of that method and figure out what each parameter is to know if you are interpreting the results in the proper order or make an assumption that headers will be returned first.

And you see here the second problem that you have to read the entire body of the method.

Now imagine the method is a big longer and does much more, you have to read it and parse what each element is and what does it contain. So the main point here is that you have to read the method to clarify the contract/API of that method multiple times.

This is an example but don’t focus on the actual code. I just added more lines there just to increase the number of lines that we need to read and use bad names:

```ruby
def parse(response)
  parsed_data = parse_data(response)
  if parsed_data.include?(:key)
    result_parse_data = post_processing(parsed_data)
  else 
    result_parse_data = parsed_data.expect(:second_key)
  end

  result = process_result
  
  return result_parse_data, result
end
```

The more this is used further away, imagine this being a public method in an object that you use in some other object. And you come back 6 months after you wrote both objects and have to debug the second one the callee. You will need to reply on correct naming and even so, seeing the `a, b = method_call` the first thing you want to clarify is about the correct order of the variables that are keeping the return values.

Moreso imagine you might want to refactor the method and from now on you always have to make sure that the order of the return remains the same.

My exploration below considers a situation when the question or the pattern is more about returning multiple values each one having the same meaning, than for example the need to return a success/failure state and a payload. Because for this situation of wanting to know if the returned value is a success/failure and then access the payload we have the pattern of using Success/Failure monads.

## A possible solution: use the return values via pattern matching

Now there is a way to use this and make the dependency problem explicit by using pattern matching.

Say we have a method that might return a String and an Integer:

```ruby
def processing
  return "Allowed", 1
end
```

You can then consume this method like this:

```ruby
processing => [ String => result, Integer => user_id ] 

puts result
puts user_id
```

This works and it makes the dependency on the order explicit but it is moving the solution for the dependency problem (dependency on the argument order and the possibility to introduce bugs by using the wrong order) to the callee to be aware of the order.

**A good API design should make it easier to do the right thing and harder to do the wrong thing. In our case we should design the return of a method in such a way that the callee should have little room for errors due to the order of returned values.**

## The simple solution: return a hash

A quick fix is to just return a hash:

```ruby
def parse(response)
  {
     header: headers(response),
     parsed_body: parse_body(response)
  }
end
```

By returning a hash we remove the dependency on the position of the arguments and also encode the knowledge about what each argument represents in the key name.

This is I think the most common solution or pattern used for this case.

There are two main concerns with returning a hash:

* It is not immutable
    
* It is hard to search for it. You can search for the keys but using the same key does not mean you wanted to group together the same data.
    

## Use Data.define

Since Ruby 3.2 in 2022 we now have in Ruby the [Data class](https://docs.ruby-lang.org/en/master/Data.html). By using this you have an immutable object with a simple interface, comparable by value and by type and you can give it a name.

```ruby
Response = Data.define(:headers, :parsed_body)

def parse(client_response)
  headers = parse_headers(client_response),
  parsed_body = parse_body(client_response)

  Response.new(headers: headers, parsed_body: parsed_body)
end
```

One main advantaged is that it can be used without the need to open the actual method and you can just look at the getters names:

```ruby
response = parse(client_response)

puts response.headers
puts response.parsed_body
```

Here is why using Data.define is a good choice:

1. It is **immutable**. If you try to do `response.headers = {}` this will raise an error.
    
2. It **removes the dependency on the position** **of the arguments**
    
3. It forces you to **give a name to the data that you want to pass** around together and this is good
    
4. It is **easily greppable** so you can find all places where you are using the Response objects. Imagine for example you want to add rate\_limit informations to that response
    

By using Data object you will get two extra capabilities: you can extend the object with computed attributes and you can compose multiple Data objects together.

Imagine that you want to carry on some rate limit information and you want to make it easily accessible.

```ruby
RateLimit = Data.define(:remaining, :reset_in)

Response = Data.define(:headers, :parsed_body, :rate_limit)

def parse(client_response)
  headers = parse_headers(client_response),
  parsed_body = parse_body(client_response)
  rate_limit = RateLimit.new(
    remaining: headers[:rate-limit-remaining],
    remaining: headers[:rate-limit-reset].to_i
  )

  Response.new(
    headers: headers, 
    parsed_body: parsed_body,
    rate_limit: rate_limit
  )
end
```

And then if you want to check if you can do more calls you can for example do this:

```ruby
RateLimit = Data.define(:remaining, :reset_in) do
  def continue?
    remaining > 0
  end
end
```

An extra souce is that you can do pattern matching against the response:

```ruby
Response = Data.define(:headers, :parsed_body, :status)

def read_posts(user_id)
  # ... doing the actual thing
  Response.new(headers: headers, parsed_body: parsed_body, status: status)
end

response = read_posts(user_id)
if response in { status: 200, parsed_body: post_body }
  puts "In case of 200 OK response"
  save_post(post_body)
end
```

---

If you like this article:

👐 Interested in learning how to improve your developer testing skills? Join my live online workshop about [**goodenoughtesting.com**](http://goodenoughtesting.com/) **\- to learn test design techniques for writing effective tests**

👉 Join my [**Short Ruby Newsletter**](https://newsletter.shortruby.com/) for weekly Ruby updates from the community and visit [**rubyandrails.info**](http://rubyandrails.info/)**, a directory with learning content about Ruby.**

🤝 Let's connect on [**Bluesky**](https://bsky.app/profile/lucianghinda.com), [**Ruby.social**](https://ruby.social/@lucian), [**Linkedin**](https://linkedin.com/in/lucianghinda), [**Twitter**](https://x.com/lucianghinda) where I post mostly about Ruby and Ruby on Rails.

🎥 Follow me on [**my YouTube channel**](https://www.youtube.com/@shortruby) for short videos about Ruby/Rails