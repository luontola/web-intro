---
layout: page
title: Templating
permalink: /templating/
---

Last time we created a navigation menu with a link to the `pictures.html` page, but that page doesn't yet exist. Create a copy of `my-page.html`, name it `pictures.html`, and change its title and heading. Find about a dozen images which you like from [Public Domain Pictures](http://www.publicdomainpictures.net/) and add their thumbnails as images on your `pictures.html` using the [`<img>`][html-img] tag.

```html
<img class="album-photo" src="http://www.publicdomainpictures.net/pictures/50000/t2/cat-looking-up.jpg">
<img class="album-photo" src="http://www.publicdomainpictures.net/pictures/30000/t2/cat-in-the-city-5.jpg">
<img class="album-photo" src="http://www.publicdomainpictures.net/pictures/30000/t2/annoyed-cat.jpg">
...
```

As shown above, give the images a unique class and use CSS to make them look like a photo album.

![Photo album](/screenshots/templating-album.png)

[View solution](https://github.com/orfjackal/web-intro-project/commit/42e94566c051ec08df4ed6e633993b44f57af07d)


## Don't Repeat Yourself

In programming we have a principle called [Don't Repeat Yourself (DRY)][dry] which says that "every piece of knowledge must have a single, unambiguous, authoritative representation within a system." If we follow that principle, our code will be easier to understand and modify.

Our little web site is already breaking the DRY principle. Both of our pages are repeating the HTML for the layout and navigation; adding another page would require changing the navigation on every page. Additionally the pictures page has lots of similar `<img>` tags with just the URL changing. To solve this problem, we will introduce *templating*, so that all the common HTML will be in one place.


## Running a Web Server

This far we have created our site with just static HTML pages, but in order to use templates, we must create a web server which will generate the HTML that will be shown to the user.

We will use the [Ruby][ruby] programming language to create the web server. If you haven't yet installed Ruby, you should do it now.

[Sinatra][sinatra] is a framework for making web applications in Ruby. Follow the [instructions][sinatra] on Sinatra's web site to install Sinatra, create a minimal web application and then run it. It should start a web server on your own machine in port 4567. To view it, visit <http://localhost:4567/hi> in your web browser.

![Your first web server is working](/screenshots/sinatra-hello-world.png)

[View solution](https://github.com/orfjackal/web-intro-project/commit/8254bc27671ac4a3236a381878bc3dcb8d3f70d5)


### Making File Changes Visible

Always when you change your application, you will need to stop and restart the web server. To stop the web server running in your terminal, press `Ctrl+C`. To start it again without having to type the command, press `Up` and then `Enter`. Try it a couple of times by changing the text that the server shows. If you get tired of doing that, Sinatra's FAQ contains instructions on [how to reload the application automatically][sinatra-reloading] when a file is changed.


## Your Site On a Web Server

As a first step towards serving our web site through Sinatra, find out how Sinatra can [serve static files][sinatra-static] and use it to serve our existing HTML and CSS files as they are. Then visit <http://localhost:4567/pictures.html> in your web browser and make sure that your site looks the same as before.

![Your first web server is working](/screenshots/sinatra-static-files.png)

[View solution](https://github.com/orfjackal/web-intro-project/commit/542566800ca115323affd8e25f548e23e7c67008)


## Fixing the Front Page

You should make sure that people can visit your web site through its front page, instead of having to know one of its sub pages. Tell Sinatra to [redirect][sinatra-redirect] the path `/` to `/my-page.html` and then visit <http://localhost:4567/> to see if it will show your web site's front page.

![Front page redirect works](/screenshots/sinatra-redirect.png)

[View solution](https://github.com/orfjackal/web-intro-project/commit/60c9efb2b60ef39ee901edbf293cb76d23c79f41)


## Generating a Page Dynamically

The idea of a templating system is to dynamically generate the HTML that will be shown to the user. To make sure that we understand how to generate pages dynamically with Sinatra, add the following code for a [parameterized route][sinatra-routes] to your application and then visit <http://localhost:4567/my-page> (Note: no `.html` at the end of the URL because else Sinatra would just serve the static file directly)

```ruby
get '/:page' do |page|
  "stuff before" + IO.read('public/' + page + '.html') + "stuff after"
end
```

![Dynamically generated HTML page](/screenshots/templating-dynamic-page.png)

Check the source code of the page in your web browser to make sure that it shows the HTML of `my-page.html` but with some stuff added to the top and bottom of the HTML file.

![Dynamically generated HTML page](/screenshots/templating-dynamic-page-source.png)

[View solution](https://github.com/orfjackal/web-intro-project/commit/a9b6ff30ae8ee5fc247a0609542489878ee327dd)


## A Bare Bones Templating System

Now that we have a proof-of-concept for dynamically generating HTML and for reading existing HTML files, we can use that to create a very basic templating system. Create a directly `views` and move all your HTML files from the `public` directory there (`views` is Sinatra's default directory for templates, though that doesn't matter yet). Create empty files `layout-top.html` and `layout-bottom.html` there as well.

Then change your application to have the following route and make sure that your application still looks the same as before. The difference is that now the pages are no more served as static files, because they are no more in the `public` directory, but they are generated dynamically using this code.

```ruby
get '/:page.html' do |page|
  IO.read('views/layout-top.html') +
  IO.read('views/' + page + '.html') +
  IO.read('views/layout-bottom.html')
end
```

Now you can move the top and bottom parts of the code from your individual HTML pages to the `layout-top.html` and `layout-bottom.html` files. Restart your application and check that it still looks the same as before.

[View solution](https://github.com/orfjackal/web-intro-project/commit/1ed8c59fea1452a8f6b6802b5408c690318eb198)

Now you have managed to apply the DRY principle and in the future it will be enough to change just one place when you change the layout or add pages to the navigation menu. You also understand the basic idea of how templating works. Next let's use a proper templating system.


## A Proper Templating System

If we would create our own templating system for every project, we would get no real work done. Thankfully other programmers have solved the templating problem already many times in the past, so we can avoid reinventing the weel and just reuse the code that they have written.

[ERB][erb] is a templating system which comes with Ruby's standard library, so it's easy to get started with and also [Sinatra supports it][sinatra-templates]. Change your application to use ERB instead of the code we wrote earlier.

* Combine `layout-top.html` and `layout-bottom.html` into a file `layout.erb`, and add the `<%= yield %>` code where the page content should be inserted
* Rename the extension of the `.html` pages in `views` directory to `.erb`
* Change your route to call the `erb` method:

```ruby
get '/:page.html' do |page|
  erb page.to_sym
end
```

After restarting your application, check that all the pages still look the same before. This time it's being powered by the ERB templating system, so later we'll be able to take advantage of some of its useful features.

[View solution](https://github.com/orfjackal/web-intro-project/commit/480bf7c0cea7183fb26a42c9ac6f9f186b9d0e3b)


## Relative and Absolute URLs

Before moving on to the next topic, let's clean up our code a little bit. The front page of a site should be at the root path (`/`) and there is a convention to call it the *index* page. Rename `my-page.erb` to `index.erb` and change your routes to render it at the root:

```ruby
get '/' do
  erb :index
end
```

You will notice that your navigation links won't work anymore. How to make links to the index page when its file name is not shown on the URL? We could use an absolute URL `<a href="http://localhost:4567/">`, but that wouldn't work when we move our site to another domain.

The recommended solution is to URLs relative to the root of the site, i.e. `<a href="/">` and `<a href="/pictures.html">`. Relative URLs which don't start with `/` are relative to the current directory, so our old navigation menu might stop working if we organize the site into subdirectories. Go change all the URLs in your templates to be relative to the root.

![Front page at the root](/screenshots/templating-index.png)

[View solution](https://github.com/orfjackal/web-intro-project/commit/9fb70ea83b984888bb39e66e0e8fd22e227a6de6)


[html-img]: https://developer.mozilla.org/en/docs/Web/HTML/Element/img
[dry]: http://programmer.97things.oreilly.com/wiki/index.php/Don't_Repeat_Yourself
[ruby]: https://www.ruby-lang.org/
[sinatra]: http://www.sinatrarb.com/
[sinatra-reloading]: http://www.sinatrarb.com/faq.html#reloading
[sinatra-static]: http://www.sinatrarb.com/intro.html#Static%20Files
[sinatra-redirect]: http://www.sinatrarb.com/intro.html#Browser%20Redirect
[sinatra-routes]: http://www.sinatrarb.com/intro.html#Routes
[sinatra-templates]: http://www.sinatrarb.com/intro.html#Views%20/%20Templates
[erb]: http://www.stuartellis.eu/articles/erb/
