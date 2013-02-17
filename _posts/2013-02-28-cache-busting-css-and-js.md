---
layout: post
title: Cache busting CSS & JS
author: ajk
excerpt: "Caching of CSS & JS is pretty standard business, and if you want your site to be fast, you should be doing it. But browser caching presents a problem: when you update the website's CSS/JS/other resources, the users' browsers won't download the new files, if they've visited your site before; the resources are cached. And your users surely won't be pushing F5."
---

Browser caching of CSS, JS, images and other website resources is pretty
standard business, and if you want your site to be fast, you should be doing it.
But browser caching presents a problem: when you update the website's
CSS/JS/other resources, the users' browsers won't download the new files, if
they've visited your site before; the resources are cached. For resources, that
change occasionally (such as CSS) this is bad. And your users surely won't be
pushing `F5`, to see if your site has new CSS or JS in store when they come for
a visit. And telling them to push `F5` is just… silly! :)

> There are only two hard things in Computer Science: cache invalidation and naming things.  
> – Phil Karlton

Invalidating (or busting) cached website resources, such as CSS & JS, is not
that hard. There are a few different techniques around, but I've settled on one
particular method, which I think is the best:

**Serve the updated resource under a new name when it has changed.** This will
bust the cache. For example, if I previously had a file called `styles.css`, I
can rename it to e.g. `styles-v2.css`, and the browser will think it's new. Of
course, all the references to it will have to be updated as so:

    <link rel="stylesheet" href="/css/styles-v2.css">

Doing this manually is okay, if the project is small, and you have the nerves.
But you could also automate it!

## Automatic cache busting

Many Web frameworks - such as Ruby on Rails and Liferay - already provide
automatic cache busting. If you're using such a framework, you're set! You don't
have to do anything! Just make sure browser caching is enabled! &lt;3

The next best thing is using a library for your cache busting needs. In this
case, you usually have to use a library specific way of writing the CSS and
other resources to your HTML

And the last resort is to roll it yourself. I'll explain my method for this with
WordPress.

### Automatic cache busting with WordPress

I have my CSS file, which I want to automatically cache bust, at `/wp-
content/themes/ajk/css/main.css`.

First, I place a line like this in the `header.php`:

    <link rel="stylesheet" href="<?php echo auto_version(get_template_directory_uri() . '/css/main.css'); ?>">

Which will give the browser something a line like:

    <link rel="stylesheet" href="/wp-content/themes/ajk/css/main.1361054840.css">

See that `1361054840`? That's a unique identifier, which will change whenever
the file changes. `auto_version()` does the heavy lifting here. It's a function
I've made & put into `functions.php`. You can get the function from [this
gist](https://gist.github.com/Darep/4627661). What `auto_version()` does here
is, it gets the last modified time of the CSS file and places it just before
`.css`. I do the same thing for JS files.

But we're not done yet! Doing just this will give us a 404, because a CSS file
like that doesn't exist. It's actually at `/wp-content/themes/ajk/css/main.css`.
To fix this, we will tell Apache that files that end with 10 numbers and `.css`
or `.js` should be treated differently. I add these line to `.htaccess`:

    # CSS/JS auto-versioning
    RewriteEngine On
    RewriteRule ^(.*)\.[\d]{10}\.(css|js)$ $1.$2 [L]

And now, when the user's browser asks for `main.1361054840.css` Apache will
serve the `main.css` file! Magic! (Note: For now, I'm only using this technique
with CSS and JS files, so I only have `(css|js)` written on the `RewriteRule`
line there)

**This technique can easily be ported to other platforms!** A co-worker recently
did this for a .NET project based on my description: he created a function for
adding the last modified time to a CSS file URL and then created a HttpHandler
to serve the actual CSS file when the URL has 10 numbers followed by a `.css`.

If you happen to have a build process in your website, you could .......

**TODO:** Write about renaming files in build. Build script has to update URLs
in templates

**TODO:** Google says to use a fingerprint instead of a timestamp. This is okay.


### Adding a Query string

Some people suggest adding a query string to the resource URL. Like here:

    <link rel="stylesheet" href="/css/styles.css?v2">

This will bust the cache, but it might have some unwanted consequences: some
proxies - and perhaps even browsers(?) - will not cache URLs that have query
strings in them. You can read about this on [Google's Performance Best
Practices](https://developers.google.com/speed/docs/best-practices/caching),
it's mentioned near the bottom.

**TODO:** some kinda summary
