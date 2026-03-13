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
reading from a Git repo.

The `allArticles` function commandeers this logic. It reads all
articles from the blog repo. This is what it currently looks like:

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
	If err != nil {
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

There are five pivotal steps that can be outlined here:

1. Create the in-memory filesystem: `fs := memfs.New()`
2. Clone the blog repo worktree into this filesystem:
   `git.Clone(memory.NewStorage(), fs, &git.CloneOptions{...}`
3. Read the given *genre* from within the in-memory worktree:
   `entries, err := fs.ReadDir("./" + genre)`. I go into more detail
   on genres in the Buildablog [README](https://github.com/BrandonIrizarry/buildablog/blob/main/README.md#frontmatter).
4. Run some code that marshals each Markdown entry under the genre
   folder into an article struct that later on gets used inside a Go
   template: `articles, err := entriesToArticles[F](fs, genre,
   entries)`
5. Return these articles, along with an error, to the REST endpoint
   handler call site.

# Flexibility

The blog repo itself is configurable via the `BLOGDIR` environment
variable. This name is a throwback from when it was using the local
filesystem directly; now, it can also be set to an `https` remote
repo. On my VPS, I have it set to `/var/git/brandons_blog`, which
indeed was my endgame all along; the only admin-type thing I had to do
was mark it as a safe repo using `git config`.

Now, the one drawback to all this is that I've gotten used to seeing
immediate feedback once I edit my content. Now, I have to remember to
commit changes first when testing locally.

Anyway, really good stuff.
