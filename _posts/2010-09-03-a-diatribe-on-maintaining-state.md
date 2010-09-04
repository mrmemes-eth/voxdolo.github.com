---
title: A Diatribe on Maintaining State
layout: post
excerpt: |- Interacting directly with the state in instance variables
  considered bad for your software system's health.
---

TL;DR
-----

If you're interacting directly with an instance variable and it's not inside
an accessor or mutator method, you're breaking encapsulation and promoting
close coupling in the design of your object system. Just because an API
encourages this behavior doesn't mean that it's a good design practice (see
[Rails][evil] for a great example of enshrining this bad habit).

The Discourse Proper
--------------------

An object instance, while resident in memory, must have somewhere to store
its state. In Ruby (as in most languages), the instance variable is our go-to
guy. It's so useful and its usage is so universally understood, in fact, that
many people don't look much further than it when determining how to store and
subsequently access state.

### A Common Approach

Let's look at an example of an approach where we directly interact with state
stored in an instance variable:

{% highlight ruby %}
  class Message
    def initialize(body)
      @body = body
    end

    def to_html
      @body.gsub(/\n/,"<br/>")
    end
  end
{% endhighlight %}

I've got some state, I initialize my object with it, and I later access it.
Clearly, this code isn't the worst thing that has ever happened to the
software world. In this simple example, it's largely forgivable to use an
approach like this.

The first principle from [_Design Patterns: Elements of Reusable
Object-Oriented Software_][patterns] is:

> Program to an interface, not an implementation.

Or as I like to dramatize it:

<q>If you're not programming to an interface, you're coupling with an
implementation.</q>

When consuming code (be it your own or someone else's) in an Object-Oriented
system the thing we are meant to be concerned with is the state of the object
itself. To wit: the fact that an object's state is (or isn't) stored in an
instance variable, should be a detail of the implementation.

### An Important Nuance

People tend to think of APIs in terms of how they're consumed externally.
It's equally important to consume them internally. Let's add the requirement
that our `Message` object must remove leading, trailing and repetitive space.

{% highlight ruby %}
  class Message
    def initialize(body)
      @body = body.squeeze(' ').strip
    end

    def to_html
      @body.gsub(/\n/,"<br/>")
    end
  end
{% endhighlight %}

Still a very straightforward implementation, but we start to see a lack of
[cohesion][cohesion] here as the initializer is now responsible for both
storing and manipulating this state. We have yet to declare an interface
though, so there's really little recourse at this point.

### Refactoring To An Interface

I mentioned earlier that interacting directly with instance variables is
largely forgivable in the simple case. That is not to be confused with an
endorsement. Ruby makes it so simple to abstract away state maintenance, that
it feels irresponsible not to. Here's how we might refactor that given our
previous case:

{% highlight ruby %}
  class Message
    attr_reader :body

    def initialize(body)
      self.body = body
    end

    def body=(body)
      @body = body.squeeze(' ').strip
    end

    def to_html
      body.gsub(/\n/,"<br/>")
    end
  end
{% endhighlight %}

With this factoring, we need never know that our state has been stored in that
instance variable. It becomes a detail of the implementation. Additionally,
we've moved the responsibility for cleaning up the excess whitespace in the
message body into the setter.

Now that we've declared an interface, it becomes significantly easier to make
substantial refactors, such as upgrading your state maintenance to allow for
persistence:

{% highlight ruby %}
  class Message
    def initialize(body)
      self.body = body
    end

    def body=(body)
      cache.transaction{|c| c[:body] = body.squeeze(' ').strip }
    end

    def body
      cache.transaction{|c| c[:body] }
    end

    def to_html
      body.gsub(/\n/,"<br/>")
    end

    private
    def cache
      @cache ||= PStore.new('message_cache.pstore')
    end
  end
{% endhighlight %}

Here we begin to see advantages of declaring an interface. By defining an
interface, you effectively declare the "surface area" of your internal API.
This also goes a long way toward making methods that are both atomic and
highly cohesive.

