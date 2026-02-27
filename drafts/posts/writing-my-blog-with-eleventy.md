+++
title = "Writing My Blog With Eleventy"
tags = ["blogging"]
date = 2025-12-03

summary = """

This is a reproduction of a post I had on my old blog, which I've \
since migrated to a custom engine.

"""

+++

# Table of Contents

+ [Introduction](#introduction)
+ [Hugo](#hugo)
+ [Eleventy: The Soup Actually Tastes Good](#eleventy:-the-soup-actually-tastes-good)
+ [Painless Deployment](#painless-deployment)
+ [Conclusion](#conclusion)


<a id="introduction"></a>
# Introduction

This is *at least* my third time trying to start a blog.

First, I experimented with using Org Mode's HTML exporting feature to
create posts; unfortunately, that didn't get me far, though there are
some [interesting attempts](https://one.tonyaldon.com/) by others to this end. I might've
published this material at some point, but at any rate it didn't stay
up long. An early topic from this time include a post about a [Java
build system](https://github.com/BrandonIrizarry/Hydraulic-Make) I once wrote that scanned a `.java` file for its
dependencies (defined by things like package imports and code syntax),
so that those would get passed into `javac` along with the target
file.

<a id="hugo"></a>
# Hugo
I then started writing a blog using Hugo. Hugo was my first encounter
with an SSG. Because of this, I was a bit impatient with Hugo, and hit
a wall every time I came across any sort of complexity. I also got
frustrated with how themes never follow a consistent template; each
does something different, with different elements, and so each one
effectively has different rules. In the end, I published a blog post
or two on GitHub pages using this setup. It was passable, but in the
end configuring it still felt wonky and cargo-culted.

Another reason for why I didn't have success with Hugo was my use of
`ox-hugo`. It's a fun package, and you can tell the author put a *lot*
of love into it. However, using Org Mode as a middleman between you
and Hugo obfuscates the nature of Hugo, something I'm realizing now as
I go deeper into using Eleventy.


<a id="eleventy:-the-soup-actually-tastes-good"></a>
# Eleventy: The Soup Actually Tastes Good

I went ahead and did a little bit of "shopping" for SSGs. I ran into
[Eleventy](https://www.11ty.dev). I watched the author's [intro video](https://www.youtube.com/watch?v=kzf9A9tkkl4), and
immediately took a liking to it. After a few false starts, I cloned
their [official starter project](https://github.com/11ty/eleventy-base-blog), tweaked it here and there, and
the rest is what you're currently looking at.

A huge shift in my thinking which made the leap from Hugo to Eleventy
possible occurred when I learned to stop worrying and love the
Markdown.

I used to think of Markdown as an icky, second-rate version of Org
Mode. Then, I eventually got the hang of writing Markdown using Emacs'
`markdown-mode` package, which is a [masterpiece](https://jblevins.org/projects/markdown-mode/) of a plugin: it
makes the experience of writing Markdown rival that of using Org, and
smoothes out a lot of Markdown's pain points (significant whitespace,
noisy links, etc.) And so I slowly let go of the attachment of using
Org Mode in all the things, and embraced the idea of writing blog
posts directly in Markdown; this also alleviated the complexity of
sundry issues arising from exporting from Org to Markdown.

At first, Eleventy looks like a deceptively complex pile of language
soup: JS, Markdown, templating languages, HTML, and CSS—at times all
occurring within the same file—all somehow live under one
roof. However, tweaking the starter project ended up being a
relatively easy, even pleasant experience.

<a id="painless-deployment"></a>
# Painless Deployment

Even deployment is simple. This site's content is version-controlled
locally. I then build the site, then simply `scp` the `_site`
directory to the appropriate directory in my VPS, where this blog is
hosted. The previous remote `_site` directory is simply overwritten
with the new files. I don't need a GitHub workflow, as I did when
using Hugo with GitHub pages; I don't even need to push to a remote
repo. Copying the files suffices.

<a id="conclusion"></a>
# Conclusion

On the one hand, I'm nowhere near able to make something like the
starter project from scratch. On the other hand, neither am I
daunted. Eleventy in a sense reminds me of Emacs, in that there's a
certain joy to be found in its eclectic complexity. I look forward to
continue using Eleventy as I grow this blog.
