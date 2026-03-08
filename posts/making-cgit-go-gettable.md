+++
title = "Making Cgit Repos Installable as Go Packages "
summary = "Steps I took to configure my private Cgit repo hub to host Go packages."
tags = ["nginx", "cgit", "go"]
date = 2026-03-07
+++

# Putting my private Git server to good use

In my quest to learn the real ins and outs of Go by going through *The
Go Programming Language*, I decided to use my newly minted private
repo [hub](https://git.brandonirizarry.xyz) to store exercises from the book as packages I can reuse
for later exercises. For example, I decided to make the lissajous
example from Chapter 1 into a separate installable [package](https://git.brandonirizarry.xyz/lissajous). The
lissajous package is used to create a GIF of a [Lissajous curve](https://en.wikipedia.org/wiki/Lissajous_curve),
which was a staple visual effect in old sci-fi movies.

However, simply "go-getting" from the repo's URL, as you would in the
case of GitHub, isn't that simple. There were multiple hiccups along
the way which I had to overcome. What follows is my best attempt to
piece together what I did to finally enable Go package installation
from my private Git server. Hopefully this account will serve two
purposes:

1. Set people straight who are looking for answers to this same
   question.
   
2. Serve as a reference for my future self in case I have to do this
   again.

# Steps

## Inject the appropriate HTML `meta` tag from Nginx

This [excellent](https://anirudh.fi/go-get-cgit ) blog post by Anirudh
Oppiliappan set me on the right path for fixing an error I encountered
early on involving a missing `go-import` something-or-other:


I ended up putting the `sub_filter` stuff inside the `location @cgit`
block:

```nginx
    location @cgit {
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME /usr/lib/cgit/cgit.cgi;
        fastcgi_param PATH_INFO $uri;
        fastcgi_param QUERY_STRING $args;
        fastcgi_param HTTP_HOST $server_name;
        fastcgi_pass unix:/run/fcgiwrap.socket;

        # Make our repos go-gettable.
        sub_filter '</head>'
         '<meta name="go-import" content="$host$uri git https://$host$uri"></head>';
        
        sub_filter_once on ;
    }
```

Remembering the semicolons here is important. You can always use `sudo
nginx -t` to test whether your current config is valid.


## Configure Git to use SSH for the Git server

Add this block to your home directory's `.gitconfig` file, making the
appropriate substitution for "example.com":

```ini
[url "git@example.com:"]
	insteadOf = https://git.example.com/
```

Alternatively, you can issue the equivalent command line invocation:

```
git config --global url."git@example.com".insteadOf "https://git.example.com/"
```

Now, whenever you invoke `go get`, you'll be prompted for your SSH
password.

Note that, in this case, the `git` in `git@example.com` refers to the
Linux *user* on your remote server that owns the directory containing
all your Git repos. This syntax mirrors the one used when logging into
the same server via SSH, viz. `ssh git@example.com`.

## Set GOPRIVATE (optional?)

I'm not sure I need to set this in my case, since AFAICT this has more
to do with close-sourcing code, which isn't my intention here. But I
threw it in just in case.

Since I want **all** my repos to be potentially installable as Go
packages for now, so I use a glob to indicate that:

`go env -w GOPRIVATE=git.brandonirizarry.xyz/*`

Initially, I had set `GOPRIVATE` to point to the `lissajous` repo
only, though this glob technique should work also (and be way easier
to maintain.)
