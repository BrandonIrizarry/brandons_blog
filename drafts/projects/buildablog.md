+++
name = "buildablog"
host_url = "https://github.com/BrandonIrizarry/buildablog"
synopsis = "The SSG used to build this site."
stack = ["Go", "HTML", "CSS"]
thumbnail = "assets/github-white.svg"
date = 2025-03-01
+++

# Motivation

While looking for options to host my blog, and having tried Hugo and
Eleventy, I decided to write my own SSG.

I had completed the [Build a Static Site Generator in Python](https://www.boot.dev/courses/build-static-site-generator-python)
course on boot.dev, and so already had some inkling of what's involved
here. That course requires the learner to manually parse Markdown into
HTML; in my project (which uses Go for the backend), Markdown parsing
is forwarded over to the [frontmatter](https://github.com/adrg/frontmatter) and [goldmark](https://github.com/yuin/goldmark) libraries
(with help from the [goldmark-highlighting](https://github.com/yuin/goldmark-highlighting/v2) extension library for
prettifying code listings.)

Basically, it felt like writing my own SSG from scratch felt like it
was going to be at most not that much harder than learning the ins and
outs of configuring an existing SSG like Hugo or Eleventy. It would
also in the end allow me to configure my site at the atomic level, so
to speak - I write and maintain the server endpoints, the HTML Go
templates, the CSS styling, you get the idea.

Using Go templates can be daunting at first, but Section 4.6 of of
*The Go Programming Language* — Text and HTML Templates — helps. Jon
Calhoun has a good [mini course](https://www.calhoun.io/intro-to-templates/) on templates, which I had already
completed in its entirety when I needed knowledge of templates for a
previous project I completed, called [Juices](https://github.com/BrandonIrizarry/Juices). Another thing I
learned while working on my SSG is that template parsing should not be
done when serving an endpoint, since this creates a significant
performance bottleneck.

Generics also came in handy when I wanted to reuse the exact same code
for differing frontmatter layouts (e.g., project posts like this one,
versus ordinary blog posts.)

# RSS

Setting up RSS for my site (once you know the tricks) turns out to be
surprisingly straightforward, especially with Go's ability to marshal
all things structured (XML, TOML, JSON, etc.) to and from itself. At
one point I was torn between using either Atom or RSS for my feed,
since the former is [touted](https://nullprogram.com/blog/2013/09/23/) as strictly superior to the latter; in
the end I decided that RSS is good enough to get things started for
now.

Two notable resources for learning about interfacing RSS with Go code
are:

1. [Create an RSS Feed (WikiHow)](https://www.wikihow.com/Create-an-RSS-Feed)  
   Super helpful for getting the details of the layout right.

2. [Build Your Own RSS Feed Generator in Go (YouTube)](https://www.youtube.com/watch?v=b2E1JpC38Pg )  
   Helpful for understanding the gory details of what the acutal
   marshalling logic looks like.

With regard to RSS, I feel *all* blogs should have an RSS/Atom
feed. There are some fantastic ones out there that don't!

When setting up RSS, I finally grokked something while reading a [blog
post](https://pluralistic.net/2024/10/16/keep-it-really-simple-stupid/) by Cory Doctorow where he advocates for RSS: RSS *is* social
media.

# Sources of Inspiration

I took (and continue to take) inspiration from blogs I've seen in the
wild, such as:

1. https://maurycyz.com/  
   I ripped off a lot of CSS (and general design decisions) from this
   site. 😁 The great thing about this site is that it proves that you
   can be both a highly intelligent and original thinker without being
   a techie conformist or all-around hipster with regard to things
   like site aesthetic, opinions, and so on. Such concerns create
   performance anxiety which get in the way of you actually making
   something *good*.  
   
   As a shout out, the author even gives some quick lessons on setting
   up a blog and writing the HTML and CSS for a simple website (which
   I also took pointers from! This guy probably did a better job of
   explaining what `width: 100%;` means then a lot of resources I've
   seen online.)

2. https://nullprogram.com/  
   For pointers on how to establish a convention for blog post
   slugs. This author uses the date-based scheme, which I adopted in
   the end.

3. https://elly.town/  
   For general compactness of aesthetic. While I don't plan on going
   that route, the nod to S-expression syntax is nice.


