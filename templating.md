---
layout: chapter
title: Templating
permalink: /templating/
next: /databases/
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


## Running a web server

This far we have created our site with just static HTML pages, but in order to use templates, we must create a web server which will generate the HTML that will be shown to the user.

We will use the [Ruby][ruby] programming language to create the web server. If you haven't yet installed Ruby, you should do it now.

[Sinatra][sinatra] is a framework for making web applications in Ruby. Follow the [instructions](/installation) to install the Ruby language and Sinatra, create a minimal web application and then run it. It should start a web server on your own machine in port 4567. To view it, visit <http://localhost:4567/hi> in your web browser.

![Your first web server is working](/screenshots/sinatra-hello-world.png)

[View solution](https://github.com/orfjackal/web-intro-project/commit/8254bc27671ac4a3236a381878bc3dcb8d3f70d5)


### Making file changes visible

Always when you change your application, you will need to stop and restart the web server. To stop the web server running in your terminal, press `Ctrl+C`. To start it again without having to type the command, press `Up` and then `Enter`. Try it a couple of times by changing the text that the server shows. If you get tired of doing that, Sinatra's FAQ contains instructions on [how to reload the application automatically][sinatra-reloading] when a file is changed.


## Your site on a web server

As a first step towards serving our web site through Sinatra, find out how Sinatra can [serve static files][sinatra-static] and use it to serve our existing HTML and CSS files as they are. Then visit <http://localhost:4567/pictures.html> in your web browser and make sure that your site looks the same as before.

![Your first web server is working](/screenshots/sinatra-static-files.png)

[View solution](https://github.com/orfjackal/web-intro-project/commit/542566800ca115323affd8e25f548e23e7c67008)


## Fixing the front page

You should make sure that people can visit your web site through its front page, instead of having to know one of its sub pages. Tell Sinatra to [redirect][sinatra-redirect] the path `/` to `/my-page.html` and then visit <http://localhost:4567/> to see if it will show your web site's front page.

![Front page redirect works](/screenshots/sinatra-redirect.png)

[View solution](https://github.com/orfjackal/web-intro-project/commit/60c9efb2b60ef39ee901edbf293cb76d23c79f41)


## Generating a page dynamically

The idea of a templating system is to dynamically generate the HTML that will be shown to the user. To make sure that we understand how to generate pages dynamically with Sinatra, add the following code for a [parameterized route][sinatra-routes] to your application and then visit <http://localhost:4567/my-page> (Note: no `.html` at the end of the URL because else Sinatra would just serve the static file directly)

TODO: introduce parameterized routes and IO.read separately

```ruby
get '/:page' do |page|
  "stuff before" + IO.read('public/' + page + '.html') + "stuff after"
end
```

![Dynamically generated HTML page](/screenshots/templating-dynamic-page.png)

Check the source code of the page in your web browser to make sure that it shows the HTML of `my-page.html` but with some stuff added to the top and bottom of the HTML file.

![Dynamically generated HTML page](/screenshots/templating-dynamic-page-source.png)

[View solution](https://github.com/orfjackal/web-intro-project/commit/a9b6ff30ae8ee5fc247a0609542489878ee327dd)


## Bare bones templating system

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


## Proper templating system

TODO: consider using .html.erb extension for the templates, will need to change the route as well

If we would create our own templating system for every project, we would get no real work done. Thankfully other programmers have solved the templating problem already many times in the past, so we can avoid reinventing the weel and just reuse the code that they have written.

[ERB][erb] is a templating system which comes with Ruby's standard library, so it's easy to get started with and also [Sinatra supports it][sinatra-templates]. Change your application to use ERB instead of the code we wrote earlier.

* Combine `layout-top.html` and `layout-bottom.html` into a file `layout.erb`, and add the `<%= yield %>` code where the page content should be inserted
* Rename the extension of the `.html` pages in `views` directory to `.erb`
* Change your route to call the `erb` method:

```ruby
get '/:page.html' do |page|
  erb page
end
```

*Note: The above code has a security vulnerability, but we'll address that in a [later chapter](/security/).*

After restarting your application, check that all the pages still look the same before. This time it's being powered by the ERB templating system, so later we'll be able to take advantage of some of its useful features.

[View solution](https://github.com/orfjackal/web-intro-project/commit/480bf7c0cea7183fb26a42c9ac6f9f186b9d0e3b)


## Front page at the root

TODO: consider moving this to the end of this chapter, together with generating the navigation menu

Before moving on to the next topic, let's clean up our code a little bit. The front page of a site should be at the root path (`/`) and there is a convention to call it the *index* page. Rename `my-page.erb` to `index.erb` and change your routes to render it at the root instead of doing a page redirect:

```ruby
get '/' do
  erb :index
end
```

You will notice that your navigation links won't work anymore because we renamed the page but did not update the navigation. How to make links to the index page when its file name is not shown on the URL? We could use an absolute URL `<a href="http://localhost:4567/">`, but that wouldn't work when we move our site to another domain.

The recommended solution is to URLs relative to the root of the site, i.e. `<a href="/">` and `<a href="/pictures.html">`. Relative URLs which don't start with `/` are relative to the current directory; a navigation menu which uses them would not work if the site is organized into subdirectories. Go change all the URLs in your templates to be relative to the root.

![Front page at the root](/screenshots/templating-index.png)

[View solution](https://github.com/orfjackal/web-intro-project/commit/9fb70ea83b984888bb39e66e0e8fd22e227a6de6)


## Parameterize the template with a list of pictures

The main benefit of a templating system is that we can parameterize it with dynamically generated data. Let's take advantage of that to avoid repeating ourselves in the pictures page.

First create a route for `/pictures.html` which stores the URLs of all your pictures in a list and gives it as a parameter the template. Note that this route must be before the `/:page.html` route, because Sinatra will use the first route that matches the requested path.

```ruby
get '/pictures.html' do
  picture_urls = [
    "http://www.publicdomainpictures.net/pictures/50000/t2/cat-looking-up.jpg",
    "http://www.publicdomainpictures.net/pictures/30000/t2/cat-in-the-city-5.jpg",
    "http://www.publicdomainpictures.net/pictures/30000/t2/annoyed-cat.jpg",
    ...
  ]
  erb :pictures, :locals => {:picture_urls => picture_urls}
end
```

Then change `pictures.erb` to use the *local variable* `picture_urls` which the above route gave the template. This is a *for loop* which repeats the contained HTML for every element in the `picture_urls` list, so that on every iteration the `url` variable has a different element of the list.

```html
<% for url in picture_urls %>
<img class="album-photo" src="<%= url %>">
<% end %>
```

Now check that the pictures page still looks the same as before. Also try adding a couple more pictures to make sure that the template works how it should.

[View solution](https://github.com/orfjackal/web-intro-project/commit/aa454fcbe2d49f9bd5c2c06631adddf651bfce58)


## Show all pictures from a directory

There is still some work involved in adding the picture URLs by hand to the list in the route. We can simplify that by making our application generate the list automatically based on what pictures there are in a directory.

Create a directory for the pictures under the `public` directory and save there all the pictures in your list.

To find out how to do something in a particular programming language, google for "[name of the language] [what you want to do]", which in this case would be "ruby list files in directory". But even if you find some snippet of code on the Internet, you should understand what it does, for example by checking the official reference documentation, before using it. Or if you know the language and its standard library already a bit, you can probably start from the reference documentation and find what you need.

In this case you can get a list of files in the `public/pictures` directory using the following code. Have a look at the documentation for the methods [glob][ruby-glob], [map][ruby-map] and [sub][ruby-sub] to understand what each of them does individually, and then try to understand this code as a whole.

```ruby
picture_urls = Dir.glob('public/pictures/**').map { |path| path.sub('public', '') }
```

Check that the pictures page still works. Try adding a couple more pictures - much easier now, isn't it?

[View solution](https://github.com/orfjackal/web-intro-project/commit/4d21a58ecc1f5d00b9115188bac32d8c3f4ab92b)


## Parameterizing the page title

Earlier when we started using templates, our web site didn't anymore have unique titles for every page. Now that we know how to parameterize templates, we can get those page titles back.

Change the `<title>` in your `layout.erb` to get the title from a local variable `title`:

```html
<title><%= title %></title>
```

Then change every route to pass a title as parameter to the template, for example:

```ruby
erb :pictures, :locals => {:picture_urls => picture_urls, :title => "Lovely Pictures"}
```

![Parameterized page title](/screenshots/templating-parameterized-title.png)

[View solution](https://github.com/orfjackal/web-intro-project/commit/5edefe49b3a8d9ec92fa5627f87024afaed9c6fd)


## Sitemap for titles

Add one more page to your web site, for example a page about your life story. It will contain just plain HTML, so it will use the `/:page.html` route. Now we have a problem - multiple pages can be rendered by the `/:page.html` route, so how can it decide what to make the page title?

One solution is to store a list of all the pages and their titles in a [hash][ruby-hash] (also known as a dictionary or map). Add the following code to your application, outside all the routes, after which you can refer its contents using the keys of the hash: `PAGES[:pictures]`

```ruby
PAGES = {
  :index => "I'm Ruby",
  :pictures => "Lovely Pictures",
  :story => "My Story",
}
```

With this it will be easy to parameterize also the title of pages which don't have custom routes.

```ruby
get '/:page.html' do |page|
  erb page.to_sym, :locals => {:title => PAGES[page.to_sym]}
end
```

[View solution](https://github.com/orfjackal/web-intro-project/commit/f63a6cc1c01afe211d9f755ba0c86f69a5ae1e09)


## Sitemap for navigation

Now that we have a sitemap of all the pages, we can use it to also generate our navigation menu, so that we can avoid having to update the layout template whenever we add more pages. Pass in the `PAGES` hash to the templates as a local variable `pages` and update your navigation menu to be generated dynamically like this:

```html
<nav class="navigation">
    <ul>
<% for page, title in pages %>
        <li><a href="<%= (page == :index ? '/' : "/#{page}.html") %>"><%= title %></a></li>
<% end %>
    </ul>
</nav>
```

Check that the navigation menu and all of its links still work.

[View solution](https://github.com/orfjackal/web-intro-project/commit/bd1811425811bcd45db76c8d57482b7f47cb4851)


## Helper method for rendering templates

Have a look at your code. All the lines which render a template with `erb` have quite much repetition, because every template needs to be parameterized with `title` and `pages`. To avoid this duplicated code, which violates the DRY principle, we can create a helper method which takes care of all the common code.

```ruby
def render_page(page, locals={})
  locals = locals.merge({:title => PAGES[page], :pages => PAGES})
  erb page, :locals => locals
end
```

With this helper method the rest of the code becomes much simpler.

```ruby
get '/pictures.html' do
  picture_urls = Dir.glob('public/pictures/**').map { |path| path.sub('public', '') }
  render_page :pictures, {:picture_urls => picture_urls}
end

get '/:page.html' do |page|
  render_page page.to_sym
end
```

Most of the time when you notice some repetition in your code, you can extract the common code into its own method, which makes the code easier to understand and change.

[View solution](https://github.com/orfjackal/web-intro-project/commit/79f3e1113d08a209db91570f1362ba85ccbfbbc9)


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
[ruby-glob]: http://docs.ruby-lang.org/en/2.2.0/Dir.html#method-c-glob
[ruby-map]: http://docs.ruby-lang.org/en/2.2.0/Enumerable.html#method-i-map
[ruby-sub]: http://docs.ruby-lang.org/en/2.2.0/String.html#method-i-sub
[ruby-hash]: http://docs.ruby-lang.org/en/2.2.0/Hash.html
