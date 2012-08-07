---
layout: post
title: "{ Rails } Render multiple tags in a helper"
description: ""
category:
tags: ['rails', 'ruby', 'views', 'helper', 'activesupport', 'safebuffer', 'rendering']
---
{% include JB/setup %}

Well, I came across this issue lately : I had this helper `render_each_microposts` which was supposed to call a partial onto each record from my Micropost model.

## First thoughts

Of course, the first thing I tried was to do something like :

{% highlight ruby %}
def render_each_microposts
  @microposts.collect do |m|
    render(partial: 'microposts/single', locals: { micropost : m }
  end.join
end
{% endhighlight %}

So. What does it do ? <br />
Obviously enough, we have a _@microposts_ instance variable, that contains all the microposts we want to display. For it to be an Array or an ActiveRecord Scoped object, we don't care, as long as it respond to _#collect_ (alias for #map, just made more sense here.) and _#join_.<br />
With this _@microposts_ variable, we go through each and every micropost and render them one by one. We assume that our partial looks like :

{% highlight erb %}
<li>
  <h2>micropost.user.name</h2>
  <p>micropost.content</p>
</li>
{% endhighlight %}

Thus, once _#collect_ has been executed, we have a beatiful array of rendered Microposts, looking probably like :

{% highlight ruby %}
# Regular stuff, you see ?
[
  "<li><h2>Gabriel Dehan</h2><p>I'am a Stegosaurus !</p></li>",
  "<li><h2>Pinky Pie</h2><p>Hmmmmmm. Nah.</p></li>"
]
{% endhighlight %}

We joined it because well, helper methods should return Strings, not Arrays.<br />
And... of course, it fails.

**We have a problem :** Our page just rendered our HTML as an escaped string.

This is a normal behavior, most of us already encountered. A common workaround to handle two rendered item in a helper would be to concatenate them :

{% highlight ruby %}
def my_helper
  content_tag(:h1, 'Hello World') + content_tag(:h2, 'Oh haiz !')
end

# Or with partials which we assume does almost the same.
def my_helper
  # Note that we need to add the parenthesis for the renders to work,
  # render is a method, + is a method.
  # When methods are chained you need not to forget parenthesis.
  render(partial: 'my_view/first_heading') + render(partial: 'my_view/second_heading')
end
{% endhighlight %}

This works because we are, in fact, concatanating not two strings, but two **ActiveSupport::SafeBuffer(s)**, as we can see :

{% highlight ruby %}
my_helper.class
# => ActiveSupport::SafeBuffer
{% endhighlight %}

## ActiveSupport::SafeBuffer

### What is it ?
SafeBuffer, is a part of [ActiveSupport Core Extensions](http://guides.rubyonrails.org/active_support_core_extensions.html#output-safety), and it's source code can be found in active_support/core_ext/string/output_safety.rb.

{% highlight ruby %}
module ActiveSupport
  class SafeBuffer < String
    # ...
  end
end
{% endhighlight %}

SafeBuffer is a subclass of string and allows us to create HTML safe strings (Strings that will be considered safe no matter whether they have been escaped or not.)

So, when we wrote down our method using _#+_, it was (obviously) an alias for _ActiveSupport::SafeBuffer_#concat, not string. We could, of course, use _#concat_ or _#+_ and a loop to do our job, but it would be ugly and slow.

### What do we do, now ?
Now we have better knowledge of _ActiveSupport::SafeBuffer_, we can now do :

{% highlight ruby %}
def render_each_microposts
  ActiveSupport::SafeBuffer.new(@microposts.collect do |m|
    render(partial: 'microposts/single', locals: { micropost : m }
  end.join)
end
{% endhighlight %}

Or even better, as _ActiveSupport::SafeBuffer_ extends _String_ with an **#html_safe** method :

{% highlight ruby %}
def render_each_microposts
  @microposts.collect do |m|
    render(partial: 'microposts/single', locals: { micropost : m }
  end.join.html_safe
end
{% endhighlight %}

Which does exactly the same if we look into the source code :

{% highlight ruby %}
class String
  def html_safe
    ActiveSupport::SafeBuffer.new(self)
  end
end
{% endhighlight %}

And now it works, as simple as that.