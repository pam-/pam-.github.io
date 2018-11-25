---
layout: post
title: "computering #01"
subtitle: "an exposing series about TIL. small and big."
type: code
date: 2018-11-24 20:24 -0500
---

## #01: CSS selectors and parents

I was working on adding some styles to this blog because it is and will always be a work in progress (just like myself, tbh), and I stumbled upon an interesting discovery.

#### The deets

This blog is built with [Jekyll][jekyll] and comes with a lot of sweet tools and shortcuts that I can, as far as I know, customize pretty easily. One of these tools is a syntax highlighter for code blocks that uses [Liquid][liquid] and [Rouge][rouge] like so:

{% highlight ruby %}
def print_hi(name)
  puts "Hi, #{name}"
end
print_hi('Tom')
#=> prints 'Hi, Tom' to STDOUT.
{% endhighlight %}

and outputs the following `html`:

```
<figure class="highlight">
  <pre>
    <code class="language-ruby">
    </code>
  </pre>
</figure>
```

I was wondering it would be possible to target the parent of the `code` element with CSS in order to style it depending on the language name. And...turns out it is [not][css-tricks]. TIL!

[liquid]: https://jekyllrb.com/docs/liquid/
[rouge]: https://github.com/jneen/rouge
[css-tricks]: https://css-tricks.com/parent-selectors-in-css/
