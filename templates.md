---
layout: chapter
title: Templates
permalink: /templates/
next: /databases/
---

In this chapter you will learn how to create a web server with Ruby, and how to use templates and a bit of programming to avoid repeating yourself. You will also get a taste of creating dynamic content, i.e. the page content is not always the same when somebody visits it.


## Pictures page

Last time we created a navigation menu with a link to the `pictures.html` page, but that page doesn't yet exist. Let's create it now.

Make a copy of `about.html`, name it `pictures.html`, and change its title and heading. Find about a dozen images which you like from [Public Domain Pictures](http://www.publicdomainpictures.net/) and add their thumbnails as images on your `pictures.html` using the [`<img>`][html-img] tag.

```html
<img class="album-photo" src="http://www.publicdomainpictures.net/pictures/50000/t2/cat-looking-up.jpg">
<img class="album-photo" src="http://www.publicdomainpictures.net/pictures/30000/t2/cat-in-the-city-5.jpg">
<img class="album-photo" src="http://www.publicdomainpictures.net/pictures/30000/t2/annoyed-cat.jpg">
...
```

As shown above, give the images a unique class and use CSS to make them look like a photo album. In case the pictures are of different size, you can set their [height][css-height]; the web browser will set their width automatically to maintain the picture's aspect ratio.

The following example uses the CSS properties [border][css-border], [background-color][css-background-color], [margin][css-margin], [padding][css-padding] and [box-shadow][css-box-shadow].

![Photo album](photo-album.png)

[View solution](https://github.com/orfjackal/web-intro-project/commit/f402219504af273d03270f51eb36d6eeab1164de)


## Don't Repeat Yourself

In programming we have a principle called [Don't Repeat Yourself (DRY)][dry] which says that *"every piece of knowledge must have a single, unambiguous, authoritative representation within a system."* If we follow that principle, our code will be easier to understand and modify.

Our little web site is already breaking the DRY principle. Both of our pages are repeating the HTML for the layout and navigation. If we would add another page, we would have to change the navigation menu in many different HTML files. Additionally the pictures page has lots of similar `<img>` tags with just the URL changing. To solve this problem, we will introduce *templates*, so that all HTML code will be written once and only once.


## Run a web server

This far we have created our site with just static HTML pages, but in order to use templates, we must create a *web server* which will dynamically generate HTML pages. Your web browser will communicate with the web server and show the HTML pages which the web server generates.

We will use the [Ruby][ruby] programming language and the [Sinatra][sinatra] web framework to create the web server. Follow the [installation guide](/install/) to install Ruby and Sinatra if you haven't yet done so.

Create a file called `app.rb` and put the following code in it. This minimal web application contains a single *route* for the *path* `/hi` which will just generate a page with the text *Hello World!*

```ruby
require 'sinatra'

get '/hi' do
  "Hello World!"
end
```

Then run the command `ruby app.rb` in your terminal. It should start a web server on your own machine in port 4567. To view it, visit <http://localhost:4567/hi> in your web browser. It should show the *Hello World!* text.

![Your first web server is working](sinatra-hello-world.png)

[View solution](https://github.com/orfjackal/web-intro-project/commit/d89cf7c04e9840963a2f21778eb7b4a5cc18c102)

Finally stop the web server by pressing `Ctrl+C` in your terminal. Try refreshing the page in your web browser. It should now show an error message because there is no more a web server to connect to.


### Making file changes visible

Always when you change your application, you will need to stop and restart the web server. To stop the web server running in your terminal, press `Ctrl+C`. To start it again without having to type the command, press `Up` and then `Enter`.

Change the text that the `/hi` route returns and check in your web browser that it shows the new text. Repeat this a couple of times. Observe how the web browser still shows the old text if you don't restart the server.

Programmers don't like doing things that the computer can do for them, so there are tools for [automatically restarting the web server][sinatra-reloading] whenever a file is changed. You should have installed the `rerun` tool as part of the install guide. Let's try it out.

Start the server with the command `rerun ruby app.rb` in your terminal. Try changing the text that the `/hi` route returns a couple more times, but notice how you don't need to restart the web server yourself. The `rerun` command will restart it automatically every time that you save the file (it may take a couple of seconds, so you might see an error message if you reload the page in your web browser quickly).


## Static site on a web server

As a first step towards serving our web site through Sinatra, we can use Sinatra's ability to [serve static files][sinatra-static] to serve our existing HTML and CSS files unmodified.

Inside the same folder as `app.rb`, create a new folder called `public` and move your HTML and CSS files and pictures there. Then start your web server with `ruby app.rb`, visit <http://localhost:4567/pictures.html> in your web browser and make sure that your site looks the same as before, but this time it's being served to you by a web server the same way as all web sites on the Internet.

* TODO: avoid åöä in path

![Your first web server is working](sinatra-static-site.png)

[View solution](https://github.com/orfjackal/web-intro-project/commit/89edaab90a65f74758abaf86c9af950bb2f4cda9)


## Fix the front page

You should make sure that people can visit your web site through its base URL, the same way as all web sites on the Internet, instead of having to know one of its sub pages.

First try going to <http://localhost:4567/> on your current site. It should show an error message "Sinatra doesn’t know this ditty." Let's fix that.

Change your `app.rb` to [redirect][sinatra-redirect] the path `/` to `/about.html` by adding the following route.

```ruby
get '/' do
  redirect to('/about.html')
end
```

Check that when you visit <http://localhost:4567/>, your web browser immediately goes to the <http://localhost:4567/about.html> address.

![Front page redirect works](front-page-redirect.png)

[View solution](https://github.com/orfjackal/web-intro-project/commit/09b915909b1bc173e6f87b8f96a73815fcb66232)


## Templates

The idea of *templates* is to sprinkle your HTML code with some placeholder marks where you can easily insert some dynamic content. We will use the [ERB][erb] templating *library* which is included in Ruby's *standard library* and which Sinatra also supports.

Create a `views` folder inside your project folder (next to the `public` folder). This is the folder where Sinatra expects to find all your template files.

Copy `public/about.html` to the `views` folder and name it `layout.erb`. In `views/layout.erb`, replace all page content (i.e. the stuff between `<section class="content">` and `</section>`) with just `<%= yield %>`

Copy `public/about.html` to `views/about.erb` and remove everything except the page content (i.e. the opposite of what you did in `views/layout.erb`). Copy `public/pictures.html` to `views/pictures.erb` and do the same thing. Remove `public/about.html` and `public/pictures.html`.

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

[View solution](https://github.com/orfjackal/web-intro-project/commit/47cb34755f57f56f478dbee80ccfa4887fba5177)


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

The above code first creates a list of the names of all Backstreet Boys and stores it in the `backstreet_boys` *variable*. The next lines calls the `sample` *method* on the list, which will randomly pick one of the items in the list, and then the code stores it in the `@who_i_marry` variable. Because the variable name starts with `@`, also the template can access that variable.

In `views/about.erb`, add the following code to show the value of `@who_i_marry` on the page.

```erb
<p>I like Backstreet Boys a lot. I'm going to marry <%= @who_i_marry %>!</p>
```

Try reloading the about page in your web browser multiple times. The name should change randomly every time that the page is reloaded.

[View solution](https://github.com/orfjackal/web-intro-project/commit/0aa84a7aee603691d888ccb4ddc218ef0a37dbff)


## Front page at the root

Normally the front page of every web site is at the root path (`/`), but on our web site it's instead at the `/about.html` path. We can solve that by rendering the about template at the root path instead of doing a redirect.

In `app.rb`, remove the old `/` route and change the `/about.html` route to be the new `/` route.

In `views/layout.erb`, change the link to the about page from `<a href="about.html">` to `<a href="/">`.

Check that when you visit <http://localhost:4567/>, your web browser stays at the <http://localhost:4567/> address and shows the front page.

![Front page at the root](front-page-root.png)

[View solution](https://github.com/orfjackal/web-intro-project/commit/e5a5a9215f1e3815efcbe7bcc217c762a8f4dcf7)


## Album photo template

In addition to inserting single items to a template, as we did a moment ago, we can insert a list of items and repeat a piece of the template for each of them. To demonstrate that, we will remove the repetition of multiple image tags on the pictures page. Using templates might seem overkill for simple image tags, but imagine that each image will also have a description, like button and a link to the full-sized picture, in which case avoiding the repetition becomes very important.

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

[View solution](https://github.com/orfjackal/web-intro-project/commit/2fd4a7cbe266f8fa56beb1b786f12db11fed12c7)


## Show all pictures from a folder

There is still some work involved in adding the picture URLs by hand to the list in the route. We can simplify that by making our application generate the list automatically based on what pictures there are in a folder.

Create a `pictures` folder under the `public` folder and save there all the pictures in your list.

Change your `/pictures.html` route to generate a list of the picture URLs based on the contents of the `public/pictures` directory.

```ruby
get '/pictures.html' do
  @picture_urls = Dir.glob('public/pictures/**').map { |path| path.sub('public', '') }
  erb :pictures
end
```

Check that the pictures page still works. Try adding a couple more pictures - much easier now, isn't it?

[View solution](https://github.com/orfjackal/web-intro-project/commit/2030d93bd772adb74a5d384a7ed4b41a723ce2c6)


### How does this code work?

The above code combines multiple operations to achieve the desired result. Always when copying code from the Internet, you should try to understand what it does. One way is to read the documentation, in this case for the [glob][ruby-glob], [map][ruby-map] and [sub][ruby-sub] methods. Another way is to run pieces of the code interactively using a [REPL][repl] and check their result. Instead of a REPL you can also [print things to the terminal][ruby-printing] when you run your application.

You can start a Ruby REPL by running the `irb` command in your terminal. When you type there a line of Ruby code and press Enter, it will print the result of that code. Here is an example session of experimenting what the above code does.

```
$ irb
irb(main):001:0> Dir.glob('public/pictures/**')
=> ["public/pictures/annoyed-cat.jpg", "public/pictures/cat-1373445873hvw.jpg", "public/pictures/cat-1382017414PaD.jpg", "public/pictures/cat-hunting.jpg", "public/pictures/lieblingskater-44421287869401VYcy.jpg"]
irb(main):002:0> "public/pictures/annoyed-cat.jpg".sub('public', '')
=> "/pictures/annoyed-cat.jpg"
irb(main):003:0> Dir.glob('public/pictures/**').map { |path| path.sub('public', '') }
=> ["/pictures/annoyed-cat.jpg", "/pictures/cat-1373445873hvw.jpg", "/pictures/cat-1382017414PaD.jpg", "/pictures/cat-hunting.jpg", "/pictures/lieblingskater-44421287869401VYcy.jpg"]
irb(main):004:0> exit
```


## Unique titles for every page

Earlier when we started using templates, our web site didn't anymore have unique titles for every page. Now that we know how use variables in templates, we can give each page its own title.

In `views/layout.erb`, get the page title from the `@title` variable.

```html
<title><%= @title %></title>
```

Add to each of your routes the code for setting the title.

```ruby
@title = "type the title here"
```

Visit every page on your site to make sure that they still work and that they have different page titles.

![Parameterized page title](parameterized-title.png)

[View solution](https://github.com/orfjackal/web-intro-project/commit/29c04f352bcce2b69eec9a8af7792b9ba986a5a3)


## Sitemap for titles

TODO: remove this chapter

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

<s>[View solution](https://github.com/orfjackal/web-intro-project/commit/ee2a9760fbfa3dac04780c2294430a3bcebf3214)</s>


## Sitemap for navigation

TODO: remove this chapter

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

<s>[View solution](https://github.com/orfjackal/web-intro-project/commit/b81b3ce39bf919cd7b88c890f45bf212cac637c0)</s>


## Helper method for rendering templates

TODO: remove this chapter

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

<s>[View solution](https://github.com/orfjackal/web-intro-project/commit/dc60a1ea4e5e01ff83acd13d6378ba80802779a6)</s>


## Dynamic route for pictures

TODO

[View solution](https://github.com/orfjackal/web-intro-project/commit/a9f6491c3d2f36e3a01e0aa8c5a1482accf3cd3d)


## Picture page template

TODO

[View solution](https://github.com/orfjackal/web-intro-project/commit/0fc93a837a82539720805214805f14701efa1e14)


## Fix relative links

TODO

In the layout it's a good practice to have all the links prefixed with `/`, because that way they will work the same way regardless of the page where the user is currently. You can read more about this in [Absolute vs. Relative Paths/Links][absolute-vs-relative-paths].

[View solution](https://github.com/orfjackal/web-intro-project/commit/b719dcc1e819a283788de845fd3eb64763c4cadd)


## Fix guessing the picture file extension

TODO

[View solution](https://github.com/orfjackal/web-intro-project/commit/41592451a607ca75ef94f183f4947ece4ab83278)


## Error when picture not found

TODO

[View solution](https://github.com/orfjackal/web-intro-project/commit/33b388bd9a3faf1004f44be4cf7e920d22302ba8)


[html-img]: https://developer.mozilla.org/en/docs/Web/HTML/Element/img
[css-height]: https://developer.mozilla.org/en-US/docs/Web/CSS/height
[css-background-color]: https://developer.mozilla.org/en-US/docs/Web/CSS/background-color
[css-margin]: https://developer.mozilla.org/en-US/docs/Web/CSS/margin
[css-padding]: https://developer.mozilla.org/en-US/docs/Web/CSS/padding
[css-border]: https://developer.mozilla.org/en-US/docs/Web/CSS/border
[css-box-shadow]: https://developer.mozilla.org/en-US/docs/Web/CSS/box-shadow
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
[ruby-printing]: http://zetcode.com/lang/rubytutorial/io/
[repl]: https://en.wikipedia.org/wiki/Read%E2%80%93eval%E2%80%93print_loop
