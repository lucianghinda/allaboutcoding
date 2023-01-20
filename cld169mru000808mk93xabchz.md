# Refactoring instance variables to local variables in Rails controllers

I like to use as much as possible the new features in Ruby. In this case, I will show how to take advantage of [shorthand hash syntax](https://dev.to/baweaver/ruby-3-1-shorthand-hash-syntax-first-impressions-19op) whileThe on changing from instance variables to local variables in controllers and views.

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

Guidelines that I will follow here:

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
    then { filter_by_author(_1) }.
    then { paginate(_1) }
end
```

### Single Responsibility Principle

I also want to keep the satisfying SRP. If I need to change, for example, something related to author search or page parameters without affecting the general algorithm, then I should not need to change this method. Thus logic for search and paginate should live in their methods:

```ruby
def books
  user.books.
    then { filter_by_author(_1) }.
    then { paginate(_1) }
end

def filter_by_author(books)
  return books unless params[:author].present?

  books.where(author: params[:author])
end

def paginate(books)
  return books unless params[:page].present?

  books.page(params[:page])
end
```

In practice, I want to make sure that when a change is needed, it should be limited to only the method related to the scope of the change.

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
        then { filter_by_author(_1) }.
        then { paginate(_1) }
    end

    def filter_by_author(books)
      return books unless params[:author].present?

      books.where(author: params[:author])
    end

    def paginate(books)
      return books unless params[:page].present?

      books.page(params[:page])
    end
end
```

## Alternatives

There was a good discussion on [/r/rails](https://www.reddit.com/r/rails/comments/10f09zp/refactoring_instance_variables_to_local_variables/) about this article with some good points against various things I presented here.

So here are some alternatives of how the cood looks like:

### Move logic from controller to `books` without going deeper

```ruby
# app/controllers/books_controller.rb

class BooksController < ApplicationController

  def index = render locals: { user:, books: }

  private

    def user
      @_user ||= User.find(params[:user_id])
    end

    def books
      @_books ||= begin
        user_books = user.books
        user_books = user_books.where(author: params[:author]) if params[:author].present?
        user_books = user_books.page(params[:page]) if params[:page].present?

        user_books
      end
    end
end
```

My review of this is that I feel somehow that `books` is doing too much, but in the same time its scope can be summarised as querying user books based on some conditions.

Usually when I have to choose between a longer method or multiple shorter methods I choose shorter methods becuase I think it allows the reader to decide if they want to go down to details or if they want to read the code at a higher level.

### Using a query object

This solution is based on the one provided by [markevich's comment](https://www.reddit.com/r/rails/comments/10f09zp/comment/j4wvlmh):

```ruby
# app/controllers/books_controller.rb
class BooksController < ApplicationController

  def index = render locals: { user:, books: }

  private

    def user = query_service.user

    def books = query_service.books

    def query_service
      @_query_service ||= UserWithBooksQuery.call(
        user_id: params[:user_id],
        author: params[:author],
        page: params[:page]
      )
    end
end

# app/services/user_with_books_query.rb
class UserWithBooksQuery
  UserWithBooksContract = Struct.new(:user, :books, keyword_init: true)

  def self.call(user_id:, author:, page:)
    user = User.find(user_id)

    books = user.books
    books = books.where(author: author) if author
    books = books.paginate(page) if page

    UserWithBooksContract.new(user:, books:)
  end
end
```

This is a good solution worth exploring if you/your team are using services. I did not present this from the start as I know quite a few teams that are using services in a specific ways (only to call external services), and they use models for everything else. For this article, it does not matter where the logic is hidden.

### Procedural action

This proposal was added by @[Paweł Świątkowski](@katafrakt) as a comment to this article and something similar was proposed by [elementboarder on /r/rails](https://www.reddit.com/r/rails/comments/10f09zp/comment/j4w5hiu)

And the main idea is that the action method should describe the logic in a procedural way

**Option 1:**

```ruby
class BooksController < ApplicationController

  def index
    user = User.find(params[:user_id])
    books = get_filtered_books

    render locals: { user:, books: }
  end

  private

    def get_filtered_books
      @_books ||= begin
        user_books = user.books
        user_books = user_books.where(author: params[:author]) if params[:author].present?
        user_books = user_books.page(params[:page]) if params[:page].present?

        user_books
      end
    end
end
```

I think this works. The only comment I have here is that it brings the need to invent these `get_<name>` or `fetch_<name>` to go eliminate the name collision with other methods or variables. So I think for example naming `get_filtered_books` to books and using short-hand syntax will remove the need to use the `get` prefix.

**Option 2**

```ruby
# app/controllers/books_controller.rb

class BooksController < ApplicationController

  def index
    user = User.find(params[:user_id])
    books = user.books.filter_by(author: params[:author]).page(params[:page])

    render locals: { user:, books: }
  end
end

# app/models/books

class Book < ApplicationRecord
  scope :filter_by, -> (author) { where(author: author) }
end
```

The move of the query logic to the model makes a lot of sense to me. I did not do that in my refactoring as I started with an example where the author put the logic in the controller and I only wanted to cleanup the controller. But moving it to model is the right way to go. This is also very short so it is a very good candidate for a good solution.

## Simplicity

If moving the logic to model is an option then I think the following solution is among the simplest and concise ones:

```ruby
# app/controllers/books_controller.rb

class BooksController < ApplicationController

  def index = render locals: { user:, books: }

  private

    def user
      @_user ||= User.find(params[:user_id])
    end

    def books
      @_books ||= user.books.filter_by(author: params[:author]).page(params[:page])
    end
end

# app/models/books

class Book < ApplicationRecord
  scope :filter_by, -> (author) { where(author: author) }
end
```

### Conclusion

The comments received show the power of code review. It is great to see many more ideas and principles that people use to refactor code and how while having the same objective (make the code maintainable and readable) we can identify different solutions.

---

*Thanks to* [*Cristian Tone*](https://www.linkedin.com/in/cristiantone) *for reading various drafts of this article and giving me valuable feedback.*

---

Updates:

1. As pointed out by pirokar13 [here](https://www.reddit.com/r/rails/comments/10f09zp/comment/j4wjwly/?context=3) I had a bug in the `then` block that returned nil, so I fixed the code. I updated the code to fix this issue
    
2. I added a whole new section called Alternatives where I added some proposal of code samples that I received from comments on reddit or on this blog sometimes adding my own twist :)
    

---

If you like this type of content, then maybe you want to consider subscribing to

* my personal newsletter [All About Coding](https://allaboutcoding.ghinda.com/newsletter)
    
    *or*
    
* my curated newsletter [Short Ruby News](https://newsletter.shortruby.com) where I cover weekly Ruby news from all around the internet.