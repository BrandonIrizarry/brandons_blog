+++
title = "Buildablog v2"
summary = """

Here I describe a major upgrade to the Buildablog project — posts are
now served from the current HEAD of the blog repo, thanks to
go-git.

"""

date = 2026-03-12
tags = ["go", "buildablog"]

+++

# I Finally Did It

This post is how I solved a problem raised in [a previous
post](/posts/2026-03-06). There, I had left things halfway: I had only installed some
infrastructure, in the form of Cgit, that was partially
suggestive of a solution.  

Here, I document how I finally leveraged that piece as part of a
*full* solution to the problem.  

Previously, the steps for updating my blog's content had been:  

1. Commit all changes.
2. Push the changes to GitHub.
3. Log in via SSH into my VPS.
4. Perform a `cd` into the `brandons_blog` directory, and run `git
   pull`.

Now, they're simply:  

1. Commit all changes.
2. Push to `https://git.brandonirizarry.xyz/brandons_blog`.

# `go-git`


