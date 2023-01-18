# Refactoring instance variables to local variables in Rails controllers

I like to use as much as possible the new features in Ruby. In this case, I will show how to take advantage of [shorthand hash syntax](https://dev.to/baweaver/ruby-3-1-shorthand-hash-syntax-first-impressions-19op) while changing from instance variables to local variables in controllers and views.

## Initial code

Say you have the following code in a controller action:

```ruby
# app/controllers/books_controller.rb

class BooksController < ApplicationController
  def index
    @user = user
    @books = books

    if params[:author].present?
      @books = @books.where(author: params[:author])
    end

    @books = @books.page(params[:page]) if params[:page].present?
  end

  private

    def user
      @_user ||= User.find(params[:user_id])
    end

    def books = user.books
end
```

Where `.search_by_title` is defined somewhere on the model (it is not essential for this part).

## Code review guidelines

Guidelines that I follow:

**The main purpose of an action in the controller is to render a response.**

I like the idea of keeping the code there at a minimum and as close as possible to everything related to `render`. Any other logic/flows should be in the method.

Why? SRP and limiting the changes. Here is a short example:

> Image you need will need to support anothe type of search, not only by title.
> 
> In this case, why change the `index` action? Did something changed with regards of what data (thinking about structure, types...) are rendered or how it is rendered? **No**. So there should not be a need to change something in the action itself.

**The data between a controller and view should only be transferred via local variables, not instance variables.** This is a bit bigger topic to cover, and I will probably write another article explaining this in detail.

Briefly, there are three main reasons:

* (1) undefined controller instance variables are `nil` thus will not throw `NameError` but silently be nil =&gt; causing business logic errors when used in conditionals (a condition if false because the variable is undefined, not because the query returned nil)
    
* (2) instance variables leak into all partials referenced, including all nested ones =&gt; it creates direct dependencies to the action in the controller instead of making partials dependent only on the call
    
* (3) instance variables can be added by using concerns and callbacks and can quickly become hard to track what instance variable is available in a view and what is the source for that data
    

## Code Review

Considering the guidelines, I can say the following things about the initial code:

* The `index` the method is doing too many =&gt; breaks SRP -&gt; has logic to create Action Record scopes, instantiates some variables from those methods, and paginates if needed
    
* The `index` method uses instance variables to pass data to the view
    
* If I need to change the search =&gt; I need to change something in `index` and it should not be the case because changing the search query (for example, from `where` to scopes) should not change the way rendering works or the logic in the view.
    
* If I need to change the way pagination works =&gt; I need to change something in the `index` and again, it should not be the case as changing the way pagination works does not change the logic of rendering nor the view logic
    

## Refactoring

### **Simplify the** `index` **method**

It should only call the render and pass along local variables. So it might look like this:

```ruby
class BooksController < ApplicationController
  def index
    render locals: { user:, books: }
  end

  private
    def user
      @_user ||= User.find(params[:user_id])
    end

    def books
    end
end
```

Now `index` only makes two calls `user` and `books` to obtain all data needed, and it will pass that data to the `app/views/books/index.html.erb` as local variables. This is a very good and simple action method. It does one thing: taking care of rendering and passing the data.

`user` method exists, and it is simple enough. But I will need to update the `books` method to return the correct query.

Also, I am using shorthand hash syntax to write shorter code:

```diff
def index
-    render locals: { user: user, books: books }
+    render locals: { user:, books: }
end
```

I will apply one more trick:

* Because I don't want other developers or myself in the future to add logic to `index` I will transform it into an endless method. This will make the method a little bit more closed to change, which is good because now this method is straightforward and brief, and we want to keep it as long as possible in this form.
    

```ruby
class BooksController < ApplicationController

  def index = render locals: { user:, books: }

  private
    def user
      @_user ||= User.find(params[:user_id])
    end

    def books
    # TBD
    end
end
```

### **Moving logic to** `books`

`books` should also be as simple as possible. Based on the initial code, this method should return the list of books based on some criteria:

* should be user books
    
* filtered by author if `params[:author]` is present
    
* paginate when if `params[:page]` is present
    

Thus I can say this method could look something like this:

```ruby
def books
  user.books.
    then { filter_by_author(_1)  if params[:author].present? }.
    then { paginate(_1) if params[:page].present? }
end
```

### Single Responsibility Principle

I also want to keep the satisfying SRP. If I need to change, for example, something related to author search or page parameters without affecting the general algorithm, then I should not need to change this method. Thus logic for search and paginate should live in their methods:

```ruby
def books
  user.books.
    then { filter_by_author(_1)  if params[:author].present? }.
    then { paginate(_1) if params[:page].present? }
end

def filter_by_author(books) = books.where(author: params[:author])

def paginate(books) = books.page(params[:page])
```

In practice, what I want to achieve is to make sure that when a change is needed, it should be limited to only the method that is related to the scope of the change.

An example of such change could be that I define on Book in a new scope like:

```ruby
class Book < ActiveRecord
  scope :sorted_by_author, -> (author) { where(author: author).order(author: :asc, created_at: :desc)}
end
```

To roll out this change to the `BooksController` I only need to change the `filter_by_author` method leaving everything else unchanged, thus limiting the scope of the change.

## Final result

Here is how the code looks at the end:

```ruby
# app/controllers/books_controller.rb

class BooksController < ApplicationController

  def index = render locals: { user:, books: }

  private
    def user
      @_user ||= User.find(params[:user_id])
    end

    def books
      user.books.
        then { filter_by_author(_1) if params[:author].present? }.
        then { paginate(_1) if params[:page].present? }
    end

    def filter_by_author(books) = books.where(author: params[:author])

    def paginate(books) = books.page(params[:page])
end
```

---

*Thanks to* [*Cristian Tone*](https://www.linkedin.com/in/cristiantone) *for reading various drafts of this article and giving me valuable feedback.*

---

If you like this type of content, then maybe you want to consider subscribing to

* my personal newsletter [All About Coding](https://allaboutcoding.ghinda.com/newsletter)
    
    *or*
    
* my curated newsletter [Short Ruby News](https://newsletter.shortruby.com) where I cover weekly Ruby news from all around the internet.