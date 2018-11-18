---
title: "Preventing Multiple Submissions"
date: 2016-06-20T16:59:16-08:00
showDate: true
draft: false
tags: ["ruby", "rails"]
---

I have a gem that has been giving me trouble lately. It saved me
countless hours of dev time upfront, but is beginning to cause me
problems now that I want to extend it. I learned quickly how to clone
and edit a gem locally using the `gem gem_name, path: 'path/to/local/gem'`,
to get the changes I personally needed.

The difficulty came when, for some yet inexplicable reason, it was causing
duplicate calls. Rather than spend more time digging deeper into the gem
to solve that problem too, I decided to make my own code more robust by
preventing duplicates all together.

I did this by wrapping my original create code in an `unless`. Now, no
matter how many times this gem submits to my controller, it'll only run
the code once.

``` ruby
def method_name
  unless organization.subscribed?
    # subscribe the org
  end

  if organization.subscribed?
    # sub succeeded, get out
  else
    # handle some errors
  end
end
```
