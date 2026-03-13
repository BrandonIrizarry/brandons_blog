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

I finally solved a problem I mentioned in [a previous post](/posts/2026-03-06). There,
I had left things halfway: I had only installed some infrastructure,
in the form of Cgit, that was partially
suggestive of a solution.  

Here, I document how I finally leveraged that piece as part of a
*full* solution to the problem.  

Previously, the steps for updating my blog's content had been:  

1. Commit all changes.
2. Push the changes to GitHub.
3. Log in via SSH into my VPS.
4. Perform a `cd` into the `brandons_blog` directory, and run `git
   pull`.

This has now been reduced to two steps:

1. Commit all changes.
2. Push to `https://git.brandonirizarry.xyz/brandons_blog`.

# How it Works
 
Previous versions of Buildablog would read posts directly from the
local filesystem. For starting out, this was an entirely intuitive and
sensible thing to do. However, I had run into a wall, since I couldn't
push to a non-bare remote, and Buildablog needs to see actual files in
order to publish them.

I also wanted to keep things conceptually simple and avoid something
like a separate call to `scp` or `rsync` — I would only rely on
Git. After all, this is how Git forges themselves work: push, and
everything is just there, present. Not just present, but presumably
usable in some form or another. I wanted my application to take
advantage of this intuitive simplicity.

Luckily, [go-git](https://go-git.github.io/docs/) comes to the rescue here. At first I tried to
implement Git-based reads alongside conventional filesystem reads, but
couldn't figure out how to make these two methods play nicely in the
same codebase. So I decided to throw out the latter, relying solely on
reading from a Git repo. The `allArticles` function reads all articles
from the given repo. This is what it currently looks like:

```go
func allArticles[F types.Frontmatter](repo string) ([]types.Article[F], error) {
	fs := memfs.New()
	genre := (*new(F)).Genre()

	_, err := git.Clone(memory.NewStorage(), fs, &git.CloneOptions{
		URL: repo,
	})
	if err != nil {
		return nil, fmt.Errorf("can't clone repository %s: %w", repo, err)
	}

	log.Printf("Successfully cloned repository %s", repo)

	entries, err := fs.ReadDir("./" + genre)
	if err != nil {
		return nil, err
	}

	log.Printf("Successfully fetched genre entries for '%s'", genre)

	articles, err := entriesToArticles[F](fs, genre, entries)
	if err != nil {
		return nil, err
	}

	return articles, nil
}
```


