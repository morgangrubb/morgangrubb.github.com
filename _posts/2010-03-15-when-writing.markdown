---
layout: post
title: ActiveRecord sugar
keywords: ruby on rails, ruby, activerecord
description: A little sugar for ActiveRecord assignments.
---

This is kind of old now and will be made redundant by Rails 3 but I thought I'd mention it anyway.

Back when [Just Landed](http://www.justlanded.com/) was still using Globalize (prior to version 2) for it's translation trickery (so this was a while ago now) we would constantly have issues chaining getters and setters on the model translated fields.

Basically, what was happening was that Globalize would make a pig's ear out of defining setters and getters on the model and you'd be left wondering what on earth was going on when you attempted to chain one of those methods and called super.

So Jorge figured out a solution involving creating modules on the fly in order to ensure that super() actually had something to call and I sugared it up thusly:

{% highlight ruby %}
# Allows for easily chaining assignment of methods that use method_missing for assignment
# such as attributes on ActiveRecord objects.
#
# Be sure to call super or shit won't work.
#
# It's just a sugared version of Jorge's idea.
#
# Usage:
#
# class Comment < ActiveRecord::Base
#   when_assigning :subject do |value|
#     value.strip!
#     super value
#   end
# end
#
module WhenWriting
  def when_reading(field, &block)
    chain_attribute field, &block
  end
  
  def when_writing(field, &block)
    assignment_field = field.to_s.ends_with?('=') ? field : :"#{field}="
    chain_attribute assignment_field, &block
  end
  
  def chain_attribute(method, &block)
    mod = Module.new do
      define_method method, &block
    end
    class_eval do
      include mod
    end
  end
end
{% endhighlight %}

This is a handy tool to have lying around for when a plugin has done something weird to your models and chaing accessors with super breaks.

