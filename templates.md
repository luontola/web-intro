---
layout: chapter
title: Templates
permalink: /templates/
next: /databases/
---

In this chapter you will learn how to create a web server with Ruby, how to use templates and a bit of programming to avoid repeating yourself. You will also get a taste of creating dynamic content, i.e. the page content is not always the same when somebody visits it.


## Pictures page

Last time we created a navigation menu with a link to the `pictures.html` page, but that page doesn't yet exist. Let's create it now.

Make a copy of `about.html`, name it `pictures.html`, and change its title and heading. Find about a dozen free images from [Public Domain Pictures](http://www.publicdomainpictures.net/) or [some similar site](https://commons.wikimedia.org/wiki/Commons:Free_media_resources/Photography). Add the images to your `pictures.html` page using the [`<img>`][html-img] tag.

```html
<img class="album-photo" src="http://www.publicdomainpictures.net/pictures/50000/t2/cat-looking-up.jpg">
<img class="album-photo" src="http://www.publicdomainpictures.net/pictures/30000/t2/cat-in-the-city-5.jpg">
<img class="album-photo" src="http://www.publicdomainpictures.net/pictures/30000/t2/annoyed-cat.jpg">
...
```

Use CSS to make the images look like a photo album. In case the images are of different size, you can set their [height][css-height]; the web browser will adjust their width automatically to maintain the image's aspect ratio.

The following example uses the CSS properties [border][css-border], [background-color][css-background-color], [margin][css-margin], [padding][css-padding] and [box-shadow][css-box-shadow].

![Pictures page with some pictures](pictures-page.png)

<aside class="solution">
    <a class="file" href="https://github.com/orfjackal/web-intro-project/blob/be1bfe190f19c5ed201fac3c30bb311aad005032/pictures.html">pictures.html</a>
    <a class="file" href="https://github.com/orfjackal/web-intro-project/blob/be1bfe190f19c5ed201fac3c30bb311aad005032/style.css">style.css</a>
    <a class="diff" href="https://github.com/orfjackal/web-intro-project/commit/be1bfe190f19c5ed201fac3c30bb311aad005032">View changes</a>
</aside>


## Don't Repeat Yourself

In programming we have a principle called [Don't Repeat Yourself (DRY)][dry] which says that *"every piece of knowledge must have a single, unambiguous, authoritative representation within a system."* If we follow that principle, our code will be easier to understand and modify.

Our little web site is already breaking the DRY principle. Both of our pages are repeating the HTML for the layout and navigation. If we would add another page, we would have to change the navigation menu in many different HTML files. Additionally the pictures page has lots of similar `<img>` tags with just the URL changing. To solve this problem, we will introduce *templates*, so that all HTML code will be written once and only once.


## Run a web server

This far we have created our site with just static HTML pages, but in order to use templates we must create a *web server* which will dynamically generate HTML pages. Your web browser will communicate with the web server and show the HTML pages which the web server generates.

We will use the [Ruby][ruby] programming language and the [Sinatra][sinatra] web framework to create the web server. Follow the [installation guide](/install/) to install Ruby and Sinatra if you haven't yet done so.

Create a file called `app.rb` and put the following code in it. This minimal web application contains a single *route* for the *path* `/hi` which will just generate a page with the text *Hello World!*

```ruby
require 'sinatra'
require 'sinatra/reloader'

get '/hi' do
  "Hello World!"
end
```

Then run the command `ruby app.rb` in your terminal. It should start a web server on your own machine in port 4567. To view it, visit <http://localhost:4567/hi> in your web browser. It should show the *Hello World!* text.

![Your first web server is working](run-a-web-server.png)

<aside class="solution">
    <a class="file" href="https://github.com/orfjackal/web-intro-project/blob/d664151308553fc3aa9cee8f65c46f640f7a4513/app.rb">app.rb</a>
    <a class="diff" href="https://github.com/orfjackal/web-intro-project/commit/d664151308553fc3aa9cee8f65c46f640f7a4513">View changes</a>
</aside>

When programming, it is often necessary to stop and restart the web server to see your changes in action.

To stop the web server by pressing `Ctrl+C` in your terminal. Try refreshing the page in your web browser. It should now show an error message because there is no more a web server to connect to.

To start the server again, without having to retype the command, press `Up` to see your previous command and then press `Enter`.


## Static site on a web server

As a first step towards a dynamic web site, we can use Sinatra's ability to [serve static files][sinatra-static] to serve our existing HTML and CSS files unmodified.

Inside the same folder as `app.rb`, create a new folder called `public` and move your HTML and CSS files and pictures there. Then start your web server with `ruby app.rb`, visit <http://localhost:4567/pictures.html> in your web browser. Your site should look the same as before, but this time it's being served to you by a web server the same way as all web sites on the Internet.

*Note: If the full path of your project folder contains non-English letters, such as åäö, Sinatra will fail to serve your static files. In that case move your project folder for example to `C:\project`*

![Your web site on a web server](static-site-on-a-web-server.png)

<aside class="solution">
    <a class="file" href="https://github.com/orfjackal/web-intro-project/blob/c9ef371e37cad51451460a5bafe9abc4b0bda7a6/public/about.html">public/about.html</a>
    <a class="file" href="https://github.com/orfjackal/web-intro-project/blob/c9ef371e37cad51451460a5bafe9abc4b0bda7a6/public/pictures.html">public/pictures.html</a>
    <a class="file" href="https://github.com/orfjackal/web-intro-project/blob/c9ef371e37cad51451460a5bafe9abc4b0bda7a6/public/style.css">public/style.css</a>
    <a class="diff" href="https://github.com/orfjackal/web-intro-project/commit/c9ef371e37cad51451460a5bafe9abc4b0bda7a6">View changes</a>
</aside>


## Fix the front page

You should make sure that people can visit your web site through its base URL, the same way as all web sites on the Internet, instead of having to know one of its sub pages.

First try going to <http://localhost:4567/> on your current site. It should show an error message "Sinatra doesn’t know this ditty." Let's fix that.

Change `app.rb` to [redirect][sinatra-redirect] the path `/` to `/about.html` by adding the following route.

```ruby
get '/' do
  redirect to('/about.html')
end
```

Check that when you visit <http://localhost:4567/>, your web browser immediately goes to the <http://localhost:4567/about.html> address.

<aside class="solution">
    <a class="file" href="https://github.com/orfjackal/web-intro-project/blob/30ce25ea147fe0306e5d1871be260220b86fd1b5/app.rb">app.rb</a>
    <a class="diff" href="https://github.com/orfjackal/web-intro-project/commit/30ce25ea147fe0306e5d1871be260220b86fd1b5">View changes</a>
</aside>


## Templates

The idea of *templates* is to sprinkle your HTML code with some placeholder marks where you can easily insert some dynamic content. We will use the [ERB templating library][erb] for that.

Create a `views` folder inside your project folder (next to the `public` folder). This is the folder where Sinatra expects to find all your template files.

Create a `layout.erb` file inside the `views` folder. Copy from `public/about.html` to `views/layout.erb` all code except the page content (i.e. the stuff between `<section class="content">` and `</section>`). Replace the page content with just `<%= yield %>`

Copy `public/about.html` to `views/about.erb` and remove everything except the page content (i.e. the opposite of what you did in `views/layout.erb`). Copy `public/pictures.html` to `views/pictures.erb` and do the same thing. Remove the old `public/about.html` and `public/pictures.html`.

Add the following routes to `app.rb` to [render][sinatra-templates] the above mentioned ERB templates.

```ruby
get '/about.html' do
  erb :about
end

get '/pictures.html' do
  erb :pictures
end
```

Check that all the pages still look the same as before in your web browser. The difference is that now all the common HTML for the layout and navigation is in a single place, so changing it and adding new pages will be easier. Templates also make it possible to have dynamic content, as we will see next.

<aside class="solution">
    <a class="file" href="https://github.com/orfjackal/web-intro-project/blob/9de139db60f73333bf71c6e02b42db1be250692b/app.rb">app.rb</a>
    <a class="file" href="https://github.com/orfjackal/web-intro-project/blob/9de139db60f73333bf71c6e02b42db1be250692b/views/about.erb">views/about.erb</a>
    <a class="file" href="https://github.com/orfjackal/web-intro-project/blob/9de139db60f73333bf71c6e02b42db1be250692b/views/layout.erb">views/layout.erb</a>
    <a class="file" href="https://github.com/orfjackal/web-intro-project/blob/9de139db60f73333bf71c6e02b42db1be250692b/views/pictures.erb">views/pictures.erb</a>
    <a class="diff" href="https://github.com/orfjackal/web-intro-project/commit/9de139db60f73333bf71c6e02b42db1be250692b">View changes</a>
</aside>


## Lottery

You might have seen on the Internet many sites which say "Click here to find out your lucky number". Here is how you can make your own using a couple of lines of code and templates.

In `app.rb`, change the `/about.html` route to contain the following code.

```ruby
get '/about.html' do
  backstreet_boys = ["A.J.", "Howie", "Nick", "Kevin", "Brian"]
  @who_i_marry = backstreet_boys.sample
  erb :about
end
```

The above code first creates a list of the names of all Backstreet Boys and stores it in the `backstreet_boys` *variable.* The next lines calls the `sample` *method* on that list, which will randomly pick one of the items in the list, and then the code stores it in the `@who_i_marry` variable. Because the variable name starts with `@`, also the template can access that variable.

In `views/about.erb`, add the following code to show the value of `@who_i_marry` on the page.

```erb
<p>I like Backstreet Boys a lot. I'm going to marry <%= @who_i_marry %>!</p>
```

Try reloading the about page in your web browser multiple times. The name should change randomly every time that the page is reloaded.

![Lottery result is shown](lottery.png)

<aside class="solution">
    <a class="file" href="https://github.com/orfjackal/web-intro-project/blob/461eafea8a9665df71852c0ef612988733e5f4c7/app.rb">app.rb</a>
    <a class="file" href="https://github.com/orfjackal/web-intro-project/blob/461eafea8a9665df71852c0ef612988733e5f4c7/views/about.erb">views/about.erb</a>
    <a class="diff" href="https://github.com/orfjackal/web-intro-project/commit/461eafea8a9665df71852c0ef612988733e5f4c7">View changes</a>
</aside>


## Front page at the root

Normally the front page of every web site is at the root path (`/`), but on our web site it's instead at the `/about.html` path. We can solve that by rendering the about template at the root path instead of doing a redirect.

In `app.rb`, remove the old `/` route and change the `/about.html` route to be the new `/` route.

In `views/layout.erb`, change the link to the about page from `<a href="about.html">` to `<a href="/">`.

Check that when you visit <http://localhost:4567/>, your web browser stays at the <http://localhost:4567/> address and shows the front page.

![Front page at the root](front-page-at-the-root.png)

<aside class="solution">
    <a class="file" href="https://github.com/orfjackal/web-intro-project/blob/4314be08624f955c28bb0efcc35ac63f772cc401/app.rb">app.rb</a>
    <a class="file" href="https://github.com/orfjackal/web-intro-project/blob/4314be08624f955c28bb0efcc35ac63f772cc401/views/layout.erb">views/layout.erb</a>
    <a class="diff" href="https://github.com/orfjackal/web-intro-project/commit/4314be08624f955c28bb0efcc35ac63f772cc401">View changes</a>
</aside>


## Album photo template

In addition to inserting single items to a template, as we did a moment ago, we can insert a list of items and repeat a piece of the template for each of them. To demonstrate that we will remove the repetition of multiple image tags on the pictures page. Using templates might seem overkill for simple image tags, but imagine that each image will also have a description, like button and a link to the full-sized picture, in which case avoiding the repetition becomes very important.

Change your `/pictures.html` route to store a list of the picture URLs in a variable.

```ruby
get '/pictures.html' do
  @picture_urls = [
    "http://www.publicdomainpictures.net/pictures/50000/t2/cat-looking-up.jpg",
    "http://www.publicdomainpictures.net/pictures/30000/t2/cat-in-the-city-5.jpg",
    "http://www.publicdomainpictures.net/pictures/30000/t2/annoyed-cat.jpg",
  ]
  erb :pictures
end
```

In `views/pictures.erb`, use a *for loop* to repeat the image tag for each url in the list.

```erb
<% for url in @picture_urls %>
<img class="album-photo" src="<%= url %>">
<% end %>
```

Now check that the pictures page still looks the same as before. Also try adding a couple more pictures to make sure that the template works how it should.

<aside class="solution">
    <a class="file" href="https://github.com/orfjackal/web-intro-project/blob/0d5309e7e8b9116e8ab5aa115382beacbde86462/app.rb">app.rb</a>
    <a class="file" href="https://github.com/orfjackal/web-intro-project/blob/0d5309e7e8b9116e8ab5aa115382beacbde86462/views/pictures.erb">views/pictures.erb</a>
    <a class="diff" href="https://github.com/orfjackal/web-intro-project/commit/0d5309e7e8b9116e8ab5aa115382beacbde86462">View changes</a>
</aside>


## Show all pictures from a folder

There is still some work involved in adding the picture URLs by hand to the list in the route. We can simplify that by making our application generate the list automatically based on what pictures there are in a folder.

Create a `pictures` folder under the `public` folder and save there all the pictures in your list.

Change your `/pictures.html` route to generate a list of the picture URLs based on the contents of the `public/pictures` folder.

```ruby
get '/pictures.html' do
  @picture_urls = Dir.glob('public/pictures/**').map { |path| path.sub('public', '') }
  erb :pictures
end
```

Check that the pictures page still works. Try adding a couple more pictures - much easier now, isn't it?

<aside class="solution">
    <a class="file" href="https://github.com/orfjackal/web-intro-project/blob/0868fa9f8d5d40d94fbb41b549f90c81ef71abd0/app.rb">app.rb</a>
    <a class="file" href="https://github.com/orfjackal/web-intro-project/blob/0868fa9f8d5d40d94fbb41b549f90c81ef71abd0/public/pictures/annoyed-cat.jpg">public/pictures/annoyed-cat.jpg</a> and other pictures
    <a class="diff" href="https://github.com/orfjackal/web-intro-project/commit/0868fa9f8d5d40d94fbb41b549f90c81ef71abd0">View changes</a>
</aside>


### How does this code work?

The above code combines multiple operations to achieve the desired result. First [glob][ruby-glob] finds all files under the `public/pictures` folder and returns a list of their paths. Each of those paths is like `public/pictures/something.jpg`, but on the pictures page we need them to be like `/pictures/something.jpg`. That's why we use [map][ruby-map] to go through each path in the list and remove the `public` prefix from the paths using [sub][ruby-sub].

For the purposes of this tutorial, it's not necessary to fully understand how the code works. But to get better at programming, one question to ask is that do I really [understand every word and symbol][the-secret-of-fast-programming-stop-thinking] in the code? One way to learn more is to read the documentation. Another way is to debug your code by [printing things to the terminal][ruby-printing] when you run your application. For some programming languages, including Ruby, there are also tools such as a debugger and a REPL.

Rant over. The tutorial must go on. No time for deep learning yet. ;-)


## Unique titles for every page

Earlier when we started using templates, our web site didn't anymore have unique titles for every page. Now that we know how to use variables in templates, we can give each page its own title.

In `views/layout.erb`, get the page title from the `@title` variable.

```html
<title><%= @title %></title>
```

Add to each of your routes the code for setting the title.

```ruby
@title = "type the title here"
```

Visit every page on your site to make sure that they still work and that they have different page titles.

![Unique page titles](unique-titles-for-every-page.png)

<aside class="solution">
    <a class="file" href="https://github.com/orfjackal/web-intro-project/blob/887bfac0ebb3eff948aed1ce40bc7fda70026eec/app.rb">app.rb</a>
    <a class="file" href="https://github.com/orfjackal/web-intro-project/blob/887bfac0ebb3eff948aed1ce40bc7fda70026eec/views/layout.erb">views/layout.erb</a>
    <a class="diff" href="https://github.com/orfjackal/web-intro-project/commit/887bfac0ebb3eff948aed1ce40bc7fda70026eec">View changes</a>
</aside>


## Pages for individual pictures

It would be nice to be able to click a picture on the pictures page to see that picture in full size and also allow people to write comments for that picture. For that we will need to have separate pages for every picture. Because our application can have an unlimited number of pictures, it's not possible to create separate routes for every picture; we need one route to handle *all* the pictures.

In `views/pictures.erb`, make each of the pictures a link to another page. The target of the link will be the URL of the picture, but with the file extension changed to `.html`. The following code does it using [regular expressions][ruby-regexp].

```erb
<% for url in @picture_urls %>
<a href="<%= url.sub(/\.\w+$/, '.html') %>"><img class="album-photo" src="<%= url %>"></a>
<% end %>
```

In `app.rb`, add the following route which uses [named parameters][sinatra-routes]. The `:picture` will match anything in that position of the path and will make it accessible as `params['picture']`.

```ruby
get '/pictures/:picture.html' do
  "Here will be a page for " + params['picture']
end
```

Go to your pictures page and click some of the pictures there. Notice how the text shown on the page is related to page's address.

![Placeholder page for a picture](pages-for-individual-pictures.png)

<aside class="solution">
    <a class="file" href="https://github.com/orfjackal/web-intro-project/blob/5fb27c7dcb8fdba729838da8316316e63ca181f3/app.rb">app.rb</a>
    <a class="file" href="https://github.com/orfjackal/web-intro-project/blob/5fb27c7dcb8fdba729838da8316316e63ca181f3/views/pictures.erb">views/pictures.erb</a>
    <a class="diff" href="https://github.com/orfjackal/web-intro-project/commit/5fb27c7dcb8fdba729838da8316316e63ca181f3">View changes</a>
</aside>


## Picture page template

Create `views/picture.erb` with the following content.

```erb
<img class="full-photo" src="<%= @picture_url %>">

<p><a href="/pictures.html">Return to album</a></p>
```

Make the route render that template.

```ruby
get '/pictures/:picture.html' do
  @title = "Picture"
  @picture_url = params['picture'] + '.jpg'
  erb :picture
end
```

Visit a few picture pages pictures and check that they show the picture. You'll notice that the CSS and some links don't work, so that needs to be solved next.

![Broken page for a picture](picture-page-template.png)

<aside class="solution">
    <a class="file" href="https://github.com/orfjackal/web-intro-project/blob/3658f94c8d259733501f3ff3140e8089cbf7c3be/app.rb">app.rb</a>
    <a class="file" href="https://github.com/orfjackal/web-intro-project/blob/3658f94c8d259733501f3ff3140e8089cbf7c3be/views/picture.erb">views/picture.erb</a>
    <a class="file" href="https://github.com/orfjackal/web-intro-project/blob/3658f94c8d259733501f3ff3140e8089cbf7c3be/public/style.css">public/style.css</a>
    <a class="diff" href="https://github.com/orfjackal/web-intro-project/commit/3658f94c8d259733501f3ff3140e8089cbf7c3be">View changes</a>
</aside>


## Fix relative links

On a picture page, if you click the "Pictures" or "Return to album" link, it will take your browser to <http://localhost:4567/pictures/pictures.html> instead of <http://localhost:4567/pictures.html>. Similarly it tries to load your CSS from <http://localhost:4567/pictures/style.css> instead of <http://localhost:4567/style.css> (you can see this with your web browser's [developer tools][browser-developer-tools]). However, the link to the front page still works.

The reason has to do with [absolute vs. relative paths][absolute-vs-relative-paths]. Absolute paths start with `http://` or similar. Other paths are relative to the page where the link appears. To make links point to the same target regardless of the page where it appears in your site, the link's path should start with `/` so that it will be relative to the site's root path instead of the current page's path.

Change all relative links in `views/layout.erb` and `views/picture.erb` to start with the `/` character. Check that the picture pages look right and all the links work.

![Working page for a picture](fix-relative-links.png)

<aside class="solution">
    <a class="file" href="https://github.com/orfjackal/web-intro-project/blob/3456d5ca1bce2dcd60619fdfa057e9eb33af7d17/views/layout.erb">views/layout.erb</a>
    <a class="file" href="https://github.com/orfjackal/web-intro-project/blob/3456d5ca1bce2dcd60619fdfa057e9eb33af7d17/views/picture.erb">views/picture.erb</a>
    <a class="diff" href="https://github.com/orfjackal/web-intro-project/commit/3456d5ca1bce2dcd60619fdfa057e9eb33af7d17">View changes</a>
</aside>


## Don't guess the picture's file extension

Not all pictures have the `.jpg` file extension. There are `.png`, `.gif` and other image types as well. But currently the `/pictures/:picture.html` route has been hard-coded to work with only `.jpg` images.

A solution for that is to check the contents of the `public/pictures` folder to find out the actual file extension. This can be implemented by taking advantage of the code that already lists all pictures in the folder, so we will extract that code into a new method for reuse.

In `app.rb`, add the methods `picture_urls` and `find_picture_url` as shown below, and change your routes to use them to set `@picture_urls` and `@picture_url`.

```ruby
get '/pictures.html' do
  @title = "Lovely Pictures"
  @picture_urls = picture_urls
  erb :pictures
end

get '/pictures/:picture.html' do
  @title = "Picture"
  @picture_url = find_picture_url(params['picture'])
  erb :picture
end

def picture_urls
  Dir.glob('public/pictures/**').map { |path| path.sub('public', '') }
end

def find_picture_url(basename)
  picture_urls.select { |path| File.basename(path, '.*') == basename }.first
end
```

<aside class="solution">
    <a class="file" href="https://github.com/orfjackal/web-intro-project/blob/fe8b2f27c46de1efcbcfa3f574a787c729000a6f/app.rb">app.rb</a>
    <a class="diff" href="https://github.com/orfjackal/web-intro-project/commit/fe8b2f27c46de1efcbcfa3f574a787c729000a6f">View changes</a>
</aside>


## Error page when the picture is not found

Currently if you go to a picture page that doesn't exist, for example <http://localhost:4567/pictures/foo.html>, it shows a page with the picture not working. But instead of that a well behaving site should give an error message that the page was not found.

In HTTP there are a bunch of numeric status codes, of which the code 404 means that a page was not found. In Sinatra we can produce that status code with `halt 404`. Additionally we can create a `not_found` route to produce an error message page (otherwise the page is empty). It could produce a full web page, but a simple "Page Not Found" text is enough.

Update your `/pictures/:picture.html` route to be as follows.

```ruby
get '/pictures/:picture.html' do
  @title = "Picture"
  @picture_url = find_picture_url(params['picture']) or halt 404
  erb :picture
end

not_found do
  "Page Not Found"
end
```

Check that <http://localhost:4567/pictures/foo.html> shows an error message. Also use the [developer tools][browser-developer-tools] to check that the status code for that page is 404.

![Page not found error shown with developer tools](error-page-when-the-picture-is-not-found.png)

<aside class="solution">
    <a class="file" href="https://github.com/orfjackal/web-intro-project/blob/04abc1bf4206d8e90f5d6d3325de3774e0d584a8/app.rb">app.rb</a>
    <a class="diff" href="https://github.com/orfjackal/web-intro-project/commit/04abc1bf4206d8e90f5d6d3325de3774e0d584a8">View changes</a>
</aside>


[html-img]: https://developer.mozilla.org/en/docs/Web/HTML/Element/img
[css-width]: https://developer.mozilla.org/en-US/docs/Web/CSS/width
[css-height]: https://developer.mozilla.org/en-US/docs/Web/CSS/height
[css-background-color]: https://developer.mozilla.org/en-US/docs/Web/CSS/background-color
[css-margin]: https://developer.mozilla.org/en-US/docs/Web/CSS/margin
[css-padding]: https://developer.mozilla.org/en-US/docs/Web/CSS/padding
[css-border]: https://developer.mozilla.org/en-US/docs/Web/CSS/border
[css-box-shadow]: https://developer.mozilla.org/en-US/docs/Web/CSS/box-shadow
[dry]: http://programmer.97things.oreilly.com/wiki/index.php/Don't_Repeat_Yourself
[ruby]: https://www.ruby-lang.org/
[sinatra]: http://www.sinatrarb.com/
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
[ruby-printing]: http://zetcode.com/lang/rubytutorial/io/
[ruby-regexp]: http://ruby-doc.org/core-2.2.0/Regexp.html
[browser-developer-tools]: https://developer.mozilla.org/en-US/Learn/Discover_browser_developer_tools
[the-secret-of-fast-programming-stop-thinking]: https://www.codesimplicity.com/post/the-secret-of-fast-programming-stop-thinking/
