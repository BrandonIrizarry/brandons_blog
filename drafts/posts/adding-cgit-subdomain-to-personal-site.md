+++
title = "Adding a CGit web interface"
tags = ["linux", "nginx", "certbot"]
summary = """

Here I document my experience setting up CGit on my VPS.

"""
+++

# Motivation

To update the content of my blog, I have to do something of a dance. I
like versioning my blog's *content* as a separate Git repo,
independent of the SSG that manages its final layout, rendering,
styling, etc. So right now what I'm doing is this:

1. Commit all changes.
2. Push the changes to GitHub, where it's currently hosted in a
   somewhat centralized manner.
3. Log in via SSH into my VPS.
4. Navigate to the cloned version of the blog repo that lives there,
   and run `git pull`.

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
populated with files. However, the buildablog server does expect to
see a working directory with blog files (not just blobs, for example),
so this doesn't solve my problem.

# Cgit

What I ended up doing in the end didn't solve this problem, but it
ended up becoming an interesting rabbit hole in its own right.

I ended up adding the Cgit web interface to my site, available via
<https://git.brandonirizarry.xyz>. I checked out a [tutorial](https://landchad.net/cgit/) on
how to do it, but their suggested Nginx setup was off in some
parts. 

After a ton of false starts, I ended up doing the following. Note that
this assumes that you've already followed the aforementioned tutorial
on setting up your `git` user and its home directory. As a quick addon
to that, I suggest adjusting the permissions for the `/var/git`
directory to 775 (I had initially found they were set to 770.) This
allows the Cgit web interface to actually read and display your hosted
repos, which is after all the point.

1. Add two new *external records* for `git.brandonirizarry.xyz` to my
   site's DNS configuration over on Epik: the A (IPv4) and AAAA (IPv6)
   records, per usual if you're already somewhat familiar with this
   thing.

2. Start out with the `http` version of the site you're serving. Not
   only does this make adding the TLS certificate later on painless,
   you can immediately verify that your new subdomain is, in fact,
   being hosted. Add the following server block to whatever nginx
   configuration you have published as your website, and then reload
   the nginx configuration with something like `sudo systemctl restart
   nginx.service`:

```
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
   someone nonlinear path in achieving my setup, but this should be as
   simple as running `sudo certbot --nginx`, and then selecting your
   subdomain from the menu options. Here I'm assuming you've already
   gotten a certificate for your main site, hence you only need one
   for your new subdomain.
   
From there you shouldn't even have to restart Nginx: you should see
that your subdomain is available over `https`.

# Five Strikes and You're Out

I learned, through banging my head against various misconfigurations
(both from the DNS and Nginx sides), that Let's Encrypt (what Certbot
uses to issue the certificate) [imposes a rate limit](https://letsencrypt.org/docs/rate-limits/#authorization-failures-per-identifier-per-account) on
certificate issues per identifier, which doesn't forgive botched
certificates. The solution here is to run Certbot with the
`--test-cert` flag, which uses Let's Encrypt's staging area, which has
no such draconian rate limit.

   
   
    
    
    


