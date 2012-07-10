---
layout: post
title: "Now presenting SimpleDecorator"
description: ""
category: 
tags: ['design-pattern', 'ruby', 'delegator', 'decorator', 'simple-decorator']
---
{% include JB/setup %}


So. I just finished the first version of the SimpleDecorator gem, now in 0.2.0.
You can find it on Github, or on [Rubygems](https://rubygems.org/gems/simple-decorator).

## What the hell is a decorator ?

Well, of course, a design pattern.<br />

A **decorator** (pattern from [[\[GOF\]](http://en.wikipedia.org/wiki/Design_Patterns)) allows you to add additional or specific behavior to an object instance,
 by wrapping this instance and providing new methods. Decorators are an easy way to provide a peculiar interface in a given context.<br />

SimpleDecorator allows you to quickly implement `Decorators` in your application.

## A true decorator
SimpleDecorator decorators are true decorators :

* They delegate unknown methods to the underlying objects
  {% highlight ruby %}
  class User
    def foo
      'bar'
    end
  end

  class Decorator < SimpleDecorator

  end

  user_decorator = Decorator.new(User.new)
  user_decorator.foo
  # => 'bar'
  {% endhighlight %}

* They can be stacked infinitely
  {% highlight ruby %}
    class User
      def initialize; @count = 1 end

      def count
        @count
      end
    end

    class CountDecorator < SimpleDecorator
     def count
       @component.count + 1
     end
    end

    class TripleCountDecorator < SimpleDecorator
      def count
        @component.count * 3
      end
    end

    user = User.new
    user.count
    # => 1

    CountDecorator.new(CountDecorator.new(user)).count
    # => 3

    TripleCountDecorator.new(CountDecorator.new(user)).count
    # => 6
  {% endhighlight %}

* They are fully transparent
  {% highlight ruby %}
    class User; end
    class Decorator < SimpleDecorator; end
    class OtherDecorator < SimpleDecorator; end

    user = User.new
    decorator = Decorator.new(OtherDecorator.new(user))
    decorator.class
    # => User
    decorator.instance_of? User
    # => true
    decorator == user
    # => true
  {% endhighlight %}

## Installation
`gem install simple-decorator`

## Usage
To create a decorator, simply inherit from SimpleDecorator.

{% highlight ruby %}
class Decorator < SimpleDecorator
end
{% endhighlight %}

Then, you just need to wrap your object inside your Decorator class
{% highlight ruby %}
class Foobar; end
# Decorator#new takes the instance you want to decorate as an argument
Decorator.new(Foobar.new)
{% endhighlight %}

It will give you access to the following instance methods :
 `source`, `decorated` which will return decorated object.

Inside your Decorator class, you can access the decorated object through the instance variable `@component`
{% highlight ruby %}
class Foobar
  def foo
    'bar'
  end
end

class Decorator < SimpleDecorator
  def foo
    'foo' + @component.foo
  end
end

Decorator.new(Foobar.new).foo
# => 'foobar'
{% endhighlight %}