---
layout: post
title: Super dirty hacks to get domains running
keywords: ruby on rails, ruby, activerecord, a2, a2hosting
description: Not a proud moment.
---

This blog is hosted on Github using their hosted pages service which requires a paid
account to use your own domain name. I don't have a paid account but I do have a couple
of domain names so I thought I could fake it.

What follows is not in any way a recommendation, it's just what I did. Would I do this
again? Well, it works, so probably. But I don't think it's a good idea at all.

The plan I had was to write a very basic Rack application, host that on AppEngine (I don't
know enough about Python or Java to quickly knock up what I wanted but I can blat it
out in Ruby in a few minutes so I'll stick with that and the JRuby deployment packages
look to be very mature).

The Rack application only has to check for the existance of a file locally, if it doesn't
exist it will fetch it from the Github pages service and cache it locally. It also should
answer a specific URL that would cause it to trash all locally cached files which could
be triggered by a callback hook from Github, helping to keep it in sync with the
repository.

It's dirty, I know, but it's simple and will work.

So I followed the guides and got AppEngine/JRuby/Rack up and running. The only problem is
that it's really slow to spin up the service. I don't think this is quite the answer
that I want - it's only HTML so we should be able to serve it up pretty damn quick.

Then it occurred to me that I pay for hosting with A2, and they do Ruby/Mongrel on
their shared hosts. That's cool, I can make that work.

Only I couldn't figure out how to set it up in the environment they provide.

They do, however, provide an instant Rails environment setup so out of frustration
I set that up, dropped my Rack app in as a Metal class, and watched everything work
perfectly.

Obviously this is daft but it works and now I have the Github hosted pages solution for
my own domain without extra monthly fees.

If anyone is interested, this is the Metal class:

{% highlight ruby %}
require 'open-uri'

class Fetch
  def self.call(env)

    if env["PATH_INFO"] =~ /reset_pages/
      Resource.destroy_all
      return [200, {"Content-Type" => "text/plain" }, "All good"]
    end

    file = (env["PATH_INFO"] || "/")[1..-1]
    file = "index.html" if file.blank?

    object = Resource.find_by_name file
    if object
      return [200, { "Content-Type" => object.content_type || "text/plain" }, object.body]
    end

    # Fetch this file from github
    response = open("http://morgangrubb.github.com/#{file}")
    if response
      object = Resource.create :name => file, :content_type => response.content_type, :body => response.read
      return [200, { "Content-Type" => object.content_type || "text/plain" }, object.body]
    end
    
    [404, {"Content-Type" => "text/html"}, "Not found"]
  end
end
{% endhighlight %}

I'm so very sorry.