This example is still very simple. In interest of fairness, an equivalent
`class` (assuming you only really care about the value of `to_html`) which
employs direct access to instance variables  might be within your threshold of
acceptable use:

{% highlight ruby %}
  class Message
    def initialize(body)
      @cache = PStore.new('message_cache.pstore')
      @cache.transaction{|c| c[:body] = body.squeeze(' ').strip }
    end

    def to_html
      @cache.transaction{|c| c[:body] }.gsub(/\n/,"<br/>")
    end
  end
{% endhighlight %}

Given the simplicity (and brevity) of this implementation, why is there any
issue with this approach? At this scale, there probably isn't. The previous
objection to this approach regarding the ease of declaring an interface in
Ruby still stands, but it's truly not until the software system exists at a
sufficient level of complexity that you'll start to experience the detriment
of this approach.

### A Slightly More Complex Use Case

Let's add support for finding and storing multiple `Message` object instances.

{% highlight ruby %}
  class Message
    def self.find(id)
      cache = PStore.new('message_cache.pstore')
      new(id,cache.transaction{|c| c[id] })
    end

    def initialize(id, body)
      @cache = PStore.new('message_cache.pstore')
      @id = id
      @cache.transaction{|c| c[@id] = body.squeeze(' ').strip }
    end

    def save
      @cache.transaction{|c| c[@id] = @body }
    end

    def to_html
      @body.gsub(/\n/,"<br/>")
    end
  end
{% endhighlight %}

Now let's see what that might look like when declaring an interface:

{% highlight ruby %}
  class Message
    attr_reader :body
    attr_accessor :id

    def self.find(id)
      new(id, cache.transaction{|c| c[id] })
    end

    def initialize(id, body)
      self.id = id
      self.body = body
    end

    def body=(body)
      self.body = body.squeeze(' ').strip
    end

    def to_html
      body.gsub(/\n/,"<br/>")
    end

    def save
      self.class.cache.transaction{|c| c[id] = body }
    end

    private
    def self.cache
      @cache ||= PStore.new('message_cache.pstore')
    end
  end
{% endhighlight %}

### Analyzing The Implementations

In the first example, you end up with some some incidental coupling in the
initializer. It takes on the responsibility for assigning internal attributes,
manipulating them where necessary, as well as initializing a persistent
reference to the cache itself. Additionally, there's no ability to manipulate
the state of an instantiated object, so another object interacting with an
instance of `Message` wouldn't be able to change the value of its `@body`.

Conversely, the implementation which has declared an interface is ready to
behave as a good citizen within an object system. It allows other objects to
interact with it at all stages of its life cycle in a specified fashion (it
has declared its "surface area"). It retains atomicity and expresses no signs
of coupling.

Another notion that I haven't addressed is that of Code Beauty. This is an
entirely subjective topic, but I contend that a well-specified interface
creates more beautiful code. I'll leave that to you to decide, but I think the
examples do a fair job of showing this.

## Wrapping Up

This is such a simple topic that it's easy to take for granted. To reiterate:
the issues with accessing instance variables directly become most obvious when
a software system attains some basic scale and complexity. Generally speaking,
once objects in a system begin to interact in any significant manner, the
coupling created by not specifying and adhering to an interface starts to
become clear. This coupling makes it more difficult to adhere to [Single
Responsibility Principle][SRP], which subsequently increases rigidity.

The intended takeaway is that you can improve the overall design of your
software system at every stage of its evolution by specifying and adhering to
an interface. One simple way you can do that is to ensure that you're not
manipulating state maintained in instance variables directly.

[evil]: http://vurl.me/UBO "Specifically, the bit about making instance
  variables from controllers available to the view."
[patterns]: http://vurl.me/UBP "The Wikipedia entry on the classic work."
[cohesion]: http://vurl.me/UBQ "The Wikipedia entry on Cohesion"
[SRP]: http://vurl.me/UBR "The Wikipedia entry on the Single Responsiblity
  Principle"
