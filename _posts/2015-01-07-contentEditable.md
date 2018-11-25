---
layout: post
title: "exploring ContentEditable"
type: code
date: 2015-01-07 18:05 -0500
---
_Originally posted on [Medium](https://medium.com/@pam_yam/contenteditable-96898be3968b)._

## Trying to tackle a mini text editor

For a while now, I’ve been meaning to build a platform where my students could exchanges questions and solutions about code. It was going to be called _Issues_. I’m using past tense because _Issues_ ended up just being an experiment and the perfect occasion to try out `contentEditable`.

#### What is contentEditable?
**A small breakdown**

Anything within a web page browser is technically editable. If they wanted, a user could drag a `div`, place it somewhere, add some headers, create an unordered list, and so on. WYSIWIG editors — such as Wix or Square, rely on this concept. With some Javascript handlers, these sites give their users the power to rearrange their web pages and style it however they’d like (to an extent). But it isn’t just about Javascript handlers. The customizable, or rather editable functionality, is rendered (lol) possible by an HTML attribute: `contentEditable`. This attribute is, by default, set to false on any web page but just like most attributes, it can be changed.

Let’s try it out in a test HTML file:

{% highlight html %}
in test.html
<p contentEditable="true"> This is a paragraph that can be changed</p>
<p> This is a paragraph that cannot be changed </p>
{% endhighlight %}

If you load this file in your browser and click on the first paragraph, you will notice that it transforms into what looks like a text-box. You can type into it, and add as much text as you’d like. Cool, right? Before we move on, let’s take a second, and picture what the web would like if contentEditable was true by default…. Pretty scary, I know.
Now that we all agree that contentEditable can be both awesome and terrible, let’s look at one way to implement it.

#### contentEditable = “true”
**the (fun) struggles**

In order to make _Issues_ fulfill its purpose, I had to provide a way for users to input questions and possible solutions. The Medium’s interface is pretty cool imo, so after doing some research (a lot of inspecting the elements) and reading this [article](https://medium.engineering/why-contenteditable-is-terrible-122d8a40e480?gi=fddde894b581) by [@nicksantos](https://twitter.com/nicksantos), I was sold on trying out contentEditable.

It all started with a single div in an html file. A div with `contentEditable` set to `true`.

{% highlight html %}
<div class="editable" contentEditable="true"></div>
{% endhighlight %}

Anybody could now write into this div, which is cool, but any new line, or text node, is contained within a div instead of a structuring text element such as a header or a paragraph tag. This is problematic because after submission, styling our `.editable` div becomes a bit of a nightmare. But more importantly, what happens if the user wants to add hierarchy to their post? It is impossible for them to do so. I soon realized that adding some structured placeholders into our `.editable` div could be a solution:

{% highlight html %}
<div class="editable" contentEditable="true">
  <h1> Enter Title </h1>
  <p> Enter Text </p>
</div>
{% endhighlight %}

The user can now write within those boundaries and create a post that has some structure to it. It’s a solution, certainly not the best, but it is sort of a start. I could then just add multiple placeholders for different ways to format the input, but unless I’m counting on the user getting annoyed, it is certainly not a good idea. This is when Javascript comes into play.

#### Javascript and contentEditable
**more (fun) struggles**

In most cases, Javascript is used to listen to any changes occurring into our document, but in our case, we need it to focus specifically onto our `.editable` div. You might have noticed earlier that when clicking on an editable element, an outline appears. This means that the focus is now on this element, which makes it inherently _selected_ on the DOM. By getting the document’s selection— here our `.editable` component, we are able to grasp all the text written within it. Luckily, getting the selection value from our document can be done with the use of a javascript function:

{% highlight js %}
window.getSelection() // document.selection for IE
{% endhighlight %}

The `.getSelection()` method (also applicable to the document) will return the range of text selected by the user. Again, in our case, this range is defined by our `.editable` div’s content whenever we click on it.
Once we get the selection, we will target the text (aka the `focusNode`) within its containing tag (aka the `parentNode`). We want the tag in order to identify it and change it when needed:

{% highlight js %}
var selection = window.getSelection()
var node = selection.focusNode.parentNode
{% endhighlight %}

With the full node in hand, we’re now able to scan it and change it depending on input. In _Issues_, I wanted the user to enter 3 spaces, then be able to write a block of code (similarly to Stack Overflow). In order to achieve that, I simply had to store the wanted input into a variable like so:

{% highlight js %}
// the following regex simply looks for 3 spaces
// at the beginning of a line
var keyShortcut = /^[^\S\r\n]{3}/gm
{% endhighlight %}

Before we can actually scan the string, we have to store the text contained by our tag. Here using jQuery:

{% highlight js %}
var input = $(node).text()
{% endhighlight %}

Alright. Now, let’s make our `contentEditable` div allow for the generation of a code tag. We’re going to scan the text as the user writes within the div, determine the current container (a new container gets created after hitting return) and then change the said container after the keyboard shortcut is entered. The jQuery event `.keyup()` will listen to the user typing into the div.

{% highlight html %}
<div class="editable" contentEditable="true">
  <p>Enter text here</p>
</div>

<script type="text/javascript">
  $('.editable').on('keyup', function(){
    var selection = window.getSelection();
    var node = selection.focusNode.parentNode;
    var keyShortcut = /^[^\S\r\n]{3}/gm;
    var input = $(node).text();

    if (input.match(keyShortcut) && $(node).is('p')){
      $(node).replaceWith('<code></code>')
    };
  });
<script>
{% endhighlight %}

The user can start typing within our placeholder `p` element. We can then check whether they enter the keyboard shortcut or not, _AND_ make sure that the current node is indeed a paragraph, before replacing the node with a code tag.

There. This is (in a nutshell) what it takes to generate a different tag within an editable component.

Big thanks to [@VeryBadHello](https://twitter.com/VeryBadHello) for helping me out with this!