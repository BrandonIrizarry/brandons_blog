+++
title = "Adding a CGit Subdomain To My Site"
tags = ["linux", "nginx", "certbot"]
summary = "Setting up CGit on my VPS."
date = 2026-03-06
+++

# Motivation

To update the content of my blog, I have to do something of a
dance. My blog's content started out life as a subdirectory of my SSG
project, but now lives in its own separate Git repo. This is super
convenient since I now can host my blog wherever I want on my VPS's
filesystem, for example in a folder called `brandons_blog`. So right
now what I'm doing is this:

1. Commit all changes.
2. Push the changes to GitHub, where it's currently hosted in a
   somewhat centralized manner.
3. Log in via SSH into my VPS.
4. Perform a `cd` into the `brandons_blog` directory, and run `git
   pull`.

However, I thought, "wouldn't it be nice if I could push **directly**
to the VPS repo?" And so I started working on that idea in the obvious
manner: add the VPS repo as a remote, such that I would be pushing to
`vps/main` alongside of `origin/main` (where `origin` points to
GitHub).

It turns out that this alone is a shade more complicated that it would
seem: I had to first add a new `git` user (see [this tutorial](https://landchad.net/git/)),
and then adjust my SSH configuration appropriately to allow for this
pseudo-user to log in via SSH (since I would be pushing into a
directory now owned by it.)

I quickly learned, to my dismay, that I couldn't push to the VPS
remote if it's not a bare repo. A bare repo is one initialized with
`git init --bare`, so that it doesn't have a working directory
populated with files. However, the buildablog server expects to see a
working directory with blog files (not just blobs, for example), so
this doesn't solve my problem.

# Cgit

What I ended up doing in the end didn't solve this problem, but it
ended up becoming an interesting rabbit hole in its own right.

I ended up adding the Cgit web interface to my site, available via
<https://git.brandonirizarry.xyz>. I checked out a [tutorial](https://landchad.net/cgit/) on
how to do it, but their suggested Nginx setup was off in some
parts. 

After a ton of false starts, I ended up doing slightly different. Note
that this assumes that you've already followed the aforementioned
tutorial on setting up your `git` user and its home directory. As a
quick addon to that, I suggest adjusting the permissions for the
`/var/git` directory to 775 (I had initially found they were set to
770.) This allows the Cgit web interface to actually read and display
your hosted repos, which is after all the point.

Here's what I did. I'm phrasing these in the imperative mood since it
reads better than beginning everything with "I" + past-tense, and the
steps themselves are also suitable as a potential HOWTO for my future
self:

1. Add two new *external records* for `git.brandonirizarry.xyz` to my
   site's DNS configuration over on Epik: the A (IPv4) and AAAA (IPv6)
   records, per usual if you're already somewhat familiar with this
   thing.

2. Start out by adding the Nginx configuration of the `http` version
   of the site. Not only does this make adding the TLS certificate
   later on painless, you can immediately verify that your new
   subdomain is, in fact, being hosted. Add the following server block
   to your published Nginx configuration, and then reload Nginx
   (e.g. `sudo systemctl restart nginx.service`):

```nginx
server {
    listen 80;
    listen [::]:80;

    # Replace this with your actual site.
    server_name git.example.org;

    root /usr/share/cgit ;
    try_files $uri @cgit ;

    location ~ /.+/(info/refs|git-upload-pack) {
        include             fastcgi_params;
        fastcgi_param       SCRIPT_FILENAME /usr/lib/git-core/git-http-backend;
        fastcgi_param       PATH_INFO           $uri;
        fastcgi_param       GIT_HTTP_EXPORT_ALL 1;

        # This part assumes your git user's home directory is /var/git.
        fastcgi_param       GIT_PROJECT_ROOT    /var/git;
        fastcgi_param       HOME                /var/git;
        fastcgi_pass        unix:/run/fcgiwrap.socket;
    }

    location @cgit {
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME /usr/lib/cgit/cgit.cgi;
        fastcgi_param PATH_INFO $uri;
        fastcgi_param QUERY_STRING $args;
        fastcgi_param HTTP_HOST $server_name;
        fastcgi_pass unix:/run/fcgiwrap.socket;
    }
}
```

3. Add a TLS certificate for your subdomain. I admit that I took a
   somewhat nonlinear path in achieving my setup, but this should be
   as simple as running `sudo certbot --nginx`, and then selecting
   your subdomain from the menu options. Here I'm assuming you've
   already gotten a certificate for your main site, hence you need a
   certificate only for your new subdomain.
   
From there you shouldn't even have to restart Nginx: you should see
that your subdomain is available over `https`.

# Five Strikes and You're Out

I learned, through banging my head against various misconfigurations
(both from the DNS and Nginx sides), that Let's Encrypt (what Certbot
uses to issue the certificate) [imposes a rate limit](https://letsencrypt.org/docs/rate-limits/#authorization-failures-per-identifier-per-account) on
certificate issues per identifier (five per hour), which doesn't
forgive botched attempts at certificate registration. The solution
here is to run Certbot with the `--test-cert` flag, which uses Let's
Encrypt's staging area, which has a much more forgiving rate limit.

In digging a bit through the Let's Encrypt [forums](https://community.letsencrypt.org/), I learned
about two super helpful sites for debugging DNS and certificate
issues:

1. <https://letsdebug.net>  

    Super helpful for figuring out issues with bad certificates.

2. <https://dnsviz.net>  

    For debugging a site's DNS config, which I was messing up since I
    wasn't sure in the beginning how to properly add a subdomain to my
    DNS records (somewhat confusingly, Epik places the word
    "subdomain" alongside the CNAME section, making me think CNAME had
    something to do with it, which it doesn't.)

# A Ghost in the Machine?

I finally managed to [host](https://git.brandonirizarry.xyz) my Cgit dashboard on my site, which
currently contains only my blog repo. I even managed to share the link
with a friend of mine, who was successfully able to view it from their
end.

However, when going through some exercises in *The Go Programming
Language* (a story for another time), I happened to cavalierly make a
GET request to that subdomain, which then reported a TLS error. In my
mind this seemed somewhat bonkers, since, after all, everything was
already up and running, no? So late that evening I had to jump back
onto the VPS and do some bespoke troubleshooting.

It looked like there were some redundant server blocks in my Nginx
config file that were added while I was throwing everything and the
kitchen sink at getting a valid TLS certificate. So what I did in the
end was remove everything Certbot had added concerning my `git`
subdomain, essentially reverting back to just the server block shown
just above, and repeating those exact steps — including first
verifying service over `http`. This part of the process for me was the
most satisfying, since it proves that the mere act of publishing a
website on the Web is, at its core, not all that difficult. One thing
different this time though was that, per the options Cerbot presents
you, it sufficed to reinstall the existing certificate, as opposed to
applying for a new one.)

After that, everything was in order! I even checked the site the next
morning just to make sure it had stayed that way. To date, everything
looks good.

# In the End...

In the end, I didn't actually solve my initial problem, but still went
down an interesting rabbit hole, and now have a convenient tool at my
disposal — my own poor-man's GitHub — for personal use. For now, I may
well only use it for throwaway Go packages, in case I don't feel like
using workspaces.



   
   
    
    
    


