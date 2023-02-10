# A method's gravity

### What is method gravity?

What I call **Method Gravity** means for me:

> The bigger the method the more new lines of code will be added to it.

I noticed this while working on various projects: once a method grows, the chances that the next developer will add more lines to it increase.

Of course, there is a tipping point (like with the gravity of a start) that when a method is too big, it will collapse, meaning someone will take it apart and split it into smaller methods.

### What is a long method and what is a short method

I am not sure there exists a definition of what is a long method and what is a short method. This is of course something subjective (depending on individual or team preferences).

I could say this (citing Sandi Metz): anything bigger than 5 lines of code could be considered a long method.

But I think a better definition could be along the following lines:

* If you need to read twice, a method to remember/understand each line of it is long.
    
* If you need to read it twice to understand what it does is (probably) too long (or could use some renaming)
    

Thus *a short method is one that can be understood quickly at a glance*.

### The main problems with long methods

There are three main problems:

1. It makes it easy to break Single Responsibility Principle: a long method tends to do many things, and to summarize it to a main purpose means to keep expanding that goal until it reaches a high complexity
    
2. It makes it easy to break Cohesion: a long method will tend to do unrelated things resulting in unwanted coupling
    
3. A long method tends to favor hard-to-read algorithms or too creative naming to avoid collision or variable name re-use
    

### Benefits of short methods

Of course, the main benefits of short methods are:

1. Simplicity
    
2. Single Responsibility Principle: short methods are easy to focus on one single thing and thus are also easy to describe in a simple way
    
3. Limits the number of changes: when you have to change something, you can isolate the change to a small method
    
4. Easy to test: a small method is easy to test
    

But for me, the biggest benefit for the developer is that using smaller methods forces us to write better names.

Let's take an example *(please read this example as pseudo-code, I will not focus here on language specifics, and the purpose of the example is to show the high-level code design and not focus on specifics)*

If you look at the following code can you say quickly what the result might be?

```ruby
# input = [{ "slug" => "one_day", "language" => "en"} => [ { "id" => 10, "status" => "booked" }]]

def transform(input)
  events_hash = input.flat_map { _1.keys }
  event_slugs = events_hash.collect { _1["slug"] }
  events = Event.where(slug: event_slugs)

  registrations_hash = input.flat_map {_1.values }.flatten
  registration_ids = registrations.collect { _1["id"] }
  registrations = Registration.where(id: registration_ids)

  hash = input.pluck(*KEYS).to_h
  hash.transform_keys! { |key| events.find { |e| e.slice(:event_type, :language) == key } }

  hash.transform_values { |value| registrations.find { |r| r.slice(:id, :status) == value} }
end
```

You can probably guess with a bit an effort. And I bet that letting 2 weeks pass and looking at this again, you might still need a small effort to remember.

What about the following code:

```ruby
def transform(input)
  keys_to_event_objects(input)
  values_to_registration_objects(input)
end
```

I think reading this second version of the function would give you a good idea about what `transform` the method is doing.

You might wonder what the full code looks like (some spaces and returns were removed to keep the code terse in this article):

```ruby
def transform(input)
  keys_to_event_objects(hash)
  values_to_registration_objects(hash)
end

def keys_to_event_objects(hash) = hash.transform_keys! { event(_1) }

def values_to_registration_objects(hash) = hash.transform_values { registration(_1) }

def event(key) = events.find { _1.slice(:slug, :language) == key }

def registration(value) = registrations.find { _1.slice(:id, :status) == value }

def events(input) = @_events ||= load_events(from_event_slugs(input))
def load_events(slugs) = Event.where(slug: slugs)
def from_event_slugs(input) = input_keys(input).collect { _1["slug"] }
def input_keys(input) = input.flat_map { _1.keys }

def registrations(input) = @_registrations ||= load_registrations(from_registration_ids(input))
def load_registrations(ids) = Registration.where(id: ids)
def from_registration_ids(input) = input_values(input).collect { _1[:id] }
def input_values(input) = input.flat_map { _1.values }.flatten
```

This code, with many endless methods, has a huge advantage: most of the changes you can think of would be limited to a small function.

Example of possible changes:

* include/eager\_load on `Event` / `Registration` if more attributes will need to be used further down the road, =&gt; change the `load_*` methods
    
* speed up the `find_*` by ordering records in some specific way (eg: if in general, the input will have the most recent registrations/events) =&gt; just change the `load_*` methods
    
* adding/removing keys from the input =&gt; just change `event` or `registration` methods
    
* say the API will decide to return `id` for events instead of `slug` =&gt; rename `from_event_slugs`, change the key inside, and then change in `events`
    

Observe that for all these changes, the main `transform` method does not change. And that is good because the main algorithm (get hash, map keys to objects, map registration to objects) is the same.

Thus with small methods, we achieve what Sandi Metz describes as the purpose of design: *reduce the cost of change*.

### Ideas to keep methods small

Some simple things that could help to keep the method gravity small in Ruby:

**Use endless methods**

```ruby
# events_controller.rb

def show = render locals: { event: }
```

```ruby
# time_rules.rb

def past?(event) = event.finish_time <= Time.current
```

Why:

* Because once a method is endless, it takes a more significant effort to add another line (you need to make it a normal method)
    

**Use guard clauses**

```ruby
def ensure_access_is_allowed
  return if current_user.admin?

  current_user.allowed_to?(:access, event)
end
```

Why:

* Because adding another branch to a guard clause takes a bit of effort, it means transforming the guard clause into a normal `if` condition
    

**Use conditional at the end of the statement**

```ruby
def show
  set_context if context_given?

  render locals: { user: }
end
```

Why:

* Because also, here, adding another branch means a bit more effort to transform the end of the statement `if` into a normal `if`
    

### More general advice

In general, I like the advice from a [paper published by Google](https://static.googleusercontent.com/media/research.google.com/en//pubs/archive/37755.pdf), called "Searching for Build Debt: Experiences Managing Technical Debt at Google"

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675946765425/2b0b9f22-2ee8-4b66-b5b9-2d1ca20c185e.png align="center")

So the main idea, if you want to keep the methods small, is to make it hard for you or a colleague to make them bigger in the future. Add a bit of antigravitational force to them.

Observe in the main example (the one with `transform` method) that it is a bit hard to make the other methods longer. You can rename them and you can change what each of them is doing with ease. But adding more lines of code to each one of them is hard.

### In case you like this recommendation from Sandi Metz:

> Methods can be no longer than five lines of code

Then you know what method has always fever than 5 lines of code?

**\-&gt; an endless method**

### Disclaimer

I wrote this as general advice. I agree there are cases when a long method makes sense. Maybe. But that should be an exception.

*Apply with care! Moderation is key. Do not abuse!*

Even Sandi Metz [says](https://www.rubypigeon.com/posts/methods-can-be-longer-than-five-lines/) (regarding her famous rules):

> There are actually six rules, and the sixth rule is that you can break any of the first five, as long as you can get your pair to agree. Why is it that we’re such cargo culters about it? It’s like, that’s the rule that people forget.

---

If you like this type of content, then maybe you want to consider subscribing to my curated newsletter [**Short Ruby News**](https://newsletter.shortruby.com/) where I cover weekly Ruby news from all around the internet.

%%[shortruby]