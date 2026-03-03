+++
title = "Smoothing Over More Markdown Pain Points"
tags = ["blogging", "emacs"]
date = 2025-12-05

summary = """

A post I had written about a small Elisp helper library I wrote for \
generating a table of contents for a Markdown file.

"""

+++

# I Couldn't Keep It Together

As I go about editing these blogs as Markdown buffers inside Emacs,
I've been running into a snag of sorts. Previously, I had been
exporting Org to Markdown one way or another. I observed how the
Markdown output inserts an anchor tag above a given section as a way
to link to it from the table of contents. I decided to continue this
practice in my now hand-wrought Markdown. However, manually keeping
the table of contents in sync with changes in the outlining of the
content itself—adding and removing sections, renaming sections, and so
on—is a pain.  And so I came up with a way to sync the two, using
Emacs Lisp. Emacs Lisp, or Elisp for short, is the Emacs editor's
extension language: the language you use to write Emacs plugins.

# Elisp For The Win
[Having written](2025-12-03) about my zany Elisp-based Java build system made
me recall those times: I could once again rise to the challenge, and
solve this new problem with Elisp. That's exactly what I did. I wrote
two functions, `bcimd-generate-toc` and `bcimd-remove-toc`. The first
one regenerates the table of contents based on the current set of
level-1 headings. The second one erases the existing table of
contents, along with the connected anchor tags. It's used by the first
function to start out with a clean slate before defining the new table
of contents.

I decided to collect these functions into an installable package. It's
currently available through Emacs' version-control installation
mechanisms (for example, `package-vc-install`.) See the [project
README](https://github.com/BrandonIrizarry/bcimd) for more details.

I find Emacs' VC-based package installation facilities extremely
convenient for writing my own bespoke stuff which I otherwise have to
manage locally. I store it remotely, and install it as an *official*
package, much like how Go packages work. In this way, I can even share
my work with the community.

# Yet Another Yasnippet Testimonial

I also decided to go the extra mile and use a [Yasnippet](Yasnippet) snippet
that generates some stock front matter. In particular, the title of a
given blog post is ripped directly from the name of the file itself,
which first undergoes some on-the-fly formatting. I got this idea from
[another blog](https://weblog.masukomi.org/2024/07/19/using-org-mode-with-hugo/) where the author runs with the whole Yasnippet idea
to set up her `ox-hugo` front matter. In fact, this is what turned me
on to the idea of Yasnippet as a useful tool in general; that is, it
isn't just a lazy man's way of inserting a for-loop into source code.

# Now I Can Keep It Together!

I now use table-of-contents regeneration frequently: writing the
package was a worthwhile investment of time.The only minor hiccup is
that I have to remember to leave two spaces in between headers, so
that the anchor tag doesn't eliminate all whitespace between sections,
an effect which looks aesthetically jarring. I may address this in the
future, but I first need to see how this package interacts with, for
example, level-2 headers. Other ideas include running
table-of-contents generation as an `after-save-hook`, and eventually
writing a full-blown minor-mode. But for now, I'm taking it easy on
this project: I still have to work on other things.
