---
title: A Simple Query Class For Searching Arrays of Hashes in Ruby 
summary: A brief-ish intro to Metaprogramming in Ruby 
---

I recently came across a scenario at work where I was fetching and processing sizeable collections of schemaless JSON objects for a new ETL job, performing many different types of searches on the collection using various key combinations. Pretty soon, I started noticing a recurring theme in my code:

{% highlight ruby %}
collection = [ # a ton of imaginary JSON objects ]

json_obj_a = collection.select { |obj| obj['key'] == 'some value' }

# do some work on json_obj_a

json_obj_b = collection.selct { |obj| obj['key'] == json_obj_a['some_key'] }

# mrege some values from json_obj_b to json_obj_a

# we also need to query with multiple params
other_obj = collection.select do |obj| 
  obj['key_a'] == 'val_a' && obj['key_b'] == 'val_b'
end
{% endhighlight %}

The main problem with this is that the code isn't very dynamic; I have to pass the `==` expression block every time I need to craft a query with new parameters. This leads to a ton of duplication in processing logic, and when the JSON collection reaches a certain level of variance and complexity, this code starts to smell *really bad*. 

But I can't easily wrap each of these JSON objects in a class without a bit of work since they're denormalized and don't always share the same structure. In addition, I expect to handle many more varied JSON payloads in the future, so I'd be tying myself to writing a new class every time I have a new JSON payload type, no bueno.  

### Setting Up the Class 

We're going to write a simple class that takes an array of hashes and exposes a set of methods that allows us to more succinctly and dynamically search the array.  Our example will revolve around the concept of a library (the books kind). There are a few fields that we know will exist across all entities in our collection, but we need to be able to be able to handle the variance between disparate fields since the data is somewhat denormalized. Here's a small sample JSON collection we'll use later:   

{% highlight javascript %}
[
  {
    "name" = "The Lord of the Rings: The Fellowship of the Ring",
    "type" = "novel",
    "pages" = 421,
    "movie" = true
    "sequels" = [
      {
        "name" = "The Lord of the Rings: The Two Towers",
        "pages" = 374 
      },
      {
        "name" = "The Lord of the Rings: The Return of the King",
        "pages" = 498
      }
    ]
  },
  {
    "name" = "Sapiens",
    "type" = "nonfiction",
    "subtype" = "philosophy",
    "pages" = "305"
  },
  {
    "name" = "V For Vendetta",
    "type" = "comic",
    "pages" = 236
  },
  { "name" = "Born To Run",
    "type" = "nonfiction",
    "subtype" = "sports"
  },
  {
    "name" = "Zen and the Art of Motorcycle Maintenance",
    "author" = "foo",
    "type" = "novel",
    "subtype" = "philosophy"
  }
]
{% endhighlight %}

We see that every entity in our `library` collection has a name and type, but beyond that, all bets are off. For our query interface, we want to be able to easily search by `type`, with the option to pass in any other key/value pairs as arguments when necessary.  

We'll set up a really simple class that takes an array of books as an argument and allows us to retrive all books of type 'novel'.

{% highlight ruby %}
# 1) Static novel retrieval

class Library
  attr_reader :books

  def initialize(collection)
    @books = collection
  end

  def novels
    @books.select { |b| b['type'] == 'novel' }
  end
end

lib = Library.new('sample.json')

lib.novels
# returns
=> [
  {
    "name" = "The Lord of the Rings: The Fellowship of the Ring",
    "type" = "novel",
    "pages" = 421,
    "movie" = true
    "sequels" = [
      {
        "name" = "The Lord of the Rings: The Two Towers",
        "pages" = 374 
      },
      {
        "name" = "The Lord of the Rings: The Return of the King",
        "pages" = 498
      }
    ]
  },
  {
    "name" = "Zen and the Art of Motorcycle Maintenance",
    "author" = "foo",
    "type" = "novel",
    "subtype" = "philosophy"
  }
]
{% endhighlight %}

This is a good start, it's now much easier and more concise to fetch all novels from our Library instance. So we can always fetch all of our books by calling `lib.books`, or just the novels using `lib.novels`.
If we wanted to be able to fetch all comic books the same way, we could create another method to wrap around a `select` method to fetch those as well:

{% highlight ruby %}
# 2) Add static comic retrieval 

class Library
  attr_reader :books

  def initialize(collection)
    @books = collection
  end

  def novels
    @books.select { |b| b['type'] == 'novel' }
  end
  
  def comics
    @books.select { |b| b['type'] == 'comic' }
  end
end
{% endhighlight %}

But something doesn't feel quite right about this: we're pretty obviously repeating our implementation of the `novels` method with a new string to match type against. It would be awesome to have a way of dymaically passing that matcher string based on our method call. Luckily for us, there is!

### Metaprogramming FTW

Metaprogramming is a great way to dynamically implement functionality on a class when you're not completely sure of your requirements before runtime. It's a pretty straightforward concept once you've written a couple of examples. The following snipped includes a `binding.pry`, which allows us to stop code anywhere during execution. If you've never used pry in your Ruby journey, I highly recommend it when you're getting lost in some strange code.

{% highlight ruby %}
# 3) Using 'pry' to explore `method_missing`

require 'pry'

class Library
  attr_reader :books

  def initialize(collection)
    @books = collection
  end

  private

    def method_missing(method)
      binding.pry
    end
end
{% endhighlight %}

When we create a new instance of `Library`, nothing new happens. However, when you try to call a non-existent method on it, like `lib.foo`, you'll notice that your console is frozen on the line where we dropped the `pry`.

Upon reading the method we're currently frozen in, you'll notice that it took one argument: `method`.  Let's inspect that argument:

{% highlight ruby %}
lib = Library.new('sample.json')
lib.foo

# inside our pry console
>> method
=> 'foo'
{% endhighlight %}

So the method argument is a string of the method we just called on the `Library` class. Whenever the `method_missing` method is implemented in a class, it will be called whenever a method is invoked that's not currently registered to the object. Since there was no method `foo` defined on `Library`, `method_missing` was automatically called, which activated the `pry` call. [Neat](https://giphy.com/embed/tLmVajLwcG7de)!

Now, we can wrap the `select` call inside of `method_missing` to reduce our code and improve the flexibility of the interface:

{% highlight ruby %}
# 4) Dynamic object type searching using `method_missing`

class Library
  attr_reader :books

  def initialize(collection)
    @books = collection
  end

  private

    def method_missing(method)
      @books.select { |b| b['type'] == method }
    end
end

lib = Library.new
lib.novel.count
# returns 2

lib.comic.count
# returns 1

lib.ryan_gosling.count
# returns []
{% endhighlight %}


Now we can easily fetch the novels and comics in our collection, with one little caveat: any other non-existent method will simply return an empty array (a la `lib.ryan_goslings`). Ryan Gosling isn't a book, so we probably don't want to return an empty collection in that case. There are several different ways to handle this, but I want to raise an error when I attempt to fetch a type that doesn't exist in my collection.

{% highlight ruby %}
# 5) Basic error-handling for non-existent keys

class Library
  attr_reader :books

  def initialize(collection)
    @books = collection
  end

  private

    def method_missing(method)
      if book_types.include?(method)
        @books.select { |b| b['type'] == method }
      else
        raise(NoMethodError, "Undefined method #{method} called on Library")
      end
    end

    def book_types
      @book_types ||= @books.pluck('type').uniq
    end
end
{% endhighlight %}

Now we've added a `book_types` method that returns an array of all types found in our books collection.  We've also memoized this method so we don't have to scan the array for types every single time we query the collection. [This](http://engineering.gusto.com/memoization-in-ruby-made-easy/) is a great intro to memoization if this a new concept for you.

###### Rails Dependency Alert!
There's one small thing I want to fix since I'm developing this class in a Rails environment. I'd rather call my query methods by their plural names since they return a collection, not a single instance. To do this easily, I'm going to use a monkey-patched method called `singularize` on `String` that ships with Rails, but not plain old Ruby.  This will allow me to call the pluralized form of the type as the method name and then dynamically singularize the method name as it enters `method_missing`.

{% highlight ruby %}
# 6) Interface semantics using `singularize` Rails helper

class Library
  attr_reader :books

  def initialize(collection)
    @books = collection
  end

  private

    def method_missing(method)
      method = method.singularize # Rails-only

      if book_types.include?(method)
        @books.select { |b| b['type'] == method }
      else
        raise(NoMethodError, "Undefined method #{method} called on Library")
      end
    end

    def book_types
      @book_types ||= @books.pluck('type').uniq
    end
end
{% endhighlight %}

This Rails helper method is great because it's smart enough to handle edge cases in English, such as "biographies" to "biography". If you're not working in a Rails environment and don't feel like adding ActiveSupport as a dependency, I don't blame you.  The interface still works really well without the extra syntactic sugar. 

### Adding Search Parameters

The next thing we want to do is add the ability to pass in keyword arguments and return results that match those arguments. When it's done, we'll be able to do stuff like `lib.novels(name: "foo")` to return all novels named "foo", or `lib.books(movie: true)` to return all books in our collection that also have a movie.

The first step to this is taking the `select` statement out of `method_missing` and wrapping in its own method that we can pass params to on the fly.

{% highlight ruby %}
# 7) Extract `select` call into its own method

class Library
  attr_reader :books

  def initialize(collection)
    @books = collection
  end

  private

    def method_missing(method)
      method = method.singularize

      if book_types.include?(method)
        search(method)
      else
        raise(NoMethodError, "Undefined method #{method} called on Library")
      end
    end

    def search(method)
      @books.select { |b| b['type'] == method }
    end

    def book_types
      @book_types ||= @books.pluck('type').uniq
    end
end
{% endhighlight %}

This is structurally closer to what we want, but we haven't actually modified any of the functionality yet.  We need to add another argument to `method_missing` that will accept a variable number of inputs using the splat operator. Then we can pass the params on to `search`:

{% highlight ruby %}
# 8) Pass params into new `search` method

class Library
  attr_reader :books

  def initialize(collection)
    @books = collection
  end

  private

    def method_missing(method, *params)
      method = method.singularize

      if book_types.include?(method)
        search(method, params)
      else
        raise(NoMethodError, "Undefined method #{method} called on Library")
      end
    end

    def search(method, params)
      @books.select { |b| b['type'] == method } # difficult to use params in this block
    end

    def book_types
      @book_types ||= @books.pluck('type').uniq
    end
end
{% endhighlight %}

There's still something wrong with this code, though. We're not injecting `params` into our search due to the hardcoded expression in the block: `b['type'] == method`. There are a few ways around this, and favorite involves simplifying the code a wee bit to make handling dynamic params a bit simpler. We accomplish this by adding the method name passed to `method_missing` to our params hash before sending to search, like so:

{% highlight ruby %}
# 9) `search` method now handles any key/val combination with ease

class Library
  attr_reader :books

  def initialize(collection)
    @books = collection
  end

  private

    def method_missing(method, *params)
      method = method.singularize

      if book_types.include?(method)
        params['type'] = method
        search(params)
      else
        raise(NoMethodError, "Undefined method #{method} called on Library")
      end
    end

    def search(params)
      @books.select { |b| b.values_at(*params.keys) == params.values }
    end

    def book_types
      @book_types ||= @books.pluck('type').uniq
    end
end
{% endhighlight %}

Now we've changed the expression to call `values_at` on the hash, passing in the param keys (using the splat operator again), and testing the result of that method against the values passed into the block. This allows us to pass as many key/value pairs as we like to the search method without having to hardcode any parameters in the block itself. 

There's only one thing left to do: expose the search functionality on `@books` as well so we can search across our entire collection, not just a subgroup. 

We can accomplish this by writing a public method `books` that takes an optional params hash. We'll write a simple control flow statement that will check for the presence of params, and pass them to the `method` if so.  If params is empty, we'll just return the `@books` instance variable.  Because of this, we can remove the `attr_reader` method since its functionality is replicated in the `books` method.

{% highlight ruby %}
# 10) Implement `books` method to take optional search params as well

class Library
  def initialize(collection)
    @books = collection
  end

  def books(params = {})
    if params
      search(params)
    else
      @books
    end
  end

  private

    def method_missing(method, *params)
      method = method.singularize

      if book_types.include?(method)
        params['type'] = method
        search(params)
      else
        raise(NoMethodError, "Undefined method #{method} called on Library")
      end
    end

    def search(params)
      @books.select { |b| b.values_at(*params.keys) == params.values }
    end

    def book_types
      @book_types ||= @books.pluck('type').uniq
    end
end
{% endhighlight %}

That's all for now, keep on Rubying!
