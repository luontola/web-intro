---
layout: default
---

This tutorial will teach the basics of making web applications. Before starting, you will need to install [Ruby](https://www.ruby-lang.org/) and a programmer's text editor such as [Atom](https://atom.io/).


```
Web Intro

Basics of html
Minimal html page
Elements and attributes, some have end tags
Some common tags

Css for colors, fonts and layouts
Can give css for elements or classes
Layou fiddling: goal is to have one big box for the content (and a smaller box for the navigation; later or now?)
- give solid borders for the boxes
- experiment with position, left, right, top, bottom, width, height, min-width, min-height, max-width, max-height
- try at different amounts on content and window size
- when boxes are like you want them to, remove borders and add some background colors and stuff

Multiple pages
Create a copy of the current file as pictures.html; put there a dozen or so pictures
Need navigation on every page
Links using the anchor tag

Removing duplication of layout and nav
Hello world backend to make sure the server is working (see docs)
Routes and handlers, method returns what we want to show
Serve current site throught server as static files to make sure it works like we think it should (see docs)
Redirect index page to our site for the base url to be useful

Instead of servind as static files, serve pages as dynamically generated html
As smallest step create a route that will read files as is and make sure it works - add a print to method or +"foo"
Then extract header and footer to separate files and concatenate them to content
Other people have already made templating systems that do the same thing and much more, so let's use one of them

Create proper index page which is at site root
Relative and absolute urls, in case we add subfolders (demonstrate?)

Remove duplication if pictures page
Just have a list of urls and render them in a template
We can even generate the list automatically based on directory contents

Good to have a title for every page
We can store titles and list of all pages for the navigation menu
Finally some removing of duplicate code by creating methods
```
