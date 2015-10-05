---
layout: chapter
title: Templating
permalink: /templating/
next: /databases/
---

In this chapter you will learn how to create a web server with Ruby, and how to use templates and a bit of programming to avoid repeating yourself.


## Pictures page

Last time we created a navigation menu with a link to the `pictures.html` page, but that page doesn't yet exist. Create a copy of `my-page.html`, name it `pictures.html`, and change its title and heading. Find about a dozen images which you like from [Public Domain Pictures](http://www.publicdomainpictures.net/) and add their thumbnails as images on your `pictures.html` using the [`<img>`][html-img] tag.

```html
<img class="album-photo" src="http://www.publicdomainpictures.net/pictures/50000/t2/cat-looking-up.jpg">
<img class="album-photo" src="http://www.publicdomainpictures.net/pictures/30000/t2/cat-in-the-city-5.jpg">
<img class="album-photo" src="http://www.publicdomainpictures.net/pictures/30000/t2/annoyed-cat.jpg">
...
```

As shown above, give the images a unique class and use CSS to make them look like a photo album.

![Photo album](photo-album.png)

[View solution](https://github.com/orfjackal/web-intro-project/commit/261d715d864c2252cc1a29f6523509cccc16e861)


## Don't Repeat Yourself

In programming we have a principle called [Don't Repeat Yourself (DRY)][dry] which says that *"every piece of knowledge must have a single, unambiguous, authoritative representation within a system."* If we follow that principle, our code will be easier to understand and modify.

Our little web site is already breaking the DRY principle. Both of our pages are repeating the HTML for the layout and navigation; adding another page would require changing the navigation on every page. Additionally the pictures page has lots of similar `<img>` tags with just the URL changing. To solve this problem, we will introduce *templating*, so that all the common HTML will be in one place.


## Running a web server

This far we have created our site with just static HTML pages, but in order to use templates, we must create a web server which will generate the HTML that will be shown to the user.

We will use the [Ruby][ruby] programming language and the [Sinatra][sinatra] web framework to create the web server. Follow the [installation guide](/install/) to install Ruby and Sinatra if you haven't yet done so.

Create a file called `app.rb` and put the following code in it.

```ruby
require 'sinatra'

get '/hi' do
  "Hello World!"
end
```

Then run the command `ruby app.rb` in your terminal. It should start a web server on your own machine in port 4567. To view it, visit <http://localhost:4567/hi> in your web browser. Finally stop the web server by pressing Ctrl+C.

![Your first web server is working](sinatra-hello-world.png)

[View solution](https://github.com/orfjackal/web-intro-project/commit/99e2f0061a1875c33dca1b0881f9c0d559b35fc4)


### Making file changes visible

Always when you change your application, you will need to stop and restart the web server. To stop the web server running in your terminal, press `Ctrl+C`. To start it again without having to type the command, press `Up` and then `Enter`. Try it a couple of times by changing the text that the server shows. If you get tired of doing that, Sinatra's FAQ contains instructions on [how to reload the application automatically][sinatra-reloading] when a file is changed.


## Static site on a web server

As a first step towards serving our web site through Sinatra, we can use Sinatra's ability to [serve static files][sinatra-static] to serve our existing HTML and CSS files unmodified.

In the same folder as `app.rb`, create a new folder called `public` and move your HTML and CSS files and pictures there. Then start your web server with `ruby app.rb`, visit <http://localhost:4567/pictures.html> in your web browser and make sure that your site looks the same as before.

* TODO: avoid åöä in path

![Your first web server is working](sinatra-static-site.png)

[View solution](https://github.com/orfjackal/web-intro-project/commit/a96758269aea70c81e8e245320ca5f0cda529c2b)


## Fixing the front page

You should make sure that people can visit your web site through its base URL instead of having to know one of its sub pages.

First try going to <http://localhost:4567/> on your current site. It should show an error message "Sinatra doesn’t know this ditty." Let's fix that.

Change your `app.rb` to [redirect][sinatra-redirect] the path `/` to `/my-page.html`. Restart the web server and check that when you visit <http://localhost:4567/>, it will send your web browser to the <http://localhost:4567/my-page.html> address.

![Front page redirect works](front-page-redirect.png)

[View solution](https://github.com/orfjackal/web-intro-project/commit/246ac703f710ae6538c48787bae937c227e9452d)


## Generating pages dynamically

The idea of a templating system is to dynamically generate the HTML that will be shown to the user. To make sure that we understand how to generate pages dynamically with Sinatra, add the following two routes to your application.

```ruby
get '/foo' do
  IO.read('public/my-page.html')
end

get '/:page' do
  "You're on page " + params['page']
end
```

When you visit <http://localhost:4567/foo>, you should see exactly the same content as on <http://localhost:4567/my-page.html>. The [`IO.read`][ruby-read] method reads a file and gives you its contents.

When you visit <http://localhost:4567/bar>, you should see the text "You're on page bar". On <http://localhost:4567/gazonk> you should see "You're on page gazonk". This is an example of a [parameterized route][sinatra-routes] which can serve multiple pages. Note that the `/:page` route must be after the `/foo` route, because Sinatra will use the first route that matches the path in the HTTP request.

[View solution](https://github.com/orfjackal/web-intro-project/commit/d94fc393487def0e1e1421166b8b3b749774c418)


## Bare bones templating system

Now that we have a proof-of-concept for dynamically generating HTML and for reading existing HTML files, we can use that to create a very basic templating system.

Create a `views` folder and move all your HTML files from the `public` folder there (`views` is Sinatra's default folder for templates). Don't move the CSS files. Create empty files `layout-top.html` and `layout-bottom.html` in the `views` folder.

Add the following route to your application. Remove the old `/foo` and `/:page` routes.  Make sure that your application still looks the same as before. The difference is that now the pages are no more served as static files, because they are no more in the `public` folder, but they are generated dynamically using this code.

```ruby
get '/:page.html' do
  IO.read('views/layout-top.html') +
  IO.read('views/' + params['page'] + '.html') +
  IO.read('views/layout-bottom.html')
end
```

Now you can move the top and bottom parts of the code from your individual HTML pages to the `layout-top.html` and `layout-bottom.html` files. Restart your application and check that it still looks the same as before.

[View solution](https://github.com/orfjackal/web-intro-project/commit/595a09999b3ce0165792712929937c60fc179542)

Now you have managed to apply the DRY principle and in the future it will be enough to change just one place when you change the layout or add pages to the navigation menu. You also understand the basic idea of how templating works. Next let's use a proper templating system.


## Proper templating system

If we would create our own templating system for every project, we would get no real work done. Thankfully other programmers have solved the templating problem already many times in the past, so we can avoid reinventing the weel and just reuse the code that they have written.

[ERB][erb] is a templating system which comes with Ruby's standard library, so it's easy to get started with and also [Sinatra supports it][sinatra-templates]. Change your application to use ERB instead of the code we wrote earlier.

* Combine `layout-top.html` and `layout-bottom.html` into a file `layout.erb`, and add the `<%= yield %>` code where the page content should be inserted
* Rename the file extension of the `.html` pages in the `views` folder to `.erb`
* Change the `/:page.html` route to call the `erb` method:

```ruby
get '/:page.html' do
  erb params['page'].to_sym
end
```

*Note: The above code has a security vulnerability, but we'll address that in a [later chapter](/security/).*

After restarting your application, check that all the pages still look the same before. This time it's being powered by the ERB templating system, so later we'll be able to take advantage of some of its useful features.

[View solution](https://github.com/orfjackal/web-intro-project/commit/1d27220c53c2633b79bd32ddbc8975c916b1e3c6)


## Front page at the root

Before moving on to the next topic, let's clean up our code a little bit. The front page of a site should be at the root path (`/`) and there is a convention to call it the *index* page.

Rename `my-page.erb` to `index.erb` and change your routes to render it at the root instead of doing a page redirect. This is easy to do now that we have templates.

```ruby
get '/' do
  erb :index
end
```

Also change the link in your navigation menu from `<a href="my-page.html">` to `<a href="/">`. The `/` path refers to the base URL of your web site.

In the layout it's a good practice to have all the links prefixed with `/`, because that way they will work the same way regardless of the page where the user is currently. You can read more about this in [Absolute vs. Relative Paths/Links][absolute-vs-relative-paths].

![Front page at the root](front-page-root.png)

[View solution](https://github.com/orfjackal/web-intro-project/commit/b31e68c559dd84094554c07f98332881d31528a3)


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

Then change `pictures.erb` to use the *local variable* `picture_urls` which the above route gave the template. This is a *for loop* which repeats the contained HTML for every element in the `picture_urls` list, so that on every iteration the `url` variable refers to a different element of the list.

```html
<% for url in picture_urls %>
<img class="album-photo" src="<%= url %>">
<% end %>
```

Now check that the pictures page still looks the same as before. Also try adding a couple more pictures to make sure that the template works how it should.

[View solution](https://github.com/orfjackal/web-intro-project/commit/e27600012a588173841e88d9802405b58bcb0707)


## Show all pictures from a folder

There is still some work involved in adding the picture URLs by hand to the list in the route. We can simplify that by making our application generate the list automatically based on what pictures there are in a folder.

Create a `pictures` folder under the `public` folder and save there all the pictures in your list.

To find out how to do something in a particular programming language, you can google for "[name of the language] [what you want to do]", which in this case would be "ruby list files in a folder". But even if you find some snippet of code on the Internet, you should understand what it does, for example by checking the official reference documentation, before using it. Or if you know the language and its standard library already a bit, you can probably start from the reference documentation and find there what you need.

In this case you can get a list of files in the `public/pictures` folder using the following code. Have a look at the documentation for the methods [glob][ruby-glob], [map][ruby-map] and [sub][ruby-sub] to understand what each of them does individually, and then try to understand this code as a whole.

```ruby
picture_urls = Dir.glob('public/pictures/**').map { |path| path.sub('public', '') }
```

Check that the pictures page still works. Try adding a couple more pictures - much easier now, isn't it?

[View solution](https://github.com/orfjackal/web-intro-project/commit/49556157d058a5ff6b7c58c167602732a9f7ed37)


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

Visit every page on your site to make sure that they still work and that they have the correct page title.

![Parameterized page title](parameterized-title.png)

[View solution](https://github.com/orfjackal/web-intro-project/commit/70d2d4cc9440147ca718e3ce3a64d0d16e181cb0)


## Sitemap for titles

Add one more page to your web site, for example a page about your life story. It will contain just plain HTML, so it will use the `/:page.html` route. Now we have a problem - multiple pages can be rendered by the `/:page.html` route, so how can it decide what to make the page title? One solution is to store a list of all the pages and their titles in a [*hash*][ruby-hash] (also known as a *dictionary* or *map*).

Add the following code to your application, outside all the routes, after which you can refer its contents using the *keys* of the hash: `PAGES[:pictures]`

```ruby
PAGES = {
  :index => "I'm Ruby",
  :pictures => "Lovely Pictures",
  :story => "My Story",
}
```

With this it will be easy to parameterize also the title of pages which don't have custom routes.

```ruby
get '/:page.html' do
  page = params['page'].to_sym
  erb page, :locals => {:title => PAGES[page]}
end
```

Change all the routes to use the title from the `PAGES` hash. Test that all your pages still work.

[View solution](https://github.com/orfjackal/web-intro-project/commit/ee2a9760fbfa3dac04780c2294430a3bcebf3214)


## Sitemap for navigation

Now that we have a sitemap of all the pages, we can use it to also generate our navigation menu, so that we can avoid having to update the layout template whenever we add more pages.

Pass in the `PAGES` hash to the templates as a local variable `pages` and update your navigation menu to be generated dynamically like this:

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

TODO: screenshot, the navigation menu and page titles are now in sync

[View solution](https://github.com/orfjackal/web-intro-project/commit/b81b3ce39bf919cd7b88c890f45bf212cac637c0)


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
get '/' do
  render_page :index
end

get '/pictures.html' do
  picture_urls = Dir.glob('public/pictures/**').map { |path| path.sub('public', '') }
  render_page :pictures, {:picture_urls => picture_urls}
end

get '/:page.html' do
  render_page params['page'].to_sym
end
```

Most of the time when you notice some repetition in your code, you can extract the common code into its own method, which makes the code easier to understand and change.

[View solution](https://github.com/orfjackal/web-intro-project/commit/dc60a1ea4e5e01ff83acd13d6378ba80802779a6)


[html-img]: https://developer.mozilla.org/en/docs/Web/HTML/Element/img
[dry]: http://programmer.97things.oreilly.com/wiki/index.php/Don't_Repeat_Yourself
[ruby]: https://www.ruby-lang.org/
[sinatra]: http://www.sinatrarb.com/
[sinatra-reloading]: http://www.sinatrarb.com/faq.html#reloading
[sinatra-static]: http://www.sinatrarb.com/intro.html#Static%20Files
[sinatra-redirect]: http://www.sinatrarb.com/intro.html#Browser%20Redirect
[sinatra-routes]: http://www.sinatrarb.com/intro.html#Routes
[sinatra-templates]: http://www.sinatrarb.com/intro.html#Views%20/%20Templates
[absolute-vs-relative-paths]: http://www.coffeecup.com/help/articles/absolute-vs-relative-pathslinks/
[erb]: http://www.stuartellis.eu/articles/erb/
[ruby-read]: http://docs.ruby-lang.org/en/2.2.0/IO.html#method-c-read
[ruby-glob]: http://docs.ruby-lang.org/en/2.2.0/Dir.html#method-c-glob
[ruby-map]: http://docs.ruby-lang.org/en/2.2.0/Enumerable.html#method-i-map
[ruby-sub]: http://docs.ruby-lang.org/en/2.2.0/String.html#method-i-sub
[ruby-hash]: http://docs.ruby-lang.org/en/2.2.0/Hash.html
